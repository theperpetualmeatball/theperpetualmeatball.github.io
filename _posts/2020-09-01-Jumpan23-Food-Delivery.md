---
title: "Analyzing Food Delivery Market Trends in New York City"
date: 2020-09-01
tags: [food delivery, data science, new york city]
header:
  image: "/images/jumpman23/banner.png"
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

After one month of launching Jumpman23's service in NYC, there have been 5214 orders places on the platform, 3192 unique customers acquired, and 898 merchant partners available on the platform. 

## Trends in Delivery Times
The graph below illustrates the number of orders per day in October. A cyclical pattern in the data shows a peak in the number of deliveries on Sundays. The increase in deliveries in the last half of September suggest that Jumpman23 may be growing in NYC.

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/total_deliveries.png" alt="total_deliveries" width="90%" height="90%">

Sunday has the highest number of deliveries, followed by Thursday and Wednesday.

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/day_of_week.png" alt="data_types" width="80%" height="80%">

The two peak hours for delivery are at 12 pm and at 7 pm, correlating to lunch and dinner!

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/hourly.png" alt="data_types" width="80%" height="80%">

## Transportation Trends

Bicycles are by far the most popular mode of transportation for jumpman, which makes sense given that they are the most economically feasible choice in NYC. 

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/vehicle_type.png" alt="data_types" width="80%" height="80%">

ORDER STATS TBD - avg delivery times, prep times, etc. 

## Customer Insights
According the graph depicting order frequency vs number of customers, only 30% of Jumpman23's customers order more than once! 
We can see that Jumpman23 has a problem with customer retention. 

- 30% of customers order more than once
- 15% of customers order more than two times
- 7% of customers order more than 3 times
- <4 % of customers order more than 4 times

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/frequency.png" alt="freq" width="90%" height="90%">

Acquiring new customers is very important when launching a business into a new market, and monitoring the number of new customers acquired per day can unlock some useful insights related to marketing performance. In October, Jumpman23 acquired an average of 106 new customers per day, and their growth rate appears to be declining over the month.  

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/new_customers.png" alt="new_customers" width="90%" height="90%">


## Merchant Insights



------------------------
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
