Case study using NHANES data
============================

Here we demonstrate how Unix command-line tools can be put to use
to carry out some practical data manipulations with the
[NHANES](https://www.cdc.gov/nchs/nhanes/index.htm) data.
NHANES is a major recurring cross-sectional survey of the US
population that collects many hundreds of variables per subject.
The variables are split across multiple files and some non-trivial
data management tasks commonly arise when working with these data.
Here we will work with a few of the files from the
[2013-2014 wave](https://wwwn.cdc.gov/nchs/nhanes/search/datapage.aspx?Component=Demographics&CycleBeginYear=2013)
of NHANES.

NHANES files are provided as SAS XPORT format files.  These are binary files and cannot
be processed directly using Unix tools.  But they can easily be converted to text/csv files using
R, Python, and other tools.  Below is a short Python program that can be used to do
the conversion of one of the files.

```
import pandas as pd
pd.read_sas("DEMO_H.XPT").to_csv("DEMO_H.csv.gz", compression="gzip")
```

**A caveat:** Since we are working with text/csv files here, it's worth pointing out that there
is a formal specification for this format called [RFC 4180](https://tools.ietf.org/html/rfc4180).
This specification deals with some special issues like delimiters (",") embedded in strings.  Not
all Unix tools are capable of processing all CSV files properly (these are tools
for working with generic text data, not for CSV specifically).  However most csv data files do
not contain complex quoted patterns, so many CSV files can be productively manipulated
using Unix tools.

```
> wget github.com/kshedden/cscar-unix-data/raw/master/NHANES/DEMO_H.csv.gz
```


