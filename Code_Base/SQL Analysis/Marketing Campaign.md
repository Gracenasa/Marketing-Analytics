## 1. Campaign Profitability
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

[untitled.csv](https://github.com/user-attachments/files/18262663/untitled.csv)

