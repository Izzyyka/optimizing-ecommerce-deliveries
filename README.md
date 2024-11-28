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
The dataset  that we used in this analysis represents  a brazilian e-commerce platform’s (Olist) relational structure, consisting of several interconnected tables that provide detail information about orders, product, payment, customer, seller, etc. This dataset spans a time range from October 2016 to October 2018, capturing transactional data within this period. The relational structure of the dataset is illustrated in the diagram below:

![OverviewImage](olist_ecommerce.png)

## Q1: What is the percentage of delayed deliveries compared to all deliveries? 

```sql
SELECT 
    YEAR(order_delivered_customer_date) AS year,
    COUNT(order_id) AS total_orders,
    COUNT(CASE WHEN order_delivered_customer_date > order_estimated_delivery_date 
    THEN 1 END) AS delayed_orders,
    -- menghitung delay_percentage
    COUNT(CASE WHEN order_delivered_customer_date > order_estimated_delivery_date 
    THEN 1 END) * 100.0 / COUNT(*) AS delayed_percentage,
    COUNT(CASE WHEN order_delivered_customer_date <= order_estimated_delivery_date 
    THEN 1 END) * 100.0 / COUNT(*) AS on_time_percentage
FROM 
    olist_orders_dataset
WHERE 
    order_status = 'delivered'
    AND order_delivered_customer_date IS NOT NULL
    AND order_estimated_delivery_date IS NOT NULL
GROUP BY 
    YEAR(order_delivered_customer_date)
ORDER BY 
    year;
```
Output:
|year |total_orders |delayed_percentage	|on_time_percentage |average_delay_days|
|---|---|---|---|---|
|2016	|267 |0.01498127341	|0.9850187266	|13 |
|2017	|40930 |0.05245541168	|0.9475445883	|8 |
|2018	|55273 |0.1026721908	|0.8973278092	|9 |

## Q2: What is the average delay time compared to the estimated delivery date?  

```sql
SELECT 
	YEAR(order_delivered_customer_date) AS year,
	-- menghitung rata-rata keterlambatan (berapa hari)
	AVG(DATEDIFF(DAY, order_estimated_delivery_date, order_delivered_customer_date)) 
	AS average_delay_days
FROM 
	olist_orders_dataset
WHERE
	order_status = 'delivered'
	AND order_delivered_customer_date > order_estimated_delivery_date
	AND order_delivered_customer_date IS NOT NULL
	AND order_estimated_delivery_date IS NOT NULL
GROUP BY
	YEAR(order_delivered_customer_date)
```
Otput:
|year	|average_delay_days |
|---|---|
|2016	|13 |
|2017	|8 |
|2018	|9 |
### Insight
Based on the data, there has been a significant increase in the total number of orders year-over-year, along with a noticeable rise in delivery delays. In 2016, there were **267 orders** with a delivery delay rate of just **1.5%** and an average delay of **13 days**. This number grew to **40,930 orders in 2017**, with a delay rate of** 5.2%** and an average delay of **8 days**. **In 2018, the orders reached 55,273**, with a delay rate of** 10.3%** and an average delay of **9 days**. The increase in delayed orders from 4 in 2016 to 5,675 in 2018 suggests that **the rise in demand has not been fully supported by an increase in logistics capacity**. Given the higher delay percentage and the larger order volume in 2018, it's important to evaluate the factors contributing to these delays and find ways to address them.

## Q3: What is the relationship between product dimensions and delivery delays?
```sql
-- menentukan perhitungan volume product
WITH ProductVolume AS (
    SELECT 
        product_id,
        product_length_cm * product_height_cm * product_width_cm AS product_volume
    FROM 
        olist_products_dataset
),
-- menentukan nilai kuartil dari persebaran volume product
VolumeQuartiles AS (
    SELECT 
        P.product_id,
        P.product_volume,
        NTILE(4) OVER (ORDER BY P.product_volume) AS quartile
    FROM 
        ProductVolume P
),
-- menentukan kategori product berdasarkan volumenya
ProductCategories AS (
    SELECT 
        product_id,
        product_volume,
        CASE 
            WHEN quartile = 1 THEN 'Small'
            WHEN quartile BETWEEN 2 AND 3 THEN 'Medium'
            WHEN quartile = 4 THEN 'Large'
        END AS dimension_category
    FROM 
        VolumeQuartiles
),
-- menentukan jumlah product yang delay
OrderDelays AS (
    SELECT 
        O.order_id,
        O.order_status,
        O.order_estimated_delivery_date,
        O.order_delivered_carrier_date,
        CASE 
            WHEN O.order_delivered_customer_date > O.order_estimated_delivery_date THEN 1
            ELSE 0
        END AS is_delayed
    FROM 
        olist_orders_dataset O
    WHERE 
        O.order_status = 'delivered'
        AND O.order_delivered_carrier_date IS NOT NULL
        AND O.order_estimated_delivery_date IS NOT NULL
        AND YEAR(O.order_delivered_customer_date) = 2018 -- Filter tahun 2018
)
-- menentukan jumlah product yang terlambat berdasarkan kategori volumenya
	SELECT 
        PC.dimension_category,
        COUNT(OD.is_delayed) AS total_orders,
        SUM(OD.is_delayed) AS delayed_orders,
        ROUND((CAST(SUM(OD.is_delayed) AS FLOAT) / COUNT(OD.is_delayed))*100,2) 
        AS delay_percentage
    FROM 
        OrderDelays OD
    JOIN olist_order_items_dataset OI ON OD.order_id = OI.order_id
    JOIN ProductCategories PC ON OI.product_id = PC.product_id
    GROUP BY 
        PC.dimension_category
```
Output:
|dimension_category	|total_orders	|delayed_orders	|delay_percentage |
|---|---|---|---|
|Large	|14614	|1617	|11.06 |
|Medium	|31393	|3039	|9.68 |
|Small	|17062	|1633	|9.57 |

