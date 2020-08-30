# Pewlett-Hackard-Analysis

## Overview of Project

Our primary customer for this project is Pewlett Hackard, a large company hosting several thousand employees, and the leadership of the company is looking forward towards the transition of it’s ‘baby boomer’ employee population in two ways: it’s building retirement packages for those getting ready to retire and it’s also looking at how to replace the positions they leave behind through mechanisms such as mentorship programs.  

In support of the company’s efforts, Bobby, an HR analyst at PH, has asked us to provide support for two assignments: determine the number of retiring employees per title, and identify employees who are eligible to participate in a mentorship program.  To execute these assignments, we must analyze several data bases with key information about employees and positions at the company.  These databases are relational databases in CSV format, and he has asked us to use Structured Query Language (SQL) to analyze them.  The databases include:
- [Employees data](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/employees.csv): employee number, first name, last name, birth date, gender, hire date 
- [Department data](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/departments.csv): department number, department name
- [Salaries for employees](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/salaries.csv): employee number, salary, start date for that salary, end date for that salary
- [Titles of employees](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/titles.csv): employee number, title, start date of that title, end date of that title
- [Department employees](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/dept_emp.csv): employee number, department they belong to, the start date of their time there, the end date of there time there
- [Department management](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/dept_manager.csv): department number, the employee number of the manager, the start date as the manager, the end date as the manager

## Results

To facilitate answering Bobby’s two key questions, we utilized SQL to build out four additional tables: titles of retiring employees, unique titles of retiring employees, the number of employees retiring by title, and employees who are eligible for a mentorship program.  The first three support Bobby’s first question and the last his second question.  Below outlines the steps we executed to build out each table in support of each question.  


### What Titles Has the Most Number of Employees Who Will be Retiring?

To answer this question, we first need to identify the titles of employees who are going to be retiring, then identify what their current role is, and then aggregate the information into a table to provide how many of each title is retiring.  

*Titles of Retiring Employees*
In order to identify the roles that retiring employees are filling, we need to create a new table that pulls employee number, first name, and last name from the Employee table, and then pull title, the date that the person assumed that title, and the date that the person relinquished that title.  We conducted an INNER JOIN on these two original tables on the employee number.  To ensure that we are only displaying employees who are retirement eligible, we limited the returns to only those employees with a birth date between 1 January, 1952 and 31 December, 1955, using a WHERE conditional statement.  Finally, we ordered by employee number with the default of ascending. 

```
-- Create table of all titles of retirement-eligible employees
SELECT e.emp_no,
    e.first_name, 
    e.last_name,
    t.title,
    t.from_date,
    t.to_date
INTO retirement_titles
FROM employees AS e
    INNER JOIN titles as t
    ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
ORDER BY e.emp_no;
```

The resulting table, the first 19 records of which are pictured below, provides a list of all the employees who are retirement eligible and their associated titles.  Of note, you can see that some employees have had multiple titles throughout their time at PH.  Employee 10004, Christian Koblick, for example was an Engineer from 1986 to 1995, and then was promoted to Senior Engineer in 1995 and is currently in that position.  While this is helpful, to truly understand what gaps in titles PH will have, we need to remove these duplicates and associate them with the most recent position of retirement-eligible employees. 

>**Retirement Titles**

![Retirement Titles](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/retirement_titles.png)


*Unique Titles of Retiring Employees*
In order to remove duplicate titles for each retirement-eligible employee and only show their current title, we took the previous table and executed SELECT DISTINCT ON to provide a unique/singular response for each employee number.  This means that each employee number can only have one response.  To ensure that the response is their current title, we utilized the ORDER BY statement with employee number ascending and then the end date for that title descending.  This means, that for each employee number the most recent ‘to date’ will be first (and thereby selected) and the older title will be second (and thereby removed).  We put this into a new table, removing the from_date and to_date column and only showing the employee number, first name, last name, and title.  

```
-- Utilize previous table to remove duplicate rows and display retirement-eligible employees current title
SELECT DISTINCT ON (emp_no) emp_no,
first_name,
last_name,
title
INTO retirement_unique_titles
FROM retirement_titles
ORDER BY emp_no, to_date DESC;
```

