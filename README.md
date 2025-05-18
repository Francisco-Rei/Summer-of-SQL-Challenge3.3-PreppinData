# Preppin Data 2023 Week 3
This is a challenge from [Summer of SQL](https://github.com/wjsutton/the_summer_of_sql) to help gain skills and confidence in SQL.

Skills tested in challenge 3.3:
- Common Table Expressions (CTEs)


## Challenge: Preppin Data 2023 Week 3

Inputs
- A list of the transactions

![Input](https://blogger.googleusercontent.com/img/a/AVvXsEi5t8Gjk3PuCXgJN9slk6ja37iyookPwAsuBVF3mPTrlH54H4Qpn1a3ailr1sA-Hb0JA8cEyPcZY8MbkflCEq9zPcLmzIimWXKxRuryUdAaqNJRMN3LDfUk5BnvEx-IiIpna4tH2NSZEAduoFhvzZbz9BC3WnGP1uYQx5TpaWUDWjvdkosAEzR017rRqQ=w640-h228).

- Quarterly Targets

![Input](https://blogger.googleusercontent.com/img/a/AVvXsEhJF9B74360AO-1fdIgkoIPnZU50hlCVfVw03PXf1HDh80iXcBsNZ3h4NGoCq9kJWwyLYKRAh9gK111L5L1T3DJ5zkdrHJM-D9RZBhBoHEVLExm5bizbkLV8qHb-utqUnnjuDox8nJbDIhrVOGWRlsX6InYCLFb5KXdZe6y-apnz8K2qmnDu3_EzI3DDg). 

Requirements:

For the transactions file:
- Filter the transactions to just look at DSB. These will be transactions that contain DSB in the Transaction Code field
- Rename the values in the Online or In-person field, Online of the 1 values and In-Person for the 2 values
- Change the date to be the quarter
- Sum the transaction values for each quarter and for each Type of Transaction (Online or In-Person) (help)

For the targets file:
- Pivot the quarterly targets so we have a row for each Type of Transaction and each Quarter (help)
- Rename the fields
- Remove the 'Q' from the quarter field and make the data type numeric (help)

Join:
- Join the two datasets together. You may need more than one join clause.
- Remove unnecessary fields
- Calculate the Variance to Target for each row


## Output
````sql
WITH CTE AS (
SELECT 
CASE 
WHEN online_or_in_person = 1 THEN 'Online'
WHEN online_or_in_person = 2 THEN 'In-Person'
END AS online_in_person,
DATE_PART('quarter',DATE(transaction_date,'dd/MM/yyyy HH24:MI:SS')) AS quarter,
SUM(value) AS total_value
FROM pd2023_wk01
WHERE SPLIT_PART(transaction_code,'-',1) = 'DSB'
GROUP BY 1,2
)

SELECT 
online_or_in_person,
REPLACE(t.quarter,'Q','')::int AS quarter,
v.total_value,
target,
v.total_value - target AS variance_from_target
FROM pd2023_wk03_targets AS t
UNPIVOT(target FOR quarter IN (Q1,Q2,Q3,Q4))
INNER JOIN CTE AS v ON t.online_or_in_person = v.online_in_person AND REPLACE(t.quarter,'Q','')::int = v.quarter;
````
| ONLINE_OR_IN_PERSON | QUARTER | TOTAL_VALUE | TARGET | VARIANCE_FROM_TARGET |
|---------------------|---------|-------------|--------|-----------------------|
| Online              | 1       | 74562       | 72500  | 2062                  |
| Online              | 2       | 69325       | 70000  | -675                  |
| Online              | 3       | 59072       | 60000  | -928                  |
| Online              | 4       | 61908       | 60000  | 1908                  |
| In-Person           | 1       | 77576       | 75000  | 2576                  |
| In-Person           | 2       | 70634       | 70000  | 634                   |
| In-Person           | 3       | 74189       | 70000  | 4189                  |
| In-Person           | 4       | 43223       | 60000  | -16777                |
