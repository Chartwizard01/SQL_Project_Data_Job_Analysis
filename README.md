## [README.md](http://README.md) contents:

# Introduction
Welcome to my SQL Portfolio Project! This initiative dives deep into the job market, particularly focusing on roles in data analytics. Through this exploration, I intend to uncover insights into the top rewarding opportunities, identify in-demand and high-value skills, and analyze the overlap between high demand and robust salaries within the data analytics domain.
Feel free to explore my SQL queries here: [project_sql](/project_sql/).
# Background
Propelled by the intention of better understanding the data analyst job market, this project was made to distinguish high-paying and in-demand skills, simplifying the process for others to find the best opportunities.
Data sourse: [data science job postings from 2023](https://www.lukebarousse.com/sql).

It details insights into job titles, salaries, locations, and essential skills.

# The questions I wanted to answer through my SQL queries were:
What are the top-paying data analyst jobs?
What skills are required for these top-paying jobs?
What skills are most in demand for data analysts?
Which skills are associated with higher salaries?
What are the most optimal skills to learn?

# Tools I Used

To properly explore the data analyst job market, I leveraged the potentialities of several essential tools:


- **SQL:** The backbone of my analysis, allowing me to query the database and unearth critical insights.
- **PostgreSQL:** The chosen database management system, ideal for handling the job posting data.
- **Visual Studio Code:** My go-to for database management and executing SQL queries.
- **Git & GitHub:** Essential for version control and sharing my SQL scripts and analysis, ensuring collaboration and project tracking.
# The Analysis

Every query in this project was designed to examine specific facets of the data analyst job market. Here's the approach I took for each question:
### 1. Top Paying Data Analyst Job
To pinpoint the highest-paying roles, I filtered data analyst positions based on average annual salary and location, with an emphasis on remote jobs. This query showcases the top earning opportunities in the field.

```sql
SELECT
	job_id,
	job_title,
	job_location,
	job_schedule_type,
	salary_year_avg,
	job_posted_date
	-- name AS company_name
FROM
	job_postings_fact
-- LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
	job_title = 'Data Analyst'
	AND salary_year_avg IS NOT NULL
	AND job_location = 'Anywhere'
ORDER BY
	salary_year_avg DESC 
LIMIT 10
```

Key Insights into Top Data Analyst Jobs in 2023:

Significant Salary Range: The top 10 highest-paying roles for data analysts boast salaries ranging from $184,000 to $650,000, highlighting the substantial earning potential in this field.

Industry Representation: High-paying opportunities are offered by a variety of companies such as SmartAsset, Meta, and AT&T, demonstrating widespread demand across diverse sectors.

Role Diversity: Job titles vary significantly, from Data Analyst to Director of Analytics, showcasing the breadth of roles and specializations available in data analytics.

### 2. Top Paying Job Skills


To identify the skills essential for top-paying jobs, I combined the job postings with the skills data, offering valuable insights into the qualifications employers prioritize for high-salary roles.

```sql 
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg
        -- name AS company_name
    FROM
        job_postings_fact
    -- LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst'
				AND salary_year_avg IS NOT NULL
        AND job_location = 'Anywhere'
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)
-- Skills required for data analyst jobs
SELECT
    top_paying_jobs.job_id,
    job_title,
    salary_year_avg,
    skills
FROM
    top_paying_jobs
	INNER JOIN
    skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
	INNER JOIN
    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC
```
Key Skills in Demand for the Top 10 Highest-Paying Data Analyst Jobs in 2023:

SQL stands out as the most sought-after skill, appearing in 8 out of 10 roles.
Python is a close second, featured in 7 roles.
Tableau ranks highly, being required in 6 roles.
Additional skills such as R, Snowflake, Pandas, and Excel also demonstrate varying levels of demand.

### 3. Top Demanded Skills
This query revealed the skills most commonly sought in job postings, highlighting key areas of high demand.

```sql
SELECT
  skills_dim.skills,
  COUNT(skills_job_dim.job_id) AS demand_count
FROM
  job_postings_fact
  INNER JOIN
    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
  INNER JOIN
    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
  -- Filters job titles for 'Data Analyst' roles
  job_postings_fact.job_title_short = 'Data Analyst'
	-- AND job_work_from_home = True -- optional to filter for remote jobs
GROUP BY
  skills_dim.skills
ORDER BY
  demand_count DESC
LIMIT 5;
```

Top Skills for Data Analysts in 2023:

SQL and Excel continue to dominate, underscoring the importance of foundational expertise in data handling and spreadsheet management.
Programming and Visualization Tools such as Python, Tableau, and Power BI are highly sought after, reflecting the growing need for technical proficiency in data-driven storytelling and decision-making.


| Skill      | Demand Count |
|------------|--------------|
| SQL        | 7,291        |
| Excel      | 4,611        |
| Python     | 4,330        |
| Tableau    | 3,745        |
| Power BI   | 2,609        |
##### *Top Skills in Demand for Data Analysts in 2023*

### 4. Top Paying Skills

Analyzing the average salaries associated with various skills identifies which skills yield the highest earnings.



| Skills       | Average Salary ($) |
|--------------|---------------------|
| PySpark      | 208,172             |
| Bitbucket    | 189,155             |
| Couchbase    | 160,515             |
| Watson       | 160,515             |
| DataRobot    | 155,486             |
| GitLab       | 154,500             |
| Swift        | 153,750             |
| Jupyter      | 152,777             |
| Pandas       | 151,821             |
| Elasticsearch| 145,000             |
##### _Table of the Average Salary for the Top 10 Paying Skills for Data Analysts_

### 5. Top Skills to Master
This query combined demand and salary data to identify skills that are both highly sought-after and well-paid, providing a strategic guide for skill development.
``` sql
WITH skills_demand AS (
  SELECT
    skills_dim.skill_id,
		skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
  FROM
    job_postings_fact
	  INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	  INNER JOIN
	    skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
  WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
		AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_work_from_home = True
  GROUP BY
    skills_dim.skill_id
),
-- Skills with high average salaries for Data Analyst roles
-- Use Query #4 (but modified)
average_salary AS (
  SELECT
    skills_job_dim.skill_id,
    AVG(job_postings_fact.salary_year_avg) AS avg_salary
  FROM
    job_postings_fact
	  INNER JOIN
	    skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
	  -- There's no INNER JOIN to skills_dim because we got rid of the skills_dim.name 
  WHERE
    job_postings_fact.job_title_short = 'Data Analyst'
		AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_work_from_home = True
  GROUP BY
    skills_job_dim.skill_id
)
-- Return high demand and high salaries for 25 skills 
SELECT
  skills_demand.skills,
  skills_demand.demand_count,
  ROUND(average_salary.avg_salary, 2) AS avg_salary --ROUND to 2 decimals 
FROM
  skills_demand
	INNER JOIN
	  average_salary ON skills_demand.skill_id = average_salary.skill_id
-- WHERE demand_count > 10
ORDER BY
  demand_count DESC, 
	avg_salary DESC
LIMIT 25
; 
```



| Skill ID | Skills       | Demand Count | Average Salary ($) |
|----------|--------------|--------------|---------------------|
| 8        | Go           | 27           | 115,320             |
| 234      | Confluence   | 11           | 114,210             |
| 97       | Hadoop       | 22           | 113,193             |
| 80       | Snowflake    | 37           | 112,948             |
| 74       | Azure        | 34           | 111,225             |
| 77       | BigQuery     | 13           | 109,654             |
| 76       | AWS          | 32           | 108,317             |
| 4        | Java         | 17           | 106,906             |
| 194      | SSIS         | 12           | 106,683             |
| 233      | Jira         | 20           | 104,918             |
##### _Table of the Most Optimal Skills for Data Analysts Sorted by Salary_

Here's an overview of the most valuable skills for Data Analysts in 2023:

- **High-Demand Programming Languages:** Python and R lead in demand, with 236 and 148 job postings respectively. Despite their widespread use, their average salaries—$101,397 for Python and $100,499 for R—suggest that while these skills are essential, they are also widely accessible.

- **Cloud Tools and Technologies:** Specialized tools like Snowflake, Azure, AWS, and BigQuery demonstrate strong demand alongside competitive salaries. This trend highlights the increasing reliance on cloud platforms and big data technologies in analytics.

- **Business Intelligence and Visualization Tools:** Tools such as Tableau and Looker, with demand counts of 230 and 49 respectively, and average salaries of $99,288 and $103,795, underscore the importance of visualizing data and driving business decisions through actionable insights.

- ** Database Technologies:** Skills in both traditional and NoSQL databases, including Oracle, SQL Server, and NoSQL, remain in demand. Salaries ranging from $97,786 to $104,534 reflect the ongoing need for expertise in data storage, retrieval, and management.
# What I Learned
During this journey, I significantly enhanced my SQL expertise, gaining proficiency in several critical areas:

- **Advanced Query Design:** Developed expertise in crafting complex SQL queries, seamlessly joining tables, and leveraging WITH clauses to create efficient temporary tables.
- **Data Aggregation:** Gained a deep understanding of GROUP BY and mastered the use of aggregate functions such as COUNT() and AVG() to effectively summarize data.
- **Analytical Problem-Solving:** Enhanced my ability to translate real-world problems into actionable and insightful SQL queries, demonstrating the practical application of analytical techniques.


# Conclusions
Insights 
from the Analysis, the following discoveries were made:
1. **Top-Paying Remote Data Analyst Roles:** Remote data analyst jobs can offer exceptionally high salaries, with the top-paying role reaching an impressive $650,000.

2. **Key Skills for High-Paying Positions:** Advanced SQL proficiency stands out as a critical requirement for securing the highest-paying data analyst roles.

3. **Most In-Demand Skills:** SQL emerges as the most sought-after skill in the data analyst job market, underscoring its importance for job seekers.

4. **Specialized Skills and High Salaries:** Niche skills like SVN and Solidity command the highest average salaries, reflecting the premium placed on specialized expertise.

5. **Optimal Skill for Market Value:** SQL not only leads in demand but also offers competitive average salaries, making it one of the most valuable skills for data analysts to master for career growth.

### Final Reflections
This project not only sharpened my SQL expertise but also unveiled key insights into the data analyst job landscape. The analysis serves as a strategic roadmap for prioritizing skill development and navigating job opportunities.

For aspiring data analysts, focusing on high-demand and high-paying skills can be a game-changer in standing out in a competitive market. This journey underscores the power of continuous learning and staying ahead of emerging trends in the ever-evolving field of data analytics.