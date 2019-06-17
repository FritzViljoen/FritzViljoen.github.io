---
title: "Cinderella Part2 - House Hunting - The Data Scientist Way"
date: 2019-06-15
tags: [data wrangling, data science, messy data]
header:
excerpt: "Data Wrangling, Data Science, Messy Data"
mathjax: "true"
---

# Property Investment

{% include figure image_path="/assets/images/cinderella.jpg" alt="Photo by Valentin Petkov on Unsplash" caption="Photo by Valentin Petkov on [**Unsplash**](https://unsplash.com)" %}

### Making Sense

So the next step was to make sense of the data, and dress it to make it useable. Taking the CSV files, we read it, clean it, analyse it, add some additional functionality, then present it in a user friendly interface. Essentially, it is the same data that is available to all on the property website, however I created a method of filtering and shortlisting the properties according to the specific user requirement.

There is still a portion of manual labour involved, having to open a deal to see if it is an outlier, or that gem you are looking for. But Cinderella provided a handy sorting tool to shortlist the listings in order of most to the least likely opportunities, based on your criteria.   

This is how I went about it:

### Data collation and clean-up
Housing listings for the city of Cape Town was collected from a well-liked property website, using a web scraper the data was collected in a batch of CSV files. The data was read with Python — Pandas as DataFrame objects. These DataFrames objects formed the backbone of the project. More on this can be found here.




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
{% include figure image_path = "assets/images/posts/hh-snippit_1.jpg" %}

Python code block:
```python
#  in order to reduce the steps in cleaning the data, the two data sets are combined, defining the source tupe allowes for splitting afterwards.
defining the data type,
rent_df['Type'] = 'to-rent'
sale_df['Type'] = 'for-sale'

df = rent_df.append(sale_df).reset_index(drop=True)

print(df.shape)
df.tail()
```
{% include figure image_path = "assets/images/posts/hh-snippit_2.jpg" %}

### Removing unnecessary characters
Cleaning of the data allows for easier time reading and visulizing the information

Python code block:
```python
df = df.replace({r'\r': ' ', r'\n': ' ', '\s+': ' '}, regex=True)
print(df.shape)
df.tail()
```

{% include figure image_path = "assets/images/posts/hh-snippit_3.jpg" %}



## Clean and Drop unnecessary columns
The source data has some additional columns that is not applicable to the project.
Dropping the columns simplifies the process and makes foe easier reading.
Additionally some of the column headers has some long naming conventions, maybe we can simplify them as well.



Python code block:
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
### Removing duplicates.
Python code block:
```python
# Removing duplicate listings based on a unique listing number
print(df.shape, 'Before')
df.drop_duplicates(subset=['Listing'], keep='first', inplace=True)
print(df.shape, 'After')
df.tail()
```

### Working with missing values
To have any hope of comparing these listings to each other, there is a minimum amount of information needed.
for this analysis, the crucial information is Beds Baths Price and Suburb.
all rows missing any one of these pieces of information is discarded.

Python code block:
```python
print(df.shape, 'Before')
df = df.dropna(axis=0, how='any',
               subset=['Beds', 'Baths', 'Price Description', 'Suburb'])


df['Price'] = df['Price Description'].replace({'R': '', ' ': ''}, regex=True)
df = df[df['Price'].astype(str).str.isnumeric()].copy()
df['Price'] = df['Price'].astype(float)


df.fillna('', inplace=True)

print(df.shape, 'After')
```

### After cleaning
The data it best slit into different data sets as the normalizing of data is subjectively different

Python code block:
```python
rent_df = df[df['Type'] == 'to-rent']
sale_df = df[df['Type'] == 'for-sale']
rent_df.tail()
```

### Normilizing data
Some of the listing are priced per week, or per day, to normilize the price, the daily and weekly rates should be scaled to a monthly rate.





Python code block:
```python
rent_df.loc[rent_df['Price Additional'] == 'PER DAY', 'Price'] *= 28
rent_df.loc[rent_df['Price Additional'] == 'PER WEEK', 'Price'] *= 4
```



## Estimating errors
Most of the data is heavily skewed to the left or lower end of the distribution, this hints at a possibility of outlier data, probably a result of input or data capturing errors.

Removing them would require insight and domain experience.

for example
the price histogram shows only one column.
Using pandas discribe lets us see some decriptive stastistics.

Python code block:
```python
print(rent_df.describe())
ax_list = rent_df.hist(bins=30, layout=(5, 5), figsize=(15, 15))
```

{% include figure image_path = "assets/images/posts/hh-snippit_4.jpg" %}


Slicing away the top 0.1% of listings, it is possible to delete to minimum listings. As the maximum monthly rentals of more than 100000 is outside of the scope of this project.

Python code block:
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




After adjusting for outlier data, the price histogram has a right skewed distribution. Expected in this case as there would be much more low rent units as high rent units available

When comparing residential properties, a couple of different strategies exists. one of the simpler ways is to compare the price per room ratio.
Using this ratio it is possible to rank the prospective properties according to possible value.

