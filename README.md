# SQL-data-cleaning
This is an educational project on data cleaning and preparation using SQL. The original database in CSV format is located in the file club_member_info.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset.
## Data Introduction
```sql
SELECT * FROM club_member_info cmi LIMIT 10;
```
The result:
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

## Copy Table
### Create new table for cleaning
```sql
-- club_member_info_cleaned definition

CREATE TABLE club_member_info_cleaned (
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone VARCHAR(50),
	full_address VARCHAR(50),
	job_title VARCHAR(50),
	membership_date VARCHAR(50)
);
```
### Copy all values from original table
```sql
INSERT INTO club_member_info_cleaned 
SELECT * FROM club_member_info;
```
## Clean Data
### Solving the Inconsistent letter case
```sql
UPDATE club_member_info_cleaned SET full_name = TRIM(full_name);
UPDATE club_member_info_cleaned SET full_name = UPPER(full_name);
```
### Solving the Age out of realistic range
```sql
UPDATE club_member_info_cleaned SET age =  CAST(SUBSTR(age, 1, 2) AS INTEGER) WHERE age > 90 AND age <> '';
UPDATE club_member_info_cleaned SET age = 'NULL' WHERE age = '';
```
### Fixing the Matial_status
```sql
UPDATE club_member_info_cleaned SET martial_status = 'divorced' WHERE martial_status  = 'divored';
UPDATE club_member_info_cleaned SET martial_status = 'unknown' WHERE martial_status  = '';
```
### Fixing the Phone_number & Job_title
```sql
UPDATE club_member_info_cleaned SET phone = 'unknown' WHERE LENGTH(phone) < 12;
UPDATE club_member_info_cleaned SET job_title = 'NULL' WHERE job_title = '';
```
### Fixing the Membership_date
```sql
UPDATE club_member_info_cleaned 
SET membership_date = '0' || SUBSTR(membership_date, 1, 2) || SUBSTR(membership_date, 3, 8) 
WHERE SUBSTR(membership_date, 1, 2) LIKE '_/';

UPDATE club_member_info_cleaned 
SET membership_date = SUBSTR(membership_date, 1, 3) || '0' || SUBSTR(membership_date, 4, 2) || SUBSTR(membership_date, 6, 8)
WHERE SUBSTR(membership_date, 4, 2) LIKE '_/';

UPDATE club_member_info_cleaned
SET membership_date = SUBSTR(membership_date, 1, 6) || '20' || SUBSTR(membership_date, 9, 2)
WHERE CAST(SUBSTR(membership_date, 7, 4) AS INTEGER) < 2000;
```
## Data after Cleaning
```sql
SELECT * FROM club_member_info_cleaned LIMIT 10;
```
The result:
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|ADDIE LUSH|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|07/31/2013|
|ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|05/27/2018|
|SYDEL SHARVELL|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/06/2017|
|CONSTANTIN DE LA CRUZ|35|unknown|co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|GAYLOR REDHOLE|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|05/29/2019|
|WANDA DEL MAR|44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|03/24/2015|
|JOANN KENEALY|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|04/17/2013|
|JOETE CUDIFF|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|MENDIE ALEXANDRESCU|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|03/12/2021|
|FEY KLOSS|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/05/2014|

## Preparing Data for Removing Duplications
### Create new table with id_column for removing duplications
```sql
-- club_member_info_duplicates_removed definition

CREATE TABLE club_member_info_duplicates_removed (
	id INTEGER PRIMARY KEY AUTOINCREMENT,
	full_name VARCHAR(50),
	age INTEGER,
	martial_status VARCHAR(50),
	email VARCHAR(50),
	phone VARCHAR(50),
	full_address VARCHAR(50),
	job_title VARCHAR(50),
	membership_date VARCHAR(50)
);
```
### Copy all values from cleaned table
```sql
INSERT INTO club_member_info_duplicates_removed (
	full_name,
	age,
	martial_status,
	email,
	phone,
	full_address,
	job_title,
	membership_date)
SELECT * FROM club_member_info_cleaned;
```
## Data Duplications Removing
### Step #1: Retrieving the min id by email
```sql
SELECT email, MIN(id) 
FROM club_member_info_duplicates_removed
GROUP BY email;
```
### Step #2: Joining the min id to original table
```sql
SELECT * FROM club_member_info_duplicates_removed as ud
JOIN (
	SELECT email, MIN(id) 
	FROM club_member_info_duplicates_removed
	GROUP BY email) as sub
ON ud.email = sub.email;
```
### Step #3: Eliminating the extra columns
```sql
SELECT ud.id, sub.id_min FROM club_member_info_duplicates_removed as ud
JOIN (
	SELECT email, MIN(id) as id_min 
	FROM club_member_info_duplicates_removed
	GROUP BY email) as sub
ON ud.email = sub.email;
```
### Step #4: Retrieving only the duplicates’ IDs
```sql
SELECT ud.id, sub.id_min FROM club_member_info_duplicates_removed as ud
JOIN (
	SELECT email, MIN(id) as id_min 
	FROM club_member_info_duplicates_removed
	GROUP BY email) as sub
ON ud.email = sub.email
WHERE ud.id > sub.id_min;
```
### Step 5: FINALE. Deleting the duplicates
```sql
DELETE FROM club_member_info_duplicates_removed 
WHERE id IN (
	SELECT ud.id, sub.id_min FROM club_member_info_duplicates_removed as ud
	JOIN (
		SELECT email, MIN(id) as id_min 
		FROM club_member_info_duplicates_removed
		GROUP BY email) as sub
	ON ud.email = sub.email
	WHERE ud.id > sub.id_min);
```
