---
title: 'Using Pandas to Parse COVID-19 Data or: How I Learned to Stop Worrying and Love the Virus'
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

df.descfribe()

# ... bunch of stuff...
# make it nice:

>>> print(f"{df.describe().__doc__}")

# get more into it:

>>> df[["state", "cases"]]

# keep going...

>>> number_of_cases_by_state = df[["state", "cases"]]

# get the goods:

>>> number_of_cases_by_state.query('state == "Washington"').groupby("state").max()
```


