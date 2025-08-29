![Netflix Logo] (https://github.com/themofaithful/NetflixDataAnalysisSQL/blob/main/NetflixLogo.png)
 
## Overview

This project requires a thorough examination of Netflix's movie and TV show data using SQL. The purpose is to extract relevant insights and answer a variety of (20+) business questions using the dataset. This paper details the project's objectives, business challenges, solutions, findings, and conclusions.
 
## Objectives
 
- Examine the distribution of content types (movies versus television shows).
- Determine the most frequent ratings for films and television shows.
- Organise and analyse content by release year, country, and duration.
- Analyse and categorise information using precise criteria and keywords.
 
## Dataset
 
Though the dataset for this project is sourced from the Kaggle dataset, it's uploaded here: Netflix_titles.csv
 
 
## Business Problems and Solutions

1.	Display the total Number of Movies vs TV Shows
```sql
	SELECT 
		type,
		COUNT(*) count_type
	FROM netflix_titles
	GROUP BY type
```
**Objective:** Determine the distribution of content types on Netflix.


2.	Count the Number of Content Items in Each Genre
```sql
		SELECT 
			Trim(Value) AS genre,  
			COUNT(*) AS total_content  
		FROM netflix_titles
		   CROSS APPLY string_split (listed_in, ',') 
		GROUP BY Trim(Value);
```
**Objective:** Count the number of content items in each genre.

3.	List All Movies Released in 2020
```sql
		SELECT * 
		FROM netflix_titles
		WHERE release_year = 2020;
```

**Objective:** Retrieve all movies released in a specific year.

4.	Find the Top 5 Countries with the Most Content on Netflix
```sql
		SELECT Top(5) * 
		FROM
		( SELECT 
			Trim(Value) AS country,  
			COUNT(*) AS total_content  
			FROM netflix_titles
			   CROSS APPLY string_split (country, ',') 
			GROUP BY Trim(Value) ) AS temp
		WHERE country IS NOT NULL
		ORDER BY total_content DESC
```
	**Objective:** Identify the top 5 countries with the highest number of content items.


 
5.	Find Content Added in the Last 5 Years
```sql

SELECT 
	* 
FROM 
	netflix_titles_filter
WHERE 
	date_added >= DATEADD(Year, -5, GetDate())
```
**Objective:** Retrieve content added to Netflix in the last 5 years.

6.	List All Movies that are Documentaries
```sql

###Method 1

SELECT * FROM netflix_titles_filter

WHERE Type = 'Movie' AND Listed_in LIKE '%Documentaries%'

###Method 2

SELECT ntf.*, nli.listed_in 

FROM netflix_titles_filter ntf

JOIN netflix_listed_in nli

ON ntf.show_id = nli.show_id

WHERE nli.Listed_in = 'Documentaries'
 
SELECT * FROM netflix_listed_in

```
**Objective:** Retrieve all movies classified as documentaries.
 
7.	Find All Content Without a Director
```sql
 
SELECT * FROM netflix_titles_filter

WHERE director = 'NA'

###Method 2

SELECT ntf.*, nd.director 

FROM netflix_titles_filter ntf

JOIN netflix_director nd

ON ntf.show_id = nd.show_id

WHERE nd.director = 'NA'
```
**Objective:** List content that does not have a director.

8.	Find How Many Movies Actor 'Salman Khan' Appeared in over the Last 10 Years
```sql

###Method 1

SELECT * FROM netflix_titles_filter

WHERE Type = 'Movie' AND cast LIKE '%Salman Khan%' AND  release_year > YEAR(GetDate()) - 10
 
###Method 2

SELECT ntf.*, nc.cast 

FROM netflix_titles_filter ntf

JOIN netflix_cast nc

ON ntf.show_id = nc.show_id

WHERE nc.cast = 'Salman Khan' AND  ntf.release_year > YEAR(GetDate()) - 10
 ```
**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years
 
9.	Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India
```sql

###Method 1

SELECT TOP (10)

	Trim(Value) AS Actor,

	COUNT(*) HighestNumber

FROM netflix_titles_filter

CROSS APPLY STRING_SPLIT(cast,',')

WHERE country LIKE '%India%' AND type = 'Movie'

GROUP BY Trim(Value)

Order BY COUNT(*) DESC
 
###Method 2, Using JOIN

SELECT TOP (10) trim(cast) Actor, Count(*) HighestNumber

FROM netflix_titles_filter ntf

JOIN netflix_cast nc

ON ntf.show_id = nc.show_id

WHERE ntf.country = 'India' AND ntf.type = 'Movie'

GROUP BY trim(cast)

Order BY COUNT(*) DESC
 ```
**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.
 
10.	Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
```sql
 
###Method 1

SELECT Category, Count(*) CategoryCounts

FROM

	(

	SELECT description,

		CASE	

			WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'

		ELSE 

			'Good'

		END AS Category

	FROM Netflix_Titles_Filter

	)AS CategorizedContents

GROUP BY Category
 
 
 
###Method 2

SELECT 

	CASE

		WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'

		ELSE 'Good'

	END as Category,

	Count(*) as TotalCount

FROM Netflix_Titles_Filter

GROUP BY 

	CASE

		WHEN description LIKE '%kill%' OR description LIKE '%violence%' THEN 'Bad'

		ELSE 'Good'

	END;
 ```
**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

11.	Identify the Longest Movie

 Our strint_split has another parameter called the ORDINAL VALUE. When you use string_split, it will split the value of the column and return the first value as Ordinal 1, means the first value and not the second value. ORDINAL is like the index of the numbering. We CAST the Value because we need the answer as INTEGER.
```sql
 
SELECT TOP 1

	Type,

	Title,

	Trim(Value) as TotalMunite,

	Duration

FROM Netflix_Titles_Filter

CROSS APPLY string_split(duration, ' ', 1)

WHERE type = 'Movie' AND ORDINAL = 1

ORDER BY CAST(Trim(Value) AS INT) DESC
 ```
**Objective:** Find the movie with the longest duration.

12.	Find All Movies/TV Shows by Director 'Rajiv Chilaka'
```sql

###Method 1

SELECT * FROM netflix_titles_filter

WHERE Type IN ('Movie', 'TV Show') AND Director LIKE '%Rajiv Chilaka%'
 
 ###Method 2

SELECT * FROM netflix_titles_filter

WHERE Director LIKE '%Rajiv Chilaka%'

###Method 3

SELECT *, ntf.type, nd.director 

FROM netflix_titles_filter ntf

JOIN netflix_director nd

ON ntf.show_id = nd.show_id

WHERE ntf.Type = 'Movie' AND nd.Director = 'Rajiv Chilaka'
 ```
**Objective:** List all content directed by 'Rajiv Chilaka'.
 
13.	List All TV Shows with More Than 5 Seasons
```sql
 
SELECT

	Title,

	TRIM(Value) Season

FROM

	netflix_titles_filter

CROSS APPLY string_split(duration,' ',1)

WHERE type = 'TV Show' and Ordinal = 1

AND TRY_CAST(TRIM(Value) AS INT) > 5

Order By CAST(TRIM(Value) AS INT) DESC
```
**Objective:** Identify TV shows with more than 5 seasons.

14.	List content items added after August 20, 2021
```sql
 
SELECT * FROM Netflix_Titles_filter

WHERE date_added > '2021-08-20'
 ```
**Objective:**Display content items added after August 20, 2021
 
15.	List movies added on June 15, 2019
```sql

SELECT * FROM Netflix_Titles_filter

WHERE type = 'Movie' AND date_added = '2019-06-15'
 ```
**Note:** If the date_added column in your table is stored as text (e.g., "June 15, 2019"), we can either:
Compare it directly as a string, or
Convert it into a DATE type using TRY_CONVERT for safer querying.
```sql
SELECT * FROM dbo.netflix_titles WHERE TRY_CONVERT(DATE, date_added, 107) = '2019-06-15';
```
15b. Show just the Count of movies added on June 15, 2019
```sql
SELECT COUNT(*) AS MoviesAddedCount
	FROM dbo.netflix_titles
	WHERE TRY_CONVERT(DATE, date_added, 107) = '2019-06-15';
 ```
15c. Count of titles by Type (Movies vs TV Shows) on June 15, 2019
```sql
SELECT 
    type,
    COUNT(*) AS TitlesAddedCount
FROM dbo.netflix_titles
WHERE TRY_CONVERT(DATE, date_added, 107) = '2019-06-15'
GROUP BY type;
 ```
**Objective:** Display movies added on June 15, 2019
 
16.	List content items added in 2021
```sql

###Method 1

SELECT * FROM Netflix_Titles_filter

WHERE date_added >= '2021-01-01' AND date_added <= '2021-12-31'

###Method 2

SELECT * FROM Netflix_Titles_filter

WHERE date_added BETWEEN '2021-01-01' AND '2021-12-31'

###Method 3

Select *  

From Netflix_Titles_filter

Where date_added LIKE '%2021%'

###Method 4

SELECT * from netflix_titles where Year(date_added) = 2021
``` 
**Objective:** Display content items added in 2021

17.	List movies added in 2021
```sql

###Method 1

SELECT * FROM Netflix_Titles_filter

WHERE type = 'Movie' AND date_added >= '2021-01-01' AND date_added <= '2021-12-31'

###Method 2

SELECT * FROM Netflix_Titles_filter

WHERE type = 'Movie' AND  date_added BETWEEN '2021-01-01' AND '2021-12-31'

###Method 3

Select *  

From Netflix_Titles_filter

Where type = 'Movie' AND date_added LIKE '%2021%'
 
###Method 4

Select *  

From Netflix_Titles_filter

Where type = 'Movie' AND Year(date_added) = 2021
```
**Objective:**Display movies added in 2021

18.	Count the number of movies and TV series that each director has produced in different columns.
This version handles multiple directors in one row (using CROSS APPLY STRING_SPLIT), so if a title has "Director A, Director B", both get credited?
```sql

SELECT Trim(value) AS Directors, Type, Count(*) MovieANDTVShow  

FROM Netflix_Titles_Filter

CROSS APPLY string_split(director, ',')

WHERE type IN('Movie', 'TV Show') AND Director <> 'NA' 

GROUP BY Trim(value), Type
```
**Objective:** Count the number of movies and TV series that each director has produced in different columns.

18a. Count the number of movies and tv series that each director has produced in different columns, include a TotalCount column Movies + TV Shows) for each director
```sql
SELECT 
    LTRIM(RTRIM(director_split.value)) AS director,
    SUM(CASE WHEN nt.type = 'Movie' THEN 1 ELSE 0 END) AS MovieCount,
    SUM(CASE WHEN nt.type = 'TV Show' THEN 1 ELSE 0 END) AS TVShowCount,
    COUNT(*) AS TotalCount
FROM dbo.netflix_titles nt
CROSS APPLY STRING_SPLIT(nt.director, ',') AS director_split
WHERE nt.director IS NOT NULL
GROUP BY LTRIM(RTRIM(director_split.value))
ORDER BY TotalCount DESC, director;
```
**Objective:** Count the number of movies and tv series that each director has produced in different columns with a TotalCount column Movies + TV Shows) for each director

