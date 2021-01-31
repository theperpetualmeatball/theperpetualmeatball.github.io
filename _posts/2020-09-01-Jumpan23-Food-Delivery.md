---
title: "Analyzing Market Trends for a Food Delivery Service"
date: 2020-09-01
tags: [food delivery, data science, new york city]
header:
  image: "/images/perceptron/percept.jpg"
excerpt: "Marketing, Expansion, Food Delivery"
mathjax: "true"
---

# Introduction

Jumpman23 is an on-demand food delivery platform connecting customers to "Jumpman", a vast network of couriers. Jumpman23 recently launched in its newest market, New York City.

We will be diving into the data to understand how Jumpman23's has performed in NY, as well as identify any key trends related to customer, merchant, and Jumpman behavior.


### Data Dictionary

The data set consists of order delivery times, vehicle types, merchant categorical data, and geographical coordinates corresponding to order pickup and drop off locations.

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/data_types.png" alt="data_types">

# Data Exploration
## Where are the orders coming from?

The heatmaps below show the geographical distribution of pickup and drop off locations in Manhattan, Brooklyn, and Queens. 




Here's some basic text.

And here's some *italics*

Here's some **bold** text.

What about a [link](https://github.com/dataoptimal)?

Here's a bulleted list:
* First item
+ Second item
- Third item

Here's a numbered list:
1. First
2. Second
3. Third

Python code block:
```python
    import numpy as np

    def test_function(x, y):
      z = np.sum(x,y)
      return z
```

R code block:
```r
library(tidyverse)
df <- read_csv("some_file.csv")
head(df)
```

Here's some inline code `x+y`.

Here's an image:
<img src="{{ site.url }}{{ site.baseurl }}/images/perceptron/linsep.jpg" alt="linearly separable data">

Here's another image using Kramdown:
![alt]({{ site.url }}{{ site.baseurl }}/images/perceptron/linsep.jpg)

Here's some math:

$$z=x+y$$

You can also put it inline $$z=x+y$$
