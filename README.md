# Problem Definition

## Business Context
The company specializes in retail sales with a presence in various regions. The company aims to evaluate its sales performance as well as customer purchase behavior. The analysis of the company is carried out by the Sales Analytics Department.

## Data Challenge
The company lacks clear visibility about the top-performing products, inactive customers, sales trends over a period of time, and customer segmentation based on revenue. The reports that are being generated are not giving any analytical insights about the sales transactions.

## Expected Outcome
The company aims to achieve analytical insights that would help the company in identifying the top products, sales trends, identifying inactive customers, and customer segmentation.

# Success Criteria 
## Rank products by revenue per region: 


This query ranks products by revenue within each region, allowing management to identify top-performing and low-performing products. The ranking functions handle ties differently, making them flexible for use in performance evaluation.

a) Running Monthly Sales Total (ROWS):
Interpretation
This query calculates cumulative sales over time for each region, which helps management analyze sales growth trends.

(b) Three-Month Moving Average (RANGE):
Interpretation
This query helps management smooth out short-term variations by calculating a moving average of sales over three months.

Month-over-month growth comparison:
Interpretation
This query calculating a month-to-month sales growth. This helps management to quickly identify sales performance increases or decreases.

Customer revenue segmentation
Interpretation
The query helps segment customers by their revenue contribution, enables businesses to focus marketing efforts on their high-value customers.

Step 3: Database Schema Design Design at least three (3) related tables with primary and foreign keys.

Step 4: Part A — SQL JOINs Implementation
INNER JOIN Retrieve transactions with valid customers and products:
Business Interpretation
The query only displays confirmed sales transactions for which customers, orders, and products exist. This query enables management to analyze what customers have bought, where they are located, and how much revenue is being generated.

LEFT JOIN Identify customers who have never made a transaction
Business Interpretation
This query shows customers who have never ordered from the business. They may be contacted with offers.

RIGHT JOIN Detect products with no sales activity:

Business Interpretation

The query shows products that have zero sales, helping management make a decision whether to withdraw such products or not.

FULL OUTER JOIN Compare customers and products including unmatched records
Business Interpretation

This query will provide a whole picture of the business, including customers who have not made any purchases and products that are not selling.


SELF JOIN Compare customers within the same region

Business Interpretation

This self-join query can help compare customers based on regions, which can help the business understand customer concentration in different regions.

