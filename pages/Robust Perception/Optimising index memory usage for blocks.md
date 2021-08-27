---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/optimising-index-memory-usage-for-blocks
author: [[Brian Brazil]] 
---
> One of the big changes in Prometheus 2.15.0 was reduced memory usage for indexes.

# Optimising index memory usage for blocks


One of the big changes in [Prometheus 2.15.0](https://www.robustperception.io/new-features-in-prometheus-2-15-0) was reduced memory usage for indexes.

To understand the improvements, we first need to talk about how block indexes work and particularly what data structures are kept in memory. If you look at [the docs](https://github.com/prometheus/prometheus/blob/v2.15.0/tsdb/docs/format/index.md) you'll see the index consists of:

-   A Symbol Table
-   Series, including their labels (using the symbol table) and chunk metadata
-   Label index (using the symbol table) from label names to label values, with a Label Offset Table pointing indicating where each label name is in the label index
-   Postings which are lists of series offsets that have a particular labelname/labelvalue pairs, with a Postings Offset Table pointing to where each labelname/labelvalue's posting is in the Postings.
-   A Table of Contents

The symbol table reduces the amount of disk space that the index file takes up, at the cost of having to do lookups to convert symbol numbers back to actual labelnames and labelvalues. Previously the symbol table was kept completely in memory in a map. So if you had a lot of churn and/or cardinality creating new label values, each unique label value was kept in memory for each block. Add in how Go garbage collection works (plus the overhead of map internals - [around 11 bytes per entry](https://stackoverflow.com/questions/15313105/memory-overhead-of-maps-in-go)) and you're more than doubling the memory usage.

The first improvement I made here was to not store the whole symbol table in memory on the heap. Instead we now only keep a list of the offsets in the file of every 32nd symbol. When we want to lookup a symbol number we can find the symbol offset before it, and walk the mmaped index file until we get the one we want. This saves a ton of memory, as we're now only storing 0.25 bytes per symbol on the heap (0.5 bytes with Go GC doubling) rather than the contents of all the symbols. Symbols are still regularly accessed so they'll probably all be resident in memory still via the mmaped file, but the doubling due to GC is gone as they're no longer on the heap. This does slow things down as you'd expect from a RAM vs CPU tradeoff, one way the impact is reduced is to have a cache of the symbols for label names kept in memory as label names are small in number but represent half of all symbol lookups.

The postings offset table is used to go from a labelname/labelvalue pair to a posting list, and then later on after all the matcher processing is done the series metadata is fetched based on the series offset in the postings. Previously the full postings offset table was kept in memory in a map. Once again for every label value there would be an entry in a map, though previous optimisations I performed shared the strings with the symbol table and introduced multi-level maps (one level for the label name, another level for the label value) so we were already down to only the cost of the map overhead, 16 bytes for the string, 8 bytes for the offset, and then all doubled due to Go GC. So somewhere around 70 bytes per posting, with the cost of label names ignored as they are small in number.

The second improvement was to not store the full postings offsets in memory. Instead we now store the a list of every 32nd labelvalue and offset for each posting for a given labelname in memory. On lookup we can binary search to the labelvalue before the one we want, and then walk the mmaped index file. I also added a batch interface so that when the posting matching code has a list of label values after applying a regex matcher that we can avoid having to walk over a given part of the postings offset table more than once. This all turned out to be a few percent faster than the previous code, which was a pleasant suprise. Memory usage is down to 1.5 bytes per posting, plus a 16th of the size of the average labelvalue - plus however much of the index offset table is resident in memory via mmap.

You may wonder, why did I choose 32 as my factor? The answer was to aim to not to have to walk more than a 4kB page of data, plus having something that was a power of 2 so the arithmetic would be faster. I haven't done extensive benchmarking to determine if this is the best choice but it appears to be satisfactory.

So to summarise before I made any improvements, we were storing all symbols twice plus 46 bytes overhead per symbol for the symbol table, and then all label pairs twice plus 70 bytes overhead per posting. Now presuming the worst case that everything is resident in memory via mmap, it's down to 1.5 bytes per symbol, all symbols, 7.5 bytes per posting, a 16th of all label values, and all label pairs once (the postings offset table includes the label name with every label value, so there's some redundancy there).

To simplify a bit we can assume that label values aren't shared across label names and that label names are insignificant in number. So that's 116 bytes overheard per label value dropped to 9 bytes which is an order of magnitude improvement. For the label values themselves we're looking at them going from being stored 4 times in memory, down to 1.0625 times. Label names go from 2 times per label value down to 1 time per label value.

This is a lot of math, so let's take a practical example. Let's say that you have label names which are 10 bytes each, and label values which are 20 bytes each on average. Previously you'd be looking at memory usage of 216 bytes per label value which goes down to 38 bytes per label value due to the symbols and postings offset tables, or around 5.5x better since 2.5.0. Other lengths of label names and values work out to a similar 4-6x improvement. This is all under the the worst case assumption that all of the symbols and postings offset tables are resident in memory, if you've high-cardinality label names which you never query by and/or never access series with those label names then the savings are even higher!

_Want to use Prometheus more efficiently? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
