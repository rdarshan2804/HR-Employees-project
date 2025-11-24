# HR-Employees-project
An HR analytics project where I cleaned messy employee data using Excel and solved 40 SQL questions focused on workforce trends, performance insights, and organizational metrics. This project highlights my data cleaning skills, SQL proficiency, and ability to derive actionable HR insights.

*What This Project Covers*
Cleaning inconsistent and unstructured employee data,
Removing duplicates, fixing formats, and standardizing fields.
*Solving 40 SQL problems across*
Workforce demographics,
Performance metrics,
Attrition insights,
Salary and department analysis,
HR operational trends.

*Tools & Skills Used*
Excel: Data cleaning, formatting, error correction,
SQL: Joins, subqueries, aggregations, window functions,
Data Analysis: Identifying patterns, generating insights.

*Key Highlights*
Improved data accuracy through systematic cleaning,
Delivered meaningful HR insights using structured SQL analysis,
Demonstrates real-world problem-solving for HR analytics roles.



--  *40 sql questions.*

-- Display the employee details along with their department name and job title.
select employee_id,first_name,last_name,job_title,department_name
from jobs as j join employees as e
on j.job_id=e.job_id
join departments as d
on d.department_id=e.department_id


-- List employees who earn more than the average salary of their department.
select department_id,
                employee_id,
                first_name,
                last_name,
                salary,
row_number() over(partition by department_id order by salary desc) as rnk  from employees
where salary>( select avg(salary) from employees)


-- Show all departments that do not have any employees.
select 
    d.department_id,
    d.department_name
from departments d
left join  employees e 
    on d.department_id = e.department_id
where e.employee_id IS NULL;


--  Find employees hired in the same month and year as their manager.
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.hire_date AS employee_hire_date,
    m.employee_id AS manager_id,
    m.first_name AS manager_first_name,
    m.last_name AS manager_last_name,
    m.hire_date AS manager_hire_date
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE MONTH(e.hire_date) = MONTH(m.hire_date)
  AND YEAR(e.hire_date) = YEAR(m.hire_date);

       
--  Display job titles along with the number of employees working in each job.
select job_title, count(e.employee_id) as no_of_employees
from employees as e 
join jobs as j on e.job_id=j.job_id
group by j.job_title



--  List employees who earn more than the minimum salary of their job.
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.salary
FROM employees AS e 
JOIN (SELECT 
		job_id,
		MIN(salary) AS minimum_salary
        FROM employees
        GROUP BY job_id) AS a
ON e.job_id = a.job_id
WHERE e.salary > a.minimum_salary;


-- Display the employee(s) who have the longest tenure in the company (oldest hire_date).
select employee_id,
       first_name,
       last_name,
	   min(hire_date) as hire_date
       from employees 
       group by  employee_id,
                 first_name,
			     last_name
       order by hire_date limit 10
        

--  Display all countries along with the total number of departments in each.
SELECT 
    c.country_id,
    c.country_name,
    COUNT(d.department_id) AS total_departments
FROM countries c
left JOIN locations l 
    ON c.country_id = l.country_id
left JOIN departments d 
    ON l.location_id = d.location_id
GROUP BY 
    c.country_id,
    c.country_name
    order by 2,total_departments 


-- Show all employees whose email address does not contain any digits.
select employee_id,
       first_name,
       last_name,
	   email
       from employees where email not regexp (0-9)


-- Show employees whose salary is between the minimum and maximum salary range of their job.
select employee_id,
	   first_name,
       last_name,
       salary,
       job_min_sal,job_max_sal
       from
(select e.*, min(salary)over(partition by job_id) as job_min_sal,
max(salary) over(partition by job_id) as job_max_sal from employees as e )as x
where salary between job_min_sal and job_max_sal


--  List employees whose first name and last name combined length is greater than 15 characters.
select employee_id,
       first_name,
       last_name,
       length(first_name)+ length(last_name) as name_length
       from employees
       where  ( length(first_name)+ length(last_name))>=15


-- Display employees who have changed jobs at least twice (based on job_history).
SELECT 
    j.employee_id,
    e.first_name,
    e.last_name,
    COUNT(*) AS job_changes
FROM job_history j JOIN employees e 
ON j.employee_id = e.employee_id
GROUP BY 
    j.employee_id,
    e.first_name,
    e.last_name
HAVING COUNT(*) >= 2;


