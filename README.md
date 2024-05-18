# Project Overview

This project is basically a walkthrough on data cleaning and analysis of the world-layoffs-dataset using SQL. It was obsereved that the past three years had a huge contribution on
the world tech layoffs as well as in many different industries. That's why it's important to understand the job market and the possible reasons behind these layoffs.

*Dataset available at*: [https://github.com/CoderFek/World-Layoffs-Dataset](https://github.com/CoderFek/World-Layoffs-Analysis-Using-SQL/blob/main/layoffs.csv) (03/20 - 03/23)

#### Metadata:-

- `company` - Companies that contributed to world-layoffs
- `location` - current location of company
- `industry` - Kind of industry where the layoffs occured the most
- `total_laid_off` - The number of people laid off
- `percentage_laid_off` - (total_laid_off / number of employees) * 100%
- others..

## Data Cleaning:- 

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
