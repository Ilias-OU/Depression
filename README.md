# Depression Medication

![Depression's](https://plantationpsychiatrist.com/wp-content/uploads/2022/07/Psychiatry-Depression-Medication-Image-pdf.jpg)


## 📄 Overview


The project analyzes a depression dataset to explore factors like demographics, medication types, and treatment outcomes. Using SQL queries, it examines patterns and correlations, such as identifying individuals with high or low medication effectiveness, comparing data across different groups, and calculating averages. The goal is to understand how various factors influence depression treatment, which could lead to more tailored and effective therapeutic strategies. This analysis highlights key trends and insights into depression medication and patient response.

---

## 📂 Dataset

Access the dataset here: [Depression’s Dataset on Kaggle](https://www.kaggle.com/datasets/mpwolke/cusersmarildownloadsdepressioncsv).

## 🔍 SQL Query Analysis

#### 1. Retrieve All Data
```
SELECT *
FROM [depression];
```

#### 2. Retrieve Distinct Values of ct
```
SELECT DISTINCT ct
FROM depression;
```

####  3. Average Values of epad, epan, and emad Where emad > 50
```
SELECT AVG(epad), AVG(epan), AVG(emad)
FROM [depression]
WHERE emad > 50;
```

####  4.Top 3 Records with the Highest ewad for Each Range of ct
```
WITH cte1 AS (
    SELECT *, ROW_NUMBER() OVER (ORDER BY ewad DESC) AS Banking
    FROM [depression]
)
SELECT *
FROM cte1
WHERE Banking < 4;
```

#### 5. Running Total of epad and epan Ordered by ct
```
SELECT *,
       SUM(epad) OVER (ORDER BY ct) AS cumulativesumepad,
       SUM(epan) OVER (ORDER BY ct) AS cumulativesumepan
FROM [depression];
```

#### 6. Rows Where epad is Greater Than the Average epad Across the Dataset
```
SELECT *
FROM [depression]
WHERE epad > (SELECT AVG(epad) FROM [depression]);
```

#### 7. Join Derived Tables to Find ct Ranges Where Average epad Exceeds 1000
```
WITH cte1 AS (
    SELECT AVG(epad) AS avgepad, ct
    FROM [depression]
    GROUP BY ct
),
cte2 AS (
    SELECT MAX(ewan) AS maxewan, ct
    FROM [depression]
    GROUP BY ct
)
SELECT C.avgepad, C.ct
FROM cte1 C
JOIN cte2 T
ON C.ct = T.ct
WHERE C.avgepad > 1000;
```

####  8. Rows Where emad is in the Top 10% Based on the 90th Percentile
```
WITH Percentile AS (
    SELECT 
        PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY emad DESC) OVER () AS NinetyPercentile
    FROM depression
)
SELECT *
FROM depression d
WHERE d.emad >= (SELECT TOP 1 NinetyPercentile FROM Percentile);
```

####  9. Recursive CTE to Find Numbers Between 100,000 and 101,000 in ct
```
WITH createnumbers AS (
    SELECT 100000 AS Mynumber
    UNION ALL 
    SELECT Mynumber + 1
    FROM createnumbers
    WHERE Mynumber < 100999
)
SELECT C.Mynumber
FROM createnumbers C
JOIN [depression] D
ON C.Mynumber = D.ct
OPTION (MAXRECURSION 1000);
```

####  10. Identify Rows Where epad is at Least Twice epan and ewad is Less Than emad
```
SELECT *
FROM depression
WHERE epad >= 2 * epan 
AND ewad < emad;
```

####  11. Convert ct into Numeric and Group by Thousands, Find Maximum eman
```
ALTER TABLE depression 
ALTER COLUMN ct INT;

SELECT ct / 1000 AS thousand,
       MAX(eman)
FROM depression
GROUP BY ct / 1000;
```

#### 12. Pivot Data to Calculate Total Sum for Multiple Columns
```
SELECT 
    SUM(epad) AS total_epad,
    SUM(epan) AS total_epan,
    SUM(ewad) AS total_ewad,
    SUM(ewan) AS total_ewan,
    SUM(emad) AS total_emad,
    SUM(eman) AS total_eman
FROM depression;
```

####  13. Row Number Partitioned by epad Ranges and Ordered by ewan
```
SELECT *,
       ROW_NUMBER() OVER (PARTITION BY 
           CASE
               WHEN epad BETWEEN 0 AND 500 THEN '0-500'
               WHEN epad BETWEEN 500 AND 1000 THEN '500-1000'
               WHEN epad BETWEEN 1000 AND 1500 THEN '1000-1500'
               ELSE '+1501'
           END
           ORDER BY ewan)
FROM depression;
```

####  14. Divide ct into Quartiles and Compute Average eman for Each Quartile
```
WITH cte1 AS (
    SELECT 
        ct,
        NTILE(4) OVER (ORDER BY ct) AS Quartile,
        eman
    FROM [depression]
)
SELECT AVG(eman), Quartile
FROM cte1
GROUP BY Quartile;
```

####  15. Cartesian Join Between epad and emad and Filter Absolute Difference Less Than 100
```
SELECT A.epad, B.emad
FROM depression A
CROSS JOIN depression B
WHERE ABS(A.epad - B.emad) < 100;
```

####  16. Rank Records Based on emad in Descending Order, Show Top 5 Records
```
WITH cte1 AS (
    SELECT *, ROW_NUMBER() OVER (ORDER BY emad DESC) AS ranking
    FROM [depression]
)
SELECT *
FROM cte1
WHERE ranking < 6;
```

####  17.Use Grouping Sets to Calculate Sum of epad and ewad for Each Combination of ct Ranges and eman Range
```
SELECT SUM(epad), SUM(ewad), ct, eman 
FROM [depression]
GROUP BY ct, eman;
```

####  18. Categorize epad into Buckets and Calculate Record Count in Each Bucket
```
WITH cte1 AS (
    SELECT *,
           CASE 
               WHEN epad < 500 THEN 'Low'
               WHEN epad BETWEEN 500 AND 1000 THEN 'Medium'
               ELSE 'High'
           END AS categorisation
    FROM [depression]
)
SELECT COUNT(categorisation), categorisation
FROM cte1
GROUP BY categorisation;
```

####  19.Calculate Average epad and emad for ct Ranges and Identify Range with Largest Difference Between Averages
```
WITH cte1 AS (
    SELECT *, 
           (ct / 1000) * 1000 AS rangestart,
           ((ct / 1000) * 1000) + 999 AS rangend
    FROM depression
),
cte2 AS (
    SELECT AVG(epad) AS avgepad, AVG(emad) AS avgemad, rangestart, rangend
    FROM cte1
    GROUP BY rangestart, rangend
)
SELECT TOP 1 avgepad - avgemad AS difference, rangestart, rangend
FROM cte2
ORDER BY difference DESC;
```

####  20. Correlated Subquery to Find Rows Where epad is Greater Than the Average epad of Rows with a Smaller ct Value
```
SELECT *
FROM depression d
WHERE d.epad > (
    SELECT AVG(epad)
    FROM depression
    WHERE ct < d.ct
);
```

####  21. Rank Each Row by ewad, Group by Ranges of ct, and Identify Top-Ranked Row for Each Group
```
WITH cte1 AS (
    SELECT *,
           (ct / 1000) * 1000 AS rangestart,
           ((ct / 1000) * 1000) + 999 AS rangend,
           ROW_NUMBER() OVER (ORDER BY ewad) AS rank
    FROM [depression]
)
SELECT *
FROM cte1
WHERE rank = 1;
```

