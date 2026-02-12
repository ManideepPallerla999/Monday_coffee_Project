# Monday Coffee Expansion SQL Project

![Company Logo](https://github.com/najirh/Monday-Coffee-Expansion-Project-P8/blob/main/1.png)

## Objective
The goal of this project is to analyze the sales data of Monday Coffee, a company that has been selling its products online since January 2023, and to recommend the top three major cities in India for opening new coffee shop locations based on consumer demand and sales performance.

## Key Questions
1. **Coffee Consumers Count**  
   How many people in each city are estimated to consume coffee, given that 25% of the population does?

2. **Total Revenue from Coffee Sales**  
   What is the total revenue generated from coffee sales across all cities in the last quarter of 2023?

3. **Sales Count for Each Product**  
   How many units of each coffee product have been sold?

4. **Average Sales Amount per City**  
   What is the average sales amount per customer in each city?

5. **City Population and Coffee Consumers**  
   Provide a list of cities along with their populations and estimated coffee consumers.

6. **Top Selling Products by City**  
   What are the top 3 selling products in each city based on sales volume?

7. **Customer Segmentation by City**  
   How many unique customers are there in each city who have purchased coffee products?

8. **Average Sale vs Rent**  
   Find each city and their average sale per customer and avg rent per customer

9. **Monthly Sales Growth**  
   Sales growth rate: Calculate the percentage growth (or decline) in sales over different time periods (monthly).

10. **Market Potential Analysis**  
    Identify top 3 city based on highest sales, return city name, total sale, total rent, total customers, estimated  coffee consumer
    

SELECT * FROM city ;  
SELECT * FROM products ;  
SELECT * FROM customers;  
SELECT * FROM sales;  
 



-- Q1 
-- how many people in each city are estimated to consume coffee, given that 25 % of population does ?

SELECT city_name,ROUND(population * 0.25/1000000,2) AS Pop_Cunsume_Coffee ,city_rank
FROM city
ORDER BY  population DESC;

-- Q2
-- what is the total revenue generated form the coffee sales across all the cities in the last quaryer of  2023?


SELECT Ci.city_name,SUM(S.total) AS Total_Sales
FROM sales AS S
LEFT JOIN  customers AS C
ON S.customer_id = C.customer_id
LEFT JOIN city AS Ci
ON C.city_id = Ci.city_id
WHERE S.sale_date > '2023-09-30' AND S.sale_date < '2024-01-01'
GROUP BY Ci.city_name
ORDER BY Total_Sales DESC
LIMIT 5;


-- Q3 
-- how many units of each coffee products have been sold?

SELECT P.product_name, COUNT(S.sale_id) AS Units_Sold
FROM  products AS P
LEFT JOIN sales AS S 
ON P.product_id = S.product_id
GROUP BY P.product_name 
ORDER BY Units_Sold DESC;


-- Q4
-- What is the average sales amount per customer in each city?

SELECT Ci.city_name, SUM( S.total) AS Total_Sales,
COUNT( DISTINCT S.customer_id) AS Total_Cus,
ROUND(
      SUM( S.total)  /COUNT( DISTINCT S.customer_id) ,2
      ) AS Avg_Sales
FROM  sales AS S
LEFT JOIN customers  AS C
ON S.customer_id = C.customer_id
LEFT JOIN city AS Ci
ON C.city_id = Ci.city_id
GROUP BY  Ci.city_name
ORDER BY Total_Sales DESC;

-- Q5 
-- Provide a list of cities along with their populations and estimated coffee consumers.

SELECT city_name,ROUND(population/1000000,2) AS Population,ROUND((population * 0.25)/1000000,2) AS Est_Population
FROM city
ORDER BY population DESC ;


-- Q6 
-- What are the top 3 selling products in each city based on sales volume?


SELECT Ci.city_name, P.product_name, COUNT(S.sale_id) AS TOTAL_ORDERS,
 DENSE_RANK() OVER(PARTITION BY Ci.city_name ORDER BY COUNT(S.sale_id) DESC ) AS rank_1
 FROM Sales AS S 
 INNER JOIN products AS P
 ON S.product_id = P.product_id 
 INNER JOIN customers AS C 
 ON C.customer_id = S.customer_id 
 INNER JOIN city AS Ci 
 ON Ci.city_id = C.city_id
 GROUP BY Ci.city_name,P.product_name
 ORDER BY Ci.city_name,TOTAL_ORDERS DESC;
 
 -- Q7
 -- How many uniqe customers are there in each city who have purchased coffee products?
 
SELECT Ci.city_name,
COUNT( DISTINCT S.customer_id) AS Total_Cus
FROM  sales AS S
LEFT JOIN customers  AS C
ON S.customer_id = C.customer_id
LEFT JOIN city AS Ci
ON C.city_id = Ci.city_id
LEFT JOIN products AS P
ON  P.product_id = S.product_id
WHERE  S.product_id <= 14
GROUP BY  Ci.city_name ;

-- Q8 
-- Find each city and their average sales per customer and avg rent per customer?

SELECT Ci.city_name, SUM( S.total) AS Total_Sales,
COUNT( DISTINCT S.customer_id) AS Total_Cus,
ROUND(
      SUM( S.total)  /COUNT( DISTINCT S.customer_id) ,2
      ) AS Avg_Sales, AVG(estimated_rent) AS Avg_rent
FROM sales AS S
INNER JOIN customers AS C
ON S.customer_id = C.customer_id
INNER JOIN  city AS Ci
ON C.city_id = Ci.city_id
GROUP BY Ci.city_name
ORDER BY AVG_Rent DESC;

-- Q9 
-- Sales growth rate: Calculate the percentage growth (or decline)  in sales over different time periods(monthly)

SELECT city_name,
EXTRACT(MONTH  FROM sale_date) AS Month_,
EXTRACT(YEAR  FROM sale_date) AS YEAR_,
SUM(S.total)
FROM sales AS S
INNER JOIN customers AS C
ON S.customer_id = C.customer_id
INNER JOIN  city AS Ci
ON C.city_id = Ci.city_id
GROUP BY city_name, YEAR_,Month_
ORDER BY city_name,YEAR_,Month_;

-- Q10
-- Identify top 3 city based on highest sales , return city name , total sales, total rent , total customers , estimated coffee consumers?

SELECT city_name,ROUND(population * 0.25,2) AS Pop_Cunsume_Coffee ,
SUM( S.total) AS Total_Sales, 
SUM(DISTINCT Ci.estimated_rent) AS Total_Rent, 
COUNT( DISTINCT S.customer_id) AS Total_Customers
FROM sales AS S
INNER JOIN customers AS C
ON S.customer_id = C.customer_id
INNER JOIN  city AS Ci
ON C.city_id = Ci.city_id
GROUP BY city_name,Pop_Cunsume_Coffee
ORDER BY Total_Sales DESC;


/*
-- Recomendation
City 1 - Pune
1.Rent is modrate 
2.sales are good 
3.rent to customer ratio is good

City 2 - Delhi 
1.Highest est coffee cunsumers
2.high no of customes which is 68
3.rent to customer ratio is good

City 3 - jaipur
1.Highest No of customers which is 69
2.rent to customer ratio is Excellent
3.Sales are good
