---
title: "Analyzing Food Delivery Trends in New York City"
date: 2020-09-01
tags: [food delivery, data science, new york city]
header:
  image: "/images/jumpman23/banner.png"
excerpt: "Marketing, Expansion, Food Delivery"
mathjax: "true"
---

# Introduction

Jumpman23 is an on-demand food delivery platform connecting customers to "Jumpman", a vast network of couriers. Jumpman23 recently launched in its newest market, New York City.

We will be diving into the data to understand how Jumpman23's has performed in NY, as well as identify any key trends related to customer, merchant, and Jumpman behavior. Finally, using the data analysis to drive our decision making, we can discuss some steps Jumpman23 can take to optimize their growth strategy.

You can view the jupter notebook links for the [data cleaning] [data_cleaning] and the [data analysis] [data_analysis].

[data_cleaning]: https://github.com/theperpetualmeatball/Jumpman23/blob/main/Jumpman23%20-%20Cleaning%20the%20Data.ipynb
[data_analysis]: https://github.com/theperpetualmeatball/Jumpman23/blob/main/Jumpman23%20-%20Analysis.ipynb

### Data Dictionary

The data set consists of order delivery times, vehicle types, merchant categorical data, and geographical coordinates corresponding to order pickup and drop off locations. 

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/data_types.png" alt="data_types">

# Data Exploration

After one month of launching Jumpman23's service in NYC, there have been 5214 orders places on the platform, 3192 unique customers acquired, and 898 merchant partners available on the platform. 

## Delivery trends
The graph below illustrates the number of orders per day in October. A cyclical pattern in the data shows a peak in the number of deliveries on Sundays. The increase in deliveries in the last half of September suggest that Jumpman23 may be growing in NYC.

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/total_deliveries.png" alt="total_deliveries" width="90%" height="90%">

Sunday has the highest number of deliveries, followed by Thursday and Wednesday.

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/day_of_week.png" alt="data_types" width="80%" height="80%">

The two peak hours for delivery are at 12 pm and at 7 pm, correlating to lunch and dinner!

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/hourly.png" alt="data_types" width="80%" height="80%">

The average prep time is almost twice as long as the average transit time. The Jumpman spends an average of 18 minutes at the pickup location waiting for the order to be prepared, which is time the Jumpman can spend deleviring another order. We will discuss how Jumpman23 can optimize it's operations by preparing meals in advance in the next section.

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/avg_times.png" alt="avg_times" width="120%" height="120%" >


## Transportation Trends

Bicycles are by far the most popular mode of transportation for a Jumpman, which makes sense given that they are the most economically feasible choice in NYC. 

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/vehicle_type.png" alt="data_types" width="80%" height="80%">

The average delivery time (in minutes) by vehicle type is shown below. The quickest method to get around is to walk!

Vehicle Type | Average Time (min)
--- | --- | --- 
truck |         61.3 
car |          51.1 
van  |         50.7 
scooter  |     46.9 
walker  |      45.6 
motorcycle  |  45.0 
bicycle   |    43.2 


## Customer Insights
According the graph depicting order frequency vs number of customers, only 30% of Jumpman23's customers order more than once! 
We can see that Jumpman23 has a problem with customer retention. 

- 30% of customers order more than once
- 15% of customers order more than two times
- 7% of customers order more than 3 times
- <4 % of customers order more than 4 times

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/frequency.png" alt="freq" width="100%" height="80%">

Acquiring new customers is very important when launching a business into a new market, and monitoring the number of new customers acquired per day can unlock some useful insights related to marketing performance. In October, Jumpman23 acquired an average of 106 new customers per day, and their growth rate appears to be declining over the month.  

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/new_customers.png" alt="new_customers" width="90%" height="90%">


## Merchant Insights
The top 10 categories and merchants are shown below. Roughly 53% of all orders are from the top 10 category, and 24% are from the top 10 merchants. 

Top 10 Categories:

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/top10categories.png" alt="top10categories" width="90%" height="90%" >

Top 10 Merhcant Partners:

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/top10merchants.png" alt="top10merchants" width="90%" height="90%">

The average prep time is 31 minutes, and the average time the Jumpman waits at the pickup location is around 18 minutes. From the histogram, we see that most merchants take between 10-45 minutes to prepare orders. 

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/histogram.png" alt="top10merchants" width="90%" height="90%">

