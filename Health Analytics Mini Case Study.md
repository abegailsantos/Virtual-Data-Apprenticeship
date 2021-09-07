# Health Analytics Mini Case Study 

### 1️⃣ How many unique users exist in the logs dataset?


```sql
SELECT
  COUNT(DISTINCT id) AS distinct_id_count
FROM health.user_logs; 
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q1.png">

### For Q2-Q8, I will need to create a temporary table. 

```sql
DROP TABLE IF EXISTS user_measure_count;

CREATE TEMP TABLE user_measure_count AS (
SELECT
  id,
  COUNT(*) AS measure_count,
  COUNT(DISTINCT measure) AS unique_measures
FROM health.user_logs
GROUP BY 1;
)
```

### 2️⃣ How many total measurements do we have per user on average?

```sql
SELECT 
  ROUND(AVG(measure_count),
  2) AS avg_measurement_count
FROM user_measure_count;
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q2.png">


### 3️⃣ What about the median number of measurements per user?
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q3.png">

### 4️⃣ How many users have 3 or more measurements?
```sql
SELECT
  COUNT(id) AS user_count
FROM user_measure_count
WHERE measure_count >= 3;
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q4.png">

### 5️⃣ How many users have 1,000 or more measurements?
```sql
SELECT
  COUNT(id) AS user_count
FROM user_measure_count
WHERE measure_count >= 1000;
```
To confirm:
```sql
SELECT * FROM user_measure_count ORDER BY measure_count DESC LIMIT 10;
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q5.png">

### 6️⃣ Looking at the logs data - what is the number and percentage of the active user base who have logged blood glucose measurements?

```sql
WITH distinct_id_table AS (
  SELECT 
    measure,
    COUNT(DISTINCT id) AS distinct_id_count,
    (SELECT COUNT(DISTINCT id) FROM health.user_logs) AS total_distinct_id_count -- Use sum when grouped by measure
  FROM health.user_logs
  GROUP BY 1)
SELECT 
  measure,
  distinct_id_count,
  total_distinct_id_count,
  ROUND(
  100 * distinct_id_count / total_distinct_id_count,
  2) AS percentage
FROM distinct_id_table
WHERE measure = 'blood_glucose';
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q6.png">

### 7️⃣ What is the number and percentage of the active user base who have at least 2 types of measurements?
```sql
SELECT
  COUNT(*) AS distinct_id_count,
  (SELECT COUNT(DISTINCT id) FROM health.user_logs) AS total_distinct_id_count,
  ROUND(
  100 * COUNT(*) / (SELECT COUNT(DISTINCT id) FROM health.user_logs),
  2) AS percentage
FROM user_measure_count
WHERE measure_count >= 2;
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q7.png">

### 8️⃣ What is the number and percentage of the active user base who have all 3 measures - blood glucose, weight and blood pressure?

```sql
SELECT
  COUNT(*) AS distinct_id_count,
  (SELECT COUNT(DISTINCT id) FROM health.user_logs) AS total_distinct_id_count,
  ROUND(
  100 * COUNT(*) / (SELECT COUNT(DISTINCT id) FROM health.user_logs),
  2) AS percentage
FROM user_measure_count
WHERE unique_measures = 3;
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q8.png">

### 9️⃣ For users that have blood pressure measurements, what is the median systolic/diastolic blood pressure values?

```sql
SELECT 
  measure,
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS NUMERIC), 
    2) AS systolic_median_val,
  ROUND(
    CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS NUMERIC), 
    2) AS diastolic_median_val
FROM health.user_logs
WHERE measure = 'blood_pressure'
GROUP BY measure
```
**Answer:**

<img height="65" alt="image" src="https://github.com/abegailsantos/Virtual-Data-Apprenticeship/blob/a17e3c2540a00d551ddd69ff23fd22c2584292aa/Images%20of%20SQL%20Snippets/Health%20Analytics%20Mini%20Case%20Study/q9.png">
