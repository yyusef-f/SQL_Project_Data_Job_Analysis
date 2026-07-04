# 📊 SQL Project: Data Job Analysis

## Introduction

Dive into the data job market! This project focuses on Data Analyst
roles, exploring 💰 top-paying jobs, 🔥 in-demand skills, and 📈 where
high demand meets high salary — all uncovered through SQL.

🔍 SQL queries used in this project: [project_sql](project_sql/)

## Background

Driven by a desire to navigate the data analyst job market more
effectively, this project set out to pinpoint top-paying and in-demand
skills, streamlining the process of finding optimal roles for aspiring
and current data analysts.

The data is sourced from `[Insert dataset source here]` and is packed
with insights on job titles, salaries, locations, and essential skills.

### The questions this project answers through SQL:

1. What are the top-paying Data Analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for Data Analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

## Tools & Technologies

| Tool | Purpose |
|---|---|
| **SQL** | The backbone of the analysis — querying the database to unearth critical insights |
| **PostgreSQL** | Database management system used to host and query the job posting data |
| **Visual Studio Code** | Primary editor for writing and managing SQL scripts |
| **pgAdmin** | GUI client for database administration and query execution |
| **Git & GitHub** | Version control and project sharing, ensuring collaboration and tracking |

---

## The Analysis

Each query in this project targets a specific aspect of the Data Analyst
job market.

### 1. Top-Paying Data Analyst Jobs

To identify the highest-paying roles, this query filters Data Analyst
positions by average yearly salary and location, focusing on remote
jobs.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

**Breakdown of results:**
- **Salary range:** the top 10 remote roles span from **$184,000** to **$650,000** a year.
- **Employers represented:** Mantys, Meta, AT&T, Pinterest, UCLA Health, SmartAsset (x2), Inclusively, Motional, and Get It Recruit — a mix of tech, healthcare, and staffing/recruiting companies.
- **Job title diversity:** roles range from individual-contributor "Data Analyst" positions to leadership titles like Director of Analytics and Associate Director of Data Insights, showing that top pay isn't limited to a single seniority level.

| Company | Job Title | Avg. Yearly Salary |
|---|---|---:|
| Mantys | Data Analyst | $650,000 |
| Meta | Director of Analytics | $336,500 |
| AT&T | Associate Director - Data Insights | $255,830 |
| Pinterest | Data Analyst, Marketing | $232,423 |
| UCLA Health | Data Analyst (Hybrid/Remote) | $217,000 |
| SmartAsset | Principal Data Analyst (Remote) | $205,000 |
| Inclusively | Director, Data Analyst - Hybrid | $189,309 |
| Motional | Principal Data Analyst, AV Performance Analysis | $189,000 |
| SmartAsset | Principal Data Analyst | $186,000 |
| Get It Recruit | ERM Data Analyst | $184,000 |

*Table of the top 10 highest-paying remote Data Analyst postings.*

---

### 2. Skills for Top-Paying Jobs

To understand what skills the top-paying jobs require, this query joins
the top 10 highest-paying postings with the skills data.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_jobs.*,
    skills_dim.skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    top_paying_jobs.salary_year_avg DESC;