Python code block:
```python

rent_df['Rent Price per Room'] = (rent_df['Price']/rent_df['Beds']).round(-2)
sale_df['Sale Price per Room'] = (sale_df['Price']/sale_df['Beds']).round(-2)
ax_list = rent_df.hist(bins=30, layout=(5,5), figsize=(15,15))
```
{% include figure image_path = "assets/images/posts/hh-snippit_6.jpg" %}


As the saying goes, Location Location Location, rental property is highly subject to area. Comparing the properties only on a price per room basis would result in only looking at opportunities in low income areas. However, we have at our disposal the Suburb of all the listings. By calculating the suburb median price and median Price per room, the value Suburbs would be ranked.


Python code block:
```python

grouping = ['Suburb', 'Region','Beds']

rentalStats_df = pd.DataFrame()
# Rental Count is a confidace indicator, as the larger the rental listings in an area the more robust analysis
rentalStats_df['Rent Count'] = rent_df.groupby(grouping).count()['Price']

# Median values are used as the median is more robust against outliers
#rentalStats_df['Rooms Median'] = (rent_df.groupby(['Suburb','Region']).median()['Beds']).round(0)
rentalStats_df['Rent Price Median'] = (rent_df.groupby(grouping).median()['Price']).round(-2)
rentalStats_df['Rent Price per Room Median'] = (rent_df.groupby(grouping).median()['Rent Price per Room']).round(-2)
rentalStats_df.reset_index(inplace=True)


export_csv = rentalStats_df.to_csv(region+'_rental-stats.csv', index=None, header=True)
rentalStats_df.tail()

df = pd.merge(sale_df, rentalStats_df, on=grouping, how='right')
df.head()
```

{% include figure image_path = "assets/images/posts/hh-snippit_7.jpg" %}





Python code block:
```python

retailStats_df = pd.DataFrame()
retailStats_df['Listing Count'] = sale_df.groupby(grouping).count()['Price']
# Median values are used as the median is more robust against outliers

retailStats_df['Sale Price Median'] = (sale_df.groupby(grouping).median()['Price']).round(-3)


retailStats_df['Sale Price per Room Median'] = (sale_df.groupby(grouping).median()['Sale Price per Room']).round(-3)
retailStats_df.reset_index(inplace=True)


export_csv = retailStats_df.to_csv(region+'_retail-stats.csv', index=None, header=True)
retailStats_df.tail()

df = pd.merge(df, retailStats_df, on=grouping, how='right')
df.head()
```

If a stratagy of rent to rent where to be followed, a heuristic of a rental ratio could be used to shortlist posible market mispricings.



Python code block:
```python
df['Address'] = df['Address'].map(str) + ' ' + df['Suburb'].map(str) + ' ' + df['Region'].map(str)
df['Listing Description'] = df['Address'].map(str) + ' ' + df['Title'].map(str)
df['Listing Description'] = df['Listing Description'].str.lower()

#df = df.apply(pd.to_numeric, errors='ignore')
df['Sale Price per Room'] = (df['Price']/df['Beds']).round(-3)

df['1% Rule'] = df['Rent Price per Room Median'] / df['Sale Price per Room']*100
df['Sale Ratio'] = df['Sale Price per Room'] / df['Sale Price per Room Median']

df = df.sort_values('1% Rule', ascending=False)

df.head()
```

Python code block:
```python
# Adding link to listing
def property_link(path):
    return '<a target="_blank" href="{}">{}</a>'.format(str(path), 'Listing')


def google_maps(Address):
    path = ('https://www.google.com/maps/place/' +
            str(Address)).replace(' ', '+')
    return '<a target="_blank" href="{}">{}</a>'.format(path, Address)


def path_to_image_html(path):
    return '<img src="' + str(path) + '" ' + 'style="width:150px;">'


displ = df[df['Listing Description'].str.contains(('stell').lower())]
displ = displ[displ['Beds'] == 3]


displ = displ.head(1000)

# print(displ.shape)

del displ['Listing Description']

displ.drop(columns=['Description', 'Sale Price per Room', 'Sale Price per Room Median',
                    'Listing Number', 'Rent Price Median',
                    'Price Description',  'Region',
                    'Type', 'Updated', 'Property Features'], inplace=True, errors='ignore')

displ.style.format({'Address': google_maps,
                    'Listing Image': path_to_image_html,
                    'Listing': property_link,
                    'Price': '{: ,.0f}'.format,
                    'Sale Price Median': '{: ,.0f}'.format,
                    '1% Rule': '{: ,.2f}'.format,
                    'Sale Ratio': '{: ,.2f}'.format,
                    })
```

{% include figure image_path = "assets/images/posts/hh-snippit_8.jpg" %}



And to my astonishment, the deals that floated to the top were spot-on! Catching even the most obscure little deals hiding in the shadows.



Property-Scrubber, my wife named “Cinderella” also allowed them to filter through all the suburbs in their reach, not only single suburbs at a time, but the best deal at any given time, anywhere!!



This is how I went about it....


Different strategies: Flips and Buy to rent. All real estate websites have varying information, most aren’t even consistent with where they include what detail — confusing floor area with site area....yeah.


Different approaches, use room to average cost ratio.


B2R — cost of room to average rent.
