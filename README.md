# RFM-Based Customer Segmentation for Strategic Insights

This project focuses on customer segmentation using RFM (Recency, Frequency, Monetary) analysis, a proven method for identifying and categorizing customers based on their purchasing behavior. By assigning scores to each RFM metric and classifying customers into meaningful segments (e.g., Champions, Loyal Customers, At Risk, Lost), businesses can optimize marketing strategies, improve customer retention, and drive revenue growth.

### Questions that can be answered through the project:

- Who are our most valuable customers?
- Which customers are at risk of churning?
- Which customers used to buy frequently but have stopped?
- Which customers should receive re-engagement campaigns?
- Which customers are most likely to respond to upsell or cross-sell offers?
- How can we optimize loyalty programs for different customer segments?

------------------------------------------------------

## Finding Number of Customers, Orders, and Average Paid Price per Country

```sql
-- Finding # of customers, # of orders and the avg paid price per country 
  SELECT  DISTINCT country                           AS country,
          COUNT(DISTINCT invoice)                    AS Number_Of_Orders,
          COUNT(DISTINCT customer_id)                AS Number_Of_Customers,
          ROUND(AVG(price * quantity), 2)            AS Avg_Payment
    FROM  tableretail
GROUP BY  1
ORDER BY  2 DESC,		
          3 DESC;
```
***Output:***

|  COUNTRY    |  NUMBER_OF_ORDERS  |   NUMBER_OF_CUSTOMERS    |   AVG_PAYMENT    |
|     :---:    |     :---:      |     :---:      |     :---:      |
| United Kingdom        | 717           |       110     |  19.89         |

------------------------------------------------------

## Figuring out the number of orders per date

```sql
-- Figuring out the number of orders per date
  SELECT  DISTINCT invoicedate            AS date,
          COUNT(invoice)                  AS Number_Of_Orders
    FROM  tableretail
GROUP BY  1
ORDER BY  2 DESC;
```

***Output Sample:***

|  DATE    |  NUMBER_OF_ORDERS    |
|     :---:    |     :---:    |
| 11/17/2011 14:26 | 154 |
| 9/28/2011 15:21 | 141 |
| 11/8/2011 14:22 | 140 |
| 9/11/2011 14:15 | 139 |
| 12/2/2011 14:26 | 127 |
| 11/25/2011 11:41 | 125 |
| 9/11/2011 15:31 | 109 |
| 12/5/2011 11:49 | 108 |
| 11/13/2011 15:30 | 105 |
| 6/15/2011 13:25 | 99 |

------------------------------------------------------

## Figuring out the quantities ordered per date

```sql
-- Figuring out the quantities ordered per date
  SELECT  DISTINCT invoicedate            AS date,
          SUM(quantity)                   AS Total_Quantities_Per_Date
    FROM  tableretail
GROUP BY  1
ORDER BY  2 DESC;

```

***Output Sample:***

|  DATE    |  TOTAL_QUANTITIES_PER_DATE    |
|     :---:    |     :---:    |
| 8/4/2011 18:06 | 11,848 |
| 8/11/2011 15:58 | 6,098 |
| 10/27/2011 12:26 | 4,936 |
| 5/23/2011 13:08 | 3,863 |
| 11/9/2011 13:56 | 3,684 |
| 6/21/2011 10:53 | 2,668 |
| 9/9/2011 15:02 | 2,352 |
| 7/28/2011 17:17 | 2,064 |
| 9/28/2011 15:21 | 1,816 |
| 9/19/2011 13:39 | 1,788 |

------------------------------------------------------

## Figuring Out the Highest Revenue Made Throughout the Day

```sql
-- Figuring out the highest revenue made throughout the day
  SELECT  DISTINCT invoicedate            AS date,	
          SUM(price * quantity)           AS Total_Price_Of_Orders
    FROM  tableretail
GROUP BY  1
ORDER BY  2 DESC;
```

***Output Sample:***


|  DATE    |  TOTAL_PRICCE_OF_ORDERS    |
|     :---:    |     :---:    |
| 8/4/2011 18:06 | 18,841 |
| 8/11/2011 15:58 | 9,350 |
| 11/9/2011 13:56 | 4,961 |
| 2/14/2011 9:47 | 3,376 |
| 5/23/2011 13:08 | 2,844 |
| 3/24/2011 18:25 | 2,279 |
| 6/21/2011 10:53 | 2,222 |
| 11/17/2011 12:39 | 2,210 |
| 9/11/2011 14:15 | 2,027 |
| 9/22/2011 15:03 | 1,937 |

