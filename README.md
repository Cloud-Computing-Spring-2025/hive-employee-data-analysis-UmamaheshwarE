# HadoopHiveHue
Hadoop , Hive, Hue setup pseudo distributed  environment  using docker compose
#Hive Queries for Employee and Department Analysis

## Introduction
This document provides a set of Hive queries for analyzing employee and department data using partitioned tables and ORC storage format. It also includes commands to execute these queries and store the output in HDFS directories.

# Docker start
```bash
docker compose up -d
```
# container up
```bash
docker exec -it resourcemanager /bin/bash
```
# output to local
```bash
hdfs dfs -get /user/hue/output ./output_local
exit 
```
# docker command
```bash
dockr cp resourcemanager:/output_local ./output
```


## Table Creation and Data Loading

### Create Partitioned Employees Table
```sql
CREATE TABLE employees (
    emp_id INT,
    name STRING,
    age INT,
    job_role STRING,
    salary FLOAT,
    project STRING,
    join_date STRING
) PARTITIONED BY (department STRING)
STORED AS ORC;
```

### Enable Dynamic Partitioning
```sql
SET hive.exec.dynamic.partition.mode=nonstrict;
```

### Load Data into Employees Table
```sql
INSERT OVERWRITE TABLE employees PARTITION (department)
SELECT emp_id, name, age, job_role, salary, project, join_date, department FROM hue__tmp_employees;
```

### Create Departments Table
```sql
CREATE TABLE departments (
    dept_id INT,
    department_name STRING,
    location STRING
) STORED AS ORC;
```

### Load Data into Departments Table
```sql
INSERT OVERWRITE TABLE departments
SELECT dept_id, department_name, location FROM hue__tmp_departments;
```

## Query Execution and Output Storage

### Retrieve Employees Who Joined After 2015
```sql
SELECT * FROM employees WHERE YEAR(join_date) > 2015;
```
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/emp_after_2015'
SELECT * FROM employees WHERE YEAR(join_date) > 2015;
```

### Find Average Salary by Department
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/average_sal'
SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department;
```

### Identify Employees Working on Project 'Alpha'
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/project_alpha'
SELECT * FROM employees WHERE project = 'Alpha';
```

### Count Employees in Each Job Role
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/cnt_emp_in_job_role'
SELECT job_role, COUNT(*) AS employee_count FROM employees GROUP BY job_role;
```

### Retrieve Employees Earning Above Average Salary in Their Department
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/emp_sal_gt_avg_sal'
SELECT e.* FROM employees e
JOIN (SELECT department, AVG(salary) AS avg_salary FROM employees GROUP BY department) d
ON e.department = d.department
WHERE e.salary > d.avg_salary;
```

### Find the Department with the Highest Number of Employees
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/dpt_high_emp'
SELECT department, COUNT(*) AS employee_count FROM employees
GROUP BY department ORDER BY employee_count DESC LIMIT 1;
```

### Exclude Employees with Null Values
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/exclude_null'
SELECT * FROM employees WHERE
    emp_id IS NOT NULL AND name IS NOT NULL AND age IS NOT NULL AND
    job_role IS NOT NULL AND salary IS NOT NULL AND project IS NOT NULL AND
    join_date IS NOT NULL AND department IS NOT NULL;
```

### Join Employees and Departments to Get Locations
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/join_emp_dept'
SELECT e.*, d.location FROM employees e
JOIN departments d ON e.department = d.department_name;
```

### Rank Employees by Salary Within Each Department
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/rank_emp'
SELECT emp_id, name, salary, department,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

### Find the Top 3 Highest-Paid Employees in Each Department
```sql
INSERT OVERWRITE DIRECTORY '/user/hue/output/3_high_paid'
SELECT * FROM (
    SELECT emp_id, name, salary, department,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
    FROM employees
) ranked_employees WHERE salary_rank <= 3;
```
  