18b. Count the number of movies and tv series that each director has produced in different columns, with TotalCount column (Movies + TV Shows) for each director, including percentage split of Movies vs TV Shows for each director.
```sql
SELECT 
    LTRIM(RTRIM(director_split.value)) AS director,
    SUM(CASE WHEN nt.type = 'Movie' THEN 1 ELSE 0 END) AS MovieCount,
    SUM(CASE WHEN nt.type = 'TV Show' THEN 1 ELSE 0 END) AS TVShowCount,
    COUNT(*) AS TotalCount,
    CAST(100.0 * SUM(CASE WHEN nt.type = 'Movie' THEN 1 ELSE 0 END) / COUNT(*) AS DECIMAL(5,2)) AS MoviePercent,
    CAST(100.0 * SUM(CASE WHEN nt.type = 'TV Show' THEN 1 ELSE 0 END) / COUNT(*) AS DECIMAL(5,2)) AS TVShowPercent
FROM dbo.netflix_titles nt
CROSS APPLY STRING_SPLIT(nt.director, ',') AS director_split
WHERE nt.director IS NOT NULL
GROUP BY LTRIM(RTRIM(director_split.value))
ORDER BY TotalCount DESC, director;
```
**Objective:** Count the number of movies and TV series that each director has produced in different columns, with a TotalCount column Movies + TV Shows) for each director

