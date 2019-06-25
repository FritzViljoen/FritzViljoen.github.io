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

Following on the previous post (see [Cinderella – Scrubbing Houses](https://fritzviljoen.github.io/cinderella?), discussing how the property listing information was collected, we then jump into how the raw data has been shaped into something the investors could use.

# Making Sense

So, the next step was to make sense of the data and dress it to make it useable. Taking the CSV files, we read it, clean it, analyse it, add some additional functionality, then present it in a user-friendly interface. Essentially, it is the same data that is available to all on the property website, however I created a method of filtering and shortlisting the properties according to the specific user requirements.

In this case, the client's brief was two-fold: As property investors, they needed to easily identify potentially under-priced properties in a specific area, specifically for refurbishment, or "flipping" opportunities. Secondly, they wanted the average rental information for each area, in order to consider properties for Buy-to-Rent opportunities.

The end product still has a portion of manual labour involved, having to open a listing to see whether it’s an outlier, or that gem they are looking for, but Cinderella 2.0 provided a handy sorting tool to shortlist the listings in order of most- to least likely opportunities, based on the specific criteria.   


For the full code and step-by-step discussion, please refer to [Cinderella – House Hunting the Data Scientist Way Jupyter Notebook](https://nbviewer.jupyter.org/github/FritzViljoen/Cinderella/blob/master/House%20Hunting%E2%80%8A%20-%20The%20Data%20Scientist%20Way.ipynb?) post.
