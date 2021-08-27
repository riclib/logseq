---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/why-is-my-snmp-string-showing-as-hexadecimal
author: [[Brian Brazil]]
---

> Why is my name showing as 0x6d79206e616d65?

# Why is my SNMP string showing as hexadecimal?

Why is my name showing as 0x6d79206e616d65?

Strings in Prometheus are full UTF-8, however not all byte sequences are valid UTF-8. For example the byte 0xff cannot appear in a valid UTF-8 string. What does this have to do with SNMP? If we get arbitrary bytes that could in principle be anything, then we have to assume that they're not UTF-8, and the best thing we can do is render them into label values as hexadecimal.

If we didn't do this, then such label values would be rejected as invalid, either by the client library or Prometheus itself at scrape time. We can't try to auto-detect this in some way and try to render what appears to be a valid ASCII/UTF-8 string (like netsnmp's command line tools appear to do), as that is not a reversible operation, and could cause clashes and confusion.

So what can we do? When sending strings on the wire, they will have a type of `OctetString` which doesn't tell us much. So we need to look at the MIB for that particular SNMP object, as MIBs explain what each object means. Extracting useful semantic information from MIBs is the job of the SNMP exporter's generator.

[RFC1213](https://tools.ietf.org/html/rfc1213#section-6) defined a `DisplayString` type for MIBs, an ASCII string of up to 255 characters. All valid ASCII is valid UTF-8, so the generator can tell the exporter that it's good to treat it as a UTF-8 string by setting its type in the snmp.yml to `DisplayString` rather than `OctetString`. If you look at IF-MIB for example, the definition of `ifName` starts with:

ifName OBJECT-TYPE
SYNTAX DisplayString

[RFC1443](https://tools.ietf.org/html/rfc1443) expanded on this with Display Hints. `DisplayString` is now defined with a display hint of `255a`, meaning a ASCII string of up to 255 bytes:

DisplayString ::= TEXTUAL-CONVENTION
DISPLAY-HINT "255a"

So the generator now also looks for any textual convention consisting of a number and then `a` and treats it as a `DisplayString`, ignoring the size limit for simplicity as the exporter doesn't statically allocate strings. It also has to special case `DisplayString`, for the sake of MIBs still depending on the original RFC1213 version that lacks the display hint.

[RFC2571](https://tools.ietf.org/html/rfc2571) introduced `SnmpAdminString` with a Display Hint of `255a` meaning up to 255 bytes worth of UTF-8 characters, and [RFC3411](https://tools.ietf.org/html/rfc3411) changed the hint to `255t` where `t` means UTF-8. As `DisplayStrings` were being handled as UTF-8 already under the covers, a `DisplayString` in the SNMP exporter is really a `SnmpAdminString` so the only adjustment this required was treating display hints of `255t` and similar as `DisplayString`.

So that's how the generator automatically detects where the device vendor has told us that a given octet string is in fact a valid ASCII or UTF-8 value. Unfortunately many MIBs do not distinguish between arbitrary binary strings and ASCII/UTF-8 strings that are intended to be human readable. For example [RFC3805](https://tools.ietf.org/html/rfc3805) defined `prtGeneralPrinterName` as a sequence of up to 127 bytes:
```
prtGeneralPrinterName OBJECT-TYPE
SYNTAX OCTET STRING (SIZE (0..127))
MAX-ACCESS read-write
STATUS current
DESCRIPTION
  "An administrator-specified name for this printer. Depending
  upon implementation of this printer, the value of this object
  may or may not be same as the value for the MIB-II 'SysName'
  object."
::= { prtGeneralEntry 16 }
```

Reading the description it seems likely that this name is intended to be an human-readable ASCII value, especially given that `sysName` is a `DisplayString`. However the generator doesn't understand English, it can only work off the formally specified parts of MIBs. If you are sure that this is indeed a valid ASCII or UTF-8 value, then you can override the type in your generator.yml to be a `DisplayString`:
```

printer_mib:
walk: - prtGeneralPrinterName
overrides:
prtGeneralPrinterName:
type: DisplayString
```

So that's how the SNMP exporter and generator deal with strings, and how you can workaround cases where MIBs don't fully represent what their strings mean.