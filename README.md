# Deforestation_SQL_Udacity
This is the project in SQL nanodegree program from Udacity

## Table of contents
1. The pdf file **Pranay_project_deforestation_exploration_solution.pdf** is the project report.
2. **images** folder contains images of the outputs of all the queries which I have ran.

## Introduction
- _Understand which countries and regions around the world seem to have forests that have been shrinking in size_.
- _Which countries and regions have the most significant forest area both in terms of total area and % of total area_.

## Aim
Making informed and well motivated decisions with respect to making the biggest impact given the few resources available.

## Create View
```sql
CREATE OR REPLACE VIEW forestation
AS
SELECT f.country_code AS forest_cc,
        f.country_name AS f_name,
        f.year AS f_year,
         f.forest_area_sqkm AS f_sq_km,
         l.total_area_sq_mi AS l_total_area_sq_mi,
         r.region AS r_region, r.income_group AS r_income_group,
          (f.forest_area_sqkm/(l.total_area_sq_mi*2.59))*100 AS perc_forest_area
  FROM forest_area f
  Join land_area l
  ON f.country_code = l.country_code
  JOIN regions r
  ON l.country_code = r.country_code
  WHERE f.year = l.year ORDER BY 1;
```

## Part 1: Global suituation

- _What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can use the country record denoted as "World" in the region table.?_

```sql
SELECT f.forest_area_sqkm
		FROM forest_area f
        WHERE f.country_name = 'World'
        	AND f.year = 1990;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic1.png)

- _What was the total forest area (in sq km) of the world in 2016? Please keep in mind that you can use the country record in the table is denoted as "World"?_
```sql
SELECT f.forest_area_sqkm
		FROM forest_area f
        WHERE f.country_name = 'World'
        	AND f.year = 2016;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic2.png)

- _What was the change (in sq km) in the forest area of the world from 1990 to 2016?_
```sql
SELECT sub1.forest_area_sqkm - sub2.forest_area_sqkm AS diff_forest_area_sq_km
      FROM (SELECT f.country_code AS cc, f.forest_area_sqkm
      	    FROM forest_area f
              WHERE f.country_name = 'World'
              	AND f.year = 1990) AS sub1
      JOIN (SELECT f.country_code AS cc,f.forest_area_sqkm
      		FROM forest_area f
              WHERE f.country_name = 'World'
              	AND f.year = 2016) AS sub2
      ON sub1.cc = sub2.cc;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic3.png)

- _What was the percent change in forest area of the world between 1990 and 2016?_
```sql
SELECT ((sub1.forest_area_sqkm-sub2.forest_area_sqkm)/sub1.forest_area_sqkm)*100  AS perc_change_fa
      FROM (SELECT f.country_code AS cc, f.forest_area_sqkm
      	    FROM forest_area f
              WHERE f.country_name = 'World'
              	AND f.year = 1990) AS sub1
      JOIN (SELECT f.country_code AS cc,f.forest_area_sqkm
      		FROM forest_area f
              WHERE f.country_name = 'World'
              	AND f.year = 2016) AS sub2
      ON sub1.cc = sub2.cc;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic4.png)

