SQL Project: Data Cleaning - Global Layoffs Dataset

Project Overview

This project involves cleaning and preparing a dataset containing information about layoffs by companies worldwide during the years 2020, 2021, 2022, and 2023. The data was sourced from Kaggle. The goal is to clean the raw data for further exploratory data analysis (EDA) and insights generation.

Dataset

The dataset contains the following key columns:

company: Name of the company.

location: Location of the layoffs.

industry: Industry sector of the company.

total_laid_off: Total number of employees laid off.

percentage_laid_off: Percentage of employees laid off.

date: Date of layoffs.

stage: Business stage of the company.

country: Country where the layoffs occurred.

funds_raised_millions: Funds raised by the company (in millions).

Cleaning Steps

The data cleaning process follows these steps:

1. Remove Duplicates

Identified duplicate rows based on all columns.

Used ROW_NUMBER() to flag duplicates.

Removed duplicate entries while preserving unique data.

2. Standardize Data

Industry Column: Fixed null and empty values by populating them with consistent industry values where available.

Country Column: Standardized variations such as "United States." to "United States".

Date Column: Converted date strings to proper DATE format using STR_TO_DATE().

Other Fields: Consolidated inconsistent labels (e.g., standardized "Crypto" variations).

3. Handle Null Values

Retained null values in certain columns (total_laid_off, percentage_laid_off, and funds_raised_millions) for future calculations.

Removed rows with null values in both total_laid_off and percentage_laid_off as they were deemed unusable.

4. Drop Unnecessary Columns

Removed the row_num column used for intermediate processing.

Key SQL Queries

Create a Staging Table

CREATE TABLE world_layoffs.layoffs_staging
LIKE world_layoffs.layoffs;

INSERT INTO layoffs_staging
SELECT * FROM world_layoffs.layoffs;

Identify and Remove Duplicates

WITH DELETE_CTE AS (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
           ROW_NUMBER() OVER (
               PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
           ) AS row_num
    FROM world_layoffs.layoffs_staging
)
DELETE FROM world_layoffs.layoffs_staging
WHERE (company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions, row_num) IN (
    SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions, row_num
    FROM DELETE_CTE
) AND row_num > 1;

Standardize Industry and Country Fields

-- Standardize Industry
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');

-- Standardize Country
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

Convert Date Column to Proper Format

UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;

Remove Unusable Data

DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
  AND percentage_laid_off IS NULL;

Outcomes

After cleaning, the dataset is:

Free from duplicates and inconsistencies.

Standardized for seamless analysis.

Ready for exploratory data analysis (EDA) and visualization.

