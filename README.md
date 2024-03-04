# Walmart-Sales-Analysis---SQL_t
This repository contains a collection of SQL scripts demonstrating various data analysis tasks, including adding new columns, exploring data characteristics, and performing advanced analytical queries using SQL window functions, CTE's

Welcome to the SQL Data Analysis Project repository! This repository aims to provide a hands-on approach to SQL data analysis, offering a diverse collection of SQL scripts tailored to address different data analysis challenges. Below is an overview of the contents and how to navigate through them:

Data Exploration and Transformation:
The initial scripts focus on adding new columns to the dataset to enhance its analytical capabilities. These include adding time-related columns like "time_of_date," "day_of_the_week," and "month," which enable time-based analysis.

Basic Analysis:
Next, we delve into basic analysis tasks such as identifying unique values in columns, determining the number of transactions per customer type, and exploring payment methods and customer demographics.
Advanced Analysis with Window Functions:
The core of this repository lies in the advanced analytical queries using SQL window functions. These queries tackle complex analysis tasks, including ranking, aggregation, and partitioning data to derive meaningful insights.
Summary and Conclusions:
Finally, we wrap up with summary queries, which provide aggregated insights such as total revenue by month, most common payment methods, and highest average VAT by product line.




select * from SalesData



----------------------------------------------------------------------------------------------------------
--ADDING TIME OF THE DAY COLUMNS

SELECT Time,
(CASE 
WHEN Time between '1899-12-30 00:00:00.000' AND '1899-12-30 12:00:00.000' THEN 'MORNING'
WHEN Time between '1899-12-30 12:00:00.000' AND '1899-12-30 16:00:00.000' THEN 'AFTERNOON'
WHEN Time between '1899-12-30 16:00:00.000' AND '1899-12-30 23:59:00.000' THEN 'EVENING'
END ) AS time_of_date
from dbo.SalesData;

select * from dbo.SalesData

ALTER TABLE SalesData 
ADD time_of_date varchar(50)

update SalesData
SET time_of_date =
(CASE 
WHEN Time between '1899-12-30 00:00:00.000' AND '1899-12-30 12:00:00.000' THEN 'MORNING'
WHEN Time between '1899-12-30 12:00:00.000' AND '1899-12-30 16:00:00.000' THEN 'AFTERNOON'
WHEN Time between '1899-12-30 16:00:00.000' AND '1899-12-30 23:59:00.000' THEN 'EVENING'
END ) 
from dbo.SalesData;


-----------------------------------------------------------------------------------------------------------
--ADDING DAY_OF_THE_WEEK COLUMN
select  TOP 100 * from SalesData
where time_of_date LIKE 'MORNING'

SELECT Date,DATENAME(dw, Date) as Day_of_the_week FROM SalesData 

Alter table SalesData
Add Day_of_the_week varchar(20);

update SalesData
Set Day_of_the_week =  DATENAME(dw, Date) 


---------------------------------------------------------------------------------------------------------
--ADDING A MONTH COLUMN

select datename(mm,Date) from SalesData

Alter table SalesData
Add Month varchar(20);

update SalesData
Set Month =  DATENAME(MM, Date) 

select * from SalesData


-----------------------------------------------------------------------------------------------------
--NUMBER OF UNIQUE CITIES?
select distinct city
from SalesData


-------------------------------------------------------------------------------------------------------
--WHICH BRANCH IS IN EACH CITY?
select  distinct Branch, City
from SalesData

---------------------------------------------------------------------------------------------------
-- NO OF UNIQUE PRODUCT_LINES ?
select distinct [Product line]
from SalesData


----------------------------------------------------------------------------------------------------
--UNIQUE PRODUCT LINES
Distinct count of unique product lines
select  Count(DISTINCT [Product line])
from SalesData

--------------------------------------------------------------------------------------------------------
--MOST COMMON PAYMENT METHOD
select  Payment , count( Payment) as no_of_transactions
from SalesData
group by Payment
order by no_of_transactions desc

---------------------------------------------------------------------------------------------------------
--WHAT IS THE MOST SELLING PRODUCT LINE?

SELECT  TOP 1 [Product line], Count([Product line])
FROM SalesData
group by [Product line]
order by [Product line] desc

----------------------------------------------------------------------------------------------------------
--WHAT IS THE TOTAL REVENUE BY MONTH

SELECT [Month], round(sum([gross income]),0) as total_revenue
FROM SalesData
group by [Month]
order by total_revenue desc

-----------------------------------------------------------------------------------------------------------
--WHAT MONTH HAD THE LARGEST COGS?

SELECT [Month], round(sum(cogs),0) as total_cogs
from SalesData
group by [Month]
order by total_cogs desc



------------------------------------------------------------------------------------------------------------
--WHAT PRODUCT LINE HAS THE LARGEST REVENUE?

SELECT TOP 1 [Product line] , round(sum([gross income]),0) as largest_revenue
FROM SalesData
group by  [Product line]
order by largest_revenue desc

--------------------------------------------------------------------------------------------------------------
-- WHAT IS THE CITY WITH LARGEST REVENUE?
SELECT TOP 1 City , round(sum([gross income]),0) as largest_revenue
FROM SalesData
group by  City
order by largest_revenue desc


--------------------------------------------------------------------------------------------------------------
--WHAT PRODUCT LINE HAS THE HIGHEST VAT?

SELECT  TOP 1 [Product line] , ROUND(AVG([tax 5%]),2) as HIGHEST_AVERAGE_VAT
FROM SalesData
group by  [Product line] 
order by HIGHEST_AVERAGE_VAT desc

---------------------------------------------------------------------------------------------------------------
WITH Average_revenue AS (
    SELECT AVG([Unit price]) as avg_unit_price FROM SalesData
), 
product_line_average_revenue AS (
    SELECT [Product line], AVG([Unit price]) as avgrev 
    FROM SalesData 
    GROUP BY [Product line]
)
SELECT [Product line], 
    (CASE 
        WHEN pl.avgrev > ar.avg_unit_price THEN 'Good'
        ELSE 'Bad'
    END) AS CASING 
FROM product_line_average_revenue pl
CROSS JOIN Average_revenue ar;



----------------------------------------------------------------------------------------------------------
-- LIST OF BRANCHES SUM OF QUANTITY IS GREATER THAN AVERAGE QUANTITY
select Branch from(
SELECT Branch, sum(Quantity) as sum_of_quantity
From SalesData
group by Branch
Having sum(Quantity) > (SELECT AVG(Quantity) from SalesData)
) as subquery



------------------------------------------------------------------------------------------------------------
--WHAT IS THE MOST COMMON PRODUCT LINE BY GENDER

SELECT Gender, [Product line], count(*) as frequency
from SalesData
group by Gender, [Product line]
order by  frequency desc

------------------------------------------------------------------------------------------------------------------
--WHAT IS THE AVERAGE RATING OF EACH PRODUCT LINE

SELECT [Product line], round(AVG(Rating),1) as average_rating
from SalesData
Group by [Product line]
order by average_rating desc


----------------------------------------------------------------------------------------------------------------
--SALES ANALYSIS

Number of sales made in each time of the day per weekday

select  Day_of_the_week,time_of_date, count([Invoice ID]) as number_of_sales
From SalesData
where Day_of_the_week =  'Friday'
group by time_of_date, Day_of_the_week
order by Day_of_the_week


-----------------------------------------------------------------------------------------------------------
--WHICH OF THE CUSTOMER TYPE BRINGS MOST REVENUE



SELECT [Customer type], format(sum(Total),'N', 'en-US') as revenue
FROM SalesData
Group by [Customer type]
order by sum(Total) desc

--------------------------------------------------------------------------------------------------------------
--Which city has the largest tax percent/ VAT (Value Added Tax)?



SELECT City, Format(AVG( [Tax 5%]), 'N', 'en-US') as VAT
FROM SalesData
group by City
Order by VAT Desc


------------------------------------------------------------------------------------------------------------
-- Which customer type pays the most in VAT?



SELECT [Customer type], FORMAT(AVG([Tax 5%]),'N', 'en-US') as VAT
FROM SalesData
Group by [Customer type]
Order by VAT Desc


-----------------------------------------------------------------------------------------------------------------
--CUSTOMER ANALYSIS
--How many unique customer types does the data have?

SELECT Distinct [Customer type]
FROM SalesData

------------------------------------------------------------------------------------------------------------
--How many unique payment methods does the data have?

SELECT DISTINCT Payment
FROM SalesData


------------------------------------------------------------------------------------------------------------
--What is the most common customer type and Buys the most?
SELECT TOP 1 [Customer type], Count([Invoice ID]) as transactions
FROM SalesData
GROUP BY [Customer type]
Order by transactions desc

------------------------------------------------------------------------------------------------------------
--What is the gender of most of the customers?
SELECT TOP 1 Gender, Count([Customer type]) as Customers_count
FROM SalesData
GROUP BY Gender
Order by Customers_count desc

------------------------------------------------------------------------------------------------------------
--What is the gender distribution per branch?
SELECT  Gender, count(*)  as gender_count
FROM SalesData
where Branch = 'C'
group by Gender

------------------------------------------------------------------------------------------------------------
--Which time of the day do customers give most ratings?

Select time_of_date, count(Rating) as Rating_count
FROM SalesData
group by time_of_date
order by Rating_count DESC


------------------------------------------------------------------------------------------------------------
--Which time of the day do customers give most ratings per branch?
WITH CTE AS (
Select time_of_date, Branch, Count(Rating) as No_of_rating,
Rank() over (partition by Branch order by Count(Rating) desc) as Ranking
FROM SalesData
group by time_of_date, Branch ) 

SELECT time_of_Date, Branch, No_of_rating, Ranking
FROM CTE
where Ranking = 1


------------------------------------------------------------------------------------------------------------
--Which day fo the week has the best avg ratings?


Select top 3 Day_of_the_week, Round(AVG(Rating),2) as Average_Rating
From SalesData
Group by Day_of_the_week
order by Average_Rating desc

------------------------------------------------------------------------------------------------------------
-- Which day of the week has the best average ratings per branch?
WITH CTE AS (
SELECT Day_of_the_week, Branch, round(Avg(Rating),2) as Average_Rating,
Rank() over (partition by branch order by round(Avg(Rating),2)  desc) as Ranking
FROM SalesData
group by Day_of_the_week, Branch) 

SELECT Day_of_the_week, Branch, Average_Rating, Ranking
FROM CTE
Where Ranking =1



































