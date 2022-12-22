 - High level metrics of the dataset:
 ```
SELECT
	COUNT(id) AS number_of_houses,
	ROUND(AVG(price)) AS avg_house_price,
	ROUND(AVG(sqft_living)) AS avg_house_square_footage,
	MIN(yr_built) AS oldest_house_year,
	MAX(yr_built) AS newest_house_year,
	STRING_AGG(DISTINCT TO_CHAR(waterfront,'9'),',') as waterfront_values,
	STRING_AGG(DISTINCT TO_CHAR(view,'9'),',') as view_values,
	STRING_AGG(DISTINCT TO_CHAR(condition,'9'),',') as condition_values

FROM housing_data
```

|number_of_houses|avg_house_price|avg_house_square_footage|oldest_house_year|newest_house_year|waterfront_values|view_values|condition_values|
|---|---|---|---|---|---|---|---|
4600|551963|2139|1900|2014| 0, 1| 0, 1, 2, 3, 4| 1, 2, 3, 4, 5|

-----

- The top 5 zip codes, based on average home price:
```
SELECT DISTINCT
	RIGHT(statezip,5) as zip_code,
	ROUND(AVG(price) OVER(PARTITION BY statezip)) AS avg_price
FROM housing_data
ORDER BY avg_price DESC
LIMIT 5
```

|zip_code|avg_price|
|---|---|
98039|2046559
98004|1317106
98040|1123818
98109|1049846
98112|1001604

-----

 - Using the list generated from above, the top 3 houses within each zip code:
```
WITH cte AS (
	SELECT
		RIGHT(statezip,5) as zip_code,
		street,
		city,
		sqft_living,
		sqft_lot,
		CASE WHEN yr_renovated <> 0 THEN 'Y' ELSE 'N' END AS is_renovated,
		condition,
		price,
		ROW_NUMBER() OVER (PARTITION BY statezip ORDER BY price DESC) AS price_rank_within_zip_code
	FROM housing_data
)

SELECT *
FROM cte
WHERE price_rank_within_zip_code <=3
AND zip_code IN 
		('98039','98004','98040','98109','98112')
```
|zip_code|street|city|sqft_living|sqft_lot|is_renovated|condition|price|price_rank_within_zip_code|
|---|---|---|---|---|---|---|---|---|
98004|4442 95th Ave NE|Bellevue|10040|37325|Y|3|7062500|1
98004|1149-1199 91st Ave NE|Bellevue|6430|27517|N|3|4489000|2
98004|1365 91st Ave NE|Clyde Hill|7050|42840|Y|4|3800000|3
98039|3222 78th Pl NE|Medina|5550|28078|N|4|3710000|1
98039|3239 78th Pl NE|Medina|4430|21000|Y|3|2750000|2
98039|1209 Evergreen Point Rd|Medina|4290|20445|N|4|2680000|3
98040|5044 Butterworth Rd|Mercer Island|9640|13068|Y|3|4668000|1
98040|5243 Forest Ave SE|Mercer Island|6980|15682|N|4|3100000|2
98040|7436 E Mercer Way|Mercer Island|3530|17450|Y|3|2367000|3
98109|1230 Warren Ave N|Seattle|6210|8856|N|5|3200000|1
98109|150 Highland Dr|Seattle|2940|5432|Y|4|1712500|2
98109|308 Prospect St|Seattle|3170|3000|N|5|1695000|3
98112|1239 Parkside Dr E|Seattle|6390|13180|Y|3|2466350|1
98112|1404 Broadmoor Dr E|Seattle|4730|13586|N|5|2453500|2
98112|173 37th Ave E|Seattle|4860|9453|N|5|2250000|3