-- List the departments that have more than 5 employees.
select d.department_name,d.department_id, 
       count(e.employee_id) as no_of_employees
       from departments d join employees e 
       on e.department_id=d.department_id
group by 
         d.department_name,
         d.department_id
having count(e.employee_id)>5  

       
--  Display the top 5 highest-paid employees with their department and job.
 select concat(e.first_name," ",e.last_name) as full_name,
       d.department_name,
       j.job_title,
       e.salary
from departments as d join employees as e
on d.department_id=e.department_id
join jobs as j on e.job_id=j.job_id
order by e.salary desc
limit 5


--  Show employees who have never worked in any other department (not present in job_history).
 select employee_id,
        first_name,
        last_name from employees
where  employee_id not in 
(select employee_id  from job_history)


--  List all job titles that do not have any employees currently assigned.
select j.job_title,
       count(e.employee_id) as total_employees from jobs as j
       left join employees as e
	   on e.job_id=j.job_id
	   group by j.job_title 
       having count(e.employee_id)<1


--  Display employees who report to a manager from a different department.
SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.department_id AS employee_department,
    m.employee_id AS manager_id,
    m.first_name AS manager_first_name,
    m.last_name AS manager_last_name,
    m.department_id AS manager_department
FROM employees e
JOIN employees m
ON e.manager_id = m.employee_id
WHERE e.department_id <> m.department_id;


-- Find city-wise employee counts using locations → departments → employees.
select city,
        count(e.employee_id) as employee_count
        from locations as l
         left join departments as d
        on l.location_id=d.location_id
        left join employees e
        on e.department_id=d.department_id
        group by city
        order by employee_count
        
	
--  Display employees hired after the average hire date of all employees
 select employee_id,
		first_name,
        last_name
        from employees where hire_date>
	   (select avg(hire_date) from employees)


--  Display the employee(s) with the highest salary in each department.
select e.department_id,
       d.department_name,
       max(salary) as maximum_salary from employees e
       join departments d on e.department_id=d.department_id
       group by e.department_id,d.department_name



--  Show departments where the average salary exceeds 10,000.
select e.department_id,
       d.department_name,
       e.salary
       from employees e
       join departments d on e.department_id=d.department_id
       join 
       (select department_id,avg(salary) avg_salary from employees
       group by department_id)x
       on e.department_id=x.department_id
       where x.avg_salary >= 10000


-- List pairs of employees working in the same department (self join).
select e.employee_id,e.first_name,e.last_name,
       m.employee_id,m.first_name,m.last_name
       from employees e  join employees m 
       on e.department_id=m.department_id
       where e.department_id =m.department_id
	   and e.employee_id < m.employee_id;  
       

--  Display employees who have the same job as their manager.
SELECT 
    e.employee_id AS employee_id,
    e.first_name  AS employee_name,
    e.job_id      AS employee_job,
    m.employee_id AS manager_id,
    m.first_name  AS manager_name,
    m.job_id      AS manager_job
FROM employees e
JOIN employees m 
ON e.manager_id = m.employee_id      
WHERE e.job_id = m.job_id;                   


--  Find how many employees were hired each year.
select year(hire_date) as hired_year,
      count(*) as no_of_employees
      from employees 
      group by year(hire_date)


--  Show job history details of employees who currently work in the same department as they previously worked.
	SELECT 
    e.employee_id,
    e.first_name,
    e.last_name,
    e.department_id AS current_department,
    j.department_id AS previous_department,
    j.job_id AS previous_job_id,
    j.start_date,
    j.end_date
FROM employees e
JOIN job_history j
ON e.employee_id = j.employee_id
WHERE e.department_id = j.department_id;


--  Display regions along with total employees working under them.
select r.region_name,
	  count(e.employee_id) as total_employees 
      from regions r left join countries as c
      on r.region_id=c.region_id
      left join locations as  l on c.country_id=l.country_id
      left join departments as d on d.location_id=l.location_id
      left join employees as e on e.department_id=d.department_id
      group by region_name
      
      
--  List employees whose salary is below their department's minimum salary (use MIN OVER window).
select employee_id,
	first_name,last_name,
	department_id,
    dept_min_salary,
	job_id,
	salary
	from
(select e.*,
min(salary) over(partition by department_id) as dept_min_salary from employees e)x
where salary < dept_min_salary
     

-- Display the longest-serving employee based on job history durations.
select e.employee_id,
       e.first_name,
       e.last_name,
       timestampdiff (year, j.start_date,end_date) as years_worked
       from employees e   join job_history j
       on e.department_id = j.department_id
       order by years_worked desc


