# 01- Data Cleaning 
<br>

**Pupose:** Need to clean the data for further processing (like visulaization) of data.
<br>

## **Procedure**
<br>

**Step 01 -** Create Database
<br>

**Step 02 -** Import Data in Database
<br>

**Step 03 -** Clean Data
<br>

## **** Step 01-Create Database of Name Worldwide_Retrenchment **** 
```sql
CREATE schema Worldwide_Retrenchment;
```
## **** Step 02-Import Data in Database Worldwide_Retrenchment **** 
Follow guide in repo "Import Data into MySQL"
<br>

### a. Check imported data to get ready for next step of data cleaning.
```sql
CREATE TABLE retrenchment_Staging LIKE retrenchment;
```
```sql
SELECT 
    *
FROM
    retrenchment_staging;
```
### b. Create Duplicate Table - Insert Data into new table.
```sql
INSERT retrenchment_staging
SELECT *
FROM retrenchment;
```
```sql
SELECT 
    *
FROM
    retrenchment_staging;
```
## **** Step 03-Data cleaning in Database Worldwide_Retrenchment by using table retrenchment_staging **** 
### a. Remove Duplicates - INSERT Row Number
```sql
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country,funds_raised_millions) AS Row_Number_Assigned
FROM retrenchment_staging;
```
### a. Remove Duplicates - Add CTE to filter the Row Number >1
```sql
WITH CTE_Duplicates_Check AS
(
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS Row_Number_Assigned
FROM retrenchment_staging
)
SELECT *
FROM CTE_Duplicates_Check 
WHERE Row_Number_Assigned > 1;
```
### a. Remove Duplicates - Verify the Provided Duplicated Values
```sql
SELECT 
    *
FROM
    retrenchment_staging
WHERE
    company = 'Casper';
```
### a. Remove Duplicates - CTE Update command not support so let's create one table, insert data into it and delete row>2
```sql
CREATE TABLE `retrenchment_staging2` (
    `company` TEXT,
    `location` TEXT,
    `industry` TEXT,
    `total_laid_off` INT DEFAULT NULL,
    `percentage_laid_off` TEXT,
    `date` TEXT,
    `stage` TEXT,
    `country` TEXT,
    `funds_raised_millions` INT DEFAULT NULL,
    `Row_Number_Assigned` INT
)  ENGINE=INNODB DEFAULT CHARSET=UTF8MB4 COLLATE = UTF8MB4_0900_AI_CI;
```
```sql
INSERT INTO  `retrenchment_staging2`
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS Row_Number_Assigned
FROM retrenchment_staging;
```
```sql
DELETE FROM `worldwide_retrenchment`.`retrenchment_staging2` 
WHERE
    worldwide_retrenchment.retrenchment_staging2.Row_Number_Assigned > 1;
```
```sql
SELECT 
    *
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`
WHERE
    worldwide_retrenchment.retrenchment_staging2.Row_Number_Assigned > 1;
```
### b. Standarize the data - Remove spaces in comapny name
```sql
UPDATE `worldwide_retrenchment`.`retrenchment_staging2` 
SET 
    company = TRIM(company);
```
### b. Standarize the data - should be Unique values of Industry like there should be either Crypto or Crypto Currency as industry type
```sql
SELECT DISTINCT
    industry
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`;
```
```sql
UPDATE `worldwide_retrenchment`.`retrenchment_staging2` 
SET 
    industry = 'Crypto'
WHERE
    industry LIKE 'Crypto%'
        OR industry LIKE '%Crypto';
```
### b. Standarize the data - for country trim the . at the end of united states
```sql
SELECT DISTINCT
    country, TRIM(TRAILING '.' FROM country)
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`
ORDER BY 1;
```
```sql
UPDATE `worldwide_retrenchment`.`retrenchment_staging2` 
SET 
    country = TRIM(TRAILING '.' FROM country);
```
```sql
SELECT 
    *
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`
ORDER BY 1;
```
### b. Standarize the data - convert date into `date` format
```sql
SELECT 
    `date`, STR_TO_DATE(`date`, '%m/%d/%Y')
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`;
```
```sql
UPDATE `worldwide_retrenchment`.`retrenchment_staging2` 
SET 
    `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```
### b. Standarize the data - Change into the date typr for 'date' column
```sql
ALTER TABLE `worldwide_retrenchment`.`retrenchment_staging2`
MODIFY COLUMN `date` DATE;
```
### c. Null/Blanks Values Check - First start from company and location
```sql
SELECT 
    company, location, industry
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`
WHERE
    industry IS NULL OR industry = '';
```
### c. Null/Blanks Values Check - in company and location there are no NULL values so let's remove the ''  and NULL values of industry
```sql
SELECT 
    *
FROM
    `worldwide_retrenchment`.`retrenchment_staging2` Stage1
        JOIN
    `worldwide_retrenchment`.`retrenchment_staging2` Stage2 ON Stage1.company = stage2.company
        AND Stage1.location = Stage2.location
WHERE
    (Stage1.industry IS NULL
        OR Stage1.industry = '')
        AND Stage2.industry IS NOT NULL;
```
```sql
UPDATE `worldwide_retrenchment`.`retrenchment_staging2` 
SET 
    industry = NULL
WHERE
    industry = '';
```
```sql
UPDATE `worldwide_retrenchment`.`retrenchment_staging2` Stage1
        JOIN
    `worldwide_retrenchment`.`retrenchment_staging2` Stage2 ON Stage1.company = stage2.company
        AND Stage1.location = Stage2.location 
SET 
    Stage1.industry = Stage2.industry
WHERE
    Stage1.industry IS NULL
        AND Stage2.industry IS NOT NULL;
```
```sql
SELECT 
    *
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`
WHERE
    industry IS NULL;
```
### c. Null/Blanks Values Check - Remove where total laid and percentage laid are NULL
```sql
DELETE FROM `worldwide_retrenchment`.`retrenchment_staging2` 
WHERE
    total_laid_off IS NULL
    AND percentage_laid_off IS NULL;
```
### d. Remove Unecessary Columns
```sql
ALTER TABLE `worldwide_retrenchment`.`retrenchment_staging2`
DROP COLUMN Row_Number_Assigned;
```
```sql
SELECT 
    *
FROM
    `worldwide_retrenchment`.`retrenchment_staging2`;
```   

`**Now "worldwide_retrenchment`.`retrenchment_staging2" is cleaned data table and ready for exploratory analysis**`
