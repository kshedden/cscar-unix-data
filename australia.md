Case study using Australia climate data
=======================================

The Australian government provides data on daily temperature extremes
(daily maximum/daily minimum) at 120 locations in Australia. The data
are available from this site:

http://www.bom.gov.au/climate/change/acorn-sat/#tabs=Data-and-networks

There are separate text files containing the minimum and maximum series
for each location, so there are around 240 files to download overall.

We can automate the download process using a combination of `wget`
and `grep`:

```
wget http://www.bom.gov.au/climate/change/acorn-sat/#tabs=Data-and-networks
cat index.html | grep -o 'href="[^"]*.txt"' | \
   sed -e "s/href=\"/http:\/\/www.bom.gov.au/g" -e"s/\"$//g" > urls
wget -i urls -P data
```
