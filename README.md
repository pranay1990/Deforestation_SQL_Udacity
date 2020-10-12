# Deforestation_SQL_Udacity
This is the project in SQL nanodegree program from Udacity

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

### What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can use the country record denoted as "World" in the region table.?

```sql
SELECT f.forest_area_sqkm
		FROM forest_area f
        WHERE f.country_name = 'World'
        	AND f.year = 1990;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic1.png)

### What was the total forest area (in sq km) of the world in 2016? Please keep in mind that you can use the country record in the table is denoted as "World"?
```sql
SELECT f.forest_area_sqkm
		FROM forest_area f
        WHERE f.country_name = 'World'
        	AND f.year = 2016;
```
![](https://github.com/pranay1990/Deforestation_SQL_Udacity/blob/main/images/GS_pic2.png)

 What was the change (in sq km) in the forest area of the world from 1990 to 2016?
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
