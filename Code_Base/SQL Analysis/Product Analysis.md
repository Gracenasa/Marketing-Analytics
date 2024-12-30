### 1. Annual Product Performance
```TSQL
SELECT
    "Product_Category",
    COUNT(DISTINCT("CustID")) as customer_base,
    COUNT(DISTINCT("OrderID")) as "sales (orders made)",
    TO_CHAR (
    	SUM("OrderQuantity"), '99,999'
    ) as "sales (quantity sold)",
    TO_CHAR (
        ROUND(
        	SUM("Amount") :: NUMERIC, 2
        	),'$99,999,999'
    ) as gross_rev,
    TO_CHAR (
        ROUND (
            (SUM("Amount") - SUM("OrderQuantity" * "ProductCost")
            ) :: NUMERIC,2
        	),'$9,999,999'
    ) as net_rev,
   	ROUND( (
   		(SUM("Amount") - SUM("OrderQuantity" * "ProductCost"))/COUNT(DISTINCT("CustID")))::NUMERIC,2
   	)as "Per-user profitability",
    TO_CHAR (
        ROUND (   			
   			(SUM("OrderQuantity"*"ProductCost")
   			):: NUMERIC,2
        	),'$99,999,999'
    ) as "COG"
FROM 
	public."Orders"
WHERE 
	"Product_Category" IS NOT NULL AND
	"Product_Category" <> 'EMPTY'
GROUP BY 
	"Product_Category"
ORDER BY
	net_rev DESC;
```
| Product Category      | Customer Base | Sales (Orders Made) | Sales (Quantity Sold) | Gross Revenue | Net Revenue | Per-User Profitability | COG        |
|-----------------------|---------------|----------------------|------------------------|---------------|-------------|------------------------|------------|
| Plants                | 8786          | 13913                | 13,913                 | $23,622,281   | $9,717,722  | 1106.05                | $13,904,559|
| Plant Care & Seeds    | 14256         | 16936                | 57,634                 | $903,981      | $568,069    | 39.85                  | $335,912   |
| Pots                  | 6437          | 6959                 | 12,406                 | $364,717      | $161,486    | 25.09                  | $203,231   |


### 2. Regional Product Popularity
```TSQL
SELECT
	"Region",
	"Product_Category",
	TO_CHAR (
        ROUND(
        	SUM("Amount") :: NUMERIC, 2
        	),'$99,999,999'
    ) as gross_rev,
    TO_CHAR (
        ROUND (
            (SUM("Amount") - SUM("OrderQuantity" * "ProductCost")
            ) :: NUMERIC,2
        	),'$9,999,999'
    ) as net_rev,
   	ROUND( (
   		(SUM("Amount") - SUM("OrderQuantity" * "ProductCost"))/COUNT(DISTINCT("CustID")))::NUMERIC,2
   	)as "Per-user profitability",
    TO_CHAR (
        ROUND (   			
   			(SUM("OrderQuantity"*"ProductCost")
   			):: NUMERIC,2
        	),'$99,999,999'
    ) as "COG"
FROM 
	public."Orders"
WHERE 
	"Product_Category" IS NOT NULL AND
	"Product_Category" <> 'EMPTY'
GROUP BY 
	"Product_Category", "Region"
ORDER BY
	"Product_Category", net_rev DESC;
```

The output is 30 rows long, thus isn't included
