# Exploratory Data Analysis: The AppleStore.
## Dataset: AppleStore.csv (attached in this same git)
## Context: 
*  Any given App in the AppStore have a description, rating, price that may or may not influence the overall rating and success.
*  An app's rating may determine its fate, KillerApp or DeadApp

## Objective:
* Run a small Market research to determine the next unexplored niche.
* We have the task, as data analysts to clean up and explore the data, gain some insights and make recommendations.

### Connect the data
The entire project has been adapted to be run in a [SQL Lite](https://sqliteonline.com/) web instance\
Since SQL Lite has limitations in uploading data, we separate the `csv` file into smaller chunks.

1. After uploading the data, we proceed to combine all chunks.

```
CREATE TABlE appleStore_description_combined AS 
SELECT * FROM appleStore_description1
	UNION ALL
SELECT * FROM appleStore_description2
	UNION ALL
SELECT * FROM appleStore_description3
	UNION ALL
SELECT * FROM appleStore_description4
```

2. Exploratory Data Analysis
*  Check unique IDs
*  Check for missing values on AppleStore
*  Check for missing values on AppleStore Description

```
SELECT COUNT(DISTINCT id) As UniqueAppIDs
FROM appleStore_description_combined

SELECT count(*) AS MissingValues
FROM AppleStore
WHERE track_name IS NULL OR user_rating IS NULL OR prime_genre IS NULL

SELECT count(*) AS MissingValues
FROM appleStore_description_combined
WHERE app_desc IS NULL
```

3. Exploration on Free and Paid Apps
* Number of Apps by Genre, Free Apps
* Number of Apps by Genre, Paid

```
SELECT 	prime_genre,
	COUNT(*) AS NumApps,
        AVG(user_rating) AS Avg_Rating
FROM AppleStore
WHERE price = 0
GROUP BY prime_genre
ORDER BY NumApps DESC

SELECT 	prime_genre,
	COUNT(*) AS NumApps,
        ROUND(avg(user_rating),2) AS Avg_Rating,
        ROUND(avg(price),2) AS Avg_Price_USD,
        MAX(price) AS Max_Price_USD
FROM AppleStore
WHERE price > 0
GROUP BY prime_genre
ORDER BY Avg_Price_USD DESC
```

4. Something about the high price of some apps caught my eye, I wanted to know more about these expensive apps.
* Highest paid apps by genre
 ```
SELECT 	prime_genre,
      	track_name,
      	price
FROM
(	SELECT
        prime_genre,
        track_name,
        price,
        RANK() OVER(PARTITION BY prime_genre ORDER BY price DESC, rating_count_tot DESC) AS rank
    	FROM AppleStore
) AS a
WHERE a.rank = 1 OR a.rank = 2
```
The most expensive apps belong to the Education genre. They are charging around **USD$ 300 and USD$ 250** a license.

---

5. Trying to get a better overview of App Ratings
* The average rating is around 3.5
```
SELECT	MIN(user_rating) AS MinRating,
	MAX(user_rating) AS MaxRating,
       	AVG(user_rating) AS AvgRating
FROM AppleStore
```
---
6. Does paid Apps have a higher rating?
* Short Answer: Yes, but slightly.
```
SELECT
CASE 
	WHEN price > 0 then 'Paid'
	ELSE 'Free'
	END AS App_Type,
AVG(user_rating) AS Avg_Rating,
COUNT(user_rating) AS AppCount
FROM AppleStore
GROUP BY App_Type
```
\
7. Do apps with more supported languages have a better rating?
* Not really, apps with 10-30 supported languages have a better rating than those with over 30 supported languages.
```
SELECT
CASE
	WHEN lang_num < 10 then '<10 languages'
	WHEN lang_num BETWEEN 10 AND 30 then '10-30 languages'
        ELSE '>30 languages'
        END AS language_bucket,
avg(user_rating) AS Avg_Rating,
avg(price) AS Avg_Price

FROM AppleStore
GROUP BY language_bucket        
ORDER BY Avg_Rating desc
```
\
8. What is the genre with the lowest average rating?
* Catalogs.
* Finance.
* Books.
Catalog Apps could be a good niche.
```
SELECT prime_genre,
	avg(user_rating) AS Avg_Rating
FROM AppleStore
GROUP by prime_genre
ORDER By Avg_Rating ASC
LIMIT 10
```
\
9. Is there any correlation between the length of the app description?
* Apps with longer descriptions tend to have better ratings. Users like to have well-documented apps.
```
SELECT
CASE
	WHEN length(b.app_desc) < 500 THEN 'Short'
	WHEN length(b.app_desc) between 500 AND 1000 THEN 'Medium'
        ELSE 'Long'
        END AS description_lenght_bucket,
avg(a.user_rating) as Avg_Rating,
COUNT(a.user_rating) AS SumCount

FROM
	AppleStore As a
JOIN
	appleStore_description_combined as b
USING (id)
GROUP BY description_lenght_bucket
ORDER by Avg_Rating DESC;
```
