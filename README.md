# Data-Cleaning

## Table of Content
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Preparation](#data-preparation)
- [Results](#results)
- [Recommendations](#recommendations)
- [Limitations](#limitations)

### Project Overview
This data analysis project involved the sorting, standardization and all-round cleaning of global layoffs from a range of companies and industries from March 2020 to April of 2023.

### Data Sources
Layoffs Data: The primary data source for this data is the “layoffs.csv” file, containing detailed information regarding how many layoffs a company made per year, per location.

### Tools
- SQL

### Data Preparation
As part of the ‘cleaning’ process, I performed the following tasks;
- Removed any duplicates
- Standardize the data, i.e. checked for inconsistencies in capitalization and trimmed spaces left on either side of the row entries
- Inspected for any null values
- Remove any unnecessary columns and rows

### Data Analysis
Highlighting the steps taken to remove duplicates.
- Step 1: In order to inspect for the presence of duplicates, I first created a "staging" version of the "layoffs.csv" so as to keep a spare in the event of any irreversible changes;
```SQL
CREATE TABLE layoffs_staging
LIKE layoffs;
```
- Step 2: Next, using the following command, I assigned row numbers/counts to each row entry;
```SQL
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, location, industry, total_laid_off, 
percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM layoffs_staging;
```
- Step 3: The previous command also created a column called "row_num" containing a number/count for each entry. Next was to inspect for duplicates using a CTE to isolate entries greater than one (>1);
```SQL
with duplicate_cte as
(
select *,
row_number() over(
partition by company, location, industry, total_laid_off, 
percentage_laid_off, `date`, stage, country, funds_raised_millions) as row_num
from layoffs_staging
)
select *
from duplicate_cte
where row_num > 1;
```
- Step 4: The previous command displayed the columns bearing the duplicates and as such, we must delete them in order to have uniform data. As with Step 1, a staging table was created since deleted data may be difficult to recover. The following code was issued;
```SQL
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
  row_num int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
* Note that this is a default 'Create Statement".

Once this new table "layoffs_staging2" was created, the same sequence to reveal the duplicates was initiated and the extra entries deleted;
```SQL
delete
from layoffs_staging2
where row_num > 1;
```

### Results
While this was a simple data cleaning project, it was clear that some of the data entered in the database were either incomplete or inconsistent across the years. This can be as a result of failing to keep track of changes and inputing them at the time of occurence, we could also assume this could be a case of companies who laid off 100% of their workforce and effectively shut down operations.

### Recommendations
When venturing into data cleaning or standardization projects, ensure to outline the basic tasks you which to accomplish. For example, as a rule of thumb, always look out for duplicates as most times, these will skew your final analysis. You should also look out for null values or empty cells and assess them, deciding whether to populate or delete such rows, as these can also affect your results as well.

### Limitations
With data sourced from the around the globe, from different companies in different countries at different stages, it is not uncommon for the reported data to only closely resemble the true story. Thus number of layoffs recorded does not accurately represent the extent to which the covid pandemic truly affected the global workforce and the global economy at large.

🃏
