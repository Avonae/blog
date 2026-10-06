---
title: Calculating Work Time in Confluence
date: '2024-03-03'
translationKey: podschet-staja-v-sql
aliases:
- /2024-03-03-Podschet-staja-v-SQL/
---

At work, I was tasked with calculating the work time for employees in a table, based on their hire date. Since we use Confluence for internal documentation, it was decided to do it there too.

We have the [Table Filter & Charts for Confluence](https://marketplace.atlassian.com/apps/27447/table-filter-charts-spreadsheets-for-confluence?hosting=datacenter&tab=overview) plugin, which lets you take a table from an article and work with it via SQL. The plugin uses some [cursed AlaSQL](https://alasql.org/) that resembles TSQL in syntax. Using the plugin, I built a new table from the employee data with four columns: number, full name, position, and tenure.

And so, everything was going fine. I found some similar code online, plugged it in… and realized that TSQL somehow thinks that if an employee was hired on 11.03.22, and today is 03.03.24, the difference is 24 MONTHS. As a result, the number of days was off by one month, showing -6 instead of 24.

Why it decided on 24 months, I still don’t know. But because of this, I had to calculate months based on the number of days between the dates, which were calculated correctly.

Anyway, here’s my code—hope it’s useful to someone. Of course, I don’t know SQL and I’m not a programmer, so feel free to share improvements in the comments.

```sql
SELECT '#','Name', 'Position',
CONCAT
(
 DATEDIFF(YEAR, 'hire date', GETDATE()), " years, ",
FLOOR(floor(DATEDIFF(DAY, 'hire date', GETDATE())/30.4375) - (DATEDIFF(YEAR, 'hire date', GETDATE()) * 12)), " months, ",
FLOOR(DATEDIFF(DAY, 'hire date', GETDATE()) - 
365*FLOOR(DATEDIFF(YEAR, 'hire date', GETDATE())) - 
30.4375*FLOOR(floor(DATEDIFF(DAY, 'hire date', GETDATE())/30.44)-DATEDIFF(YEAR, 'hire date', GETDATE()) * 12)), " days"
)
AS 'experience' FROM T1;
```