18c. Count the number of movies and TV series that each director has produced in different columns, with TotalCount column (Movies + TV Shows) for each director, including the percentage split of Movies vs TV Shows for each director. Show only the Top 5 directors ranked by their total number of titles
```sql
SELECT TOP 5
    LTRIM(RTRIM(director_split.value)) AS director,
    SUM(CASE WHEN nt.type = 'Movie' THEN 1 ELSE 0 END) AS MovieCount,
    SUM(CASE WHEN nt.type = 'TV Show' THEN 1 ELSE 0 END) AS TVShowCount,
    COUNT(*) AS TotalCount,
    CAST(100.0 * SUM(CASE WHEN nt.type = 'Movie' THEN 1 ELSE 0 END) / COUNT(*) AS DECIMAL(5,2)) AS MoviePercent,
    CAST(100.0 * SUM(CASE WHEN nt.type = 'TV Show' THEN 1 ELSE 0 END) / COUNT(*) AS DECIMAL(5,2)) AS TVShowPercent
FROM dbo.netflix_titles nt
CROSS APPLY STRING_SPLIT(nt.director, ',') AS director_split
WHERE nt.director IS NOT NULL
GROUP BY LTRIM(RTRIM(director_split.value))
ORDER BY TotalCount DESC, director;
```
**Objective:** Count the number of movies and TV series that each director has produced in different columns, with a TotalCount column Movies + TV Shows) for each director

 
19.	Which country has the highest number of comedy movies?
###Method 1: This version handles without considering ties 
```sql
SELECT TOP 1 Trim(value) All_Country, Count(*) COMEDIES

FROM Netflix_Titles_Filter

CROSS APPLY string_split(country, ',')

WHERE Type = 'Movie' AND listed_in LIKE '%comedies%' AND Trim(value) <> 'NA'

GROUP BY Trim(value)

ORDER BY Count(*) DESC
 ```
