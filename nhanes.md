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

We have converted the three files used here to text/csv and included them with the rest
of this tutorial.  You can download them with the following commands:

```
> wget github.com/kshedden/cscar-unix-data/raw/master/NHANES/DEMO_H.csv.gz
> wget github.com/kshedden/cscar-unix-data/raw/master/NHANES/BMX_H.csv.gz
> wget github.com/kshedden/cscar-unix-data/raw/master/NHANES/BPX_H.csv.gz
```

In general, it is better to leave data files compressed rather than decompressing
them.  Most Unix tools are capable of working directly with compressed files.  For
example, type

```
less DEMO_H.csv.gz
```

and you will see the uncompressed contents of this file, even though the file remains
compressed on disk.

The `zcat` or `gzip -dc` commands can be used to decompress a file "on the fly", which
can be used to decompress and pipe a stream into another program:

```
zcat DEMO_H.csv.gz | wc -l
gzip -dc DEMO_H.csv.gz | wc -l
```

While we could leave the files compressed, for simplicity, here we will decompress the files
before proceeding

```
> gunzip DEMO_H.csv.gz
> gunzip BMX_H.csv.gz
> gunzip BPX_H.csv.gz
```

## How many records are there?

A basic task is to determine how many records (lines of text) are present in
each of the three files:

```
wc -l DEMO_H.csv
wc -l BMX_H.csv
wc -l BPX_H.csv
```

## How many unique identifiers are there?

```
cut -d, -f2 DEMO_H.csv | sort -u | wc -l
cut -d, -f2 BMX_H.csv | sort -u | wc -l
cut -d, -f2 BPX_H.csv | sort -u | wc -l
```

## Who is in the demographics file but not in the BMX file?

Use process substitution to compose complex commands:

```
comm -23 <(cut -d, -f2 DEMO_H.csv | sort) <(cut -d, -f2 BMX_H.csv | sort)
```

## How many people have each distinct age?

```
cut -d, -f6 DEMO_H.csv | sort | uniq -c | sort -k2 -n
```

## Remove '.0' from the trailing end of numbers

```
cat DEMO_H.csv | sed -e 's/.0$//g' -e 's/.0,/,/g'
```

## Get a frequency table of genders in the file

```
awk -F "," 'NR > 1 {count[$5]++}END{for(j in count) print j,count[j]}' DEMO_H.csv
```

## Merge the demographic and body measurements data on the SEQN key

```
join -1 2 -2 2 -t, <(sort -k2 -t, DEMO_H.csv) <(sort -k2 -t, BMX_H.csv)
```

## Split file by gender

`NR == 1` always prints the header row

```
awk -F, 'NR == 1 || int($5)==2' DEMO_H.csv > females.csv
awk -F, 'NR == 1 || int($5)==1' DEMO_H.csv > males.csv
```

## Split file by age and gender

Retain only women over 40:

```
awk -F, 'NR == 1 || ((int($5) == 2) && (int($6) > 40))' DEMO_H.csv
```