```

**Breakdown of results** (across the top-paying postings with skill data):
- **SQL** leads, appearing in every one of these postings.
- **Python** is close behind, appearing in nearly all of them.
- **Tableau** is the most requested visualization tool.
- **R** shows up consistently as a secondary programming language.
- Cloud and engineering tools — **Azure, AWS, Snowflake, Databricks, PySpark** — and collaboration tools — **Jira, Confluence, Bitbucket, Atlassian** — appear repeatedly, reflecting the seniority of these roles.

| Skill | Appearances |
|---|---:|
| SQL | 8 |
| Python | 7 |
| Tableau | 6 |
| R | 4 |
| Pandas | 3 |
| Excel | 3 |
| Snowflake | 3 |

*Skill frequency among the top-paying postings returned with matching skill records.*

---

### 3. Most In-Demand Skills for Data Analysts

This query identifies the skills most frequently requested across all
remote Data Analyst postings.

```sql
SELECT
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_work_from_home = TRUE
GROUP BY skills_dim.skills
ORDER BY demand_count DESC
LIMIT 5;
```

**Breakdown of results:**
- **SQL and Excel** remain the clear foundation, underscoring the importance of core data-processing and spreadsheet skills.
- **Python, Tableau, and Power BI** round out the top 5, confirming that programming and visualization skills are essential alongside the fundamentals.

| Skill | Demand Count |
|---|---:|
| SQL | 7,291 |
| Excel | 4,611 |
| Python | 4,330 |
| Tableau | 3,745 |
| Power BI | 2,609 |

*Table of the top 5 most in-demand skills for Data Analyst postings.*

---

### 4. Skills Based on Salary

This query calculates the average salary associated with each skill to
identify which command the highest pay.

```sql
SELECT
    skills_dim.skills,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS average_salary
FROM
    job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_postings_fact.salary_year_avg IS NOT NULL AND
    job_work_from_home = TRUE
GROUP BY skills_dim.skills
ORDER BY average_salary DESC
LIMIT 25;
```

**Breakdown of results:**
- **Big data & ML tools** dominate the top of the list — **PySpark ($208K)**, **Databricks**, **NumPy**, and **Jupyter** all rank highly, reflecting a premium on large-scale data processing and modeling skills.
- **DevOps/engineering crossover tools** — **Bitbucket ($189K)**, **GitLab**, **Kubernetes**, **Airflow**, and **Jenkins** — show that analysts with deployment and pipeline experience earn significantly more.
- **Specialized/niche platforms** — **Couchbase**, **Watson**, and **DataRobot** — all average above $150K, suggesting rare expertise commands a strong premium even with lower overall demand.

| Skill | Average Salary |
|---|---:|
| PySpark | $208,172 |
| Bitbucket | $189,155 |
| Couchbase | $160,515 |
| Watson | $160,515 |
| DataRobot | $155,486 |
| GitLab | $154,500 |
| Swift | $153,750 |
| Jupyter | $152,777 |
| Pandas | $151,821 |
| Elasticsearch | $145,000 |
| Golang | $145,000 |
| NumPy | $143,513 |
| Databricks | $141,907 |
| Linux | $136,508 |
| Kubernetes | $132,500 |
| Atlassian | $131,162 |
| Twilio | $127,000 |
| Airflow | $126,103 |
| Scikit-learn | $125,781 |
| Jenkins | $125,436 |
| Notion | $125,000 |
| Scala | $124,903 |
| PostgreSQL | $123,879 |
| GCP | $122,500 |
| MicroStrategy | $121,619 |

*Table of the top 25 highest-paying skills for remote Data Analysts.*

---

### 5. Most Optimal Skills to Learn

Combining demand and salary data, this query pinpoints skills that offer
both strong market demand and strong pay.

```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM
        job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_work_from_home = TRUE AND
        job_postings_fact.salary_year_avg IS NOT NULL
    GROUP BY skills_dim.skill_id
),

average_salary AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS average_salary
    FROM
        job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_postings_fact.salary_year_avg IS NOT NULL AND
        job_work_from_home = TRUE
    GROUP BY skills_dim.skill_id
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    skills_demand.demand_count,
    average_salary.average_salary
FROM skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE demand_count > 10
ORDER BY
    average_salary DESC,
    demand_count DESC
