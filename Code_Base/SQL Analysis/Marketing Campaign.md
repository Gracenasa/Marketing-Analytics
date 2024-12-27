### 1. Campaign Profitability
Profitability is defined by per-user profitability, customers attracted, gross revenue, net revenue, COG and sales (order quantity).

```TSQL
SELECT
    "AcquisitionSource",
    COUNT(DISTINCT("CustID")) as customer_base,
    TO_CHAR (
    	SUM("OrderQuantity"), '99,999'
    ) as sales, 
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
	"AcquisitionSource" <> 'EMPTY'
GROUP BY 
	"AcquisitionSource"
ORDER BY
	net_rev DESC;
```

| AcquisitionSource | Customer Base | Sales   | Gross Revenue | Net Revenue | Per-User Profitability | COG        |
|-------------------|---------------|---------|---------------|-------------|------------------------|------------|
| Google-ads        | 16,658        | 67,042  | $19,840,204   | $8,328,057  | 499.94                 | $11,512,147|
| Meta-ads          | 4,731         | 8,415   | $2,572,408    | $1,079,291  | 228.13                 | $1,493,116 |
| Yt-Campaign       | 4,756         | 8,496   | $2,478,366    | $1,039,928  | 218.66                 | $1,438,438 |

### 2. Customer Acquisition Cost
```TSQL
WITH RevenueData AS (
    SELECT 
        "AcquisitionSource",
        SUM("Amount") AS "Total Revenue",
        COUNT("OrderID") AS "Total Orders"
    FROM public."Orders"
    GROUP BY "AcquisitionSource"
),
CACData AS (
    SELECT 
        "AcquisitionSource",
        "Total Revenue",
        "Total Orders",
        ("Total Revenue" * 0.10 / NULLIF("Total Orders", 0)) AS "CAC" -- Assume 10% of revenue is acquisition spend
    FROM RevenueData
)
SELECT 
    "AcquisitionSource",
    TO_CHAR(
    	ROUND(("Total Revenue")::NUMERIC, 2), '$99,999,999'
    )AS "Total Revenue",
    TO_CHAR("Total Orders", '99,999'),
    TO_CHAR(
    	ROUND(("CAC")::NUMERIC, 2), '99.99%'
    )AS "CAC"
FROM CACData
ORDER BY "CAC" ASC;
```


### 3. Fraud rate for each campaign strategy

```TSQL
SELECT 
	*,
	(ROUND(fraudulent::NUMERIC,2)/ROUND(all_payments::NUMERIC,2))*100 AS fraud_rate
FROM ( 
	SELECT
		"AcquisitionSource",
		COUNT(
			CASE WHEN "Fraud" = 'FALSE' THEN "OrderID" END
		 	) AS authentic,
		COUNT(CASE WHEN "Fraud" = 'TRUE' THEN "OrderID" END
			) AS Fraudulent,
		COUNT(
			CASE WHEN "Fraud" = 'NA' THEN "OrderID" END
			) AS non_confirmed,
		COUNT("OrderID") as all_payments
	FROM public."Orders"
	GROUP BY "AcquisitionSource"
	)
GROUP BY
	"AcquisitionSource", authentic, fraudulent, all_payments, non_confirmed
ORDER BY 
	fraud_rate DESC;
```
| AcquisitionSource | Authentic | Fraudulent | Non-Confirmed | All Payments | Fraud Rate         |
|-------------------|-----------|------------|---------------|--------------|--------------------|
| Meta-ads          | 5,288     | 268        | 62            | 5,618        | 4.770380918476320  |
| Google-ads        | 41,964    | 2,107      | 578           | 44,649       | 4.719030661380994  |
| Yt-Campaign       | 5,349     | 230        | 63            | 5,642        | 4.076568592697625  |


### 4. Campaign Demographics

```TSQL
SELECT 
	"AcquisitionSource",
	"Country",
	COUNT("OrderID") as sales
FROM
	public."Orders"
GROUP BY
	"AcquisitionSource", 
	"Country"
ORDER BY 
	"AcquisitionSource", 
	sales DESC;
```

| AcquisitionSource | Country         | Sales  |
|-------------------|-----------------|--------|
| Google-ads        | United States   | 15,715 |
| Google-ads        | Australia       | 9,929  |
| Google-ads        | Canada          | 5,464  |
| Google-ads        | United Kingdom  | 5,111  |
| Google-ads        | Germany         | 4,235  |
| Google-ads        | France          | 4,195  |
| Meta-ads          | United States   | 2,040  |
| Meta-ads          | Australia       | 1,235  |
| Meta-ads          | Canada          | 686    |
| Meta-ads          | United Kingdom  | 632    |
| Meta-ads          | France          | 524    |
| Meta-ads          | Germany         | 501    |
| Yt-Campaign       | United States   | 2,012  |
| Yt-Campaign       | Australia       | 1,230  |
| Yt-Campaign       | Canada          | 700    |
| Yt-Campaign       | United Kingdom  | 662    |
| Yt-Campaign       | Germany         | 538    |
| Yt-Campaign       | France          | 500    |


### 5. Campaign Popularity Annually
```TSQL
SELECT 
	"AcquisitionSource",
	EXTRACT(YEAR FROM TO_DATE("OrderDate",'DD/MM/YYYY')) as "Year",
	COUNT(DISTINCT("OrderID")) as "Orders Made"
FROM 
	public."Orders"
WHERE 
	"AcquisitionSource" IS NOT NULL
GROUP BY
	"AcquisitionSource",
	"Year"
ORDER BY 
	"Year", "Orders Made" DESC;
```

