# Task - Employee Records

The major corporation BusinessCorp&#8482; wants to do some analysis of varioius metrics around its employees and have contracted the job out to you. They have provided you with two SQL files, you will need to write the queries which will extract the data they are looking for.

## Setup

- Create a database to run the files against
- Use the `psql -d database -f file` command to run the SQL files
- Before starting to write the queries take some time to look at the data and figure out the relationships between the tables - maybe even draw an ERD to help.

## Tasks

### CTEs

1) Calculate the average salary of all employees

```sql
SELECT 
	ROUND(AVG(salary), 0) AS avg_salary
FROM employees
```

2) Calculate the average salary of the employees in each team (hint: you'll need to `JOIN` and `GROUP BY` here)

```sql
SELECT 
	departments.name,
	ROUND(AVG(salary), 0) AS avg_department_salary
FROM employees
JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
```

3) Using a CTE find the ratio of each employees salary to their team average, eg. an employee earning £55000 in a team where the average is £50000 has a ratio of 1.1

```sql
WITH department_avg (id, department_name, avg_department_salary) AS (
    SELECT 
	    departments.id,
	    departments.name,
	    ROUND(AVG(salary), 0) AS avg_department_salary
    FROM employees
    JOIN departments
    ON employees.department_id = departments.id
    GROUP BY departments.id
)
SELECT 
	employees.first_name,
	employees.last_name,
	ROUND(employees.salary / department_avg.avg_department_salary, 1) AS ratio_salary_to_dept_avg
FROM employees
JOIN department_avg
ON department_avg.id = employees.department_id
```

4) Find the employee with the highest ratio in Argentina

```sql
WITH department_avg (id, department_name, avg_department_salary) AS (
SELECT 
	departments.id,
	departments.name,
	ROUND(AVG(salary), 0) AS avg_department_salary
FROM employees
JOIN departments
ON employees.department_id = departments.id
GROUP BY departments.id
)
SELECT 
	employees.first_name,
	employees.last_name,
	ROUND(employees.salary /department_avg.avg_department_salary, 1) AS ratio_salary_to_dept_avg
FROM employees
JOIN department_avg
ON department_avg.id = employees.department_id
WHERE country = 'Argentina'
ORDER BY ratio_salary_to_dept_avg DESC
```

5) **Extension:** Add a second CTE calculating the average salary for each country and add a column showing the difference between each employee's salary and their country average

```sql
WITH department_avg (id, department_name, avg_department_salary) AS (
	SELECT 
		departments.id,
		departments.name,
		ROUND(AVG(salary), 0) AS avg_department_salary
	FROM employees
	JOIN departments
	ON employees.department_id = departments.id
	GROUP BY departments.id
), country_avg (country, avg_country_salary) AS (
	SELECT
		country,
		ROUND(AVG(salary), 0) AS avg_country_salary
	FROM employees
	GROUP BY country
)
SELECT 
	employees.first_name,
	employees.last_name,
	ROUND(employees.salary / department_avg.avg_department_salary, 1) AS ratio_salary_to_dept_avg,
	ROUND(employees.salary / country_avg.avg_country_salary, 1) AS ratio_salary_to_country_avg
FROM employees
JOIN department_avg
ON department_avg.id = employees.department_id
JOIN country_avg
ON country_avg.country = employees.country
```

---

### Window Functions

1) Find the running total of salary costs as the business has grown and hired more people

```sql
SELECT 
	start_date,
	salary,
	SUM(salary) OVER (ORDER BY start_date) AS running_salary_costs
FROM employees
```

2) Determine if any employees started on the same day (hint: some sort of ranking may be useful here)

```sql
SELECT 
	id,
	first_name,
	last_name,
	RANK() OVER (ORDER BY start_date) AS order_started
FROM employees
-- check to see if any employees have the same rank
```

3) Find how many employees there are from each country

```sql
SELECT
	DISTINCT country,
	COUNT(*) OVER (PARTITION BY country)
FROM employees
```

4) Show how the average salary cost for each department has changed as the number of employees has increased

```sql
SELECT
	departments.name AS department,
	start_date AS date,
	RANK() OVER (PARTITION BY departments.id ORDER BY start_date) AS number_of_employees,
	AVG(salary) OVER (ORDER BY start_date) AS average_salary_costs
FROM employees
JOIN departments
ON employees.department_id = departments.id
```

5) **Extension:** Research the `EXTRACT` function and use it in conjunction with `PARTITION` and `COUNT` to show how many employees started working for BusinessCorp&#8482; each year. If you're feeling adventurous you could further partition by month...

```sql
SELECT
	DISTINCT EXTRACT(year FROM start_date) AS year,
	COUNT(*) OVER (PARTITION BY EXTRACT(year FROM start_date)) AS number_of_employees_starting
FROM employees
JOIN departments
ON employees.department_id = departments.id
ORDER BY year
```

---

### Combining the two

1) Find the maximum and minimum salaries

```sql
SELECT
	MIN(salary),
	MAX(salary)
FROM employees
```

2) Find the difference between the maximum and minimum salaries and each employee's own salary

```sql
WITH salary_min_max (id, min_salary, max_salary) AS (
	SELECT
		id,
		MIN(salary) OVER(),
		MAX(salary) OVER()
	FROM employees
)
SELECT 
	first_name,
	last_name,
	salary,
	salary - salary_min_max.min_salary AS difference_from_min,
	salary_min_max.max_salary - salary AS difference_from_max
FROM employees
JOIN salary_min_max
ON employees.id = salary_min_max.id	
```

3) Order the employees by start date. Research how to calculate the **median** salary value and the **standard deviation** in salary values and show how these change as more employees join the company

```sql
SELECT 
	first_name,
	last_name,
	start_date,
	(SELECT PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY salary) FROM employees) AS median_salary,
	STDDEV(salary) OVER (ORDER BY start_date)
FROM employees
ORDER BY start_date
-- can't work out how to make median rolling
```

4) Limit this query to only Research & Development team members and show a rolling value for only the 5 most recent employees.

```sql
<!--Copy solution here-->
```

