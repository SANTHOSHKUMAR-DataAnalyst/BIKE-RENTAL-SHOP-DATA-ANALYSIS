select * from customer;
select * from bike;
select * from rental;
select * from membership;
select * from membership_type;

/* 1. Santy would like to know how many bikes the shop owns by category. Can
you get this for her? 
Display the category name and the number of bikes the shop owns in
each category (call this column number_of_bikes ). Show only the categories
where the number of bikes is greater than 2 .  */

select category 
from bike
group by 1
Having count(model) > 2;

/*2. Santy needs a list of customer names with the total number of
memberships purchased by each.
For each customer, display the customer's name and the count of
memberships purchased (call this column membership_count ). Sort the
results by membership_count , starting with the customer who has purchased
the highest number of memberships. Keep in mind that some customers may not have purchased any
memberships yet. In such a situation, display 0 for the membership_count .*/

select name
,count(mem.id) membership_count
from customer cust 
left join membership mem 
on cust.id = mem.customer_id
group by 1
order by 2 DESC;

/* 3. Santy is working on a special offer for the winter months. Can you help her
prepare a list of new rental prices?
For each bike, display its ID, category, old price per hour (call this column 
old_price_per_hour ), discounted price per hour (call it new_price_per_hour ), old
price per day (call it old_price_per_day ), and discounted price per day (call it
new_price_per_day ).
Electric bikes should have a 10% discount for hourly rentals and a 20%
discount for daily rentals. Mountain bikes should have a 20% discount for
hourly rentals and a 50% discount for daily rentals. All other bikes should
have a 50% discount for all types of rentals.
Round the new prices to 2 decimal digits.   */

with discount as (
select model,
id,
category,
price_per_hour as old_price_per_hour,
case when category = 'electric' then price_per_hour * 10/100 
     when category = 'mountain bike' then price_per_hour * 20/100 
     else price_per_hour * 50/100 end as hourly_discount,
case when category = 'electric' then price_per_day * 20/100 
     when category = 'mountain bike' then price_per_day * 50/100 
     else price_per_day * 50/100 end as day_discount,    
price_per_day as old_price_per_day
from bike
)
select model,
id,
category,
old_price_per_hour,
round(old_price_per_hour - hourly_discount,2) as new_price_per_hour,
old_price_per_day,
round(old_price_per_day - day_discount,2) as new_price_per_day
from discount;

/* 4. Santy is looking for counts of the rented bikes and of the available bikes in
each category.
Display the number of available bikes and the number of rented bikes by bike category. */

select category 
,count(case 
		 when status = 'available' 
			then 1 end) as available_bikes_count
,count(case 
		when status = 'rented' 
            then 1 end) as rented_bikes_count
from bike 
group by 1;

/* 5. Santy is preparing a sales report. he needs to know the total revenue
from rentals by month, the total by year, and the all-time across all the
years. 
Display the total revenue from rentals for each month, the total for each
year, and the total across all the years.
Sort the results chronologically. Display the year total after all the month
totals for the corresponding year. Show the all-time total as the last row */

select year(start_timestamp) as year
,month(start_timestamp) as month
,sum(total_paid) as total_revenue
from rental
group by 1,2 with rollup
order by (year is null),1,2;

/* This part uses a boolean expression (year IS NULL) to check if year is NULL.
When year IS NULL is TRUE , it is treated as 1 in sorting.
When year IS NULL is FALSE , it is treated as 0.
Since 0 comes before 1 in ascending order, 
this expression places all rows where year is not NULL (regular rows) 
before the row where year is NULL (the grand total row).
*/


/* 6. Santy has asked you to get the total revenue from memberships for each
combination of year, month, and membership type.
Display the year, the month, the name of the membership type , and
 the total revenue for every combination of year, month, and membership type.
Sort the results by year, month, and name of membership type. */

select year(m.start_date) as year
,month(m.start_date) as month
,mt.name
,sum(m.total_paid) as total_revenue
from membership m 
join membership_type mt 
on m.membership_type_id = mt.id 
group by 1,2,3
order by 1,2,3;

/* 7. Next, Santy would like data about memberships purchased in 2023, with
subtotals and grand totals for all the different combinations of membership
types and months.
Display the total revenue from memberships purchased in 2023 for each
combination of month and membership type. Generate subtotals and
grand totals for all possible combinations. There should be 3 columns: 
membership_type_name , month , and total_revenue .
Sort the results by membership type name alphabetically and then 
chronologically by month. */

select mt.name as membership_type_name 
,month(m.start_date) as month
,sum(m.total_paid) as total_revenue
from membership as m 
join membership_type as mt 
on m.membership_type_id = mt.id
where year(m.start_date) = '2023'
group by 1,2 with rollup
order by 1, 2;  

/* 8. Now it's time for the final task.
Santy wants to segment customers based on the number of rentals and
see the count of customers in each segment.
Categorize customers based on their rental history as follows:
Customers who have had more than 10 rentals are categorized as 'more
than 10' .
Customers who have had 5 to 10 rentals (inclusive) are categorized as 
'between 5 and 10' .
Customers who have had fewer than 5 rentals should be categorized as
'fewer than 5' .
Calculate the number of customers in each category. Display two columns: 
rental_count_category (the rental count category) and customer_count (the
number of customers in each category). */

with customers_based_on_number_of_rentals as 
	(
		select customer_id ,count(bike_id) as total_count_bike
		from rental as r , bike as b 
		where r.bike_id = b.id
		group by 1
	)		
select case when total_count_bike > 10 then 'more than 10' 
            when total_count_bike between 5 and 10 then 'between 5 and 10 '
            else 'fewer than 5' end as rental_count_category
,count(customer_id) as customer_count
from customers_based_on_number_of_rentals 
group by 1
order by 2;

the followings are tables 

CREATE TABLE `bike` (
  `id` int NOT NULL,
  `model` varchar(50) DEFAULT NULL,
  `category` varchar(50) DEFAULT NULL,
  `price_per_hour` decimal(10,0) DEFAULT NULL,
  `price_per_day` decimal(10,0) DEFAULT NULL,
  `status` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE TABLE `customer` (
  `id` int NOT NULL,
  `name` varchar(30) DEFAULT NULL,
  `email` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE TABLE `membership` (
  `id` int NOT NULL,
  `membership_type_id` int DEFAULT NULL,
  `customer_id` int DEFAULT NULL,
  `start_date` date DEFAULT NULL,
  `end_date` date DEFAULT NULL,
  `total_paid` decimal(10,0) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE TABLE `membership_type` (
  `id` int NOT NULL,
  `name` varchar(50) DEFAULT NULL,
  `description` varchar(500) DEFAULT NULL,
  `price` decimal(10,0) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE TABLE `rental` (
  `id` int NOT NULL,
  `customer_id` int DEFAULT NULL,
  `bike_id` int DEFAULT NULL,
  `start_timestamp` timestamp NULL DEFAULT NULL,
  `duration` int DEFAULT NULL,
  `total_paid` decimal(10,0) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