- _If you compare the amount of forest area lost between 1990 and 2016, to which country's total area in 2016 is it closest to?_
```sql
SELECT l.country_name,
       l.total_area_sq_mi*2.59 AS total_area_sqkm,
       ABS((l.total_area_sq_mi*2.59)- (SELECT sub1.forest_area_sqkm - sub2.forest_area_sqkm AS diff_forest_area_sq_km
                                       FROM (SELECT f.country_code AS cc, f.forest_area_sqkm
      	                                     FROM forest_area f
                                             WHERE f.country_name = 'World'
              	                             AND f.year = 1990) AS sub1
                                       JOIN (SELECT f.country_code AS cc,f.forest_area_sqkm
      		                                   FROM forest_area f
                                             WHERE f.country_name = 'World'
              	                              AND f.year = 2016) AS sub2
                                        ON sub1.cc = sub2.cc)) AS diff_fa_la_sqkm
    FROM land_area l
    WHERE l.year = 2016
    ORDER BY 3 LIMIT 1;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic5.png)

## Part 2: Regional Outlook
- _Create a table that shows the Regions and their percent forest area (sum of forest area divided by sum of land area) in 1990 and 2016. (Note that 1 sq mi = 2.59 sq km)._
```sql
CREATE OR REPLACE VIEW regional_distr
AS
SELECT r.region,
       l.year,
       SUM(f.forest_area_sqkm) total_forest_area_sqkm,
       SUM(l.total_area_sq_mi*2.59) AS total_area_sqkm,
        (SUM(f.forest_area_sqkm)/SUM(l.total_area_sq_mi*2.59))*100 AS percent_fa_region
      FROM forest_area f
      JOIN land_area l
      ON f.country_code = l.country_code AND f.year = l.year
      JOIN regions r
      ON l.country_code = r.country_code
      GROUP BY 1,2
      ORDER BY 1,2;
```
- _What was the percent forest of the entire world in 2016?_ 
```sql
SELECT ROUND(CAST(percent_fa_region AS numeric),2) AS percent_fa_region
	   FROM regional_distr
     WHERE year = 2016 AND region = 'World';
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic1_part1.png)

