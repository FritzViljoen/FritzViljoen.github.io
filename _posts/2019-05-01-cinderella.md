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

Not really having any better idea, they would take a best guess at a suburb where they think the type of deals they were looking for could be found. They had a rough idea about budget, but individually had to draw up a spreadsheet gathering all the variables to see whether the Return on Investment (ROI) was within target.

Knowing my skills, they asked me whether I could help de-pain the process, some form of web-scrubber to isolate the information they were looking for...so I did.

### The Run Down
Housing listings for the city of Cape Town were collected from a well-liked property website. Using a web scraper the data was collated in a batch of CSV files, and read with Python — Pandas as DataFrame objects. These DataFrames objects formed the backbone of the project.

For the full code and step-by-step discussion, please refer to [Cinderella – Scrubbing Houses Jupyter Notebook](https://nbviewer.jupyter.org/github/FritzViljoen/Cinderella/blob/master/Scrubber.ipynb) post.


I you are interested in the next chapter to see where these csv files were used, be sure to have a look at [House Hunting - The Data Scientist Way](https://fritzviljoen.github.io/cinderella-part-2/).
