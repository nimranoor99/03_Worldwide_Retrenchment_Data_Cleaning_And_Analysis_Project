# Exploratory Data Analysis
Please find below some question and their related queries applied on previously cleaned data.
<br>

**Q1. Find those company which have done the 100% laid off.**
```sql
SELECT 
    *
FROM
    worldwide_retrenchment.retrenchment_staging2
WHERE
    percentage_laid_off = 1
ORDER BY total_laid_off DESC;
```

SELECT 
    *
FROM
    worldwide_retrenchment.retrenchment_staging2
WHERE
    percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
    
SELECT 
    company, SUM(total_laid_off)
FROM
    worldwide_retrenchment.retrenchment_staging2
GROUP BY company
ORDER BY 2 DESC;
    
SELECT 
    MIN(`Date`), MAX(`Date`)
FROM
    worldwide_retrenchment.retrenchment_staging2;

SELECT 
    YEAR(`Date`), SUM(total_laid_off)
FROM
    worldwide_retrenchment.retrenchment_staging2
GROUP BY YEAR(`Date`)
ORDER BY 1 DESC;
SELECT 
    SUBSTRING(`Date`, 1, 7) AS `Month`, SUM(total_laid_off)
FROM
    worldwide_retrenchment.retrenchment_staging2
WHERE
    SUBSTRING(`Date`, 1, 7) IS NOT NULL
GROUP BY `Month`
ORDER BY 1 ASC;
#Answer: 2023-03	4470
-- 2023-02	36493
-- 2023-01	84714
-- 2022-12	10329
-- 2022-11	53451
-- 2022-10	17406
-- 2022-09	5881
-- 2022-08	13055
-- 2022-07	16223
-- 2022-06	17394
-- 2022-05	12885
-- 2022-04	4128
-- 2022-03	5714
-- 2022-02	3685
-- 2022-01	510
-- 2021-12	1200
-- 2021-11	2070
-- 2021-10	22
-- 2021-09	161
-- 2021-08	1867
-- 2021-07	80
-- 2021-06	2434
-- 2021-04	261
-- 2021-03	47
-- 2021-02	868
-- 2021-01	6813
-- 2020-12	852
-- 2020-11	237
-- 2020-10	450
-- 2020-09	609
-- 2020-08	1969
-- 2020-07	7112
-- 2020-06	7627
-- 2020-05	25804
-- 2020-04	26710
-- 2020-03	9628

-- Q.What is the rolling total of laid off?
WITH Rolling_Total_CTE
AS
(
SELECT 
    SUBSTRING(`Date`,1,7) AS `Month`, SUM(total_laid_off) AS Laid_Off
FROM
    worldwide_retrenchment.retrenchment_staging2
WHERE SUBSTRING(`Date`,1,7) IS NOT NULL
GROUP BY `Month`
ORDER BY 1 ASC
)
SELECT `Month`,
Laid_Off,
SUM(Laid_Off) OVER (ORDER BY `Month`) AS Rolling_Total 	
FROM Rolling_Total_CTE ;


# Answer: 2020-03	9628	9628
-- 2020-04	26710	36338
-- 2020-05	25804	62142
-- 2020-06	7627	69769
-- 2020-07	7112	76881
-- 2020-08	1969	78850
-- 2020-09	609	79459
-- 2020-10	450	79909
-- 2020-11	237	80146
-- 2020-12	852	80998
-- 2021-01	6813	87811
-- 2021-02	868	88679
-- 2021-03	47	88726
-- 2021-04	261	88987
-- 2021-06	2434	91421
-- 2021-07	80	91501
-- 2021-08	1867	93368
-- 2021-09	161	93529
-- 2021-10	22	93551
-- 2021-11	2070	95621
-- 2021-12	1200	96821
-- 2022-01	510	97331
-- 2022-02	3685	101016
-- 2022-03	5714	106730
-- 2022-04	4128	110858
-- 2022-05	12885	123743
-- 2022-06	17394	141137
-- 2022-07	16223	157360
-- 2022-08	13055	170415
-- 2022-09	5881	176296
-- 2022-10	17406	193702
-- 2022-11	53451	247153
-- 2022-12	10329	257482
-- 2023-01	84714	342196
-- 2023-02	36493	378689
-- 2023-03	4470	383159


-- Q.What is company wise laid off rate ?

WITH Company_Laid_Off_Rate(Company, Year, Total_Laid_Off)
AS 
(
SELECT 
    company,YEAR(`date`), SUM(total_laid_off)
FROM
    worldwide_retrenchment.retrenchment_staging2
GROUP BY company,YEAR(`date`)
)
SELECT Company, Year, Total_Laid_Off
FROM Company_Laid_Off_Rate
;
# Answer: 

-- Q.What is company wise laid off rate per year? WHich company laid off high i 2022?
WITH Company_Laid_Off_Rate(Company, Year, Total_Laid_Off)
AS 
(
SELECT 
    company,YEAR(`date`), SUM(total_laid_off)
FROM
    worldwide_retrenchment.retrenchment_staging2
GROUP BY company,YEAR(`date`)
)
SELECT Company, Year, Total_Laid_Off,
DENSE_RANK() OVER (PARTITION BY Year ORDER BY Total_Laid_off DESC) AS Ranking
FROM Company_Laid_Off_Rate
WHERE Year IS NOT NULL
ORDER BY Ranking ASC;
# Answer: 

-- Q. What is the top 5 high laid off rankings for Year 2021?
WITH Company_Laid_Off_Rate(Company, Year, Total_Laid_Off)
AS 
(
SELECT 
    company,YEAR(`date`), SUM(total_laid_off)
FROM
    worldwide_retrenchment.retrenchment_staging2
GROUP BY company,YEAR(`date`)
),
Company_Year_Rank 
AS
(
SELECT Company, Year, Total_Laid_Off,
DENSE_RANK() OVER (PARTITION BY Year ORDER BY Total_Laid_off DESC) AS Ranking
FROM Company_Laid_Off_Rate
WHERE Year IS NOT NULL
ORDER BY Ranking ASC)

SELECT 
    *
FROM
    Company_Year_Rank
WHERE
    Ranking <= 5
        AND (Year = '2021' OR Year = '2022')
ORDER BY Year;
