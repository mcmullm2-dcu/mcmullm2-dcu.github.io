---
layout: post
title: Year-on-year queries
tags: [sql,d3,data visualisation]
includes: [d3]
localscripts: [yearonyear]
---
A common requirement for most organisations is to pick some metric (say, sales
totals) and view them over time. This is a great way to spot trends, but
year-on-year figures go a little further by allowing you to compare one year's
figures with the same period from the previous year. In this post, I'll go
through one way that I approached it for our company intranet.

## SQL Queries
SQL is a great tool for aggregating data. Let's start by looking at a typical
SQL query for grouping and summing sales totals by month. This example is
presented as a stored procedure for MS SQL Server, but the general idea is
similar across most major platforms:

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

This code assumes there is a table called `OrderDetails` which contains (among
others) columns called `OrderDate` and `OrderPrice`.

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
the calculation. The next column `LastYearSalesTotal` applies a similar logic to
the previous year.

The `WHERE` clause adds another restriction so it only includes the months
January up until the current month for both years. This ensures that each row
has something to compare to. However, by getting rid of the second comparison,
you could still get the remaining months of the previous year, which could be
useful if you want to figure out what to aim for!

The `CASE` function can be as complex as you need it to be, so if your criteria
differs from this example, play around with them until you get at the right
data. For example, I adjusted it to process the previous 12 months, rather than
working with calendar years.

## Visualising the data
Once the SQL query was working correctly, I added it to our internal API. This
uses ASP.NET Core, and outputs the data as JSON. Using the
[D3.js](https://d3js.org/) library, I presented the results in a browser like
this:

<svg id="monthlyJobSales" viewBox="0 0 750 450" class="svg-barchart img-fluid">
</svg>

The graph above then tries to convey as much information as possible without
overwhelming the user. Colour is used to highlight the best and worst months,
while the previous year's figures are overlaid as a discrete line. Hovering
over a bar give you a popover window displaying the actual figures. The version
I developed in work also makes each bar clickable, leading to a breakdown of
the figures by customer.

Choosing a histogram seems like an odd choice at first: time-based data is
often presented as a line graph. I tested both with my colleagues and this
version won out for a variety of reasons. Firstly, the previous year's figures
are secondary information, so the contrast is more pronounced by making this
year's figures a physically larger area. Secondly, the colour-coding is easier
to see at a glance. Finally, because the bars are clickable, mobile users found
it more convenient to use, again because of the larger area.

## Conclusion
From the seemingly simple task of retrieving year-on-year figures, the end
result is a utility that takes in a variety of disciplines, from SQL to data
visualisation. I've applied the same process to other metrics in the company
too, particularly the number of orders per month. For us, this is an important
figure to track, as it directly affects both our staffing and automation
requirements. I hope you can apply similar techniques to your organisation!