Method 2: This version finds the country (or countries, in case of a tie) with the highest number of comedy movies:
```sql
SELECT country, ComedyMovieCount
FROM (
    SELECT 
        country,
        COUNT(*) AS ComedyMovieCount,
        RANK() OVER (ORDER BY COUNT(*) DESC) AS rnk
    FROM dbo.netflix_titles
    WHERE type = 'Movie'
      AND listed_in LIKE '%Comedy%'
      AND country IS NOT NULL
    GROUP BY country
) ranked
WHERE rnk = 1;
 ```
**Objective:** Which country has the highest number of comedy movies?
 
20.	For each year, which director has the maximum number of movies released
Method 1: This version handles without considering ties
```sql
 
SELECT 
        nt.release_year,
        TRIM(d.value) AS director,
        COUNT(*) AS MovieCount 
    FROM dbo.netflix_titles nt
    CROSS APPLY STRING_SPLIT(nt.director, ',') d
    WHERE nt.type = 'Movie'
      AND nt.director IS NOT NULL
      AND nt.release_year IS NOT NULL
    GROUP BY nt.release_year, TRIM(d.value)
```
 ###Method 2:
```sql
SELECT release_year, director, MovieCount
FROM (
    SELECT 
        nt.release_year,
        TRIM(d.value) AS director,
        COUNT(*) AS MovieCount,
        MAX(COUNT(*)) OVER (PARTITION BY nt.release_year) AS MaxMoviesInYear
    FROM dbo.netflix_titles nt
    CROSS APPLY STRING_SPLIT(nt.director, ',') d
    WHERE nt.type = 'Movie'
      AND nt.director IS NOT NULL
      AND nt.release_year IS NOT NULL
    GROUP BY nt.release_year, TRIM(d.value)
) sub
WHERE MovieCount = MaxMoviesInYear
ORDER BY release_year, director;
```
**Objective:** For each year, which director has the maximum number of movies released


 
21.	What is the average running length of movies in each genre?
```sql
 
SELECT Type, Title, Trim(value) as AVGERAGELEN

FROM netflix_titles_filter

CROSS APPLY string_split(Duration, ' ', 1)

WHERE type = 'Movie' and Ordinal = 1
```
**Objective:** What is the average running length of movies in each genre?

