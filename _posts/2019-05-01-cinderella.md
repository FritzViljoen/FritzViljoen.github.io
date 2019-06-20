---
title: "Cinderella - Web Scrubbing Houses "
date: 2019-05-01
tags: [data wrangling, data science, messy data, web scrubbing]
header:
excerpt: "Data Wrangling, Data Science, Messy Data, Web Scrubbing"
mathjax: "true"
---

# Property Investment

{% include figure image_path="/assets/images/cinderella.jpg" alt="Photo by Valentin Petkov on Unsplash" caption="Photo by Valentin Petkov on [**Unsplash**](https://unsplash.com)" %}

### Where do we start?
A couple of my closest friends recently joined a group encouraging property investment, highlighting the different strategies, how to get investors, where to look, how to go about it. Their enthusiasm was contagious, and before we knew it, my wife and I were neck-deep in it too. The group went ahead and started a company, got the web domain, printed some business cards, sorting out all the admin .... and then what?

Their approach was simple look for DEALS, DEALS, DEALS!! Not very concerned with where the property was located, there are many places where one can make money in property. So, they started trawling real-estate websites, and painfully, one by one, started going through all listings, hoping to spot that unicorn but finding only donkeys!

Not really having any better idea, they would take a best guess at a suburb where they think the type of deals, they were looking for could be found. They had a rough idea about budget, but individually had to draw up a spreadsheet gathering all the variables to see whether the Return on investment ROI was within target.

Knowing my skills, they asked me whether I could help de-pain the process, some form of web-scrubber to isolate the information they were looking for...so I did.


So for this project we’re going to use Python and its community driven libraries like, Pandas & BeautifulSoup.
Python code block:
```python
import datetime
import pandas as pd
from urllib.request import urlopen as uReq
from bs4 import BeautifulSoup as soup
```

The idea is to loop through the listing pages of the property website. So, we need a couple of things.
Firstly, where to store the parsed listings, and secondly a way to get the web page html.
```python
df = pd.DataFrame()
pg = 1
while True:
        url = 'https://www.immoafrica.net/residential/' + status + \
            '/south-africa/western-cape/cape-metropol/?sort=newest' + \
            '&pg=' + str(pg)
        uClient = uReq(url)
        page_html = uClient.read()
        response = uClient.close
        page_soup = soup(page_html, "html.parser")
```

Next issue to resolve is to let the loop know when to stop. As with all things in life there are trade-offs, depending on which route you take. I went with the option of looking for the maximum page number in the pagination section which breaks the while loop when the current page count equals the max page count. Simple stuff.
```python
pagination = page_soup.find('div', class_='pagination-holder').ul.find_all("a")
page_count = max([int(tag.text) if (tag.text).isdigit() else 0 for tag in pagination])
print(url,'of ' + str(page_count))
        if pg >= page_count:
            break
        pg += 1
```

Next step is to identify the portions of the html that covers only the listings on each page. For each of the pages a listing holder is defined that contains only the listings on that page.

Then, looping through the listings, we fetch the specific information of each listing visible on the listing holder page. The Property information is temporarily stored in a python dictionary “parsedListing”.
```python
listingHolder = page_soup.find_all('article', class_='block')

for listing in listingHolder:
    parsedListing = {}
    for a in listing.find_all('a', href=True, text=True):
        link_text = a['href']
        parsedListing['Link'] = link_text

    title = listing.find_all('img')[0].get('title')
    parsedListing['Title'] = title
    title = title.split(' in ')
    parsedListing['Suburb'] = " ".join(title[1].split())

```

Each listing has a number of parameters which we are trying to capture, so we run the loop through them again and save the information to the dictionary.
```python
    parameters = listing.findAll('li')
    for para in parameters:
        try:
            value = para.findAll('strong')[0].text
            key = " ".join(para.find('strong').next_sibling.split())
            if key in ['Bed', 'Bath']:
                key = key+'s'
            parsedListing[key] = value
        except:
            None
```

From here each parsed listing is appended to the Pandas DataFrame
```python
df = df.append(parsedListing, ignore_index=True)
```

Saving the DataFrame as a CSV file allows for further analysis, and data sharing.
```python
export_csv = df.to_csv(status+'.csv', index=None, header=True)
```

I you are interested where these csv files are used be sure to have a look at [House Hunting - The Data Scientist Way](https://fritzviljoen.github.io/cinderella-part-2/).
