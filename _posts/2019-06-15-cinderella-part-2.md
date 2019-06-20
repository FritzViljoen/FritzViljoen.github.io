---
title: "House Hunting - The Data Scientist Way"
date: 2019-06-15
tags: [data wrangling, data science, messy data]
header:
excerpt: "Data Wrangling, Data Science, Messy Data"
mathjax: "true"
---

# Property Investment

{% include figure image_path="/assets/images/cinderella.jpg" alt="Photo by Valentin Petkov on Unsplash" caption="Photo by Valentin Petkov on [**Unsplash**](https://unsplash.com)" %}

Housing listings for the city of Cape Town was collected from a well-liked property website. Using a web scraper the data was collated in a batch of CSV files, and read with Python — Pandas as DataFrame objects. These DataFrames objects formed the backbone of the project. For more on this, please see the [Cinderella – Scrubbing Houses](https://fritzviljoen.github.io/cinderella/) post.

# Making Sense

So, the next step was to make sense of the data and dress it to make it useable. Taking the CSV files, we read it, clean it, analyse it, add some additional functionality, then present it in a user-friendly interface. Essentially, it is the same data that is available to all on the property website, however I created a method of filtering and shortlisting the properties according to the specific user requirements.

In this case, the client's brief was two-fold: As a property investor, she wanted to be able to easily identify potentially under-priced properties in a specific area, specifically for refurbishment, or "flipping" opportunities. Secondly, she wanted the average rental information for each area, in order to consider properties for Buy-to-Rent opportunities.

The end product still has a portion of manual labour involved, having to open a listing to see whether it’s an outlier, or that gem you are looking for, but Cinderella 2.0 provided a handy sorting tool to shortlist the listings in order of most- to least likely opportunities, based on the specific criteria.   

This is how I went about it:

## Data collation and clean-up
In the CSV files, we collected the property information for different regions in South Africa and saved them separately. For this specific client, using Pandas again, we call in the region of Cape Town, both the rental and sales listings.  

Python code block:
```python
import pandas as pd

region = 'cape-town'

rent_df = pd.read_csv(region+'_to-rent.csv', low_memory=False)
sale_df = pd.read_csv(region+'_for-sale.csv', low_memory=False)
print(rent_df.shape, 'to-rent')
print(sale_df.shape, 'for-sale')
rent_df.tail()
```

This is what it looked like:

{% include figure image_path = "assets/images/posts/hh-snippit_2.jpg" %}

In order to reduce the steps when cleaning the data, the two data sets are combined, but not before labelling the source data type for each to enable easy splitting afterwards.

```python
# defining the data type
rent_df['Type'] = 'to-rent'
sale_df['Type'] = 'for-sale'

df = rent_df.append(sale_df).reset_index(drop=True)

print(df.shape)
df.tail()
```

### Removing Redundant Characters
Cleaning of the data allows for easier time reading and visualization of the information.

```python
df = df.replace({r'\r': ' ', r'\n': ' ', '\s+': ' '}, regex=True)
print(df.shape)
df.tail()
```

{% include figure image_path = "assets/images/posts/hh-snippit_3.jpg" %}

### Clean and Drop Unnecessary Columns
The source data has some additional columns that is not applicable to this project.
Dropping the columns simplifies the process and makes for easier reading and data exploration.
Additionally, some of the column headers has some long naming conventions, shorter names would be better.

```python
df.drop(columns=['SpacerForProfilePic', 'StatusFlags row',
                 'StatusFlag flagOnShow', 'StatusFlag flagNew',
                 'Included', 'Garage', 'StatusFlag flagAvailableFrom',
                 'StatusFlag flagHdMedia', 'StatusFlag flagOfferPending',
                 'ImageHolder', 'Number', 'StatusFlag flagMatterport',
                 'StatusFlag flagHdMedia', 'StatusFlag flagPriceReduced',
                 'StatusFlag flagAvailableNow', 'Floor Area', 'Land Area', 'Search Area',
                 ], inplace=True, errors='ignore')

df.rename(columns={'UspsString': 'Property Features',
                   'PriceAdditionalDescriptor': 'Price Additional',
                   'PriceDescription': 'Price Description',
                   'PropertyType': 'Property Type',
                   'ListingImage': 'Listing Image',
                   'ListingNumber': 'Listing Number'
                   }, inplace=True)

print(df.columns)
print(df.shape)
df.tail()
```
### Removing Duplicates

```python
# Removing duplicate listings based on a unique listing number
print(df.shape, 'Before')
df.drop_duplicates(subset=['Listing'], keep='first', inplace=True)
print(df.shape, 'After')
df.tail()
```

### Working with Missing Values
To have any hope of comparing these listings with each other, there is a minimum amount of information required. All real estate websites have varying information, most aren’t consistent with where they include what detail, for example confusing floor area / building footprint with site area, which distorted the output quite severely.

For this analysis, the crucial information we decided on was the number of Beds, Baths, Listed Sales or Rental Price and Suburb, as this is the most consistently accurate.

All rows missing any one of these pieces of information is discarded as this breaks the calculation.  
Listing price also had some non-numeric values in there. As this is a crucial column its prudent to drop the record.

```python
print(df.shape, 'Before')
df = df.dropna(axis=0, how='any',
               subset=['Beds', 'Baths', 'Price Description', 'Suburb'])

df['Price'] = df['Price Description'].replace({'R': '', ' ': ''}, regex=True)
df = df[df['Price'].astype(str).str.isnumeric()].copy()
df['Price'] = df['Price'].astype(float)

df.fillna('', inplace=True)

df['Address'] = df['Address'].map(str) + ' ' + df['Suburb'].map(str) + ' ' + df['Region'].map(str)

print(df.shape, 'After')
```

We could have replaced the missing values with averages based on other listings, but this could influence the integrity of the data, and as we had sufficient data, decided to rather discard them instead, and collect them on a separate page without any calculations, to manually browse, just in case the unicorn was hidden there.

### After Cleaning
The data is best split into different data sets as the normalizing of rental vs sales data is subjectively different.

```python
rent_df = df[df['Type'] == 'to-rent']
sale_df = df[df['Type'] == 'for-sale']
rent_df.tail()
```

### Normalizing the Data
To further complicate things, some of the listings are priced per week, or per day instead of per month. So, to normalize the price, the daily and weekly rates were scaled to a monthly rate.

```python
rent_df.loc[rent_df['Price Additional'] == 'PER DAY', 'Price'] *= 28
rent_df.loc[rent_df['Price Additional'] == 'PER WEEK', 'Price'] *= 4
```

## Getting to Work
### Estimating Errors
When plotting the information, most of the data is heavily skewed to the left or lower end of the distribution, which hints at a possibility of outlier data, most likely a result of input or data capturing errors.

Removing them requires insight and domain experience.

Using Pandas Describe, we are able to see some descriptive statistics: For example, the price histogram shows only one column.

```python
print(rent_df.describe())
ax_list = rent_df.hist(bins=30, layout=(5, 5), figsize=(15, 15))
```

{% include figure image_path = "assets/images/posts/hh-snippit_4.jpg" %}

Using the mean average, we iteratively slice away the top 0.1% of listings, until finally it is possible to build a proper data set while losing the minimum number of listings.

```python
# Removing the highest 0.1% of rental listings
for column in ['Baths', 'Beds', 'Price']:
    while rent_df[column].max() > rent_df[column].mean()*10:
        rent_df = rent_df[rent_df[column] < rent_df[column].quantile(0.999)]
        rent_df = rent_df[rent_df[column] > rent_df[column].quantile(0.001)]

print(rent_df.describe())
ax_list = rent_df.hist(bins=30, layout=(5, 5), figsize=(15, 15))

```
{% include figure image_path = "assets/images/posts/hh-snippit_5.jpg" %}


After adjusting for outlier data, the price histogram has an expected skewed distribution, in this case as expected there would be more low rent units available than high rent units.

When comparing residential properties, a couple of different strategies are possible. One of the simpler ways to compare different sized properties with each other, we calculate the price per room ratio per area. Using this ratio, we able to rank the prospective properties according to possible investor potential.

This is not a fool proof system, as the rental price of a one bedroom to a two bedroom is not priced on a one to two ratio. However, as a first sifting process, it provides the user with a general picture of the price points for the different suburbs and where high rental potential can be expected. It is also more useful when looking into commune type Buy-to-Rent properties.  

In the next episode, we will approach the room ratios per units slightly differently, applying some machine learning to evaluate these properties on more equal footing, but for now this is sufficient to get going.

```python
rent_df['Rent Price per Room'] = (rent_df['Price']/rent_df['Beds']).round(-2)
sale_df['Sale Price per Room'] = (sale_df['Price']/sale_df['Beds']).round(-2)
ax_list = rent_df.hist(bins=30, layout=(5,5), figsize=(15,15))
```
{% include figure image_path = "assets/images/posts/hh-snippit_6.jpg" %}


### Location, Location, Location
As the saying goes, rental yield is highly subject to Location. Comparing the properties only on a price per room basis would result in only looking at opportunities in low socio-economic areas. However, by calculating the suburb median price and median price per room, the properties can be ranked comparatively in each suburb.

```python
grouping = ['Suburb', 'Region', 'Beds']

rentalStats_df = pd.DataFrame()
# Rental Count is a confidence indicator, as the larger the rental listings in an area the more robust analysis
rentalStats_df['Rent Count'] = rent_df.groupby(grouping).count()['Price']

# Median values are used as the median is more robust against outliers
#rentalStats_df['Rooms Median'] = (rent_df.groupby(['Suburb','Region']).median()['Beds']).round(0)
rentalStats_df['Rent Price Median'] = (rent_df.groupby(grouping).median()['Price']).round(-2)
rentalStats_df['Rent Price per Room Median'] = (rent_df.groupby(grouping).median()['Rent Price per Room']).round(-2)
rentalStats_df.reset_index(inplace=True)

export_csv = rentalStats_df.to_csv(region+'_rental-stats.csv', index=None, header=True)
rentalStats_df.tail()
```

{% include figure image_path = "assets/images/posts/hh-snippit_7.jpg" %}

The process is then repeated to establish the sale price per room:

```python
retailStats_df = pd.DataFrame()
retailStats_df['Listing Count'] = sale_df.groupby(grouping).count()['Price']
# Median values are used as the median is more robust against outliers

retailStats_df['Sale Price Median'] = (sale_df.groupby(grouping).median()['Price']).round(-3)

retailStats_df['Sale Price per Room Median'] = (sale_df.groupby(grouping).median()['Sale Price per Room']).round(-3)
retailStats_df.reset_index(inplace=True)

export_csv = retailStats_df.to_csv(region+'_retail-stats.csv', index=None, header=True)
retailStats_df.tail()
```

## Shortlisting
With our data set nicely calculated, we now have a few possibilities to sort our listings to find deals for the three basic strategies of property investment: Rent-to-Rent, Buy-to-Rent, and Buy-to-Flip.

1) For Rent-to-Rent, we rank according to a Rent per Room versus the Median Rent per Room ratio for the suburb.  
2) For Buy-to-Rent, we use the heuristic 1% rule, stating that your rental income per month should ideally be higher than 1% of the purchase price.
3) For Buy to flip, we're looking for the cheapest property relative to a suburb. For this, we rank the properties based on the Purchase Price versus the Median Purchase Price for the suburb.  

