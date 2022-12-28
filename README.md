# Explorative Data Analysis: Inside AirBnb Berlin (SQL)

## Table of Contents

- [Summary](README.md#summary)
- [Implications](README.md#implications)
- [Explorative Data Analysis](README.md#explorative-data-analysis)
- [References](README.md#references)

## Summary

Inside Airbnb is an investigative website that provides detailed data on Airbnb listings in dozens of cities and countries around the world. The data is scraped from publicly available information on the Airbnb website. In this project, I'll be working with their [dataset for the city of Berlin](http://insideairbnb.com/berlin) to perform some exploratory data analysis using SQL in [MySQL Workbench](https://www.mysql.com/products/workbench/).

## Implications

In recent years, the impact of Airbnb and other peer-to-peer short-term rental platforms has increasingly come into focus. Cities worldwide have started to regulate those platforms more strictly claiming that they cause illegal renting and gentrification, leading to an increase in rents.

The data provided by Inside Airbnb provides a useful basis for studying and monitoring Airbnb's practices and penetration across housing markets.

In a [recent study](https://www.diw.de/de/diw_01.c.796644.de/publikationen/diskussionspapiere/2020_1890/airbnb_and_rents__evidence_from_berlin.html), researchers examined the impact of Airbnb on rents in the city of Berlin. Their findings suggest that each additional listing on Airbnb increases average monthly rents by at least seven cents per square meter.

## Explorative Data Analysis

```SQL
-- Selects the airbnb_berlin database.
USE airbnb_berlin;


-- QUERY 1: Returns a list of all neighbourhood groups included in the dataset.
SELECT DISTINCT
	neighbourhood_group
FROM listings
ORDER BY
	neighbourhood_group;


-- QUERY 2: Returns the average daily price per neighbourhood group.
SELECT
	neighbourhood_group,
    ROUND(AVG(price), 2) AS average_daily_price
FROM listings
GROUP BY
	neighbourhood_group
ORDER BY
	average_daily_price DESC;


-- QUERY 3: Returns the number of listings that consist of entire homes or apartments and the percentage.
-- Calculates the percentage by using a subquery.
SELECT
	neighbourhood_group,
    COUNT(id) AS number_of_homes_apartments,
    COUNT(id) * 100 / (SELECT COUNT(id) FROM listings) AS percentage
FROM listings
WHERE
	room_type = 'Entire home/apt'
GROUP BY
	neighbourhood_group
ORDER BY
	number_of_homes_apartments DESC;


-- QUERY 4: Returns the number of listings and percentage per room type.
-- Calculates the percentage by using a subquery.
SELECT
	room_type,
    COUNT(id) AS number_of_listings,
    ROUND(COUNT(*) * 100 / (SELECT COUNT(*) FROM listings), 2) AS percentage
FROM listings
GROUP BY
	room_type
ORDER BY
	number_of_listings DESC;


-- QUERY 5: Returns the top 10 hosts by number of listings.
SELECT
    host_name,
	COUNT(*) AS number_of_listings
FROM listings
GROUP BY
	host_id,
    host_name
ORDER BY
	number_of_listings DESC LIMIT 10;


-- QUERY 6: Returns the average, minimum, and maximum daily price per room type.
SELECT
	room_type,
    AVG(price) AS average_daily_price,
    MIN(price) AS minimum_daily_price,
    MAX(price) AS maximum_daily_price
FROM listings
GROUP BY
	room_type
ORDER BY
	average_daily_price DESC;


-- QUERY 7: Returns the host id, host name, and number of listings per host.
SELECT
	host_id,
    host_name,
	COUNT(host_id) AS number_of_listings
FROM listings
GROUP BY
	host_id,
    host_name
ORDER BY
	number_of_listings DESC;


-- QUERY 8: Returns the top 10 hosts by number of listings, broken down by room type.
SELECT
    host_name,
    COUNT(*) AS number_of_listings,
    SUM(CASE WHEN room_type = 'Entire home/apt' THEN 1 ELSE 0 END) AS entire_home_apts,
    SUM(CASE WHEN room_type = 'Private room' THEN 1 ELSE 0 END) AS private_rooms,
    SUM(CASE WHEN room_type = 'Shared room' THEN 1 ELSE 0 END) AS shared_rooms,
    SUM(CASE WHEN room_type = 'Hotel room' THEN 1 ELSE 0 END) AS hotel_rooms
FROM listings
GROUP BY
	host_id,
    host_name
ORDER BY
	number_of_listings DESC LIMIT 10;


-- QUERY 9: Returns a list of hosts with multiple listings, broken down by the number of listings (1-10+).
-- Calculates the number of listings per host by using a Common Table Expression (CTE).
WITH listings_per_host AS (
	SELECT host_id, COUNT(host_id) AS number_of_listings
    FROM listings
    GROUP BY host_id)
SELECT
	SUM(CASE WHEN lph.number_of_listings = 1 THEN 1 ELSE 0 END) AS '1',
    SUM(CASE WHEN lph.number_of_listings = 2 THEN 1 ELSE 0 END) AS '2',
    SUM(CASE WHEN lph.number_of_listings = 3 THEN 1 ELSE 0 END) AS '3',
    SUM(CASE WHEN lph.number_of_listings = 4 THEN 1 ELSE 0 END) AS '4',
    SUM(CASE WHEN lph.number_of_listings = 5 THEN 1 ELSE 0 END) AS '5',
    SUM(CASE WHEN lph.number_of_listings = 6 THEN 1 ELSE 0 END) AS '6',
    SUM(CASE WHEN lph.number_of_listings = 7 THEN 1 ELSE 0 END) AS '7',
    SUM(CASE WHEN lph.number_of_listings = 8 THEN 1 ELSE 0 END) AS '8',
    SUM(CASE WHEN lph.number_of_listings = 9 THEN 1 ELSE 0 END) AS '9',
    SUM(CASE WHEN lph.number_of_listings >= 10 THEN 1 ELSE 0 END) AS '10+'
FROM listings l
	INNER JOIN listings_per_host lph
		ON l.host_id = lph.host_id;


-- QUERY 10: Returns a list of hosts with multiple listings, broken down by the number of listings (1-10+).
-- Calculates the number of listings per host by using a temporary table.
DROP TABLE IF EXISTS listings_per_host;

CREATE TEMPORARY TABLE listings_per_host
SELECT
	host_id,
    COUNT(host_id) AS number_of_listings
FROM listings
GROUP BY
	host_id;

SELECT
	SUM(CASE WHEN lph.number_of_listings = 1 THEN 1 ELSE 0 END) AS '1',
    SUM(CASE WHEN lph.number_of_listings = 2 THEN 1 ELSE 0 END) AS '2',
    SUM(CASE WHEN lph.number_of_listings = 3 THEN 1 ELSE 0 END) AS '3',
    SUM(CASE WHEN lph.number_of_listings = 4 THEN 1 ELSE 0 END) AS '4',
    SUM(CASE WHEN lph.number_of_listings = 5 THEN 1 ELSE 0 END) AS '5',
    SUM(CASE WHEN lph.number_of_listings = 6 THEN 1 ELSE 0 END) AS '6',
    SUM(CASE WHEN lph.number_of_listings = 7 THEN 1 ELSE 0 END) AS '7',
    SUM(CASE WHEN lph.number_of_listings = 8 THEN 1 ELSE 0 END) AS '8',
    SUM(CASE WHEN lph.number_of_listings = 9 THEN 1 ELSE 0 END) AS '9',
    SUM(CASE WHEN lph.number_of_listings >= 10 THEN 1 ELSE 0 END) AS '10+'
FROM listings l
	INNER JOIN listings_per_host lph
		ON l.host_id = lph.host_id;


-- QUERY 11: Returns the number of listings broken down by short-term and long-term rentals.
-- The threshold for short-term rentals is 30 days.
SELECT
	COUNT(CASE WHEN minimum_nights < 30 THEN 1 ELSE NULL END) AS short_term_rentings,
    COUNT(CASE WHEN minimum_nights < 30 THEN 1 ELSE NULL END) * 100 / (SELECT COUNT(id) FROM listings) AS percentage_str,
    COUNT(CASE WHEN minimum_nights >= 30 THEN 1 ELSE NULL END) AS long_term_rentings,
    COUNT(CASE WHEN minimum_nights >= 30 THEN 1 ELSE NULL END) * 100 / (SELECT COUNT(id) FROM listings) AS percentage_ltr
FROM listings;
```

## References

- [Inside Airbnb Berlin Dataset](http://insideairbnb.com/berlin)
- [Airbnb and Rents: Evidence from Berlin Discussion Paper](https://www.diw.de/de/diw_01.c.796644.de/publikationen/diskussionspapiere/2020_1890/airbnb_and_rents__evidence_from_berlin.html)
- [MySQL Workbench](https://www.mysql.com/products/workbench/)
