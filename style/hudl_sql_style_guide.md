---
layout: default
title: SQL Style Guide
description: A guide to writing clean, clear, and consistent SQL.
---

# Purpose


# Principles

* We take a disciplined and practical approach to writing code.
* We regularly check-in code to Github
* We believe consistency in style is very important.
* We demonstrate intent explicitly in code, via clear structure and comments where needed.

# Rules

## General

* No tabs. 2 spaces per indent.
* No trailing whitespace.
* 80 character limit per line
* Include links to hudl id constants or references to important links
* All tables and views should adhere to all lowercase, words separated with underscores
* Always use AS when creating aliases for columns, tables, subqueries
* Always capitalize SQL keywords (e.g., `SELECT` or `AS`)
* Always capitalize SQL datatypes (e.g., `VARCHAR` or `DATE`)
* Variable names and identifiers should be lower case and underscore separated and all other text should be capitalized:

  __GOOD__:
  `SELECT COUNT(*) AS backers_count`

  __BAD__:
  `SELECT COUNT(*) AS backersCount`

* Comments should go near the top of your query, or at least near the closest `SELECT`
* Try to only comment on things that aren't obvious about the query (e.g., why a particular ID is hardcoded, etc.)
* Don't use single letter variable names be as descriptive as possible given the context:

  __GOOD__:
  `SELECT ksr.backings AS backings_with_creators`

  __BAD__:
  `SELECT ksr.backings AS b`

* Use [Common Table Expressions](http://www.postgresql.org/docs/8.4/static/queries-with.html) (CTEs) early and often, and name them well.
* All dates should be formatted as 'YYYY-MM-DD'
* Use != for the inequality operator

## `SELECT`

Align all columns to the first column on their own line:

```sql
SELECT
  projects.name,
  users.email,
  projects.country,
  COUNT(backings.id) AS backings_count
FROM ...
```

`SELECT` goes on its own line:

```sql
SELECT [DISTINCT]
  name,
  ...
```

Always rename aggregates and function-wrapped columns:

```sql
SELECT
  name,
  SUM(amount) AS sum_amount
FROM ...
```


Long Window functions should be split across multiple lines: one for the `PARTITION`, `ORDER` and frame clauses, aligned to the `PARTITION` keyword. Partition keys should be one-per-line, aligned to the first, with aligned commas. Order (`ASC`, `DESC`) should always be explicit. All window functions should be aliased.

```sql
SUM(1) OVER (
  PARTITION BY 
    category_id,
    year
  ORDER BY 
    pledged DESC
  ROWS UNBOUNDED PRECEDING
) AS category_year
```

## `FROM`

Only one table should be in the `FROM`. Never use `FROM`-joins:

__GOOD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
JOIN ksr.backings AS backings ON backings.project_id = projects.id
...
```

__BAD__:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects, ksr.backings AS backings
WHERE
  backings.project_id = projects.id
...
```

## `JOIN`
Additional filters in the `JOIN` go on new indented lines:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
JOIN ksr.backings AS backings ON projects.id = backings.project_id
  AND backings.project_country != 'US'
...
```

The `ON` keyword and condition goes on the `JOIN` line:

```sql
SELECT
  projects.name AS project_name,
  COUNT(backings.id) AS backings_count
FROM ksr.projects AS projects
JOIN ksr.backings AS backings ON projects.id = backings.project_id
...
```

Begin with `JOIN`s and then list `LEFT JOIN`s, order them semantically, and do not intermingle `LEFT JOIN`s with `JOIN`s unless necessary:

__GOOD__:

```sql
JOIN ksr.backings AS backings ON ...
JOIN ksr.users AS users ON ...
JOIN ksr.locations AS locations ON ...
LEFT JOIN ksr.backer_rewards AS backer_rewards ON ...
LEFT JOIN ...
```

__BAD__:

```sql
LEFT JOIN ksr.backer_rewards AS backer_rewards ON backings
JOIN ksr.users AS users ON ...
LEFT JOIN ...
JOIN ksr.locations AS locations ON ...
```

## `WHERE`

Multiple `WHERE` clauses should go on different lines and begin with the SQL operator:

```sql
SELECT
  name,
  goal
FROM ksr.projects AS projects
WHERE country = 'US'
  AND deadline >= '2015-01-01'
...
```

## `GROUP BY`

GROUP BY/ORDER BY (group by and order by on line by itself, list of fields start on next line 2 spaces in. 

```SQL
GROUP BY 
  schoolid, 
  name, 
  organization_type
```


## `CASE`

`CASE` statements aren't always easy to format but try to align `WHEN` and `ELSE` together inside `CASE` and `END`:

```sql
CASE 
  WHEN category = 'Art' THEN backer_id
  ELSE NULL
END
```

## Parentheses
Open parentheses will start on same line and then start code on next line 2 spaces in
Closing parentheses will be on the line after end of code, alias can be on same line as closing parentheses
One exception is nested parentheses in where (any boolean comparisons) clause. If it can fit on one line put it on one line otherwise stick with parentheses rules
Examples below

```SQL
SUM(1) OVER (
  PARTITION BY
    category_id,
    year
  ORDER BY pledged DESC
  ROWS UNBOUNDED PRECEDING
) AS category_year


SELECT 
FROM
WHERE sport in (
  ‘Football’,
  ‘Basketball’
)
WITH table_cte AS (
  SELECT
    ..
  FROM
)


SELECT
  …
FROM (
  SELECT
    ..
  FROM
)
```

## IN/NOT IN Operator
If you have 4 or less items you have the option to list out on one single line, otherwise use parentheses guidance shown below: 

```SQL
AND etas.state NOT IN (
  'Rhode Island',
  'South Dakota',
   'Alaska',
)
```

## Tips