LIMIT 25;
```

| Skill | Demand Count | Average Salary |
|---|---:|---:|
| Go | 27 | $115,320 |
| Confluence | 11 | $114,210 |
| Hadoop | 22 | $113,193 |
| Snowflake | 37 | $112,948 |
| Azure | 34 | $111,225 |
| BigQuery | 13 | $109,654 |
| AWS | 32 | $108,317 |
| Java | 17 | $106,906 |
| SSIS | 12 | $106,683 |
| Jira | 20 | $104,918 |
| Oracle | 37 | $104,534 |
| Looker | 49 | $103,795 |
| NoSQL | 13 | $101,414 |
| Python | 236 | $101,397 |
| R | 148 | $100,499 |
| Redshift | 16 | $99,936 |
| Qlik | 13 | $99,631 |
| Tableau | 230 | $99,288 |
| SSRS | 14 | $99,171 |
| Spark | 13 | $99,077 |
| C++ | 11 | $98,958 |
| SAS | 63 | $98,902 |
| SQL Server | 35 | $97,786 |
| JavaScript | 20 | $97,587 |

*Table of the most optimal skills for Data Analysts, sorted by salary. Note: SAS appears twice in the source data under two different skill IDs (7 and 186) with identical figures; both are collapsed into a single row above.*

**Breakdown of results:**
- **High-demand programming languages:** Python (236 postings) and R (148 postings) show by far the highest demand of any skill on this list, though their average salaries (~$101K and ~$100K) sit in the middle of the range — reflecting how widely available these skills are.
- **Cloud tools and technologies:** Snowflake, Azure, BigQuery, and AWS all combine solid demand (32–37 postings) with above-average salaries ($108K–$113K), pointing to cloud proficiency as a strong differentiator.
- **BI and visualization tools:** Looker (49 postings, $103,795) and Tableau (230 postings, $99,288) confirm that visualization skills are both widely required and consistently well compensated.
- **Database technologies:** Oracle, SQL Server, and NoSQL average between roughly $97,800 and $104,500, showing steady demand for traditional and NoSQL database expertise.

---

## What I Learned

Throughout this project, I strengthened my SQL toolkit with some real
firepower:

- **🧩 Complex Query Crafting:** Practiced advanced SQL — joining
  multiple tables and using `WITH` clauses to build clean, reusable CTEs.
- **📊 Data Aggregation:** Got comfortable with `GROUP BY`, turning
  `COUNT()` and `AVG()` into go-to tools for summarizing large datasets.
- **💡 Analytical Thinking:** Sharpened my ability to turn open-ended
  questions into precise, actionable SQL queries — and query results into
  clear business insight.

---

## Conclusions

### Insights

From this analysis, the following insights emerged:

1. **Top-Paying Data Analyst Jobs:** Remote Data Analyst roles can pay
   as much as **$650,000** a year, with the top 10 ranging from
   $184,000 to $650,000 — and high pay isn't limited to senior titles.
2. **Skills for Top-Paying Jobs:** SQL and Python are near-universal
   requirements at the top of the pay scale, with Tableau and R as
   strong secondary skills.
3. **Most In-Demand Skills:** SQL is the single most requested skill
   in the market (7,291 postings), ahead of Excel, Python, Tableau, and
   Power BI.
4. **Skills with Higher Salaries:** Specialized big-data and DevOps
   tools — PySpark, Bitbucket, Couchbase, Watson — command the highest
   average salaries, all above $150,000.
5. **Optimal Skills for Job Market Value:** Cloud platforms (Snowflake,
   Azure, AWS) and BI tools (Looker, Tableau) offer the strongest
   combination of solid demand and above-average pay.

### Closing Thoughts

This project strengthened my SQL skills while providing practical
insight into the Data Analyst job market. The findings offer a clear
guide for prioritizing skill development and job search strategy —
SQL and Python for baseline employability, and cloud or BI tools like
Snowflake, Azure, or Looker for standing out and commanding a higher
salary. It also reinforced the value of continuous learning in a field
where in-demand tools keep evolving.


## Author

**Name:** `Youssef El Marghany`

**GitHub:** [yyusef-f](https://github.com/yyusef-f)

**LinkedIn:** [Youssef El Marghany](https://www.linkedin.com/in/youssef-el-marghany-592a55375)
