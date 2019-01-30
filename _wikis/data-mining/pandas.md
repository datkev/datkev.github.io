---
layout: article
title: Pandas
permalink: /wikis/data-mining/pandas
aside:
    toc: true
sidebar:
    nav: wikis
---


Data exploration, analysis, and visualization for tabular data.

See <a href="https://github.com/justmarkham/python-data-science-workshop.git" target="_blank">https://github.com/justmarkham/python-data-science-workshop.git</a> for original code.

## Basics

Pandas Basics: Reading Files, Summarizing, Handling Missing Values, Filtering, and Sorting

```py
# imports
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# read in the CSV file
drinks = pd.read_csv('../directory/drinks.csv')
drinks = pd.read_csv('https://some-link.csv')
type(drinks)

# examine the data
drinks                  # print the first 30 and last 30 rows
drinks.head()           # print the first 5 rows
drinks.tail(10)         # print last 10 rows
drinks.describe()       # describe any numeric columns
drinks.shape            # dimensions of DF

# find missing values in a DataFrame
drinks.isnull()         # DataFrame of booleans
drinks.isnull().sum()   # convert booleans to integers and add

# handling missing values
drinks.dropna()                   # drop a row if ANY values are missing
print drinks.dropna().shape       # returns shape of DF after dropping rows with ANY missing value
drinks.dropna(how='all')          # drop row only if ALL values missing
print ufo.dropna(how='all').shape # returns shape of DF after dropping rows with ALL missing value
drinks.fillna(value='NA')         # fill in missing values
drinks.fillna(value='NA', inplace=True)
drinks.isnull().sum()             # returns count of missing values in all columns

# selecting a column ('Series')
drinks['continent']
drinks.continent            # equivalent
type(drinks.continent)

# summarizing a non-numeric column
drinks.continent.describe()
drinks.continent.value_counts()                         # excludes missing values
drinks.continent.value_counts(dropna=False)             # includes missing values
dataset['laebl'].value_counts(normalize=True).head()    # gives percentages

# selecting multiple columns
drinks[['country', 'beer_servings']]
my_cols = ['country', 'beer_servings']
drinks[my_cols]

# add a new column as a function of existing columns
drinks['total_servings'] = drinks.beer_servings + drinks.spirit_servings + drinks.wine_servings
drinks['total_servings'] = drinks['beer_servings'] + drinks['spirit_servings']
drinks.head()

# rename a column
ufo.rename(columns={'Colors Reported':'Colors_Reported', 'Shape Reported':'Shape_Reported'}, inplace=True)

# delete a column
del ufo['City']
del ufo['State']

# logical filtering and sorting
drinks[drinks.continent=='NA']
drinks[['country', 'total_servings']][drinks.continent=='NA']
drinks[['country', 'total_servings']][drinks.continent=='NA'].sort_index(by='total_servings')
drinks[['country', 'total_servings']][drinks.continent=='NA'].sort_index(by='total_servings', ascending=False)
drinks[drinks.wine_servings > drinks.beer_servings]
drinks[(drinks.wine_servings > 300) & (drinks.total_litres_of_pure_alcohol > 12)]
drinks.sort_index(by='beer_servings').tail(6)
drinks.beer_servings[drinks.continent=='NA'].mean()
smaller_dataset.groupby('label').duration.mean().sort_values(ascending=False).plot(kind='bar')
```

## Split, Apply, Combine

Apply a function to a DataFrame that is split by different types of keys.

```py
# for each continent, calculate mean beer servings
drinks.groupby('continent').beer_servings.mean()

# for each continent, count number of occurrences
drinks.groupby('continent').continent.count()
drinks.continent.value_counts()
```

## Lambda Functions

Lambda Functions are anonymous functions

```py
'''
f = lambda x: x+1   SAME AS 
def f(x): return x + 1
'''

# Apply any function using .apply
drinks.groupby('continent').total_servings.apply(lambda x: x.mean())    # mean
drinks.groupby('continent').total_servings.apply(lambda x: x.std())     # st dev
print drinks.groupby('continent').total_servings.min()
print drinks.groupby('continent').total_servings.max()
drinks.groupby('continent').total_servings.apply(lambda x: x.max()-x.min())    # range

# make new month columns
ufo['Month'] = ufo['Time'].apply(lambda x:int(x.split('/')[0]))
# day columns
ufo['Day'] = ufo['Time'].apply(lambda x:int(x.split('/')[1]))
# year columns
ufo['Year'] = ufo['Time'].apply(lambda x:int(x.split('/')[2][:4]))
```

## Plotting
```py
# bar plot of number of countries in each continent
drinks.continent.value_counts().plot(kind='bar', title='Countries per Continent')
plt.xlabel('Continent')
plt.ylabel('Count')
plt.show()

# bar plot of average number of beer servings by continent
drinks.groupby('continent').beer_servings.mean().plot(kind='bar')

# histogram of beer servings
drinks.beer_servings.hist(bins=20)

# grouped histogram of beer servings
drinks.beer_servings.hist(by=drinks.continent)
drinks.beer_servings.hist(by=drinks.continent, sharex=True)     # same x axes
drinks.beer_servings.hist(by=drinks.continent, sharex=True, sharey=True)    # same x and y axes

# boxplot of beer servings by continent
drinks.boxplot(column='beer_servings', by='continent')

# scatterplot of beer servings versus wine servings
drinks.plot(x='beer_servings', y='wine_servings', kind='scatter', alpha=0.3)

# same scatterplot, except all European countries are colored red
colors = np.where(drinks.continent=='EU', 'r', 'b')
drinks.plot(x='beer_servings', y='wine_servings', kind='scatter', c=colors)

# line plot of number of ufo sightings per year
sightings_per_year = ufo.groupby('Year').City.count()
sightings_per_year.plot(kind='line',
                        color='r',
                        linewidth=1,
                        title='UFO Sightings by year')

# plot ufo sightings in July
ufo[(ufo.Year==2014) & (ufo.Month ==7)].groupby('day').City.count().plot(kind='bar',
                                                                         color='b',
                                                                         title='UFO Sightings in July 2014')
```

