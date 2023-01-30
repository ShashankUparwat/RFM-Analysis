# RFM-Analysis
RFM Data Analysis in SQL and Tableau

##### R - Recency
##### F - Frequency
##### M - Monetary

##### RFM analysis is a marketing technique used to quantitatively rank and group customers based on the recency, frequency and monetary total of their recent transactions to identify the best customers and perform targeted marketing campaigns.


## SQL Code

#### Inspecting Data
```SQL
SELECT *
FROM [dbo].[sales_data];
```
#### hecking Unique Values
```SQL
SELECT DISTINCT STATUS FROM dbo.sales_data;--------Plot

SELECT DISTINCT YEAR_ID FROM dbo.sales_data;
SELECT DISTINCT PRODUCTLINE FROM dbo.sales_data;---Plot
SELECT DISTINCT COUNTRY FROM dbo.sales_data;-------Plot
SELECT DISTINCT DEALSIZE FROM dbo.sales_data;------Plot
SELECT DISTINCT TERRITORY FROM dbo.sales_data;-----Plot
```

## Analysis

### Total sales by Productline

```SQL
SELECT PRODUCTLINE, SUM(sales) AS Revenue
FROM [dbo].[sales_data]
GROUP BY PRODUCTLINE
ORDER BY 2 DESC;
```

#### Total sales by Year
```SQL
SELECT YEAR_ID, SUM(sales) AS Revenue
FROM [dbo].[sales_data]
GROUP BY YEAR_ID
ORDER BY 2 DESC;
```

#### Operating Months in each Year
```SQL
SELECT DISTINCT MONTH_ID
FROM [dbo].[sales_data]
WHERE YEAR_ID = 2004;
```

#### Total Sales by Deal size
```SQL
SELECT DEALSIZE, SUM(sales)
FROM [dbo].[sales_data]
GROUP BY DEALSIZE
ORDER BY 2 DESC;
```

#### Best month for sale in a specific year and total revenue earned respectively
```SQL
SELECT MONTH_ID, SUM(sales) as Revenue, COUNT(ORDERNUMBER) AS Total_Orders, SUM(QUANTITYORDERED) AS Quantity
FROM [dbo].[sales_data]
WHERE YEAR_ID = 2004---Change Year
GROUP BY MONTH_ID
ORDER BY 2 DESC;
```

#### November is the most profitable month in the year 2003 and 2004 respectively

#### Products sold in November
```SQL
SELECT MONTH_ID, PRODUCTLINE, SUM(sales), COUNT(ORDERNUMBER) AS Total_Orders, SUM(QUANTITYORDERED) AS Quantity
FROM [dbo].[sales_data]
WHERE MONTH_ID = 11 ---Change month
GROUP BY MONTH_ID, PRODUCTLINE
ORDER BY 3 DESC;
```

## RFM Analysis 

#### Create CTE 
```SQL
DROP TABLE IF EXISTS #rfm;
WITH rfm as
(
SELECT CUSTOMERNAME,
  SUM(sales) AS Monetary_Value,
  AVG(sales) AS Avg_Monetary_Value,
  COUNT(ORDERNUMBER) AS Frequency,
  MAX(ORDERDATE) AS Last_Order_Date,
  (
SELECT MAX(ORDERDATE)
FROM [dbo].[sales_data]
  ) AS Max_Order_Date,
  DATEDIFF(DD, MAX(ORDERDATE), (SELECT MAX(ORDERDATE) FROM [dbo].[sales_data]))  AS Recency
FROM [RFMAnalysis].[dbo].[sales_data]
GROUP BY CUSTOMERNAME
),
rfm_calc as
(
SELECT r.*,
NTILE(4) OVER (ORDER BY Recency DESC) AS rfm_Recency,
NTILE(4) OVER (ORDER BY Frequency) AS rfm_Frequency,
NTILE(4) OVER (ORDER BY Monetary_Value) AS rfm_Monetary
FROM rfm r
)
SELECT c.*, rfm_Recency+rfm_Frequency+rfm_Monetary as rfm_cell,
CAST(rfm_Recency AS varchar)+CAST(rfm_Frequency AS varchar)+CAST(rfm_Monetary AS varchar) AS rfm_cell_string
INTO #rfm
FROM rfm_calc AS c
```

#### Temporary table created
```SQL
SELECT* FROM #rfm
```

#### Best Customer Case Scenario
```SQL
SELECT CUSTOMERNAME, rfm_Recency, rfm_Frequency, rfm_Monetary,
CASE
WHEN rfm_cell_string IN (111, 112, 121, 122, 123, 132, 211, 212, 114, 141) THEN 'Lost Customer'
WHEN rfm_cell_string IN (133, 134, 143, 144, 244, 243, 234, 334, 343, 344) THEN 'Slipping Away' --- Big spenders who haven't purchased lately
WHEN rfm_cell_string IN (311, 411, 331) THEN 'New Customer'
WHEN rfm_cell_string IN (222, 223, 233, 322) THEN 'Potential Churners'
WHEN rfm_cell_string IN (323, 333, 321, 422, 332, 432) THEN 'Active' --- Customers who buy often but at a low price
WHEN rfm_cell_string IN (433, 434, 443, 444) THEN 'Loyal'
END rfm_segment
FROM #rfm
```

#### Orders Shipped
```SQL
SELECT ORDERNUMBER, COUNT(*) as Total_Items
FROM [dbo].[sales_data]
WHERE STATUS = 'Shipped'
GROUP BY ORDERNUMBER
```

#### Products bought by customers
```SQL
SELECT *
FROM [dbo].[sales_data]
WHERE ORDERNUMBER = 10411
ORDER BY 4 ASC
```

#### Products sold together
```SQL
SELECT DISTINCT ORDERNUMBER, STUFF(

(SELECT ',' + PRODUCTCODE
FROM [dbo].[sales_data] AS P
WHERE ORDERNUMBER IN
 (
SELECT ORDERNUMBER
FROM(
 SELECT ORDERNUMBER, COUNT(*) as Total_Items
 FROM [dbo].[sales_data]
 WHERE STATUS = 'Shipped'
 GROUP BY ORDERNUMBER
) as Orders
WHERE Total_Items = 2
 )
 AND p.ORDERNUMBER = S. ORDERNUMBER
 FOR XML PATH ('')),
 1, 1, '') AS Product_Codes
FROM [dbo].[sales_data] AS S
ORDER BY 2 DESC
```


## Tableau Dashboards

#### Dashboard 1
```Tableau
https://public.tableau.com/views/SalesDashboard1_16750276362460/SalesDashboard1?:language=en-GB&:display_count=n&:origin=viz_share_link
```

#### Dashboard 2
```Tableau
https://public.tableau.com/views/SalesDashboard1_16750274816340/SalesDashboard2?:language=en-GB&:display_count=n&:origin=viz_share_link
```
