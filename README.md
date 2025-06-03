# MoMA Collection SQL Analysis

This project utilizes **SQL** to perform data preparation and in-depth analysis on a dataset from the **Museum of Modern Art (MoMA) collection**. It covers essential data cleaning techniques, table normalization, and analytical queries to answer key questions about the collection's characteristics, artist representation, and acquisition patterns over time.

---

## Database Schema

The analysis implies an initial database schema with `artworks` and `artist` tables. Through the cleaning and transformation process, three new tables are created: `Mcollection`, `MOMA`, and `Artwork_fact`.

* **`artworks`**: Contains raw data about individual artworks.
* **`artist`**: Contains raw data about artists.
* **`Mcollection`**: Stores collection-related metadata (`AccessionNumber`, `Classification`, `Department`, `DateAcquired`, `Cataloged`).
* **`MOMA`**: Stores physical attributes and URLs (`ObjectID`, `URL`, `ImageURL`, `OnView`, `dimensions`, `duration`).
* **`Artwork_fact`**: Stores core artwork details (`Title`, `ConstituentID`, `AccessionNumber`, `ObjectID`, `Medium`, `Dimensions`, `CreditLine`).

---

## Setup

To run these SQL queries, you'll need:

* A **SQL Server instance** (or a compatible SQL database).
* The **MoMA collection dataset** loaded into your database. Ensure your raw tables are named `artworks` and `artist` as per the implicit schema.

**Note**: The provided queries are primarily written for **SQL Server syntax**. Minor adjustments might be required for functions like `CONVERT`, `CHARINDEX`, `DATENAME`, and `COALESCE` if you are using a different database system (e.g., PostgreSQL, MySQL).

---

## SQL Queries

### Initial Data Exploration

These queries provide a quick overview of the `artworks` and `artist` tables.
```sql
SELECT * FROM artworks;
SELECT * FROM artist;
```

### Data Cleaning

This section details the SQL queries used to clean and standardize data within the `artworks` and `artist` tables.

* **Standardize Date Formats**: Ensures the `DateAcquired` column in the `artworks` table is in a consistent `DATE` format.
```sql
SELECT DATEACQUIRED, CONVERT(DATE, DATEACQUIRED)
FROM artworks;

UPDATE artworks
SET DateAcquired = CONVERT(DATE, DateAcquired);
```
* **Populating Nationality from Artistbio**: Identifies and populates the `Nationality` column in the `artist` table where it's `NULL` but can be extracted from the `Artistbio` field.
* Finding The Problem
```sql
SELECT Artistbio, nationality
FROM artist
WHERE artistbio IS NOT NULL AND nationality IS NULL;
```
   * Testing the Solution
```sql
SELECT
 	Artistbio,
 	LEFT(Artistbio, CHARINDEX(',', Artistbio) - 1) AS newnationality
FROM
 	Artist
WHERE
 	CHARINDEX(',', Artistbio) > 0
 	AND Nationality IS NULL;
```
   * Updating the Table
```sql
UPDATE Artist
SET nationality = LEFT(Artistbio, CHARINDEX(',', Artistbio) - 1)
WHERE CHARINDEX(',', Artistbio) > 0
AND nationality IS NULL;
```
* **Handling Gender Null Values**: Updates `NULL` values in the `Gender` column of the `artist` table to `'gender non-conforming'`.
```sql
SELECT DISTINCT(gender)
FROM artist;

UPDATE Artist
SET Gender = 'gender non-conforming'
WHERE Gender IS NULL;
```
* **Replacing Nulls in Wiki_QID and ULAN**: Replaces `NULL` values in `Wiki_QID` and `ULAN` columns of the `artist` table with `'Unknown'`.
```sql
SELECT ConstituentID, ULAN
FROM artist
WHERE ULAN IS NULL;

UPDATE Artist
SET Wiki_QID = 'Unkown'
WHERE Wiki_QID IS NULL;

UPDATE Artist
SET ULAN = 'Unkown'
WHERE ULAN IS NULL;
```
* **Removing Null Values from Artistbio and Nationality**: Replaces `NULL` values in `Artistbio` and `Nationality` columns of the `artist` table with `'Unknown'`.
```sql
SELECT constituentid, DisplayName
FROM Artist
WHERE Artistbio IS NULL AND Nationality IS NULL;

UPDATE Artist
SET Artistbio = 'Unkown'
WHERE Artistbio IS NULL;

UPDATE Artist
SET Nationality = 'Unkown'
WHERE Nationality IS NULL;
```
---

### Creating Multiple Tables from Artworks

