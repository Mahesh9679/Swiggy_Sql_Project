# Swiggy_Sql_Project
# Swiggy Data Analysis - A Food Delivery Company

![swiggy Logo](https://github.com/Mahesh9679/Swiggy_Sql_Project/blob/main/swiggy-logo%20(1).png)

## Overview

This project demonstrates my SQL problem-solving skills through the analysis of data for Swiggy, a popular food delivery company in India. The project involves setting up the Database, Importing data, Handling null values, and Solving a variety of business problems using complex SQL queries, Creating Stored Procedures, Query Optimization and generatiing Insights and Actionable recommendation.

## Project Structure

- **Database Setup:** Creation of the `Swiggy_db` database and the required tables.
- **Data Import:** Inserting sample data into the tables.
- **Data Cleaning:** Handling null values and ensuring data integrity.
- **Business Problems:** Solving 20 specific business problems using SQL queries.
- **Stored Procedure:** Creating Stored Procedures

## ERD
![ERD](https://github.com/Mahesh9679/Swiggy_Sql_Project/blob/main/ER_Diagram.png)

## Database Setup
```sql
CREATE DATABASE Swiggy_db;
```
-- 1. Creating Tables
```sql
create table customers (
    Customer_id int primary key,
    Customer_name varchar(30),
    Reg_date date
);

Create table restaurants (
	Restaurant_id int primary key,
    Restaurant_name varchar(55),
    City varchar(30),
    Opening_hours varchar(55)
);
create table orders(
    Order_id int primary key,
    Customer_id int ,
    Restaurant_id int,
    Order_item varchar(55),
    Order_date date,
    Order_time time,
    Order_status varchar(30),
    Total_amount float,
    foreign key (Customer_id) references customers (Customer_id),
    foreign key (Restaurant_id) references restaurants (restaurant_id)
);

create table riders(
    Rider_id int primary key,
    Rider_name varchar(30),
    Sign_up date
);
create table deliveries(
    delivery_id int primary key,
    order_id int ,
    delivery_status varchar(35),
    delivery_time time,
    rider_id int,
    foreign key (order_id) references orders(order_id),
    foreign key (rider_id) references riders(rider_id)
);
```
## Data Import

## Data Cleaning and Handling Null Values
```sql
select * from customers
where customer_name is null or reg_date is null;

select * from restaurants
where Restaurant_name is null or city is null or Opening_hours is null ;

select * from orders
where customer_id is null or Restaurant_id is null or Order_item is null
or order_date is null  or order_time is null or Order_status is null  or
Total_amount is null  ;

select * from riders
where rider_name  is null or sign_up is null;

select * from deliveries
where order_id  is null or delivery_status is null or delivery_time is null or rider_id is null;
```
## Feature Engineering
```sql
set sql_safe_updates=0;

Alter table orders 
add column Time_of_Day varchar(20);

Update orders 
set time_of_day =(
                 case 
                    when order_time between "00:00:00" and "12:00:00" then "Morning"
					when order_time between "12:01:00" and "16:00:00" then "Afternoon"
					else "Evening"
				  end );
                  
Alter table deliveries
add column Time_of_Day varchar(20);

Update deliveries 
set time_of_day =(
                 case 
                    when delivery_time between "00:00:00" and "12:00:00" then "Morning"
					when delivery_time between "12:01:00" and "16:00:00" then "Afternoon"
					else "Evening"
				  end );
 ```     
## Business Problems Solved

-- 1.Popular Time Slots
-- Identify the time slots during which the most orders are placed, based on 2-hour intervals.
```sql
select 
case when hour(order_time) between 0 and 1 then "00:00:00 AM-02:00:00AM"
		  when hour(order_time) between 2 and 3 then "02:00:00 AM-04:00:00 AM"
          when hour(order_time) between 4 and 5 then "04:00:00 AM-06:00:00 AM"
          when hour(order_time) between 6 and 7 then "06:00:00 AM-08:00:00 AM"
		  when hour(order_time) between 8 and 9 then "08:00:00 AM-10:00:00 AM"
          when hour(order_time) between 10 and 11 then "10:00:00 AM-12:00:00 PM"
          when hour(order_time) between 12 and 13 then "12:00:00 PM-02:00:00 PM"
          when hour(order_time) between 14 and 15 then "02:00:00 PM-04:00:00 PM"
          when hour(order_time) between 16 and 17 then "04:00:00 PM-06:00:00 PM"
          when hour(order_time) between 18 and 19 then "06:00:00 PM-08:00:00 PM"
          when hour(order_time) between 20 and 21 then "08:00:00 PM-10:00:00 PM"
          when hour(order_time) between 22 and 23 then "10:00:00 PM-00:00:00 AM"
		end as time_slot,
		count(order_id) as Orders_count from orders
group by time_slot 
order by Orders_count desc ;

 ```                  
## Business
-- High-Value Customers
-- 2.List the customers who have spent more than 3000 in total on food orders. -- Return: customer_name, customer_id.

```sql
select customer_id, customer_name ,
round(sum(total_amount),2) as Total_order_amt
from customers 
join orders using (customer_id) 
group by customer_id, customer_name 
having Total_order_amt >3000 
order by Total_order_amt desc ;
``` 
-- Most Popular Dish by City
-- 3. Identify the most popular dish in each city based on the number of orders.
```sql
select city, Order_item ,count(order_id) as No_of_orders
from restaurants_india
join orders using (Restaurant_id) 
group by city,Order_item
order by  No_of_orders desc ;

``` 
 -- 4. Q15. Order Frequency by Day
-- Analyze order frequency per day of the week and identify the peak day for each restaurant.
```sql
select restaurants_india.Restaurant_id,Restaurant_name,
count(order_id) as Total_orders,
 dayname(order_date) as Day_of_week 
from orders  join restaurants_india
on orders.Restaurant_id=restaurants_india.Restaurant_id
where order_status="Delivered"
group by restaurants_india.Restaurant_id,Day_of_week
order by Total_orders desc ;

``` 
-- 5. Ranking City by Revenue
-- Rank each city based on the total revenue for the last year (2024)
```sql
select city , round(sum(Total_amount),2) as Total_revenue ,
row_number() over (order by sum(Total_amount) desc ) as city_Ranking
from restaurants_india
join orders using (Restaurant_id)
where year(Order_date)="2024"
group by city;

```
-- 6. Order Item Popularity
-- Track the popularity of specific order items over time and identify seasonal demand spikes.
```sql
 select order_item,count(order_id) as Total_orders,
    case 
       when month(order_date) between 3 and 5 then "Spring"
       when month(order_date) between 6 and 8 then "Summer"
       when month(Order_date) between 9 and 11 then "Autumn"
       else "Winter"
       end as Seasons
from orders 
where order_status="Delivered"
group by order_item,Seasons
order by Total_orders desc;

```
-- 7.  Restaurant Revenue Ranking
-- Rank restaurants by their total revenue from the last 1 year within their city
```sql
select Restaurant_id , Restaurant_name , City ,
round(sum(Total_amount),2) as Revenue , 
dense_rank() over (partition by city 
order by sum(Total_amount) desc )as Ranking
from restaurants_india 
join orders using (Restaurant_id) 
where year(order_date)=year(current_date())-1
group by Restaurant_id , Restaurant_name ,City;

```
-- 8. Order Value Analysis
-- Find the average order value (AOV) per customer who has placed more than 5 orders. 
```sql
select customer_id ,customer_name , 
round(avg(Total_amount),2) as AOV ,
count(order_id) as total_orders from orders
join customers using (customer_id)
group by customer_id,customer_name 
having total_orders >5;

```
## Stored Procedure

-- 9. create a stored procedure to show the full order history of a customer sorted by most recent-- 
-- return customer name, order id, restaurant name, order date, Total amount
```sql
delimiter //
create procedure customer_details(in customer_id int )
begin 
select Customer_name, order_id, Restaurant_name,
Order_item, order_date,Total_amount
from customers c
join orders using (Customer_id)
join restaurants_india using (Restaurant_id)
where c.Customer_id = customer_id
order by order_date desc ;
end //
delimiter //

call customer_details(1187);
```

-- Extra Sample Questions Solved

-- Orders Without Delivery
-- Write a query to find orders that were placed but not delivered. -- Return: restaurant_name, city, and the number of not delivered orders.
```sql
select R.Restaurant_id ,R.restaurant_name , R.city , sum(case when delivery_status="Cancelled" then 1 else 0 end ) as Not_delivered_orders
from restaurants_india R 
join orders using (Restaurant_id )
join deliveries using ( Order_id )
where Order_status ="Placed" and delivery_status ="Cancelled"
group by R.Restaurant_id ,restaurant_name , city 
order by Not_delivered_orders ;

```
-- Restaurant Revenue Ranking
-- Rank of restaurants by their total revenue from the last 1 year. -- Return: restaurant_name, total_revenue, and their rank within their city.
```sql
select restaurant_name ,city, round(sum(Total_amount),2) as total_revenue , 
row_number() over (order by sum(Total_amount) desc) as ranking
from restaurants_india 
join orders using (Restaurant_id)
where year(order_date)=year(current_date())-1
group by restaurant_name ,city ;

```
-- Customer Churn
-- Find customers who haven’t placed an order in 2024 but did in 2025.
```sql
select distinct customer_id,customer_name from customers
join orders using (customer_id)
where year(order_date)=2025 and customer_id not in 
(select customer_id  from customers
join orders using (customer_id)
where year(order_date)=2024)
group by customer_id,customer_name;

```
-- Late deliveries:
-- Find all orders where the delivery time was more than 60 minutes after the order time .
```sql
SELECT 
    o.Order_id,
    o.Order_time,
    d.delivery_time,
    TIMESTAMPDIFF(MINUTE, o.Order_time, d.delivery_time) AS delivery_delay_minutes
FROM orders o
JOIN deliveries d ON o.Order_id = d.order_id
WHERE TIMESTAMPDIFF(MINUTE, o.Order_time, d.delivery_time) > 60;

```

-- Find if there’s a correlation between order value and delivery delay (create bins of Total_amount and analyze average delay).
```sql
SELECT 
    CASE 
        WHEN Total_amount BETWEEN 0 AND 100 THEN '0-100'
        WHEN Total_amount BETWEEN 101 AND 200 THEN '101-200'
        WHEN Total_amount BETWEEN 201 AND 300 THEN '201-300'
        WHEN Total_amount BETWEEN 301 AND 400 THEN '301-400'
        WHEN Total_amount > 400 THEN '400+'
        ELSE 'Unknown'
    END AS amount_range,
    COUNT(*) AS total_orders,
    ROUND(AVG(timestampdiff(MINUTE, Order_time, delivery_time)), 2) AS avg_delay_minutes
FROM orders
join deliveries using (order_id)
WHERE delivery_time IS NOT NULL
GROUP BY amount_range
ORDER BY 
    amount_range desc;

```
-- Top 5 Most Frequently Ordered Dishes
-- Write a query to find the top 5 most frequently ordered dishes from the last 1 year.
```sql
select  order_item , count(*) as counts from orders
where  year(order_date)>= year(current_date())-1
group by Order_item
order by counts desc ;
 
```
## Key Insights 

 • Peak Ordering Time: Most orders are placed between 4 PM– 6 PM, while 2 AM – 4 AM sees minimal activity.

 • Top Customers: Customer id 1187,1869 placed over 5  orders each with an average order value of ₹400–₹600.

 • Seasonal Trends: Winter has the highest dish sales, while Spring has the lowest, indicating seasonal food demand.
 
 • Top-Selling Items by City: Dishes like vada Pav ,Pani puri , Fish Fry , Mutton curry and Masala Dosa are best-sellers across major cities.
 
 • High Non-Deliveries: Restaurant id 573,1690,1884 show the highest non-delivery rates.

 
## Actionable Recommendations

 • Introduce Late-Night Offers: Encourage more orders during 2 AM – 4AM with special discounts or combos.
 
 • Reward Loyal Customers: Launch a VIP program or exclusive deals for high-frequency users to boost retention
 
 • Plan Seasonal Campaigns: Launch season-specific menus or offers during Winter, and create Spring discounts to balance demand.
 
 • Promote Best-Selling Dishes: Use regional best-sellers in targeted ads and bundled offers to drive repeat sales.
 
 • Improve Restaurant Reliability: Audit and support restaurants with high non-deliveryrates to reduce cancellations.
 
## Conclusion

This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems in the context of a food delivery service like Swiggy. The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

## Notice 
All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with Swiggy or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.


