![Netflix Logo] (https://github.com/themofaithful/NetflixDataAnalysisSQL/blob/main/NetflixLogo.png)
 
## Overview

This project requires a thorough examination of Netflix's movie and TV show data using SQL. The purpose is to extract relevant insights and answer a variety of (20+) business questions using the dataset. This paper details the project's objectives, business challenges, solutions, findings, and conclusions.
 
## Objectives
 
- Examine the distribution of content types (movies versus television shows).
- Determine the most frequent ratings for films and television shows.
- Organise and analyse content by release year, country, and duration.
- Analyse and categorise information using precise criteria and keywords.
 
## Dataset
 
Though the dataset for this project is sourced from the Kaggle dataset, but its uploaded here: Netflix_titles.csv
 
 
## Business Problems and Solutions

###1.	Display the total Number of Movies vs TV Shows
	SELECT 
		type,
		COUNT(*) count_type
	FROM netflix_titles
	GROUP BY type

	--Objective: Determine the distribution of content types on Netflix.


###2.	Count the Number of Content Items in Each Genre
		SELECT 
			Trim(Value) AS genre,  
			COUNT(*) AS total_content  
		FROM netflix_titles
		   CROSS APPLY string_split (listed_in, ',') 
		GROUP BY Trim(Value);

		--Objective: Count the number of content items in each genre.

###3.	List All Movies Released in a 2020
		SELECT * 
		FROM netflix_titles
		WHERE release_year = 2020;

		--Objective: Retrieve all movies released in a specific year.

###4.	Find the Top 5 Countries with the Most Content on Netflix
		SELECT Top(5) * 
		FROM
		(
			SELECT 
			Trim(Value) AS country,  
			COUNT(*) AS total_content  
			FROM netflix_titles
			   CROSS APPLY string_split (country, ',') 
			GROUP BY Trim(Value)

		) AS temp
		WHERE country IS NOT NULL
		ORDER BY total_content DESC

		--Objective: Identify the top 5 countries with the highest number of content items.


 
--5.	Find Content Added in the Last 5 Years

SELECT 
	* 
FROM 
	netflix_titles_filter
WHERE 
	date_added >= DATEADD(Year, -5, GetDate())
 
 
 
--6.	List All Movies that are Documentaries

--Method 1

SELECT * FROM netflix_titles_filter

WHERE Type = 'Movie' AND Listed_in LIKE '%Documentaries%'
 
 
--Method 2

SELECT ntf.*, nli.listed_in 

FROM netflix_titles_filter ntf

JOIN netflix_listed_in nli

ON ntf.show_id = nli.show_id

WHERE nli.Listed_in = 'Documentaries'
 
SELECT * FROM netflix_listed_in
 
 
 
--7.	Find All Content Without a Director
 
SELECT * FROM netflix_titles_filter

WHERE director = 'NA'
 
 
--Method 2

SELECT ntf.*, nd.director 

FROM netflix_titles_filter ntf

JOIN netflix_director nd

ON ntf.show_id = nd.show_id

WHERE nd.director = 'NA'
 
 
 
--8.	Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

--Method 1

SELECT * FROM netflix_titles_filter

WHERE Type = 'Movie' AND cast LIKE '%Salman Khan%' AND  release_year > YEAR(GetDate()) - 10
 
--Method 2

SELECT ntf.*, nc.cast 

FROM netflix_titles_filter ntf

JOIN netflix_cast nc

ON ntf.show_id = nc.show_id

WHERE nc.cast = 'Salman Khan' AND  ntf.release_year > YEAR(GetDate()) - 10
 
 
 
--9.	Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

--Method 1

SELECT TOP (10)

	Trim(Value) AS Actor,

	COUNT(*) HighestNumber

FROM netflix_titles_filter

CROSS APPLY STRING_SPLIT(cast,',')

WHERE country LIKE '%India%' AND type = 'Movie'

GROUP BY Trim(Value)

Order BY COUNT(*) DESC
 
--Method 2, Using JOIN

SELECT TOP (10) trim(cast) Actor, Count(*) HighestNumber

FROM netflix_titles_filter ntf

JOIN netflix_cast nc

ON ntf.show_id = nc.show_id

WHERE ntf.country = 'India' AND ntf.type = 'Movie'

GROUP BY trim(cast)

Order BY COUNT(*) DESC
 
 
--10.	Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.
 
--Method 1

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
 
 
 
--Method 2

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
 
 
 
 
--11.	Identify the Longest Movie

-- Our strint_split has another parameter called the ORDINAL VALUE. That when you use string_split, it will split the value of the column and return the first value as Ordinal 1, means the first value and not the second value. ORDINAL is like the index of the numbering. We CAST the Value because we need the answer as INTEGER.
 
SELECT TOP 1

	Type,

	Title,

	Trim(Value) as TotalMunite,

	Duration

FROM Netflix_Titles_Filter

CROSS APPLY string_split(duration, ' ', 1)

WHERE type = 'Movie' AND ORDINAL = 1

ORDER BY CAST(Trim(Value) AS INT) DESC
 
 
 
--12.	Find All Movies/TV Shows by Director 'Rajiv Chilaka'

--Method 1

SELECT * FROM netflix_titles_filter

WHERE Type IN ('Movie', 'TV Show') AND Director LIKE '%Rajiv Chilaka%'
 
 
--Method 2

SELECT * FROM netflix_titles_filter

WHERE Director LIKE '%Rajiv Chilaka%'
 
 
--Method 3

SELECT *, ntf.type, nd.director 

FROM netflix_titles_filter ntf

JOIN netflix_director nd

ON ntf.show_id = nd.show_id

WHERE ntf.Type = 'Movie' AND nd.Director = 'Rajiv Chilaka'
 
 
--13.	List All TV Shows with More Than 5 Seasons
 
SELECT

	Title,

	TRIM(Value) Season

FROM

	netflix_titles_filter

CROSS APPLY string_split(duration,' ',1)

WHERE type = 'TV Show' and Ordinal = 1

AND TRY_CAST(TRIM(Value) AS INT) > 5

Order By CAST(TRIM(Value) AS INT) DESC
 
 
 
 
--14.	List content items added after August 20, 2021
 
SELECT * FROM Netflix_Titles_filter

WHERE date_added > '2021-08-20'
 
 
--15.	List movies added to on June 15, 2019

SELECT * FROM Netflix_Titles_filter

WHERE type = 'Movie' AND date_added = '2019-06-15'
 
 
--16.	List content items added in 2021

--Method 1

SELECT * FROM Netflix_Titles_filter

WHERE date_added >= '2021-01-01' AND date_added <= '2021-12-31'
 
 
--Method 2

SELECT * FROM Netflix_Titles_filter

WHERE date_added BETWEEN '2021-01-01' AND '2021-12-31'
 
 
--Method 3

Select *  

From Netflix_Titles_filter

Where date_added LIKE '%2021%'
 
 
--Method 4

SELECT * from netflix_titles where Year(date_added) = 2021
 
 
--17.	List movies added in 2021

--Method 1

SELECT * FROM Netflix_Titles_filter

WHERE type = 'Movie' AND date_added >= '2021-01-01' AND date_added <= '2021-12-31'
 
 
--Method 2

SELECT * FROM Netflix_Titles_filter

WHERE type = 'Movie' AND  date_added BETWEEN '2021-01-01' AND '2021-12-31'
 
 
--Method 3

Select *  

From Netflix_Titles_filter

Where type = 'Movie' AND date_added LIKE '%2021%'
 
 
 
--Method 4

Select *  

From Netflix_Titles_filter

Where type = 'Movie' AND Year(date_added) = 2021
 
 
 
--18.	Count the number of movies and tv series that each director has produced in different columns.

SELECT Trim(value) AS Directors, Type, Count(*) MovieANDTVShow  

FROM Netflix_Titles_Filter

CROSS APPLY string_split(director, ',')

WHERE type IN('Movie', 'TV Show') AND Director <> 'NA' 

GROUP BY Trim(value), Type
 
 
 
 
--19.	Which country has highest number of comedy movies?
 
SELECT TOP 1 Trim(value) All_Country, Count(*) COMEDIES

FROM Netflix_Titles_Filter

CROSS APPLY string_split(country, ',')

WHERE Type = 'Movie' AND listed_in LIKE '%comedies%' AND Trim(value) <> 'NA'

GROUP BY Trim(value)

ORDER BY Count(*) DESC
 
 
 
--20.	For each year, which director has maximum number of movies released
 
SELECT release_year, Trim(value) DIRECTOR, Count(*) MOVIES

FROM Netflix_Titles_Filter

CROSS APPLY string_split(director, ',')

WHERE Type = 'Movie' AND director <> 'NA'

GROUP BY release_year, Trim(value)

ORDER BY Count(*) DESC
 
 
 
 
--21.	What is the average running length of movies in each genre?
 
SELECT Type, Title, Trim(value) as AVGERAGELEN

FROM netflix_titles_filter

CROSS APPLY string_split(Duration, ' ', 1)

WHERE type = 'Movie' and Ordinal = 1

 
 
 
--22.	List directors who have directed both comedies and horror films.
 
SELECT DISTINCT Director, Trim(value) as ComedyAndHorror

FROM Netflix_Titles_Filter

CROSS APPLY string_split(Listed_in, ',')

WHERE Listed_in LIKE '%Comedies%' AND Listed_in LIKE '%Horror%' AND Director <> 'NA'

GROUP BY Trim(value), Director

HAVING Trim(value) <> 'Independent Movies' AND Trim(value) <> 'Sci-Fi & Fantasy' AND Trim(value) <> 'International Movies' AND Trim(value) <> 'Action & Adventure' AND Trim(value) <> 'Cult Movies'
 
 
 
--23.	List the director's name and the number of horror and comedy films that he or she has directed.

SELECT Director, Count(*) MovieNumbers, Trim(value) as ComedyAndHorror

FROM Netflix_Titles_Filter

CROSS APPLY string_split(Listed_in, ',')

WHERE Listed_in LIKE '%Comedies%' AND Listed_in LIKE '%Horror%' AND Director <> 'NA'

GROUP BY Trim(value), Director

HAVING Trim(value) <> 'Independent Movies' AND Trim(value) <> 'Sci-Fi & Fantasy' AND Trim(value) <> 'International Movies' AND Trim(value) <> 'Action & Adventure' AND Trim(value) <> 'Cult Movies'

ORDER BY Count(*) DESC
 
 
 
--24.	Find the Most Common Rating for Movies and TV Shows
 
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
 
 
 
 
--25.	Find each year and the average numbers of content release in India on netflix and return top 5 year with highest avg content release!
 
 
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
 
 