This section demonstrates how the original `artworks` table is denormalized into three more focused tables for better organization and analytical querying.
```sql
SELECT * FROM Artworks;

SELECT COUNT(objectID) AS cnt, COUNT(DISTINCT(objectid)) AS uniq FROM Artworks;

SELECT COUNT(AccessionNumber) AS cnt, COUNT(DISTINCT(AccessionNumber)) AS uniq
FROM Artworks;
```

* **`Mcollection` Table**: Created to store collection-specific metadata.
```sql
SELECT AccessionNumber, Classification, Department, DateAcquired, Cataloged
INTO Mcollection
FROM Artworks;

SELECT * FROM Mcollection;
```
* **`MOMA` Table**: Created to store physical dimensions and URLs related to the artworks.
```sql
SQL
SELECT
 	ObjectID, URL, ImageURL, OnView, Circumference_cm, Depth_cm,
 	Diameter_cm, Height_cm, Length_cm, Weight_kg, Width_cm, Seat_Height_cm, Duration_sec
INTO MOMA
FROM Artworks;

SELECT * FROM MOMA;
```
* **`Artwork_fact` Table**: Created to store core factual information about each artwork.
```sql
SELECT Title, ConstituentID, AccessionNumber, ObjectID, Medium, Dimensions, CreditLine
INTO Artwork_fact
FROM Artworks;

SELECT * FROM Artwork_fact;
```

---

### Data Cleaning of New Tables

Further cleaning operations are applied to the newly created tables to ensure data quality.

* **`Artwork_fact` Table Cleaning**: Handles inconsistent or missing `Title`, `Medium`, `Dimensions`, and `CreditLine` values.
```sql
SELECT COUNT(Title) FROM Artwork_fact WHERE Title = '!!?!!!';

UPDATE Artwork_fact
SET Title = COALESCE(Title, 'Untitled');

UPDATE Artwork_fact
SET Title ='Untitled'
WHERE Title ='!';

UPDATE Artwork_fact
SET Title ='Untitled'
WHERE Title ='!!?!!!';

UPDATE Artwork_fact
SET Medium = 'Unkown'
WHERE Medium IS NULL;

UPDATE Artwork_fact
SET Dimensions = 0
WHERE Dimensions IS NULL;

UPDATE Artwork_fact
SET CreditLine = 'Unkown'
WHERE CreditLine IS NULL;
```
* **`Mcollection` Table Cleaning**: Checks and fills `NULL` `DateAcquired` values with a default date.
```sql
SELECT * FROM Mcollection;

SELECT DISTINCT(Classification) FROM Mcollection;
SELECT DISTINCT(Department) FROM Mcollection;
SELECT DISTINCT(Cataloged) FROM Mcollection;

SELECT AccessionNumber, DateAcquired
FROM Mcollection
WHERE DateAcquired IS NULL;

SELECT AccessionNumber, COALESCE(CONVERT(VARCHAR(10), DateAcquired),'1900-01-01')
FROM Mcollection
WHERE DateAcquired IS NULL;

UPDATE Mcollection
SET DateAcquired = COALESCE(CONVERT(VARCHAR(10), DateAcquired),'1900-01-01');
```
* **`MOMA` Table Cleaning**: Replaces `NULL` values in `URL`, `ImageURL`, `OnView`, and various dimension/duration columns with `'NA'` or `'0.0'`.
```sql
SELECT * FROM MOMA;

SELECT DISTINCT(Duration_sec) FROM MOMA;

UPDATE MOMA
SET URL = 'NA'
WHERE URL IS NULL;

UPDATE MOMA
SET ImageURL = 'NA'
WHERE ImageURL IS NULL;

UPDATE MOMA
SET OnView = COALESCE(OnView, 'NA');

UPDATE MOMA
SET Circumference_cm = COALESCE(Circumference_cm, '0.0');

UPDATE MOMA
SET Depth_cm = COALESCE(Depth_cm, 0.0);

UPDATE MOMA
SET Diameter_cm = COALESCE(Diameter_cm, 0.0);

UPDATE MOMA
SET Length_cm = COALESCE(Length_cm, 0.0);

UPDATE MOMA
SET Weight_kg = COALESCE(Weight_kg, 0.0);

UPDATE MOMA
SET Width_cm = COALESCE(Width_cm, 0.0);

UPDATE MOMA
SET Seat_Height_cm = COALESCE(Seat_Height_cm, 0.0);

UPDATE MOMA
SET Duration_sec = COALESCE(duration_sec, 0);
```

---

### Creating Columns for Analysis

