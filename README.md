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

### Data Cleaning

This section details the SQL queries used to clean and standardize data within the `artworks` and `artist` tables.

* **Standardize Date Formats**: Ensures the `DateAcquired` column in the `artworks` table is in a consistent `DATE` format.
* **Populating Nationality from Artistbio**: Identifies and populates the `Nationality` column in the `artist` table where it's `NULL` but can be extracted from the `Artistbio` field.
    * Finding The Problem
    * Testing the Solution
    * Updating the Table
* **Handling Gender Null Values**: Updates `NULL` values in the `Gender` column of the `artist` table to `'gender non-conforming'`.
* **Replacing Nulls in Wiki_QID and ULAN**: Replaces `NULL` values in `Wiki_QID` and `ULAN` columns of the `artist` table with `'Unknown'`.
* **Removing Null Values from Artistbio and Nationality**: Replaces `NULL` values in `Artistbio` and `Nationality` columns of the `artist` table with `'Unknown'`.

---

### Creating Multiple Tables from Artworks

This section demonstrates how the original `artworks` table is denormalized into three more focused tables for better organization and analytical querying.

* **`Mcollection` Table**: Created to store collection-specific metadata.
* **`MOMA` Table**: Created to store physical dimensions and URLs related to the artworks.
* **`Artwork_fact` Table**: Created to store core factual information about each artwork.

---

### Data Cleaning of New Tables

Further cleaning operations are applied to the newly created tables to ensure data quality.

* **`Artwork_fact` Table Cleaning**: Handles inconsistent or missing `Title`, `Medium`, `Dimensions`, and `CreditLine` values.
* **`Mcollection` Table Cleaning**: Checks and fills `NULL` `DateAcquired` values with a default date.
* **`MOMA` Table Cleaning**: Replaces `NULL` values in `URL`, `ImageURL`, `OnView`, and various dimension/duration columns with `'NA'` or `'0.0'`.

---

### Creating Columns for Analysis

Adds a `Decade` column to the `Mcollection` table for time-based analysis.

---

### Data Analysis Queries

These queries provide valuable insights into the MoMA collection.

* **How Modern are the Artworks?**: Analyzes the number of artworks acquired per decade to understand the collection's modernity.
* **Which Artists are Featured the Most?**: Identifies artists with the highest number of artworks in the collection.
* **What Types of Artworks Are the Most Common?**: Examines the distribution of artworks by classification, department, and medium.
    * Classification Wise
    * Department Wise
    * Medium Wise
* **Are there Any Trends in the Date of Acquisition?**: Investigates yearly and monthly acquisition trends.
    * Yearly Trends (Note: All `NULL` values in the date column were converted to `1900-01-01` during cleaning.)
    * Monthly Trends

---

### Recommended Analysis: Department-wise Artwork Acquisition by Decade

This query provides valuable insight into the acquisition trends of each department over multiple decades, which can help in predicting future resource requirements and strategic planning.
