---
title: "Cinderella - Scrubbing Houses"
date: 2019-05-01
tags: [data wrangling, data science, messy data]
header:
excerpt: "Data Wrangling, Data Science, Messy Data"
mathjax: "true"
---

# Property Investment

{% include figure image_path="/assets/images/cinderella.jpg" alt="Photo by Valentin Petkov on Unsplash" caption="Photo by Valentin Petkov on [**Unsplash**](https://unsplash.com)" %}

### Where do we start?
A couple of my closest friends recently joined a group encouraging property investment, highlighting the different strategies, how to get investors, where to look, how to go about it. Needless to say, their enthusiasm was contagious, and before we knew it, my wife and I were neck-deep in it too. The group went ahead and started a company, got the web domain, printed some business cards, sorting out all the admin .... and then what?

Their approach was fairly simple look for DEALS, DEALS, DEALS!! Not very concerned with where the property was located, there are many places where one can make money in property. So they started trawling real-estate websites, and painfully, one by one, started going through all listings, hoping to spot that unicorn but finding only donkeys!

Not really having any better idea, they would take a best guess at a suburb where they think the type of deals they were looking for could be found. They had a rough idea about budget, but individually had to draw up a spreadsheet gathering all the variables to see whether the Return on investment ROI was within target.

Knowing my skills, they asked me whether I could help de-pain the process, some form of web-scrubber to isolate the information they were looking for...so I did.

Python code block:
```python
def get_immoafrica(status):
    import datetime
    import pandas as pd
    from urllib.request import urlopen as uReq
    from bs4 import BeautifulSoup as soup

    df = pd.DataFrame()
    page_count = int(input('pg= '))
    for pg in range(page_count):
        try:
            url = 'https://www.immoafrica.net/residential/' + status + \
                '/south-africa/western-cape/cape-metropol/?sort=newest' + \
                '&pg=' + str(pg+1)

            print('pg= ' + str(pg+1) + ' of ' + str(page_count))
            uClient = uReq(url)
            page_html = uClient.read()
            response = uClient.close
            page_soup = soup(page_html, "html.parser")

            listingHolder = page_soup.find_all('article', class_='block')

            for listing in listingHolder:
                parsedListing = {}

                for a in listing.find_all('a', href=True, text=True):
                    link_text = a['href']
                    parsedListing['Link'] = link_text

                title = listing.find_all('img')[0].get('title')
                parsedListing['Title'] = title

                if title.find('Bedroom', 0) < 0:
                    continue

                title = title.split(' in ')
                parsedListing['Suburb'] = " ".join(title[1].split())

                parsedListing['Type'] = parsedListing['Title'][parsedListing['Title'].find(' ', 9):
                                                               parsedListing['Title'].find(' ', 12)]

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

                df = df.append(parsedListing, ignore_index=True)

        except:
            continue
    export_csv = df.to_csv('immoafrica_'+status+'.csv',
                           index=None, header=True)


# get_immoafrica('to-rent')
# get_immoafrica('for-sale')

```

And to my astonishment, the deals that floated to the top were spot-on! Catching even the most obscure little deals hiding in the shadows.



Property-Scrubber, my wife named “Cinderella” also allowed them to filter through all the suburbs in their reach, not only single suburbs at a time, but the best deal at any given time, anywhere!!



This is how I went about it....


Different strategies: Flips and Buy to rent. All real estate websites have varying information, most aren’t even consistent with where they include what detail — confusing floor area with site area....yeah.


Different approaches, use room to average cost ratio.


B2R — cost of room to average rent.


There is still a portion of manual labour involved, having to open every deal to see if it is an outlier, or that gem you are looking for. But Cinderella provided a handy sorting tool to shortlist the listings in order of most to the least likely opportunities.  