## End-to-End Data Analysis Project For Netflix Movies/TV Shows Dataset Using SQL

![Netflix Logo](https://github.com/themofaithful/NetflixDataAnalysisSQL/blob/main/NetflixLogo.png)
 
## Overview

This project requires a thorough examination of Netflix's movie and TV show data using SQL. The purpose is to extract relevant insights and answer a variety of (20+) business questions using the dataset. This paper details the project's objectives, business challenges, solutions, findings, and conclusions.
 
## Objectives
 
- Examine the distribution of content types (movies versus television shows).
- Determine the most frequent ratings for films and television shows.
- Organise and analyse content by release year, country, and duration.
- Analyse and categorise information using precise criteria and keywords.
 
## Dataset
The dataset for this project is sourced from the [Kaggle](https://www.kaggle.com/datasets?fileType=csv) dataset; however, you can send an email to Themofaithful@gmail.com to request it.

## Processes and Stages of Dataset Importation

Step 1: Download the dataset from: [Netflix Dataset](https://greaterheight.tech/NetflixContent.csv)

Step 2: Open the NetflixContent.csv file and explore it to determine the column names.

Step 3: Execute the SQL code below:
```sql
CREATE DATABASE MoviesDB
GO
USE MoviesDB
GO
CREATE TABLE NetflixContent
(
	ShowID nvarchar(MAX) NOT NULL,
	Type nvarchar(MAX) NULL,
	Title nvarchar(MAX) NULL,
	Director nvarchar(MAX) NULL,
	Cast nvarchar(MAX) NULL,
	Country nvarchar(MAX) NULL,
	DateAdded nvarchar(MAX) NULL,
	ReleaseYear smallint NULL,
	Rating nvarchar(MAX) NULL,
	Duration nvarchar(MAX) NULL,
	ListedIn nvarchar(MAX) NULL,
	Description nvarchar(MAX) NULL
)
GO
```
**Result:**
- MoviesDB database will be created
- NetflixContent table will be created

Step 4: Execute the SQL code below. The code inserts all the records into NetflixContent
```sql
BULK INSERT NetflixContent
FROM 'C:\...\NetflixContent.csv'		--specify the folder where you downloaded the file
WITH (
   FORMAT = 'CSV',
	FIELDTERMINATOR = ',',  -- Specifies the column delimiter
    ROWTERMINATOR = '\n',   -- Specifies the row terminator (newline character)
    FIRSTROW = 2,            -- Optional: Skips the header row if present
	TABLOCK
);
Go
```
Step 5: We now run the SQL code below to identify the maximum column size for the content in each column of the netflix_titles table that has been imported. Run the SQL statements below.
USE Movies
```sql
 SELECT
	MAX(LEN(showid)) showid,
	MAX(LEN(type)) type,
	MAX(LEN(title)) title,
	MAX(LEN(director))director,
	MAX(LEN(cast)) cast,
	MAX(LEN(country))country,
	MAX(LEN(dateadded)) dateadded,
	MAX(LEN(releaseyear)) releaseyear,
	MAX(LEN(rating))rating,
	MAX(LEN(duration))duration,
	MAX(LEN(listedin)) listedin,
	MAX(LEN(description))description
FROM 
	netflixContent
```
**Objective:** To identify the maximum column size in each of the columns of the netflixContent table so we can remodify the datatype sizes of the column. This is for performance optimization.

**Result:** When you execute the code above, the result gives the maximum column size of each column in the netflixContent. 

<img width="671" height="91" alt="image" src="https://github.com/user-attachments/assets/4d9029a2-4cd3-4e37-be61-c3f50f813f99" />

Step 6: Using the results above, we now alter the table netflixContent and change the data sizes of the columns.
```sql
ALTER TABLE NetflixContent ALTER COLUMN showid nvarchar(5) NOT NULL
ALTER TABLE NetflixContent ALTER COLUMN type nvarchar(7) NULL
ALTER TABLE NetflixContent ALTER COLUMN title nvarchar(104) NULL
ALTER TABLE NetflixContent ALTER COLUMN director nvarchar(208) NULL
ALTER TABLE NetflixContent ALTER COLUMN cast nvarchar(771) NULL
ALTER TABLE NetflixContent ALTER COLUMN country nvarchar(123) NULL
ALTER TABLE NetflixContent ALTER COLUMN dateadded nvarchar(19) NULL
ALTER TABLE NetflixContent ALTER COLUMN releaseyear smallint NULL
ALTER TABLE NetflixContent ALTER COLUMN rating nvarchar(8) NULL
ALTER TABLE NetflixContent ALTER COLUMN duration nvarchar(10) NULL
ALTER TABLE NetflixContent ALTER COLUMN listedin nvarchar(79) NULL
ALTER TABLE NetflixContent ALTER COLUMN description nvarchar(250) NULL
```


## Processes and Stages of Dataset Cleaning

Step 1: Execute the SQL code below. Make a backup copy of NetflixContent
```sql
--Before you do anything on the imported data, we make a backup copy of NetflixContent

SELECT * INTO NetflixContent_stagging
FROM NetflixContent
```
**Objective:** To avoid fatal errors like mistakenly deleting the dataset. It is the professional best practice to first make a duplicate copy of the dataset to be analysed.

Step 2: Create a Primary Key for the Showid column in the table
```sql
ALTER TABLE 
	netflixStaging
ADD CONSTRAINT pk_ntc
PRIMARY KEY (show_id)
```
**Objective:** To begin to find duplicates, we have to create a primary key for the table. We chose the showid column because it has no Null values.

Step 3: Check for and replace all Null values in the columns of the table netflixStaging with 'NA' (Not Available)
```sql
UPDATE 
	netflixStaging
SET 
	type='NA' 
WHERE type Is Null

UPDATE 
	netflixStaging
SET 
	title='NA' 
WHERE title Is Null

UPDATE 
	netflixStaging
SET 
	director='NA' 
WHERE director Is Null

UPDATE 
	netflixStaging
SET 
	cast='NA' 
WHERE cast Is Null

UPDATE 
	netflixStaging
SET 
	country='NA' 
WHERE country Is Null

UPDATE 
	netflixStaging
SET 
	dateadded='NA' 
WHERE dateadded Is Null

UPDATE 
	netflixStaging
SET 
	rating = 'NA' 
WHERE rating Is Null

UPDATE 
	netflixStaging
SET 
	duration = 'NA' 
WHERE duration Is Null

UPDATE 
	netflixStaging
SET 
	listedin = 'NA' 
WHERE listedin Is Null

UPDATE 
	netflixStaging
SET 
	description = 'NA' 
WHERE description Is Null
```
**Objective:** To make the table easier to view by identifying and replacing Null values with 'NA'

Step 4: We begin to run several SQL queries to check for duplicate records
Checking for duplicate show_id
```sql
SELECT 
    Show_id,
	COUNT(*) duplicate_count
FROM 
    netflixStaging
GROUP BY 
		show_id
HAVING COUNT(*)>1
```

Step 5: After the duplicates have been identified, the next task depends on the requirements of the client. Some clients may want them stored in another table, and some may want them deleted. For this project, we are deleting the duplicates.

Method 1: Creating a table to store the duplicate records.
```sql
SELECT * INTO DeleledNetflixStagingRecs
FROM
	netflixStaging
WHERE title in 
(
	SELECT
		title
	FROM
		netflixStaging  
	GROUP BY
		title, type
	HAVING COUNT(*) > 1
)
```
**Objective:** Identifying the duplicates and moving them into a new table

Method 2: Delete duplicates. To use this method, we have to create a new column named ID, make it a PRIMARY KEY of datatype int, and let it auto increase by 1. 
For this project, this is the recommended method.
```sql
ALTER TABLE netflixStaging
DROP CONSTRAINT [pk_ntc]

ALTER TABLE netflixStaging
ADD ID INT IDENTITY(1,1) NOT NULL;

ALTER TABLE netflixStaging
ADD CONSTRAINT PK_netflixStaging_ID PRIMARY KEY (ID)
```
**Result:** We removed the showid PRIMARY KEY by dropping the constraint name associated with it. We then created a column ID with a datatype of INT, that autoincreases by 1 starting from 1, and does not accept null values.
Finally, we made the column the new PRIMARY KEY of the table.
```sql
DELETE FROM netflixStaging WHERE ID NOT IN
(
	SELECT MIN(ID) FROM netflixStaging
	GROUP BY Title, Type
)
```

Step 6: Before we start analysing the content of the table, we need to normalize it. We can see from the cast, listedin, directors, and country columns that several cells contain multiple values separated by commas. Below is a process of generating tables out of the columns since the values in the named columns have the same unique ID, using the relational operator CROSS APPLY and the function String_split()
```sql
SELECT 
  	showid, 
	TRIM(value) as cast
INTO
	netflixStaging_Cast
FROM
	netflixStaging
CROSS APPLY string_split(cast,',')
 
SELECT 
	showid, 
	TRIM(value) as listedin
INTO
	netflixStaging_ListedIn
FROM
	netflixStaging
CROSS APPLY string_split(listedin,',')
 
SELECT 
	showid, 
	TRIM(value) as country
INTO
	netflixStaging_Country
FROM
	netflixStaging
CROSS APPLY string_split(country,',')

SELECT 
	showid, 
	TRIM(value) as director
INTO
	netflixStaging_Director 
FROM
	netflixStaging
CROSS APPLY string_split(director,',')
```
**Result:** Tables netflixStaging_Director, netflixStaging_Country, netflixStaging_ListedIn and netflixStaging_Cast are generated.


Step 7: We have normalized the table, and the column showid is common to all the tables netflixStaging_Director, netflixStaging_Country, netflixStaging_ListedIn, netflixStaging_Cast, and netflixStaging. So, we do not need the columns listedin, directors, cast, and country in NetflixStaging.

Method 1: We use the code below if we want to select certain columns from a table into a temporary table and then drop the original table. Then we later rename the temporary table using the name of the original table. 
Note: We first make a copy of netflixStaging named netflixStagingCopy. We will call NetflixStagingCopy the unnormalized table.
```sql
SELECT
	*
INTO
	netflixStagingCopy
FROM
	netflixStaging

 SELECT  
	showid, type, title, dateadded, releaseyear, rating, duration, description
INTO 
	netflixStaging_temp
FROM 
	netflixStaging
DROP TABLE netflixStaging
EXEC sp_rename 'netflixStaging_temp', 'netflixStaging'
```

Method 2: Another way we can get rid of the unwanted columns is to delete them or drop the columns by altering the table.
 ```sql
 ALTER TABLE netflixStaging DROP Column director
 ALTER TABLE netflixStaging DROP Column country
 ALTER TABLE netflixStaging DROP Column cast
 ALTER TABLE netflixStaging DROP Column listed_in
```
**Result:** We now have a completely normalized table, netflixStaging. We can get any content we want (analyze) about the directors, cast, listedin, and country from the tables netflixStaging_Director, netflixStaging_Country, netflixStaging_ListedIn, netflixStaging_Cast, and netflixStaging using relational operators like join and inner join.


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

SELECT * FROM 
	netflix_titles_filter
WHERE 
	date_added >= DATEADD(Year, -5, GetDate())
```
**Objective:** Retrieve content added to Netflix in the last 5 years.


6.	List All Movies that are Documentaries

Method 1

```sql

SELECT * FROM netflix_titles_filter
WHERE Type = 'Movie' AND Listed_in LIKE '%Documentaries%'
```

Method 2

```sql

SELECT ntf.*, nli.listed_in 
FROM netflix_titles_filter ntf
JOIN netflix_listed_in nli
ON ntf.show_id = nli.show_id
HERE nli.Listed_in = 'Documentaries'
 SELECT * FROM netflix_listed_in

```

**Objective:** Retrieve all movies classified as documentaries.
 

7.	Find All Content Without a Director
   
 Method 1
 
```sql
 
SELECT * FROM netflix_titles_filter
WHERE director = 'NA'

```
Method 2

```sql

SELECT ntf.*, nd.director 
FROM netflix_titles_filter ntf
JOIN netflix_director nd
ON ntf.show_id = nd.show_id
WHERE nd.director = 'NA'

```
**Objective:** List content that does not have a director.


8.	Find How Many Movies Actor 'Salman Khan' Appeared in over the Last 10 Years

Method 1
  	
```sql

SELECT * FROM netflix_titles_filter
WHERE Type = 'Movie' AND cast LIKE '%Salman Khan%' AND  release_year > YEAR(GetDate()) - 10

 ```
 
Method 2

```sql
SELECT ntf.*, nc.cast 
FROM netflix_titles_filter ntf
JOIN netflix_cast nc
ON ntf.show_id = nc.show_id
WHERE nc.cast = 'Salman Khan' AND  ntf.release_year > YEAR(GetDate()) - 10
 ```
**Objective:** Count the number of movies featuring 'Salman Khan' in the last 10 years
 

9.	Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

Method 1

```sql

SELECT TOP (10)
Trim(Value) AS Actor,
COUNT(*) HighestNumber
FROM netflix_titles_filter
CROSS APPLY STRING_SPLIT(cast,',')
WHERE country LIKE '%India%' AND type = 'Movie'
GROUP BY Trim(Value)
Order BY COUNT(*) DESC
``` 
Method 2: Using JOIN

```sql
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

Method 1

```sql

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
	) AS CategorizedContents
      GROUP BY Category
 ```
 Method 2
 
 ```sql
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
    
Method 1

```sql

SELECT * FROM netflix_titles_filter

WHERE Type IN ('Movie', 'TV Show') AND Director LIKE '%Rajiv Chilaka%'
 ```
Method 2

```sql
SELECT * FROM netflix_titles_filter

WHERE Director LIKE '%Rajiv Chilaka%'
```

Method 3

```sql
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
**Objective:** Display content items added after August 20, 2021
 

15.	List movies added on June 15, 2019
```sql

SELECT * FROM Netflix_Titles_filter
WHERE type = 'Movie' AND date_added = '2019-06-15'
 ```
**Note:** If the date_added column in your table is stored as text (e.g., "June 15, 2019"), we can either:

- Compare it directly as a string, or
- Convert it into a DATE type using TRY_CONVERT for safer querying.

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

Method 1

```sql

SELECT * FROM Netflix_Titles_filter
WHERE date_added >= '2021-01-01' AND date_added <= '2021-12-31'
```
Method 2

```sql

SELECT * FROM Netflix_Titles_filter
WHERE date_added BETWEEN '2021-01-01' AND '2021-12-31'
```
Method 3

```sql
SELECT *  FROM Netflix_Titles_filter
WHERE date_added LIKE '%2021%'
```
Method 4

```sql
SELECT * from netflix_titles where Year(date_added) = 2021
``` 
**Objective:** Display content items added in 2021


17.	List movies added in 2021

Method 1

```sql

SELECT * FROM Netflix_Titles_filter
WHERE type = 'Movie' AND date_added >= '2021-01-01' AND date_added <= '2021-12-31'
```
Method 2

```sql

SELECT * FROM Netflix_Titles_filter
WHERE type = 'Movie' AND  date_added BETWEEN '2021-01-01' AND '2021-12-31'
```
Method 3
```sql

SELECT *  FROM Netflix_Titles_filter
WHERE type = 'Movie' AND date_added LIKE '%2021%'

```
Method 4

```sql

SELECT *  FROM Netflix_Titles_filter
Where type = 'Movie' AND Year(date_added) = 2021

```
**Objective:** Display movies added in 2021


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

18a. Count the number of movies and TV series that each director has produced in different columns, including a TotalCount column Movies + TV Shows) for each director
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
**Objective:** Count the number of movies and TV series that each director has produced in different columns with a TotalCount column Movies + TV Shows) for each director

18b. Count the number of movies and TV series that each director has produced in different columns, with TotalCount column (Movies + TV Shows) for each director, including percentage split of Movies vs TV Shows for each director.
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

Method 1: This version handles without considering ties 
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
Method 2:

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