22.	List directors who have directed both comedies and horror films.
```sql
 
SELECT DISTINCT Director, Trim(value) as ComedyAndHorror

FROM Netflix_Titles_Filter

CROSS APPLY string_split(Listed_in, ',')

WHERE Listed_in LIKE '%Comedies%' AND Listed_in LIKE '%Horror%' AND Director <> 'NA'

GROUP BY Trim(value), Director

HAVING Trim(value) <> 'Independent Movies' AND Trim(value) <> 'Sci-Fi & Fantasy' AND Trim(value) <> 'International Movies' AND Trim(value) <> 'Action & Adventure' AND Trim(value) <> 'Cult Movies'
``` 
**Objective:** List directors who have directed both comedies and horror films.

23.	List the director's name and the number of horror and comedy films that he or she has directed.
```sql

SELECT Director, Count(*) MovieNumbers, Trim(value) as ComedyAndHorror

FROM Netflix_Titles_Filter

CROSS APPLY string_split(Listed_in, ',')

WHERE Listed_in LIKE '%Comedies%' AND Listed_in LIKE '%Horror%' AND Director <> 'NA'

GROUP BY Trim(value), Director

HAVING Trim(value) <> 'Independent Movies' AND Trim(value) <> 'Sci-Fi & Fantasy' AND Trim(value) <> 'International Movies' AND Trim(value) <> 'Action & Adventure' AND Trim(value) <> 'Cult Movies'

ORDER BY Count(*) DESC
```
**Objective:** List the director's name and the number of horror and comedy films that he or she has directed.

24.	Find the Most Common Rating for Movies and TV Shows
```sql
 
SELECT Rating, COUNT(*) AS CountRating

FROM (

    SELECT Rating

    FROM Netflix_Titles_filter

    UNION ALL 

    SELECT Rating

    FROM Netflix_Titles_filter

) AS combined

GROUP BY Rating
ORDER BY COUNT(*) DESC
``` 
**Objective:** Identify the most frequently occurring rating for each type of content.

25.	Find each year and the average number of content releases in India on Netflix, and return the top 5 years with the highest average content release!
```sql

SELECT TOP 5 release_year, COUNT(show_id) AS total_release, 

    ROUND(COUNT(show_id) * 1.0 / (SELECT COUNT(show_id) 

	FROM Netflix_Titles_filter 

	WHERE country = 'India') * 100, 2) AS avg_release

FROM 

    Netflix_Titles_filter

WHERE 

    country = 'India'

GROUP BY 

    release_year

ORDER BY 

    avg_release DESC
 ```
**Objective:** Calculate and rank years by the average number of content releases by India.
