---
layout: post
title: SQL Year-on-year queries
tags: [sql]
includes: [d3]
localscripts: [yearonyear]
---
SQL is great for aggregating data, but sometimes we want to compare the results
with another time period. A common example would be to compare year-on-year
monthly sales totals. Let's start by looking at a typical SQL query for grouping
and summing totals by month. This example is presented as a stored procedure for
MS SQL Server, but the general idea is similar across most major platforms:

{% highlight sql %}
-- For every month this year, get the total sales value
CREATE PROCEDURE GetMonthlyTotals
AS
BEGIN
  SET NOCOUNT ON

  -- Get the first day of the current year
  DECLARE @StartDate DATETIME;
  SET @StartDate = DATEADD(yy, DATEDIFF(yy, 0, GETDATE()), 0)

  -- Now run the query...
  SELECT
    MONTH(OrderDate) AS SalesMonth,
    SUM(OrderPrice) AS SalesTotal
  FROM [dbo].[OrderDetails]
  WHERE OrderDate >= @StartDate
  GROUP BY MONTH(OrderDate)
  ORDER BY MONTH(OrderDate);
END
{% endhighlight %}

This code assumes there is a table called `OrderDetails` which contains at least
columns called `OrderDate` and `OrderPrice`.

So far so good, but it would be even better if we could see at a glance how the
sales figures for June 2018 compared to June 2017.

One way would be to simply adjust `@StartDate` to begin a year previously. While
that would certainly give us enough figures to work with, they'd all be on
separate rows:

```
Month   Year    Total
01      2017    452908
02      2017    542021
...
12      2018    630948
01      2018    482093
02      2018    551530
...
```
What we *really* need is something like this:
```
Month    Total    LastYearTotal
01       482093   452908
02       551530   542021
...
```
Now it's clear that this year's figures seem to be improving over the same time
last year. But how do we do it? The trick is to adjust the `SUM` function to
include a `CASE` function. Here's the complete procedure:

{% highlight sql %}
-- Get each monthly sales total for this calendar year, and a comparison
-- total with last year.
CREATE PROCEDURE GetMonthlyTotals
AS
BEGIN
  SET NOCOUNT ON

  -- Get the first day of the current year
  DECLARE @StartDate DATETIME, @EndDate;
  SET @EndDate = GETDATE();
  SET @StartDate = DATEADD(yy, DATEDIFF(yy, 0, @EndDate)-1, 0);
  
  -- Get information about the current date
  DECLARE @CurrentMonth INT, @CurrentYear INT;
  SET @CurrentMonth = MONTH(@EndDate);
  SET @CurrentYear = YEAR(@EndDate);

  -- Now run the query...
  SELECT
    MONTH(OrderDate) AS SalesMonth,
    SUM(CASE WHEN YEAR(OrderDate) = @CurrentYear THEN OrderPrice ELSE NULL END) AS SalesTotal,
    SUM(CASE WHEN YEAR(OrderDate) = @CurrentYear - 1 THEN OrderPrice ELSE NULL END) AS LastYearSalesTotal
  FROM [dbo].[OrderDetails]
  WHERE OrderDate >= @StartDate AND MONTH(OrderDate) <= @CurrentMonth
  GROUP BY MONTH(OrderDate)
  ORDER BY MONTH(OrderDate);
END
{% endhighlight %}

With this procedure, the first column `SalesTotal` only adds up the `OrderPrice`
values if the order comes from the current year. The `ELSE` clause throws in
nulls for orders that don't match that criteria, effectively excluding them from
the calculation. The next column `LastYearSalesTotal` applies a similar logic.

The `WHERE` clause adds another restriction so it only includes the months
January up until the current month for both years. This ensures that each row
has something to compare to. However, by getting rid of the second comparison,
you could still get the remaining months of the previous year, which could be
useful if you want to figure out what to aim for!

The `CASE` function can be as complex as you need it to be, so if your criteria
differs from this example, play around with them until you get at the right
data. For example, I adjusted it to process the previous 12 months, rather than
working with calendar years. I then presented the results as a D3 graph on our
company intranet, similar to the graph below:

<svg id="monthlyJobSales" viewBox="0 0 750 450" class="svg-barchart img-fluid">
</svg>

The graph above then tries to convey as much information as possible without
overwhelming the user. Colour is used to highlight the best and worst months,
while the previous year's figures are overlaid as a discrete line. Hovering
over a bar give you a popover window displaying the actual figures. The version
I developed in work also makes each bar clickable, leading to a breakdown of
the figures by customer.

So from the seemingly simple task of retrieving year-on-year figures, the end
result is a utility that takes in a variety of disciplines, from SQL to data
visualisation.

