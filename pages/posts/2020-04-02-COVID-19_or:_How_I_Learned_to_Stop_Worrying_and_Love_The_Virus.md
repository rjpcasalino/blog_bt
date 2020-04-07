---
title: 'Using Pandas or wget, sed, and awk to Parse COVID-19 Data or: How I Learned to Stop Worrying and Love the Virus'
layout: post
---

The New York Times (in their illustrious wisdom) have begun to share data files with cumulative counts of the virus in the United States of America. 

That data can be found [here](https://github.com/nytimes/covid-19-data).

Install pandas:

`pip install pandas`

Download the csv...

```
from pandas import read_csv

df = read_csv("us-states.csv")

```

play around some:

```
df.cases.max()

df.deaths.max()

df.describe()

# make it nice:

>>> print(f"{df.describe().__doc__}")

# get more into it:

>>> df[["state", "cases"]]

# keep going...

>>> number_of_cases_by_state = df[["state", "cases"]]

# maybe this?

>>> number_of_cases_by_state.query("state == 'Washington'").groupby("state").max()

# aha!
>>> number_of_cases_by_state.query("state == 'Washington'").groupby("state").max().to_json()
'{"cases":{"Washington":5588}}'

# go deeper now...

pip install urllib3

>>> import urllib3

>>> http = urllib3.PoolManager()

>>> r = http.request("GET", "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv")

>>> r.status

200

>>> df = read_csv(str(r.data))

>>> df.cases.max()

### as of 4/1 @ 9AM PST

83889
```

### wget, sed, awk

It appears the Times doesn't update the csv with the current day's stats until the next day. That is somewhat annoying. Anyway, I wrote a small bash script to query the csv by date (rudimentary):

```
#!/usr/bin/env bash

if [ -z "$1" ]; then
        printf "Please supply a date range!\n (e.g., 2 days ago)";
        exit 1;
fi;

frame=`date --date="$1" +%F`

awk -v time=$frame -F, '$1 == time { deaths[$2] += $5 }
END      { for (name in deaths) print name, deaths[name] }'
```

`awk` is quite the powerful little tool. Combine this with `sed` and you can get the results rather painlessly. `-v` allows for environment vars and `-F,` denotes that the separator is a comma. Otherwise `awk` defaults to blank spaces.

I wonder if there is a better way to get `sed` to "know" how many lines are within a given file. In any case, I just did this:

```
sed `wc -l < us-states.csv`q
```
aside: psst, I had forgotten that I wrapped `wc -l $*` into a cmd named `lc` and that was confusing for a bit.

the `q` here stands for quit. So, print all these lines and then quit. Stream editors are cool like that. The code above is rather ugly so please forgive me. The reason for the `<` is to disregard the file name in the output. Why that works, I don't really know but I think I'd better wise up.

Moving on, let's tie in wget, awk, and cron. Then we can really get going:

```
#!/usr/bin/env bash
wget -q -O data.csv "http://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv"

sed `wc -l < data.csv`q data.csv | awk -v date=$(date --date="yesterday" +%F), -F, '$1 == date { printf("%s\n\t cases: %d\n\t deaths: %d\n", $2, $4, %5) }' > /dev/tty1

$ Teaxs
	cases: 8115
	deaths: 160
...

crontab -e

*/30 * * * * bash /home/you/fetch-nytimes-covid-19-csv.sh
 
```

`-q` do it quietly and save it (`-O`) to a file named `data.csv` or whatever works best for you.

this will dump to stdout. We can do better perhaps? What could we do with this? Feed it along the wire to some other service? The world is my oyster. 