------------------------------------------------------

## Figuring Out the Highest Paid Customers per Interval

```sql
-- Figuring out the highest paid customers per interval
  SELECT  DISTINCT customer_id            AS customer_id,
          invoicedate                     AS date,
          SUM(price * quantity)           AS Total_Price
    FROM  tableretail
GROUP BY  1, 2
ORDER BY  3 DESC;
```
***Output Sample:***


|  CUSTOMER_ID    |  DATE    |  TOTAL_PRICE    |
|     :---:    |     :---:    |     :---:    |
| 12931 | 8/4/2011 18:06 | 18,841 |
| 12931 | 8/11/2011 15:58 | 9,350 |
| 12931 | 11/9/2011 13:56 | 4,961 |
| 12939 | 2/14/2011 9:47 | 3,376 | 
| 12901 | 5/23/2011 13:08 | 2,844 |
| 12901 | 3/24/2011 18:25 | 2,279 |
| 12830 | 6/21/2011 10:53 | 2,222 |
| 12931 | 11/17/2011 12:39 | 2,210 |
| 12748 | 9/11/2011 14:15 | 2,027 |
| 12906 | 9/22/2011 15:03 | 1,937 |

### Quick Insights:
- There's a total order of 717 and 110 customers, They are all based in the UK.
- The avg payment paid per order is 19.89 Pounds
- The majority of orders are made during morning to noon with an interval of approximately 09:00 AM to 4:30 PM.
- The highest amount paid also fits the same interval with an exception of a single occurrence **8/4/2011 18:06**
- Based on the quantities ordered and total paid price, Customers tend to order frequently multiple goods ***which we will try to validate throught the next step***



## Customer Segmentation using RFM

```sql
WITH rfm_base AS(
    SELECT   DISTINCT customer_id                              AS customer_id,
             MAX(TO_DATE(invoicedate, 'MM/DD/YYYY HH24:MI'))   AS Last_order_Date,
             COUNT(invoice)                                    AS order_count,
             SUM(price * quantity)                             AS total_price
      FROM   tableretail
  GROUP BY   1
),

rfm AS(
    SELECT   customer_id                                       AS customer_id,
             CURRENT_DATE - Last_order_Date                    AS recency,
             order_count                                       AS order_count,
             total_price                                       AS total_price
      FROM   rfm_base
),

customer_segment_base AS(
    SELECT   customer_id                                       AS customer_id,
             NTILE(5) OVER(ORDER BY recency)                   AS recency,
             NTILE(5) OVER(ORDER BY order_count)               AS frequency,
             NTILE(5) OVER(ORDER BY total_price)               AS monetary
      FROM   rfm
)

  SELECT   customer_id,
           recency,
           frequency,
           monetary,
           CASE WHEN (recency + frequency + monetary) = 15             THEN 'Champions'
                WHEN (recency + frequency + monetary) >= 12            THEN 'Loyal Customers'
                WHEN (recency + frequency + monetary) BETWEEN 9 AND 11 THEN 'Potential Loyalists'
                WHEN (recency + frequency + monetary) BETWEEN 6 AND 8  THEN 'Customers Needing Attention'
                WHEN (recency + frequency + monetary) BETWEEN 4 AND 5  THEN 'Hibernating'
                WHEN (recency + frequency + monetary) <= 3             THEN 'Lost'
           END AS customer_segment
     FROM  customer_segment_base
 ORDER BY  2 DESC,
           3 DESC,
           4 DESC;
```

***Final Output Sample:***


| customer_id | recency | frequency | monetary | customer_segment               |
|------------|---------|-----------|----------|--------------------------------|
| 12868      | 5       | 4         | 4        | Loyal Customers                |
| 12872      | 5       | 4         | 3        | Loyal Customers                |
| 12878      | 5       | 3         | 3        | Potential Loyalists            |
| 12857      | 5       | 3         | 3        | Potential Loyalists            |
| 12967      | 5       | 2         | 4        | Potential Loyalists            |
| 12845      | 5       | 2         | 2        | Potential Loyalists            |
| 12908      | 5       | 1         | 3        | Potential Loyalists            |
| 12891      | 5       | 1         | 2        | Customers Needing Attention    |

------------------------------------------------------

## 🔗 Get In Touch
<a href="mailto:mustafaa7med@gmail.com"><img alt="Gmail" title="Moustafa Ahmed's Gmail" src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"></a>
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/mustafaa7med)