- _Which region had the HIGHEST percent forest in 2016, and which had the LOWEST, to 2 decimal places?_
```sql
SELECT region,
       ROUND(CAST(total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
       ROUND(CAST(percent_fa_region AS NUMERIC),2) AS percent_fa_region
       FROM regional_distr
       WHERE ROUND(CAST(percent_fa_region AS NUMERIC),2) = (SELECT MAX(
                                                                        ROUND(
                                                                               CAST(percent_fa_region AS numeric),2
                                                                             )
                                                                       ) AS max_percent
                                      	   					         FROM regional_distr
                                                             WHERE year = 2016
                                                            )
              AND year=2016;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic1_part2.png)
```sql
SELECT region,
      ROUND(CAST(total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
      ROUND(CAST(percent_fa_region AS NUMERIC),2) AS percent_fa_region
      FROM regional_distr
      WHERE ROUND(CAST(percent_fa_region AS NUMERIC),2) = (SELECT MIN(
                                                                      ROUND(
                                                                            CAST(percent_fa_region AS numeric),2
                                                                                )
                                                                      ) AS max_percent
                                           	   					         FROM regional_distr
                                                                 WHERE year = 2016
                                                             )
            AND year = 2016;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic1_part3.png)

- _What was the percent forest of the entire world in 1990?_
```sql
SELECT ROUND(CAST(percent_fa_region AS numeric),2) AS percent_fa_region
	   FROM regional_distr
     WHERE year = 1990 AND region = 'World';
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic2_part1.png)

- _Which region had the HIGHEST percent forest in 1990, and which had the LOWEST, to 2 decimal places?_

```sql
     SELECT region,
            ROUND(CAST(total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
            ROUND(CAST(percent_fa_region AS NUMERIC),2) AS percent_fa_region
            FROM regional_distr
            WHERE ROUND(CAST(percent_fa_region AS NUMERIC),2) = (SELECT MAX(
                                                                             ROUND(
                                                                                    CAST(percent_fa_region AS numeric),2
                                                                                  )
                                                                            ) AS max_percent
                                           	   					         FROM regional_distr
                                                                  WHERE year = 1990
                                                                 )
                   AND year=1990;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic2_part2.png)

```sql
     SELECT region,
           ROUND(CAST(total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
           ROUND(CAST(percent_fa_region AS NUMERIC),2) AS percent_fa_region
           FROM regional_distr
           WHERE ROUND(CAST(percent_fa_region AS NUMERIC),2) = (SELECT MIN(
                                                                           ROUND(
                                                                                 CAST(percent_fa_region AS numeric),2
                                                                                     )
                                                                           ) AS max_percent
                                                	   					         FROM regional_distr
                                                                      WHERE year = 1990
                                                                  )
                 AND year = 1990;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic2_part3.png)

- _Based on the table you created, which regions of the world DECREASED in forest area from 1990 to 2016?_
```sql
WITH table1990 AS (SELECT * FROM regional_distr WHERE year =1990),
	   table2016 AS (SELECT * FROM regional_distr WHERE year = 2016)
SELECT table1990.region,
       ROUND(CAST(table1990.percent_fa_region AS NUMERIC),2) AS fa_1990,
       ROUND(CAST(table2016.percent_fa_region AS NUMERIC),2) AS fa_2016
    FROM table1990 JOIN table2016 ON table1990.region = table2016.region
    WHERE table1990.percent_fa_region > table2016.percent_fa_region;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/RO_pic3.png)

## Part 3: Country Oulook
- _Which 5 countries saw the largest amount decrease in forest area from 1990 to 2016? What was the difference in forest area for each?_
```sql
WITH table1990 AS (SELECT f.country_code,
                          f.country_name,
                          f.year,
                          f.forest_area_sqkm
	                     FROM forest_area f
                       WHERE f.year = 1990 AND f.forest_area_sqkm IS NOT NULL AND f.country_name != 'World'
                     ),

      table2016 AS (SELECT f.country_code,
                           f.country_name,
                           f.year,
                           f.forest_area_sqkm
	                     FROM forest_area f
                       WHERE f.year = 2016 AND f.forest_area_sqkm IS NOT NULL AND f.country_name != 'World'
                     )

 SELECT table1990.country_code,
        table1990.country_name,
        r.region,
        table1990.forest_area_sqkm AS fa_1990_sqkm,
        table2016.forest_area_sqkm AS fa_2016_sqkm,
        table1990.forest_area_sqkm-table2016.forest_area_sqkm AS diff_fa_sqkm
      FROM table1990
      JOIN table2016
      ON table1990.country_code = table2016.country_code
      AND (table1990.forest_area_sqkm IS NOT NULL AND table2016.forest_area_sqkm IS NOT NULL)
      JOIN regions r ON table2016.country_code = r.country_code
      ORDER BY 6 DESC
      LIMIT 5;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/CO_pic1.png)

- _Which 5 countries saw the largest percent decrease in forest area from 1990 to 2016? What was the percent change to 2 decimal places for each_
```sql
WITH table1990 AS (SELECT f.country_code,
                          f.country_name,
                          f.year,
                          f.forest_area_sqkm
	                     FROM forest_area f
                       WHERE f.year = 1990 AND f.forest_area_sqkm IS NOT NULL AND f.country_name != 'World'
                     ),

      table2016 AS (SELECT f.country_code,
                           f.country_name,
                           f.year,
                           f.forest_area_sqkm
	                     FROM forest_area f
                       WHERE f.year = 2016 AND f.forest_area_sqkm IS NOT NULL AND f.country_name != 'World'
                     )

 SELECT table1990.country_code,
        table1990.country_name,
        r.region,
        table1990.forest_area_sqkm AS fa_1990_sqkm,
        table2016.forest_area_sqkm AS fa_2016_sqkm,
        table1990.forest_area_sqkm-table2016.forest_area_sqkm AS diff_fa_sqkm,
        ABS(ROUND(CAST(((table2016.forest_area_sqkm-table1990.forest_area_sqkm)/table1990.forest_area_sqkm*100) AS NUMERIC),2)) AS perc_change
      FROM table1990
      JOIN table2016
      ON table1990.country_code = table2016.country_code
      AND (table1990.forest_area_sqkm IS NOT NULL AND table2016.forest_area_sqkm IS NOT NULL) JOIN regions r ON table2016.country_code = r.country_code
      ORDER BY ROUND(CAST(((table2016.forest_area_sqkm-table1990.forest_area_sqkm)/table1990.forest_area_sqkm*100) AS NUMERIC),2)
      LIMIT 5;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/CO_pic2.png)

