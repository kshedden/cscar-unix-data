Case study using Australia climate data
=======================================

The Australian government provides data on daily temperature extremes
(daily maximum/daily minimum) at 120 locations in Australia. The data
are available from this site:

http://www.bom.gov.au/climate/change/acorn-sat/#tabs=Data-and-networks

There are separate text files containing the minimum and maximum temperature
series for each location, so there are around 240 files to download overall.

### Automating download of the data files

We can automate the download process using a combination of `wget`
and `grep`:

```
wget http://www.bom.gov.au/climate/change/acorn-sat/#tabs=Data-and-networks
cat index.html | grep -o 'href="[^"]*.txt"' | \
   sed -e "s/href=\"/http:\/\/www.bom.gov.au/g" -e"s/\"$//g" > urls
wget -i urls -P data
```

The first call to `wget` downloads the html source for the page given above.  We can
inspect this html (open it with `less`) to see that the data file names end in
`.txt`, and are stated in the html file as relative links with this form:

```
href="/climate/change/acorn/sat/data/acorn.sat.minT.002012.daily.txt"
```

Therefore, we can use `grep` to locate all the relevant URLs in the html file using
`grep`.  We then use `sed` to restructure each relative URL as a full URL.  Finally,
we use `wget` with the `-i` option to download all the data files into a directory
called "data".

### Consolidating the data

Now suppose that we would like to join (merge) the minimum and maximum series
for each station into a single file.

To do this, first we get the distinct station ids:

```
ls -lt | grep -oP "acorn.sat.minT.\K([0-9]+)(?=.daily.txt)" | sort -u > ids
```

Now we can use a loop to merge all the pairs of files.  Note that the top two
lines of each file do not contain data, so we use `tail` below to remove these
lines prior to joining.  Also, these source files contain windows-style newlines
(\r\n) so we use `sed` to remove the extra `\r` characters that are present
after the merge.

```
while read ID
    do
        f1="acorn.sat.minT.${ID}.daily.txt"
        f2="acorn.sat.maxT.${ID}.daily.txt"
        join <(tail -n +2 ${f1}) <(tail -n +2 ${f2}) | \
            sed -e "s/\r//g" > combined.${ID}.txt
done < ids
```