The resulting table, the first 18 records of which are show below, is now the ‘cleaned’ version of the first table.  It shows only the current title for each retirement-eligible employee and therefore provides Bobby and his leadership an accurate view of which titles will be empty when each employee retires.  You can verify the accuracy of our code by again looking at Employee 10004, Christian Koblick.  In the first table he had to titles, but the most recent and current was Senior Engineer.  In our second table, he only has one title and it is now Senior Engineer.  

>**Retirement Unique Titles**

![Retirement Unique Titles](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/retirement_unique_titles.png)


*Retiring Titles*
While a list of each retirement-eligible employee and their current title is helpful at a micro level, to look at the impact at a macro level it will be helpful to aggregate the information in the unique titles table and show how many employees of each title is retiring.  This will indicate to executives where the impact of retirement will be felt most across their business.  To accomplish this, we utilized a combination of the COUNT and the GROUP BY statements.  By doing this, we were able to group each title together (e.g. Senior Engineer title for employee number 10001 and for employee number 10004) and count how many retirement-eligible employees have that same title.  Then we ordered the responses in descending order based upon the number of employees.

```
-- Provide count of how many employees are in each title 
SELECT title, COUNT (title)
INTO retiring_titles
FROM retirement_unique_titles
GROUP BY title
ORDER BY count DESC;
```

As you can see from the table below, there is a significant population of retirement-eligible employees that are Senior Engineers and Senior Staff.  Closely following those groups are employees that are Engineers and Staff.  Considering these positions are in the same group but of different seniority, it is fair to conclude that there will be a significant impact on engineering and staff roles, with 2x the amount of impact in the senior skills in those two groups.  

>**Retiring Titles**

![Retiring Titles](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/retiring_titles.png)

*Analysis: Retiring Titles*
- There will be a significant impact to the engineering and staff roles with 93% of all retirement eligible employees in those role categories
- The senior engineer and senior staff roles will also be significantly	impacted with 64% of all retirement eligible employees in those two roles combined
- There are not many managers within the retirement-eligible employees with only 2 listed out of the total 9 departments available, and they make up an extremely small percentage of the overall population of retirement eligible employees

Of note, additional analysis on how many Senior Engineers and Senior Staff employees are not retirement eligible would provide context into the true impact of the retirees on the entire business.  More specifically, we could answer how many of the total Senior Engineers and Senior Staff are retirement eligible.  


### Who is eligible for mentorship in each of these titles?

Answering this question is a bit more straightforward that the previous question, and uses both the SELECT DISTINCT ON statement as well as two INNER JOIN statements.  To start, we want to create a new table that includes the employee number, first name, last name, and date of birth from the Employees table.  The date of birth is needed for filtering responses to ONLY include employees who were born between 1 January, 1965 and 31 December, 1965.   We also need to pull the dates from the Department Employee table that the employee joined a particular department and the date that they left that department.  The date that they left that department is key to ensuring that we ONLY get responses that include personnel who are still in that department (e.g. their to_date = ‘9999-01-01’).  Finally, we need to pull the title from the Titles table.  To merge the data from these tables together, we use INNER JOIN because we want to find the common data between each of them.  We use the SELECT DISTINCT ON statement to ensure that we only get one response per employee number.  Finally, using the WHERE clause, we ensure we are filtering for candidates who were born in 1965 AND who are currently employed.   

```
Part 2:  Who is eligible for mentorship 
SELECT DISTINCT ON (emp_no) e.emp_no,
	e.first_name, 
	e.last_name,
	e.birth_date,
	de.from_date,
	de.to_date,
	t.title
INTO mentorship_eligibility
FROM employees AS e
	INNER JOIN dept_emp as de
	ON (e.emp_no = de.emp_no)
	INNER JOIN titles as t
	ON (e.emp_no = t.emp_no)
WHERE (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31') AND (de.to_date = '9999-01-01')
ORDER BY e.emp_no;
```

In looking at the resulting table, you can see the results are ordered by employee number in ascending form and with each employee only having one title and their end date in their department is ‘9999-01-01’ indicating that they are still in their role.  Of note, there are only 1,548 mentorship eligible candidates that meet the criterial outlined by Bobby and the PH leadership team.  This is a small number when compared to the 90K retirement-eligible employees.
 
