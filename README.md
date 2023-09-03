# HR Data Exploration in SQL

This is a case study hosted by DataInMotion, designed to utilize intermediate and advanced SQL skills.

Case Study information found here:  https://d-i-motion.com/courses/sql-case-studies/

### Questions to answer

#### 1. Find the longest ongoing project for each department.
   
First I'm inserted extra data into the `projects` table for a better test of the longest project for each department.
```
INSERT INTO projects (name, start_date, end_date, department_id)
VALUES ('HR Project 2', '2023-01-12', '2023-07-30', 1),
       ('IT Project 2', '2023-02-14', '2023-08-30', 2),
       ('Sales Project 2', '2023-03-01', '2023-09-30', 3);
```
Next I created a temp table to calculate and store the number of days between the start and end date.
```
DROP TABLE IF EXISTS #project_days
CREATE TABLE #project_days (
	DepartmentID int,
	Department varchar(100),
	ProjectName varchar(100),
	Days int
)
INSERT INTO #project_days
SELECT p.department_id, d.name, p.name, DATEDIFF(DAY,p.start_date,p.end_date) Days
FROM projects p
JOIN departments d
ON p.department_id = d.id
```
I created another temp table to calculate the maximum difference in start and end date for each department.
```
DROP TABLE IF EXISTS #max_project_days
CREATE TABLE #max_project_days (
	Department varchar(100),
	MaxProjectDays int
)
INSERT INTO #max_project_days
SELECT Department, MAX(Days) AS MaxProjectDays
FROM #project_days
GROUP BY Department
```
Finally, I used a `JOIN` to include the name of the longest projects in a results table.
```
SELECT max.Department, p_days.ProjectName, max.MaxProjectDays AS LongestProjectDays
FROM #max_project_days AS max
LEFT JOIN #project_days as p_days
ON max.Department = p_days.Department
AND max.MaxProjectDays = p_days.Days
```
The results table with the longest project for each department is below:
| Department	|ProjectName		| LongestProjectDays |
|---------------|-----------------------|--------------------|
| HR		| HR Project 2		| 199
| IT		| IT Project 2		| 197
| Sales		| Sales Project 2	| 213

#### 2. Find all employees who are not managers.

For this question, I assumed that all managers would have the word 'manager' somewhere in their job title.
```
SELECT name, job_title, department_id
FROM employees
WHERE LOWER(job_title) NOT LIKE '%manager%'
```
| name          | job_title       | department_id |
|---------------|-----------------|---------------|
| Bob Miller    | HR Associate    | 1             |
| Charlie Brown | IT Associate    | 2             |
| Dave Davis    | Sales Associate | 3             |

#### 3. Find all employees who have been hired after the start of a project in their department.

For this question, I did a join on `employees` and `projects` to compare employee hire dates to project start dates.
```
SELECT em.name AS EmployeesHiredAfterProjectStart
FROM employees em
FULL JOIN projects pr
ON em.department_id = pr.department_id
WHERE DATEDIFF(DAY,pr.start_date,em.hire_date) > 0
GROUP BY em.name
```
| EmployeesHiredAfterProjectStart |
|---------------------------------|
| Dave Davis                      |

#### 4. Rank employees within each department based on their hire date (earliest hire gets the highest rank).

#### 5. Find the duration between the hire date of each employee and the hire date of the next employee hired in the same department.