## Geographical Distribution of Deliveries
Comparing the heatmap of the pickup locations vs the dropoff locations, we see that drop-off locations are more spread out in Upper Manhattan and Brooklyn/Queens, and pickup locations are centred in lower manhatten and midtown. From first glance, it seems like there is a lack of merchant partners in Brooklyn, where this is a higher dropoff vs pickup rate. 
 
<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/heatmap.png" alt="top10merchants">

I decided to dive deeper into the data to compare Jumpman23's performance in Manhatten, Brooklyn, and Queens.
To do this, I first clipped the geographical data into boroughs using geopandas. I downloaded a dataset from New York City Open Data, which contained all the latitude and longtitude data and polygon information for all three boroughs. Once I successfully clipped the data and sorted the orders by borough, it was finally ready for analysis.

It appears that only account for 4% of all orders are delivered to Brooklyn, and there are almost no oders delivered to Queens. The merchants:orders ratio is ~16.7% in Manhattan and ~31.2% in Brooklyn. The heatmap suggested that there was a lack of merchant partners in Brooklyn; however, we are now seeing that there is actually fewer merchants per customer in Manhattan!

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/borough_analysis.png" alt="top10merchants" width="80%" height="80%">



# Analysis
## Why do some customers order more frequently?

Acquiring new customers can become very expensive over time, and it may be more cost-effective to retain existing customers. According to the data, only 30% of customers order more than once, which is very low!

What factors lead to customers ordering more than once? I decided to divivde the data into two classes: Class 1 include customers who have ordered only once, and Class2+ include customers ordering more than once.

The following Class 1 and Class 2+ comparisons did not show any significant differences:
 - The average delivery time for Class 1 and Class 2+ customers was 45.6 and 45 mins respectively
 - The average transit time for both classes was ~ 14 mins (i.e food temperature was not an affect) 
 - Percentage of customers ordering more than once were similar for Manhattan and Brooklyn
 - The most common vehicle type for jumpman in both classes was bikes, followed by cars and walking
 
I tried to think about my own food delivery habits, and realized I only order once or twice a month from Ubereats because it can get pretty expensive. Income might play an important factor to customers who order frequently, such as once a week. Being the determined data scientist that I am, I gathered some ancillary data to support my hypothesis - the NYC median household income map!

The graph of customers who ordered more than 3 times is shown on the right, next to a map of NYC, depicting the median household income by neighbourhood (to the left). Comparing the two graphs side-by-side, we can see that customers who ordered almost once a week are clustered in the light blue to dark blue regions, suggesting a median household income of greater than 100K per year. 


<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/income.png" alt="top10merchants" >

Legend:

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/legend.png" alt="top10merchants" width="40%" height="40%">

# What do repeating customers order?
Do customers who order 3 or 4 times a month order differently than customers who order only once? The top 10 merchants for both classes are shown along with the price range pulled from Yelp.

Price ranges does not appear to differ between the two groups; however, we see more healthy food options, such sweetgreen and Whole Foods move up the list for customers who order frequently. Extravagant meal options like Suhi and BBQ also moved up the list. Interestingly, we see that McDonald's is a popular option for frequent orderers, which may suggest that busy individuals with lower budgets may also be ordering frequently. Either that, or everyone wants a Happy Meal now and then!

<img src="{{ site.url }}{{ site.baseurl }}/images/jumpman23/class3orders.png" alt="top10merchants">


# Conclusion
There we have it - a full breakdown of Jumpman23's food delivery operations in NYC! We can use our data analysis to make some reccomendations to Jumpman23's marketing team to help with customer acquisition and retention, and improve upon the platform's service. 

### Customer Acquisition
 - Run more marketing campaigns in Brooklyn with a focus on high median household income neighbourhoods
 - Good days to run campaigns are Sundays and Thursdays; best times are around noon and dinner 
 - Run promotional offers for popular restaurants like Shake Shack
 
### Customer Retention
 - Focus retention in areas with high median household income
 - Partner with more merchants offering healthy food options (prices in '$$' range are most popular)
 
### Improving Service Efficiency
 - Order most popular meals ahead of time, for merchants in the top 10 list, to cut down on delivery time 
 - Partner with more merchants in locations where there are higher drop-offs v.s pickups (see heatmap)
 - Partner with more merchants in dense manhattan dropoff locations to increase the merchant:customer ratio
 
 