- _If countries were grouped by percent forestation in quartiles, which group had the most countries in it in 2016?_
```sql
With table1 AS (SELECT f.country_code,
                       f.country_name,
                       f.year,
                       f.forest_area_sqkm,
                       l.total_area_sq_mi*2.59 AS total_area_sqkm,
                        (f.forest_area_sqkm/(l.total_area_sq_mi*2.59))*100 AS perc_fa
                        FROM forest_area f
                        JOIN land_area l
                        ON f.country_code = l.country_code
                        AND (f.country_name != 'World' AND f.forest_area_sqkm IS NOT NULL AND l.total_area_sq_mi IS NOT NULL)
                        AND (f.year=2016 AND l.year = 2016)
                        ORDER BY 6 DESC
                  ),
      table2 AS (SELECT table1.country_code,
                        table1.country_name,
                         table1.year,
                         table1.perc_fa,
                         CASE WHEN table1.perc_fa >= 75 THEN 4
                              WHEN table1.perc_fa < 75 AND table1.perc_fa >= 50 THEN 3
                              WHEN table1.perc_fa < 50 AND table1.perc_fa >=25 THEN 2
                              ELSE 1
                         END AS percentile
                         FROM table1 ORDER BY 5 DESC
                  )

SELECT table2.percentile,
       COUNT(table2.percentile)
       FROM table2
       GROUP BY 1
       ORDER BY 2 DESC;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/CO_pic3.png)

- _List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016._
```sql
With table1 AS (SELECT f.country_code,
                       f.country_name,
                       f.year,
                       f.forest_area_sqkm,
                       l.total_area_sq_mi*2.59 AS total_area_sqkm,
                        (f.forest_area_sqkm/(l.total_area_sq_mi*2.59))*100 AS perc_fa
                        FROM forest_area f
                        JOIN land_area l
                        ON f.country_code = l.country_code
                        AND (f.country_name != 'World' AND f.forest_area_sqkm IS NOT NULL AND l.total_area_sq_mi IS NOT NULL)
                        AND (f.year=2016 AND l.year = 2016)
                        ORDER BY 6 DESC
                  ),
      table2 AS (SELECT table1.country_code,
                        table1.country_name,
                         table1.year,
                         table1.perc_fa,
                         CASE WHEN table1.perc_fa >= 75 THEN 4
                              WHEN table1.perc_fa < 75 AND table1.perc_fa >= 50 THEN 3
                              WHEN table1.perc_fa < 50 AND table1.perc_fa >=25 THEN 2
                              ELSE 1
                         END AS percentile
                         FROM table1 ORDER BY 5 DESC
                  )
SELECT table2.country_name,
       r.region,
       ROUND(CAST(table2.perc_fa AS NUMERIC),2) AS perc_fa,
       table2.percentile
       FROM table2
       JOIN regions r
       ON table2.country_code = r.country_code
       WHERE table2.percentile = 4
       ORDER BY 1;
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/CO_pic4.png)

- _How many countries had a percent forestation higher than the United States in 2016?_

```sql
With table1 AS (SELECT f.country_code,
                       f.country_name,
                       f.year,
                       f.forest_area_sqkm,
                       l.total_area_sq_mi*2.59 AS total_area_sqkm,
                        (f.forest_area_sqkm/(l.total_area_sq_mi*2.59))*100 AS perc_fa
                        FROM forest_area f
                        JOIN land_area l
                        ON f.country_code = l.country_code
                        AND (f.country_name != 'World' AND f.forest_area_sqkm IS NOT NULL AND l.total_area_sq_mi IS NOT NULL)
                        AND (f.year=2016 AND l.year = 2016)
                        ORDER BY 6 DESC
                  )
SELECT COUNT(table1.country_name)
      FROM table1
      WHERE table1.perc_fa > (SELECT table1.perc_fa
                                     FROM table1
                                     WHERE table1.country_name = 'United States'
                              )
```

![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/CO_pic5.png.png)