--  Show departments located in countries where the region name contains ‘Europe’.
select d.department_name,
       d.department_id,
       c.country_name,
       r.region_name
       from departments d left join locations l
	   on d.location_id = l.location_id
       left join countries c on c.country_id = l.country_id
       left join regions r on r.region_id = c.region_id
       where r.region_name = "Europe"


--  Find employees who moved to a different job but stayed in the same department.
select e.employee_id, 
       e.first_name,e.last_name,
       e.job_id as current_job,
       e.department_id,
       j.job_id as previous_job
       from employees e join job_history j 
       on e.department_id=j.department_id
       where e.department_id=j.department_id
       and 
       e.job_id <> j.job_id
       

-- Display the number of employees under each manager.
select  m.manager_id,
        m.first_name as manager_first_name,
        m. last_name as manager_last_name,
       count(e.employee_id) as no_of_employees
       from employees as m
       left join employees as e
       on e.manager_id=m.employee_id
       group by  m.manager_id,
                 manager_first_name,
                 manager_last_name
	   order by no_of_employees  desc   


--  List salaries that appear more than once in the employees table.
select salary, count(*) as salary_count from employees 
group by salary 
having count(*)>1
order by salary_count


-- Display job-wise average salary and compare it with overall average salary.
SELECT j.job_id,
       j.job_title,
       ROUND(AVG(e.salary), 2) AS job_wise_avg_salary,
       ROUND((SELECT AVG(salary) FROM employees), 2) AS overall_average_salary
FROM employees e
JOIN jobs j
    ON e.job_id = j.job_id
   GROUP BY j.job_id, j.job_title;


--  For each department, find the employee who was hired most recently.
SELECT e.department_id,
       e.employee_id,
       e.first_name,
       e.last_name,
       e.hire_date
FROM employees e
JOIN (
    SELECT department_id, MAX(hire_date) AS max_hire_date
    FROM employees
    GROUP BY department_id)t
	ON e.department_id = t.department_id
    AND e.hire_date = t.max_hire_date;
  

--  Display employees whose job has a max_salary more than double their current salary.
select e.employee_id,
       e.first_name,e.last_name,
       j.job_title,
       e.salary,j.max_salary
       from employees e join jobs j
       on e.job_id=j.job_id
       where 2*e.salary<=j.max_salary


--  List employees who have never had a salary change (no job_history entries).
select employee_id,first_name,last_name,
       salary
       from employees 
       where employee_id  not in
		(select employee_id from job_history)
	   

--  Display employees who work in locations where no other departments exist except theirs.
SELECT e.employee_id,e.first_name,
       e.last_name,e.department_id, d.location_id
       FROM employees e
       jOIN departments d
    ON e.department_id = d.department_id
WHERE d.location_id IN (
    SELECT location_id
    FROM departments
    GROUP BY location_id
    HAVING COUNT(department_id) = 1)

       
--  Show department_id and the number of employees who have the same last name.
select department_id, last_name,
count(last_name) as no_of_emp
from employees 
group by department_id,
         last_name
having count(*)>1


--  Display employees who have worked in more than one region (via job_history → departments → locations → countries → regions).
 select e.employee_id,first_name,last_name,count( distinct c.region_id) as job_count
 from employees e join job_history jh
 on e.employee_id = jh.employee_id
 join departments d on jh.department_id = d.department_id
 join locations l on d.location_id = l.location_id
 join countries c on l.country_id = c.country_id
 join regions r on c.region_id = r.region_id
group by employee_id,first_name,last_name
having count( distinct c.region_id)>1


--  Find jobs where the salary range (max_salary − min_salary) is greater than 5000.
select job_id,job_title,
          min_salary,max_salary,
          ( max_salary-min_salary) as diff_amount
from jobs where ( max_salary-min_salary) > 5000


--  List employees hired on weekends (Saturday or Sunday).
select employee_id,
       first_name,
       last_name,
       hire_date
       from employees 
       where dayofweek(hire_date) in (1,7)


-- Display all employees whose manager reports to a manager earning less salary (nested manager hierarchy).
SELECT e.employee_id,
       e.first_name,
       e.last_name,
       e.manager_id
FROM employees e
JOIN employees m ON e.manager_id = m.employee_id        
JOIN employees mm ON m.manager_id = mm.employee_id       
WHERE mm.salary < m.salary;
