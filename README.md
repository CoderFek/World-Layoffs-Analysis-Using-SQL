# Project Overview

This project is basically a walkthrough on data cleaning and analysis of the world-layoffs-dataset using SQL. It was obsereved that the past three years had a huge contribution on
the world tech layoffs as well as in many different industries. That's why it's important to understand the job market and the possible reasons behind these layoffs.

<br>
<p align = "center">
 <img src = "https://github.com/CoderFek/World-Layoffs-Analysis-Using-SQL/assets/93084608/1866cf6a-e1d1-41c5-a42f-d0933dbe1841" width = "700" height = "400">
</p>
<br>

*Dataset available at*: [https://github.com/CoderFek/World-Layoffs-Dataset](https://github.com/CoderFek/World-Layoffs-Analysis-Using-SQL/blob/main/layoffs.csv) (03/20 - 03/23)

#### Metadata:-

- `company` - Companies that contributed to world-layoffs
- `location` - current location of company
- `industry` - Kind of industry where the layoffs occured the most
- `total_laid_off` - The number of people laid off
- `percentage_laid_off` - (total_laid_off / number of employees) * 100%
- others..

## Data Cleaning

```
SELECT *
FROM layoffs_staging;


-- Creating a duplicate table for analysis

CREATE TABLE layoffs_staging
LIKE layoffs;


-- Finding duplicate rows

WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`) AS row_num
FROM layoffs_staging
)

SELECT *
FROM duplicate_cte
WHERE row_num > 1;

CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT *
FROM layoffs_staging2
WHERE row_num >1;

INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;


-- Turn off safe updates

SET SQL_SAFE_UPDATES = 0;


-- Deleting duplicate rows

DELETE
FROM layoffs_staging2
WHERE row_num >1;

SELECT *
FROM layoffs_staging2
WHERE row_num >1;


-- Fixing typos

SELECT company, TRIM(company)
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET company = TRIM(company);

SELECT DISTINCT industry
FROM layoffs_staging2
ORDER BY industry;

SELECT *
FROM layoffs_staging2
WHERE industry like 'Crypto%';

UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';

SELECT DISTINCT country
FROM layoffs_staging2
ORDER BY 1;

UPDATE layoffs_staging2
SET country = 'United States'
WHERE country LIKE 'United States%';


-- Changing the data type

SELECT `date`,
STR_TO_DATE(`date`, '%m/%d/%Y')
FROM layoffs_staging2;

UPDATE layoffs_staging2
SET `date`= STR_TO_DATE(`date`, '%m/%d/%Y');

SELECT `date`
FROM layoffs_staging2;

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;


-- Removing null values

SELECT COUNT(*)
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL OR industry = '';

SELECT *
FROM layoffs_staging2
WHERE company = 'Airbnb';


-- Change all blank values to null

UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

-- Self Join

Select *
FROM layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;


-- Update values

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
	ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;


-- There is still a null value

SELECT *
FROM layoffs_staging2
WHERE company LIKE 'Bally%';
```

## Exploratory Data Analysis

```
SELECT *
FROM layoffs_staging2;


-- MAX layoff in 1 day
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging2;


-- Companies that laid off all the employees
SELECT *
FROM layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;


-- Companies with the highest number of layoffs
SELECT company, SUM(total_laid_off) AS max_layoffs
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;


-- Layoffs by stages
SELECT COUNT(company), stage, SUM(total_laid_off) AS max_layoffs
FROM layoffs_staging2
GROUP BY stage
ORDER BY 3 DESC;


-- Timeframe in which the layoffs have occured
SELECT MIN(`date`), MAX(`date`)
FROM layoffs_staging2;
-- It started in March 2020 when covid hit.


-- Which regions where inpacted the most
SELECT country, SUM(total_laid_off) AS max_layoffs
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
-- USA was impacted the most with 256 thousand layoffs


-- Which industry has the maximum contribution on layoffs
SELECT industry, SUM(total_laid_off) AS max_layoffs
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
-- Consumer and Retail, makes sense.


-- Indian industries vs layoffs
SELECT industry,
SUM(CASE WHEN YEAR(`date`) = 2020 THEN total_laid_off ELSE 0 END) AS layoffs_2020,
SUM(CASE WHEN YEAR(`date`) = 2021 THEN total_laid_off ELSE 0 END) AS layoffs_2021,
SUM(CASE WHEN YEAR(`date`) = 2022 THEN total_laid_off ELSE 0 END) AS layoffs_2022,
SUM(CASE WHEN YEAR(`date`) = 2023 THEN total_laid_off ELSE 0 END) AS layoffs_2023,
SUM(total_laid_off) AS total_layoffs
FROM layoffs_staging2
WHERE country LIKE '%ndia'
GROUP BY industry
ORDER BY 6 DESC;


-- Indian companies and layoffs
SELECT company, industry,
SUM(CASE WHEN YEAR(`date`) = 2020 THEN total_laid_off ELSE 0 END) AS layoffs_2020,
SUM(CASE WHEN YEAR(`date`) = 2021 THEN total_laid_off ELSE 0 END) AS layoffs_2021,
SUM(CASE WHEN YEAR(`date`) = 2022 THEN total_laid_off ELSE 0 END) AS layoffs_2022,
SUM(CASE WHEN YEAR(`date`) = 2023 THEN total_laid_off ELSE 0 END) AS layoffs_2023,
SUM(total_laid_off) AS total_layoffs
FROM layoffs_staging2
WHERE country LIKE '%ndia'
GROUP BY company, industry
ORDER BY 3 DESC;

-- In India education industry was hugely affected by layoffs followed by transportation and food


-- Time Series analysis

SELECT YEAR(`date`), SUM(total_laid_off) AS max_layoffs
FROM layoffs_staging2
WHERE YEAR(`date`) IS NOT NULL
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;


-- Monthly analysis
SELECT SUBSTRING(`date`,1,7) AS `Month`, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY `Month`
ORDER BY `Month`;
-- 84714 of layoffs in just first month of 2023. WOW


-- Monthly analysis with industry
SELECT SUBSTRING(`date`,1,7) AS `Month`, industry, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY `Month`, industry
ORDER BY `Month`;


-- Now let's see the maximum layoffs by month and which industry has contributed the most month wise
WITH monthly_sums AS (
    SELECT 
        SUBSTRING(`date`, 1, 7) AS `Month`, 
        industry, 
        SUM(total_laid_off) AS highest_laid_off
    FROM 
        layoffs_staging2
    WHERE 
        SUBSTRING(`date`, 1, 7) IS NOT NULL
    GROUP BY 
        `Month`, industry
),
max_monthly_sums AS (
    SELECT 
        `Month`, 
        MAX(highest_laid_off) AS max_laid_off
    FROM 
        monthly_sums
    GROUP BY 
        `Month`
)
SELECT 
    ms.`Month`,
    ms.industry,
    ms.highest_laid_off
FROM 
    monthly_sums ms
JOIN 
    max_monthly_sums mms
ON 
    ms.`Month` = mms.`Month`
    AND ms.highest_laid_off = mms.max_laid_off
ORDER BY 
    ms.`Month`;
    
-- Let's see the companies that laid off in 2023
SELECT company, total_laid_off
FROM layoffs_staging2
WHERE YEAR(`date`) = 2023
ORDER BY total_laid_off DESC;

/* 
Hmm, just as i predicted. Most of these layoffs happened in the beginning of 2023 and it is safe say
that these layoffs are mostly accompanied by the corporate industry.
*/
```
