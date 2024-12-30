### 1. Top 3 customers annually by orders made and amount spent
```TSQL
WITH CustomerMetrics AS (
    SELECT 
        "Customer_Name" AS Customer_name,
        EXTRACT(YEAR FROM TO_DATE("OrderDate", 'DD/MM/YYYY')) AS Order_Year,
        COUNT(DISTINCT("OrderID")) AS Total_Orders,
        SUM("Amount") AS Total_Amount_Spent
    FROM public."Orders"
    GROUP BY Customer_name, Order_Year
),
RankedByOrders AS (
    SELECT 
        Customer_name,
        Order_Year,
        Total_Orders,
        Total_Amount_Spent,
        ROW_NUMBER() OVER (PARTITION BY Order_Year ORDER BY Total_Orders DESC) AS Order_Rank
    FROM CustomerMetrics
),
RankedByAmount AS (
    SELECT 
        Customer_name,
        Order_Year,
        Total_Orders,
        Total_Amount_Spent,
        ROW_NUMBER() OVER (PARTITION BY Order_Year ORDER BY Total_Amount_Spent DESC) AS Amount_Rank
    FROM CustomerMetrics
)
-- Top 3 Customers by Number of Orders
SELECT 
    Customer_name,
    Order_Year,
    Total_Orders,
    Total_Amount_Spent,
    'Orders' AS Ranking_Type
FROM RankedByOrders
WHERE Order_Rank <= 3
UNION ALL

-- Top 3 Customers by Amount Spent
SELECT 
    Customer_name,
    Order_Year,
    Total_Orders,
    Total_Amount_Spent,
    'Amount' AS Ranking_Type
FROM RankedByAmount
WHERE Amount_Rank <= 3
ORDER BY Order_Year, Ranking_Type, Total_Orders DESC, Total_Amount_Spent DESC;
```
| customer_name              | order_year | total_orders | total_amount_spent      | ranking_type  |
|--------------------|------|-------|-------------|--------|
| RAFAEL XU          | 2020 | 2     | 4624.9125   | Amount |
| RUBEN PATEL        | 2020 | 1     | 3578.27     | Amount |
| LANCE GILL          | 2020 | 1     | 3578.27     | Amount |
| RAFAEL XU          | 2020 | 2     | 4624.9125   | Orders |
| SCOTT RODGERS      | 2020 | 1     | 3578.27     | Orders |
| TIFFANY LI         | 2020 | 1     | 3578.27     | Orders |
| JANET MUNOZ        | 2021 | 5     | 9597.6887   | Amount |
| MAURICE SHAN       | 2021 | 4     | 8755.0445   | Amount |
| KAITLYN HENDERSON  | 2021 | 3     | 6846.8949   | Amount |
| SAMANTHA JENKINS   | 2021 | 17    | 1084.9853   | Orders |
| MASON ROBERTS      | 2021 | 14    | 975.5973    | Orders |
| APRIL SHAN         | 2021 | 13    | 791.6015    | Orders |
| JORDAN TURNER      | 2022 | 5     | 6801.9568   | Amount |
| FRANKLIN XU        | 2022 | 4     | 7982.7797   | Amount |
| WILLIE XU          | 2022 | 3     | 6577.5578   | Amount |
| ASHLEY HENDERSON   | 2022 | 20    | 1260.4429   | Orders |
| FERNANDO BARNES    | 2022 | 17    | 1250.2608   | Orders |
| NANCY CHAPMAN      | 2022 | 17    | 823.5372    | Orders |


### 2. Year-over-Year (YoY) growth rate by customer count
```TSQL
SELECT
    "Year",
    "Customers",
    TO_CHAR (
    	ROUND(
        ("Customers" - LAG("Customers") OVER (ORDER BY "Year")) * 100.0 /
        LAG("Customers") OVER (ORDER BY "Year"),
        2), '999.99%'
    ) AS "%GE Increase"
FROM
    (
        SELECT
            EXTRACT(YEAR FROM TO_DATE("OrderDate", 'DD/MM/YYYY')) AS "Year",
            COUNT(DISTINCT "CustID") AS "Customers"
        FROM
            public."Orders"
        GROUP BY
            "Year"
    ) AS subquery
ORDER BY
    "Year" ASC;
```

| Year | Customers | %GE Increase|
|------|-----------|---------------------|
| 2020 | 2,630     | NULL                |
| 2021 | 9,091     | 245.67%             |
| 2022 | 10,502    | 15.52%              |


### 3. Customer distibution per region