### Calculating the Ratios
```python
df = pd.merge(rent_df, rentalStats_df, on=grouping, how='right')

# Rent to Rent Ranking
df['Rental Ratio'] = df['Rent Price per Room'] / df['Rent Price per Room Median']

rental_df = df.sort_values('Rental Ratio', ascending=False)
rental_df.head()
```

```python
df = pd.merge(sale_df, rentalStats_df, on=grouping, how='right')
df = pd.merge(df, retailStats_df, on=grouping, how='right')

# Buy to Rent Ranking
df['1% Rule'] = df['Rent Price per Room Median'] / df['Sale Price per Room']*100

# Buy to Flip Ranking
df['Sale Price per Room'] = (df['Price']/df['Beds']).round(-3)
df['Sale Ratio'] = df['Sale Price per Room'] / df['Sale Price per Room Median']

retail_df = df.sort_values('1% Rule', ascending=False)
retail_df.head()
```


### Presenting the Data

```python
# Adding link to listing
def property_link(path):
    return '<a target="_blank" href="{}">{}</a>'.format(str(path), 'Listing')

# Adding Google maps address link
def google_maps(Address):
    path = ('https://www.google.com/maps/place/' +
            str(Address)).replace(' ', '+')
    return '<a target="_blank" href="{}">{}</a>'.format(path, Address)

# Adding listing Image
def path_to_image_html(path):
    return '<img src="' + str(path) + '" ' + 'style="width:150px;">'

displ = retail_df.head(1000)
displ.drop(columns=['Description', 'Sale Price per Room', 'Region',
                    'Listing Number', 'Rent Price Median', 'Type',
                    'Price Description', 'Sale Price per Room Median',
                    'Property Features'], inplace=True, errors='ignore')

displ.style.format({'Address': google_maps,
                    'Listing Image': path_to_image_html,
                    'Listing': property_link,
                    'Price': '{: ,.0f}'.format,
                    'Sale Price Median': '{: ,.0f}'.format,
                    '1% Rule': '{: ,.2f}'.format,
                    'Sale Ratio': '{: ,.2f}'.format,
                    })
```

So the end product looks like this, which for ease of use, includes a link to the listing, link to Maps highlighting the suburb, all the various columns on which one can sort, depending on the strategy, and as a final touch - a thumbnail image of the main listing picture, to easily identify what property you're dealing with.

{% include figure image_path = "assets/images/posts/hh-snippit_8.jpg" %}


And to the client's delight, the deals that floated to the top were spot-on! Catching even the most obscure little deals hiding in the shadows. By providing the data in this format, the client was able to sort the listings on may more parameters than what the property websites allow, and ultimately significantly cut down on the time spent trawling these websites!
