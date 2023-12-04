# Exploratory Data Analysis: The AppleStore.
## Dataset: AppleStore.csv (attached in this same git)
## Context: 
*  Any given App in the AppStore have a description, rating, price that may or may not influence the overall rating and success.
*  An app's rating may determine its fate, KillerApp or DeadApp

## Objective:
* Run a small Market research to determine the next unexplored niche.
* We have the task, as data analysts to clean up and explore the data, gain some insights and make recommendations.

### Connect the data
The entire project has been adapted to be run in [a ](https://sqliteonline.com/) \
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
SELECT prime_genre,
		COUNT(*) AS NumApps,
        avg(user_rating) AS Avg_Rating
FROM AppleStore
Where price = 0
Group by prime_genre
ORDER By NumApps DESC

SELECT prime_genre,
		COUNT(*) AS NumApps,
        ROUND(avg(user_rating),2) AS Avg_Rating,
        ROUND(avg(price),2) AS Avg_Price_USD,
        max(price) AS Max_Price_USD
FROM AppleStore
Where price > 0
Group by prime_genre
ORDER By Avg_Price_USD DESC
```

4. 
* Highest paid apps by genre
 
```
SELECT prime_genre,
      track_name,
      price
FROM (
  SELECT
          prime_genre,
          track_name,
          price,
          RANK() OVER(PARTITION BY prime_genre ORDER BY price DESC, rating_count_tot DESC) AS rank
    FROM
          AppleStore
  	) AS a
WHERE 
a.rank = 1 	
```
-- Overview of App Ratings

SELECT min(user_rating) AS MinRating,
	   max(user_rating) AS MaxRating,
       avg(user_rating) AS AvgRating
From AppleStore

-- Paid Apps have a higher rating?

SELECT CASE 
			When price > 0 then 'Paid'
            Else 'Free'
            End as App_Type,
       avg(user_rating) AS Avg_Rating,
       count(user_rating) AS AppCount
From AppleStore
GROUP By App_Type

-- Supported languages have better rating?

SELECT CASE 
			When lang_num < 10 then '<10 languages'
            When lang_num BETWEEN 10 AND 30 then '10-30 languages'
            ELSE '>30 languages'
            End as language_bucket,
       avg(user_rating) AS Avg_Rating,
       avg(price) AS Avg_Price
From AppleStore
GROUP By language_bucket        
Order By Avg_Rating desc

-- Genre with lowest ratings?

SELECT prime_genre,
	avg(user_rating) AS Avg_Rating
From AppleStore
GROUP by prime_genre
ORDER By Avg_Rating ASC
limit 10

-- Correlation between lenght of description?

SELECT CASe
			When length(b.app_desc) < 500 THen 'Short'
            When length(b.app_desc) between 500 AND 1000 THen 'Medium'
            Else 'Long'
            End as description_lenght_bucket,
            avg(a.user_rating) as Avg_Rating,
            COUNT(a.user_rating) AS SumCount
            
From 
	AppleStore As a
join
	appleStore_description_combined as b
using (id)
GROUP BY description_lenght_bucket
ORDER by Avg_Rating DESC;