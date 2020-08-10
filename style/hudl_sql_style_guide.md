---
layout: default
title: SQL Style Guide
description: A guide to writing clean, clear, and consistent SQL.
---

# Purpose
This document exists as a resource of guiding principles for those who build, refactor and work with data models. It contains Decision Science’s requirements and best practices for data modeling, SQL style and data testing.

# Principles
* We take a disciplined and practical approach to writing code
* We regularly check-in code to Github
* We believe consistency in style is very important
* We demonstrate intent explicitly in code, via clear structure and comments where needed
* We value DRY (do not repeat yourself) code and readability, but will favor readability over DRY when the two are in conflict
* We believe everyone is responsible for code and modeling quality

## Data Modeling
### Data Modeling: The process of using business logic to aggregate or otherwise transform raw data
### Data Model: A structure that organizes and standardizes data elements and how they relate to each other and a real-world entity or concept. In our case, a data model is a standardized representation of a business entity or concept. 

When it has been determined that a new data model is required, the person building the new model will be required to hold a data model design review. This is a process in which the builder of the data model will write-up their design plans and hold a meeting for these plans to be peer reviewed. 

The purpose of this is not to block or slow work. Rather, this is a means to ensure data quality, prevent painful re-work and identify dependencies that may have been overlooked. Each data model design will require the following:

* __State the purpose__: 
    Every new data model that is introduced must have a clear purpose linked to business value. All models added should be done so with a high amount of rigor to       ensure data quality.
* __Start With the End in Mind__: 
    Test Driven Development https://www.guru99.com/test-driven-development.html is when the tests are written before the code. While we don’t require a full set of     all tests in the Data Model Design process, an overview of test cases that will be considered and implemented should be included as a part of the data model       design review.
* __Identify the Grains__: 
    Identify what the grains will be of the final table and of any tables from which you will be creating your final table.
* Sample of table output with examples (can be hypothetical)
* Column names and descriptions defined 

### File Orgainzation
* __Business-centric__ models and __source-centric__ models should be organizationally distinct. __Business-centric__ models define business logic. __Source-centric__ models select from dbt sources and their primary function is to clean and group like-data (ie: payment data may come from two different sources, but need to be joined).

  __Business-centric__ models should be created within the marts directory. __Source-centric__ models should be created within the staging directory. 

> Many pre-existing business-centric models do not exist within the marts directory.
            However, all new ones should be created there.

* Define raw data as dbt sources: No raw data sources should be referenced other than in sources. If a source and staging layer do not exist for a model you want to build, you are required to build the needed sources and staging layers.
* __Staging__ 
  - Used to clean sources
  - Rename and Recast fields in this layer
  - There should only be one staging file per one source. Only seed files should be joined at this layer if necessary.
  - No business metrics/values should be created at this layer

* __dbt seed__
  - All seed files belong in the `data` directory.
  - Seed files are best used for mappings (ie. country codes to country names)
* __dbt snapshots__
  - Use to track slowly changing dimensions
  - As a general rule should be built on top of sources
* __dbt macros__
  - Best for trusted, reusable pieces of logic (ie.fiscal year creation).

### File Materialization
* __Table__
  - Use the table materialization for any models being queried by BI tools (ie. Looker), to give your end user a faster experience
  - Also use the table materialization for any slower transformations that are used by many downstream models
  - Are fast to query but take longer to build, especially when they contain complex transformations and or a large amount of data, so keep this in mind when     selecting this materialization
* __View__
  - Builds faster, but requires more time for the end user to query. 
  - Is dbt’s default materialization if you do not specify one
  - Generally start with views for your models, and only change to another materialization when you're noticing performance problems.
  - Views are best suited for models that do not do significant transformation, e.g. renaming, recasting columns.
  - As a general rule, all staging models should be materialized as views
  - As a general rule, you will want the view to be late-binding. This is currently the default in our dbt project and does not need to be specified in every file.     For  more information on why we do this, you can check out this article: https://blog.getdbt.com/using-redshift-s-late-binding-views-with-dbt/

* __Ephemeral__
  - Use for very light-weight transformations that are early on in your DAG
  - Use when code will be re-used in multiple files to help keep code DRY. However, if this code won’t be re-used, consider making it a       CTE rather than it’s own file.
  - Should be used when the piece of logic needs to be referenced in multiple downstream files. 
  - This data should not need to be queried by end users and only needs to be referenced for the purposes of data modeling

* __Incremental__
  - Allows dbt to insert or update records into a table since the last time that dbt was run
  - Incremental models are best for event-style data (ie: the user activity model)
  - Use incremental models when your dbt runs are becoming too slow (don't start with incremental models)
  - Be aware that the need for full-refreshing will add complication to this model and make sure that the benefits outweigh the potential downfalls that may come       with higher maintenance.
  
### Sort and Dist Keys
More in-depth documentation on this topic can be found here: https://docs.aws.amazon.com/redshift/latest/dg/c_designing-tables-best-practices.html

* __Sort Key Selection__: Amazon Redshift stores your data on disk in sorted order according to the sort key. The Amazon Redshift query optimizer uses sort order when it determines optimal query plans. Some suggestions for best approach include:
  - If recent data is queried most frequently, specify the timestamp column as the leading column for the sort key. Queries are more efficient because they can         skip entire blocks that fall outside the time range.
  - If you do frequent range filtering or equality filtering on one column, specify that column as the sort key. Amazon Redshift can skip reading entire blocks of     data for that column. It can do so because it tracks the minimum and maximum column values stored on each block and can skip blocks that don't apply to the         predicate range.
  - If you frequently join a table, specify the join column as both the sort key and the distribution key. Doing this enables the query optimizer to choose a sort     merge join instead of a slower hash join. Because the data is already sorted on the join key, the query optimizer can bypass the sort phase of the sort merge       join.
  
 * __Dist Key Selection__: When you execute a query, the query optimizer redistributes the rows to the compute nodes as needed to perform any joins and aggregations. The goal in selecting a table distribution style is to minimize the impact of the redistribution step by locating the data where it needs to be before the query is run. Some suggestions for best approach include:
 
    - Distribute the fact table and one dimension table on their common columns. Your fact table can have only one distribution key. Any tables that join on another     key aren't collocated with the fact table. Choose one dimension to   collocate based on how frequently it is joined and the size of the joining rows. Designate     both the dimension table's primary key and the fact table's corresponding foreign key as the DISTKEY.
    - Choose the largest dimension based on the size of the filtered dataset. Only the rows that are used in the join need to be distributed, so consider the size of     the dataset after filtering, not the size of the table.
    - Choose a column with high cardinality in the filtered result set. If you distribute a sales table on a date column, for example, you should probably get fairly     even data distribution, unless most of your sales are seasonal. However, if you commonly use a range-restricted predicate to filter for a narrow date period,       most of the filtered rows occur on a limited set of slices and the query workload is skewed.
    - Change some dimension tables to use ALL distribution. If a dimension table cannot be collocated with the fact table or other important joining tables, you can     improve query performance significantly by distributing the entire table to all of the nodes. Using ALL distribution multiplies storage space requirements and     increases load times and maintenance operations, so you should weigh all factors before choosing ALL distribution.

### Naming Conventions

* Rename and recast fields once. The earlier in the DAG this can be done, the better.
* Use prefixes in table names (for example, stg_, fct_ and dim_) to indicate which relations should be queried by end users.
* All tables and views should adhere to all lowercase, words separated with underscores
* Models with objects in the name should be plural (i.e. dim_product**s**)
* Avoid using numbers or special characters in CTE names
* Variable names and identifiers should be lower case and underscore separated and all other text should be capitalized:

  __GOOD__:
  `SELECT COUNT(*) AS backers_count`

  __BAD__:
  `SELECT COUNT(*) AS backersCount`

* Don't use single letter variable names be as descriptive as possible given the context:

  __GOOD__:
  `SELECT ksr.backings AS backings_with_creators`

  __BAD__:
  `SELECT ksr.backings AS b`

## SQL Style Guide

### General Rules

* No tabs. 2 spaces per indent.
* No trailing whitespace.
* 80 character limit per line
* Include links to hudl id constants or references to important links
* Always use AS when creating aliases for columns, tables, subqueries
* Always capitalize SQL keywords (e.g., `SELECT` or `AS`)
* Always capitalize SQL datatypes (e.g., `VARCHAR` or `DATE`)
* Comments should go near the top of your query, or at least near the closest `SELECT`
* Try to only comment on things that aren't obvious about the query (e.g., why a particular ID is hardcoded, etc.)
* All dates should be formatted as 'YYYY-MM-DD'
* Use `!=` for the inequality operator


### Common Table Expressions (CTEs)

* Use [Common Table Expressions](http://www.postgresql.org/docs/8.4/static/queries-with.html) often, and place them at the top of code
* CTEs should be used to represent **ALL** table/model references to make code more readable
* Only use full CTE names when referencing them in the `SELECT` statement. Only use aliases to rename columns.

```sql
WITH project_costs AS (
  SELECT 
    *
  FROM ksr.projects
) 

SELECT
  *
FROM project_costs
```


### `SELECT`

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

Always rename aggregates and function-wrapped columns, and leave a space between dimensions and aggregate columns:

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

### `FROM`

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

### `JOIN`
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

### `WHERE`

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

### `GROUP BY`

GROUP BY/ORDER BY (group by and order by on line by itself, list of fields start on next line 2 spaces in.

```SQL
GROUP BY
  schoolid,
  name,
  organization_type
```


### `CASE`

`CASE` statements aren't always easy to format but try to align `WHEN` and `ELSE` together inside `CASE` and `END`:

```sql
CASE
  WHEN category = 'Art' THEN backer_id
  ELSE NULL
END
```

### Parentheses
Open parentheses will start on same line and then start code on next line 2 spaces in
Closing parentheses will be on the line after end of code, alias can be on same line as closing parentheses
One exception is nested parentheses in where (any boolean comparisons) clause. If it can fit on one line put it on one line otherwise stick with parentheses rules
Examples below

```SQL
SUM(1) OVER (
  PARTITION BY
    category_id,
    year
  ORDER BY
    pledged DESC
  ROWS UNBOUNDED PRECEDING
) AS category_year


SELECT
FROM
WHERE sport IN (
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

### IN/NOT IN Operator
If you have 4 or less items you have the option to list out on one single line, otherwise use parentheses guidance shown below:

```SQL
AND etas.state NOT IN (
  'Rhode Island',
  'South Dakota',
  'Alaska',
)
```

## Data Testing and Quality
### Things to consider
* Source Data: Think outside the box, not just dbt. If you are working with a product team, what can they assist you with on their end to ensure data quality?  Think doomsday scenarios. What could go wrong and how would you catch it if it did? Types of dbt tests that may help:
  - Accepted values 
  - Null Tests
  - At least one
  - Unique
  - Relationships
* Recency:  How will you be alerted if this data isn’t updated?
* Null values: Can this column contain null values and still be accurate?
* Uniqueness: What is the unique key? 
* Load: How much time are your additions adding to DBT Master build time? How long do the queries that end users will write take? Test the load by querying your  table in both Redash and Looker. Consider your sort and dist keys to improve these metrics

### Requirements 
* __New Models__ : New models will be held to the data design review standards and should contain at minimum the following tests:
  - Unique Key
  - Null tests on unique key and on any column that is used to join on
  - Tests for recency

* __Additions/Changes to Existing Models__: Any time you make changes to an existing model, you are responsible for standardizing the code to meet current guidelines. This includes the SQL style, testing and other convention guidelines found in this document.  
  
* __Fixing Bugs in Old Models__: Additions of tests are absolutely required when a bug is fixed to ensure that our model is resistant to the similar issue repeating in the future. In this instance, the tests added need to account for the bug(s) being fixed.
  





## Tips
