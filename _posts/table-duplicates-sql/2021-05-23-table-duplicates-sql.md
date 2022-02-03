---
title: Finding Table Duplicates with SQL
date: 2021-05-25 10:00:00 +07:00
tags: [programming]
description: Finding all duplicated records in table with SQL
---

# Finding all duplicated records in table with SQL

## We are given code

```sql
DECLARE @testTable table (name nvarchar(50), createdDate datetime);
insert into @testTable (name, createdDate)
Values (NULL,'20160101 20:36:12'),
('ADAM','20160101 20:38:27'),
('Zbyszek','20160101 20:39:02'),
('ADAM','20160101 20:41:55'),
('ADAM','20160101 20:42:31'),
('ADAM','20160101 20:51:11'),
('ADAM','20160101 20:51:24'),
(NULL,'20160101 20:52:01'),
('Zbyszek','20160101 20:53:07'),
('Zbyszek','20160101 20:53:44'),
('Renata','20160101 20:54:52'),
(NULL,'20160101 20:54:57'),
('Zygmunt','20160101 20:54:59'),
(NULL,'20160101 20:55:03'),
(NULL,'20160101 20:56:12'),
('Renata','20160101 20:57:22'),
('Zygmunt','20160101 20:58:31'),
('Zygmunt','20160101 20:59:41'),
('Zygmunt','20160101 21:02:02'),
('Zygmunt','20160101 21:03:59'),
('ADAM','20160101 21:42:31'),
('ADAM','20160101 21:51:11'),
('ADAM','20160101 21:51:24');
```

## The result should be:

<div class="overflow-table" markdown="block">

| name                    | count  | Minimal date              |  Maximal date           |
| :---------------------- | :----- | :-----------------------: |:------------------------: 
| ADAM                    | 4      | 2016-01-01 20:41:55.000   | 2016-01-01 20:51:24.000 |
| Zbyszek                 | 2      | 2016-01-01 20:53:07.000   | 2016-01-01 20:53:44.000 |
| ADAM                    | 3      | 2016-01-01 21:42:31.000   | 2016-01-01 21:51:24.000 |
| Zygmunt                 | 4      | 2016-01-01 20:58:31.000   | 2016-01-01 21:03:59.000 |


</div>

We want to record in the table the records that indicate which names repeated one after another — and write down the first and last recorded dates of such a set of repeating records.

## Solution

```sql
WITH C AS (
 SELECT createddate, name,
  --grouping by combining properties of name and creation date
 ROW_NUMBER() OVER(ORDER BY createddate) - ROW_NUMBER() OVER(ORDER BY name, createddate) AS groupped
 FROM testTable
  --blank names are omitted
WHERE name IS NOT NULL
)
SELECT name, COUNT(name) as ilosc, MIN(createddate) AS minimal_date, MAX(createddate) AS maximal_date
FROM C
GROUP BY name, groupped
--at least two grouped occurrences
HAVING Count(name) > 1
ORDER BY maximal_date;
```

Returns us the desired result.