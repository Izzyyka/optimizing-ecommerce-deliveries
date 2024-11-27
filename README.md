# Optimizing Ecommerce Deliveries
Tools: SQL Server Management Studio\
Dataset: You can access the dataset [here](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

## Understanding the Business Challenge
Delivery delays present a significant challenge for e-commerce businesses, especially when logistics capacity fails to meet the growing demand for shipments. Delays not only disrupt the supply chain but also risk damaging retailer credibility, as customers may leave negative reviews or lose trust in the service. This problem directly impacts the customer experience, potentially reducing customer retention and loyalty.\
To address this issue, we identified several key business questions to guide our analysis using SQL. These questions aim to uncover the root causes of delivery delays and their impact on different dimensions of the e-commerce business, including:
1. What is the percentage of delayed deliveries compared to all deliveries?
2. What is the average delay time compared to the estimated delivery date? 
3. What is the relationship between product dimensions and delivery delays?
4. How does the probability of delay vary across months? Which specific month has the highest delay percentage?
5. Which product categories have the highest delay probability?
6. How does delay probability vary by customer location?
7. How does delay probability vary by seller location?

By answering these questions, we aim to gain actionable insights to optimize delivery performance and address critical delay factors affecting the business.

## Data Overview
The dataset  that we used in this analysis represents  a brazilian e-commerce platform’s (Olist) relational structure, consisting of several interconnected tables that provide detail information about orders, product, payment, customer, seller, etc. This dataset spans a time range from October 2016 to October 2018, capturing transactional data within this period. Here is an overview of each tables and its relationship:

