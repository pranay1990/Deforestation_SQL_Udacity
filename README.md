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
- _What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can use the country record denoted as ``World`` in the region table._?
```sql
SELECT f.forest_area_sqkm
		FROM forest_area f
        WHERE f.country_name = 'World'
        	AND f.year = 1990;
```