>**Mentorship Eligibility**

![Mentorship Eligibility](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/mentorship_eligibility.png)

*Analysis: Mentorship*
- The number of employees eligible for mentorship is significantly smaller than that needed to replace the retirement-eligible employee community (1,548 to 90,389).  PH could expand the age range for mentorship eligible candidates to increase the candidate pool.

Of note, additional analysis could be done on the exact count per title similar to the table we created in question 1.  In doing so, we can identify how well the percentage break down of mentorship eligible employees matches the retirement-eligible employee titles.  

## Summary

*What is the impact of the ‘silver tsunami’ and how many roles need to be filled?*
The ‘silver tsunami’ will have a significant impact to the engineering and staff roles at PH, with a total of 84,133 retirement-eligible employees currently in these two roles.  More specifically, the Senior Engineers and Senior Staff positions will be hardest hit with 29,414 and 28,254 retirement-eligible employees currently holding those titles respectively.  The mid-level role of Engineer and Staff titles has 14,222 and 12,243 retirement-eligible employees respectively.  Together, these four titles make up 93% of all current retirement-eligible employees at PH.  Additional roles include Technique Leader (4,502 employees with this title), Assistant Engineer (1,761 employees with this title), and manager (2 employees with this title).  

*Are there enough qualified, retirement-ready employees to mentor the next generation of PH employees?*
Based upon the criterial for mentorship program participation, there are a total of 1,548 PH employees who are eligible to participate in this program.  This is significantly less than the overall population of retirement-eligible employees which is 90,398.  Assuming each retirement-eligible employee is qualified to act as a mentor, the ratio of participants vs. mentors is 2:100.  This means that the impact of removing retirement-eligible employees to support the mentorship program will be minimal but it will also limit the overall impact it has on the remaining workforce.  By expanding the eligibility age range to personnel born in years other than 1965, PH could increase it’s participants and still have a limited impact on productivity.    


*What additional analysis could we conduct to assess the impact of the ‘silver tsunami’?*
To better understand the overall impact of the ‘silver tsunami’, we could conduct analysis to identify how much of the total workforce and how much of each title is retirement eligible.  Simply by removing the WHERE statement in the first table of the first question, you can identify the total employees of each title type and then identify how many are retirement eligible.  In doing this, for example, you could see that there are 97,743 total Senior Engineers at PH, which means that 30% of the entire Senior Engineers at PH are retirement eligible.  

```
SELECT e.emp_no,
	e.first_name, 
	e.last_name,
	t.title,
	t.from_date,
	t.to_date
INTO all_titles
FROM employees AS e
	INNER JOIN titles as t
	ON (e.emp_no = t.emp_no)
ORDER BY e.emp_no;

-- Use Dictinct with Orderby to remove duplicate rows
SELECT DISTINCT ON (emp_no) emp_no,
first_name,
last_name,
title
INTO all_unique_titles
FROM all_titles
ORDER BY emp_no, to_date DESC;

SELECT title, COUNT (title)
INTO compiled_all_titles
FROM all_unique_titles
GROUP BY title
ORDER BY count DESC;
```

>**Compiled Titles at PH**

![Compiled Titles at PH](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/compiled_titles.png)

Additionally, you could utilize the same breakdown of the mentorship eligible employees to see if the break down in percentages matches that of the actual retirement-eligible employees.  For example, do Senior Engineers make up 33% of the total mentorship eligible candidates like they do for retirement-eligible employees?   

```
SELECT title, COUNT (title)
FROM mentorship_eligibility
GROUP BY title
ORDER BY count DESC;
```

>**Title Breakdown of Mentorship Candidates**

![Title Breakdown of Mentorship Candidates](https://github.com/MaureenFromuth/Pewlett-Hackard-Analysis/blob/master/mentorship_eligible_breakdown.png)

As you can see, the break down of mentorship eligible employees puts a greater emphasis on Senior Staff roles and engineers over Senior Engineers.  While this may be deliberate, it does not adequately match where the retirement eligible employees will leave an impact.  Discussing this further and if they should make mentorship eligibility based on role rather than age is a recommended next step.
