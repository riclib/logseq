title:: Grafana/Merge Data Between Data Sources
type:: howto
tags:: #grafana

# Howto
	- In the query Use Mixed and chhose both data sources
	  > Note: This is for instant queries, not sure how performant it will be for ranges
	  > Note: Tested with Table kind of data and with Table and bar chart datasources
	- ## Create A Mixed Query
		- ![CleanShot 2021-08-07 at 12.26.13.png](../assets/CleanShot_202021-08-07_20at_2012.26.13_1628328416452_0.png)
		  > Make sure to summarize by the fields you want to merge)
	- ## Filter on the data you need
	  > if you are mixing a main table with a lookup table, there usually will be more values in the lookup that query returns.
		- ![CleanShot 2021-08-07 at 12.31.15.png](../assets/CleanShot_202021-08-07_20at_2012.31.15_1628328685329_0.png) 
		  > For instant queries filter by not null is not working as of Grafana 8.1.0
		-
	- ## Sort
		- ![CleanShot 2021-08-07 at 12.34.36.png](../assets/CleanShot_202021-08-07_20at_2012.34.36_1628328890714_0.png)
	- ## Format
		- If you want to display in a table you're ready. If in a panel that expects time series data...
		- ![CleanShot 2021-08-07 at 12.34.18.png](../assets/CleanShot_202021-08-07_20at_2012.34.18_1628328866657_0.png)