Adds a `Decade` column to the `Mcollection` table for time-based analysis.
```sql
ALTER TABLE Mcollection
ADD Decade INT;

UPDATE Mcollection
SET Decade = CASE
    WHEN YEAR(DateAcquired) BETWEEN 1900 AND 1920 THEN 1910
    WHEN YEAR(DateAcquired) BETWEEN 1921 AND 1930 THEN 1920
    WHEN YEAR(DateAcquired) BETWEEN 1931 AND 1940 THEN 1930
    WHEN YEAR(DateAcquired) BETWEEN 1941 AND 1950 THEN 1940
    WHEN YEAR(DateAcquired) BETWEEN 1951 AND 1960 THEN 1950
    WHEN YEAR(DateAcquired) BETWEEN 1961 AND 1970 THEN 1960
    WHEN YEAR(DateAcquired) BETWEEN 1971 AND 1980 THEN 1970
    WHEN YEAR(DateAcquired) BETWEEN 1981 AND 1990 THEN 1980
    WHEN YEAR(DateAcquired) BETWEEN 1991 AND 2000 THEN 1990
    WHEN YEAR(DateAcquired) BETWEEN 2001 AND 2010 THEN 2000
    WHEN YEAR(DateAcquired) BETWEEN 2011 AND 2020 THEN 2010
    WHEN YEAR(DateAcquired) BETWEEN 2021 AND 2030 THEN 2020
    ELSE NULL
END;
```

---

### Data Analysis Queries

These queries provide valuable insights into the MoMA collection.
```sql
SELECT * FROM Artist;
SELECT * FROM MOMA;
SELECT * FROM Artwork_fact;
SELECT * FROM Mcollection;
```

* **How Modern are the Artworks?**: Analyzes the number of artworks acquired per decade to understand the collection's modernity.
```sql
SQL
SELECT Decade, COUNT(AccessionNumber) AS Number_of_Artworks
FROM Mcollection
GROUP BY Decade
ORDER BY Number_of_Artworks DESC;
```
* **Which Artists are Featured the Most?**: Identifies artists with the highest number of artworks in the collection.
```sql
SELECT a.DisplayName, COUNT(b.AccessionNumber) AS Number_of_Artworks
FROM Artist a
INNER JOIN Artwork_fact b
ON a.ConstituentID = b.ConstituentID
GROUP BY a.DisplayName
ORDER BY Number_of_Artworks DESC;
```
* **What Types of Artworks Are the Most Common?**: Examines the distribution of artworks by classification, department, and medium.
    * Classification Wise
```sql
SELECT Classification, COUNT(AccessionNumber) AS Number_of_Artworks
FROM Mcollection
GROUP BY Classification
ORDER BY Number_of_Artworks DESC;
```
  * Department Wise
```sql
SELECT Department, COUNT(AccessionNumber) AS Number_of_Artworks
FROM Mcollection
GROUP BY Department
ORDER BY Number_of_Artworks DESC;
```
  * Medium Wise
```sql
SELECT Medium, COUNT(DISTINCT(AccessionNumber)) AS Number_of_Artworks
FROM Artwork_fact
GROUP BY Medium
ORDER BY Number_of_Artworks DESC;
```
* **Are there Any Trends in the Date of Acquisition?**: Investigates yearly and monthly acquisition trends.
    * Yearly Trends (Note: All `NULL` values in the date column were converted to `1900-01-01` during cleaning.)
```sql
SELECT YEAR(DateAcquired) AS AcquisitionYear,
 	COUNT(*) AS NumberOfAcquisitions
FROM Mcollection
GROUP BY YEAR(DateAcquired)
ORDER BY AcquisitionYear;
```
  * Monthly Trends
```sql
SELECT MONTH(DateAcquired) AS AcquisitionMonth,
 	DATENAME(month, DateAcquired) AS MonthName,
 	COUNT(*) AS NumberOfAcquisitions
FROM Mcollection
GROUP BY MONTH(DateAcquired),
 	DATENAME(month, DateAcquired)
ORDER BY AcquisitionMonth;
```

---

### Recommended Analysis: Department-wise Artwork Acquisition by Decade

This query provides valuable insight into the acquisition trends of each department over multiple decades, which can help in predicting future resource requirements and strategic planning.
```sql
SELECT
 	mcollection.decade,
 	mcollection.department,
 	COUNT(mcollection.AccessionNumber) AS NumberOfArtworks
FROM
 	mcollection
GROUP BY
 	mcollection.decade,
 	mcollection.department
ORDER BY
 	mcollection.decade,
 	mcollection.department;
```

