# Zomato Data Analysis Project - A SQL Based Business Intelligence Exploration

![](https://github.com/ArunJp0/Zomato_Sql/blob/main/zomato-white-logo.png)
![](https://github.com/ArunJp0/Zomato_Sql/blob/main/Zom-1.PNG)
![](https://github.com/ArunJp0/Zomato_Sql/blob/main/Zom-2.PNG)
![](https://github.com/ArunJp0/Zomato_Sql/blob/main/Zom-3.PNG)
![](https://github.com/ArunJp0/Zomato_Sql/blob/main/Zom-4.PNG)

## Project Overview

- **Project Title** : Zomato Data Analysis
- **Level** : Advanced
- **Database** : zomato_db

This project is a comprehensive data analysis of Zomato's food delivery platform. It focuses on evaluating customer behavior, restaurant performance, rider efficiency, seasonal trends, and order patterns using SQL for data querying and Power BI for data visualization. The goal is to extract actionable business insights that can help improve customer experience, optimize operations, and support strategic decisions for Zomato's growth.

## Tools & Technologies Used

- **SQL (MySQL Workbench)** : Data cleaning, transformation, and advanced querying.
- **Power BI** : Data visualization and dashboard building
- **Excel** : Initial data cleaning and NULL handling.
- **GitHub** : Version control and project documentation.

## Project Structure

- **Database Creation**: The project starts by creating a database named zomato_db.
- **Data Import**: Inserting data into the tables.
- **Data Cleaning** : Handling null values and ensuring the data accuracy.
- **Business Problems**: Solving 25 specific business problems using SQL queries.
- **Stored Procedure**: Creating Stored Procedures to save reusable SQL code for repeated use.
- **Query Optimization**: Optimized the queries using cte's, index and used joins.

## Entity Relationship Diagram
![](https://github.com/ArunJp0/Zomato_Sql/blob/main/Zomato_ER.PNG)

## Database and Table Creation

```sql

create database zomato_db;

create table customers (
     customer_id int primary key,
     customer_name varchar(25),
     reg_date date
);

create table restaurants (
     restaurant_id int primary key,
     restaurant_name varchar(55), 
     city varchar(20),
     opening_hours varchar(55)
);

create table orders (
     order_id int primary key,
     customer_id int,
     restaurant_id int,
     order_item varchar(55),
     order_date date,
     order_time time,
     order_status varchar(55),
     total_amount float
);

-- fk constraints 
alter table orders
add constraint fk_customers
foreign key (customer_id)
references customers(customer_id);

-- fk constraints 
alter table orders
add constraint fk_restaurants
foreign key (restaurant_id)
references restaurants(restaurant_id);

create table riders (
	 rider_id int primary key,
     rider_name varchar(55),
     sign_up date
);

create table deliveries (
	 delivery_id int primary key,
     order_id int,
     delivery_status varchar(55),
     delivery_time time,
     rider_id int
);

-- fk constraints 
alter table deliveries
add constraint fk_orders
foreign key (order_id)
references orders(order_id);

-- fk constraints 
alter table deliveries
add constraint fk_riders
foreign key (rider_id)
references riders(rider_id);

```

## Data Cleaning

```sql

select * from customers
where customer_id is null or customer_name is null or reg_date is null;

select * from restaurants
where restaurant_id is null or restaurant_name is null or city is null or opening_hours is null;

select * from orders
where order_id is null or customer_id is null or restaurant_id is null or order_item is null
or order_date is null  or order_time is null or order_status is null  or
total_amount is null;

select * from riders
where rider_id is null or rider_name is null or sign_up is null;

select * from deliveries
where delivery_id is null or order_id is null or delivery_status is null or delivery_time is null or rider_id is null;

```

## Enhancements

**Based on order time assigning 'traffic conditions' and then based on that assigning 'estimated delivery time'.**
```sql
      alter table orders
	    add column traffic_conditions varchar(10),
      add column estimated_delivery_time_status varchar(20);
      
      update orders                                                                                                     
      set traffic_conditions = case
      when order_time between '07:00:00' and '10:00:00' then 'High'
      when order_time between '12:00:00' and '14:00:00' then 'Medium'
      else 'Low'
      end;
      
      update orders
      set estimated_delivery_time_status = case
      when traffic_conditions = 'High' then '15mins delay'
      when traffic_conditions = 'Medium' then '10mins delay'
      else 'on-time'
      end;
);
```

## Used Stored Procedure for Two Use Cases

1. **Get Restaurant Performance Summary for a Given Year.**
   ```sql
   delimiter //
   create procedure get_restaurant_summary_by_year(in input_year int)
   begin
    select r.restaurant_id, r.restaurant_name, count(o.order_id) as total_orders,        
    round(sum(o.total_amount), 2) as total_sales,
    round(avg(o.total_amount), 2) as avg_order_value
    from restaurants r
    join orders o on r.restaurant_id = o.restaurant_id
    where year(o.order_date) = input_year
    group by r.restaurant_id, r.restaurant_name
    order by total_sales desc;
   end //
   delimiter;
   call get_restaurant_summary_by_year(2023);
   ```

2. **Identify Top N Customers in a Given Month & Year.**
   ```sql
   delimiter //
   create procedure get_top_customers_by_month (
    in input_month int,
    in input_year int,
    in limit_n int )
   begin
    select c.customer_id, c.customer_name, count(o.order_id) as total_orders,
    round(sum(o.total_amount), 2) as total_spent,
    round(avg(o.total_amount), 2) as avg_order_value
    from customers c
    join orders o on c.customer_id = o.customer_id
    where month(o.order_date) = input_month and year(o.order_date) = input_year
    group by c.customer_id, c.customer_name
    order by total_spent desc
    limit limit_n;
    end //
   delimiter ;
   call get_top_customers_by_month(5, 2023, 5);
   ```

## Data Analysis & Business Key Problems and Answers

1. **Find the total number of orders each restaurant has received.**
   ```sql
   select r.restaurant_id, r.restaurant_name, count(o.order_id) as total_orders
      from restaurants r
      join orders o on
      r.restaurant_id = o.restaurant_id
      group by r.restaurant_id, r.restaurant_name
      order by total_orders desc;
   ```

2. **Calculate the average amount spent by each customer.**
   ```sql
   select c.customer_name, round(avg(o.total_amount),2) as avg_amnt
      from customers c
      join orders o on
      c.customer_id = o.customer_id
      group by c.customer_name
      order by avg_amnt desc;
   ```

3. **Identify the order with the highest number of items.**
   ```sql
   select order_item, count(order_id) highest_orders
      from orders
      group by order_item
      order by highest_orders desc;
   ```

4. **Get the number of orders per city.**
   ```sql
   sselect r.city, count(o.order_id) no_of_orders
      from restaurants r
      join orders o on
      r.restaurant_id = o.restaurant_id
      group by r.city
      order by no_of_orders desc;
   ```

5. **Find out how many deliveries each rider has completed.**
   ```sql
   select rd.rider_id, rd.rider_name,  count(dl.delivery_id) as completed_delivery
      from riders rd
      join deliveries dl on
      rd.rider_id = dl.rider_id
      where dl.delivery_status= 'Delivered'
      group by rd.rider_id, rd.rider_name
      order by rd.rider_id;
   ```

6. **Calculate and compare the order cancellation rate for each restaurant between the current year and the previous year.**
   ```sql
   with cc as
      (
      select r.restaurant_id, r.restaurant_name, count(o.order_id) as cancel_count
      from restaurants r
      join orders o on
      o.restaurant_id = r.restaurant_id
      where year(o.order_date) between '2023' and '2024'
      and o.order_status = 'Not Fulfilled'
      group by 1,2
      ),
      
      total_count as
      (
      select r.restaurant_id, r.restaurant_name , count(o.order_id)  as total_counts
      from orders o join restaurants r
      on r.restaurant_id = o.restaurant_id
      group by 1,2
      )
      
      select cc.restaurant_id, cc.restaurant_name,
      round(cc.cancel_count / total_count.total_counts * 100, 2) as cancellation_rate
      from cc join total_count on
      cc.restaurant_id = total_count.restaurant_id
      group by 1,2
      order by 3 desc;
   ```

7. **Determine each rider's average delivery time.**
   ```sql
    with adt as
       (
       select rd.rider_id, rd.rider_name, 
       case 
       when delivery_time > order_time 
       then round(timestampdiff(minute, order_time, delivery_time), 2)
       else round(timestampdiff(minute, order_time, delivery_time + interval 1 day), 2)
       end as delivery_time
       from riders rd
       join deliveries dl on
       rd.rider_id = dl.rider_id
       join orders o on
       o.order_id = dl.order_id
       )
       select rider_id, rider_name, concat(round(avg(delivery_time),2), " ", "mins") as avg_delivery_time
       from adt
       group by rider_id, rider_name 
       order by avg_delivery_time desc;
       
       create index idx_rider_id on deliveries(rider_id);
       create index idx_order_id_ on orders(order_id);
   ```

8. **Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining.**
   ```sql
   with rgr as
       (
       select r.restaurant_id, r.restaurant_name, date_format(o.order_date, "%m/%Y") as mnth, count(o.order_id) as total_orders, 
       lag(count(o.order_id) ,1) over(partition by restaurant_id order by date_format(o.order_date, "%m/%Y")) as previous_month_order
       from restaurants r
       join orders o on
       r.restaurant_id = o.restaurant_id
       join deliveries dl on
       o.order_id = dl.order_id
       where dl.delivery_status = 'Delivered'
       group by r.restaurant_id, r.restaurant_name, mnth
       order by r.restaurant_id, mnth 
       )
       select rgr.restaurant_id, rgr.restaurant_name, rgr.mnth, rgr.total_orders, previous_month_order,
       round((rgr.total_orders - previous_month_order) / previous_month_order * 100, 2) as growth_rate
       from rgr;
   ```

9. **Identify the time slots during which the most orders are placed based on 2-hour intervals.**
    ```sql
      select 
        case 
          when hour(order_time) between 0 and 1 then "00:00:00 AM-02:00:00AM"
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
          end as time_slots,
	 count(order_id) as total_order
     from orders
     group by time_slots
     order by total_order desc
     limit 1;
   ```

10. **List the customers who have spent more than 100K in total on food orders.**
    ```sql
    select * from orders;
      
      select o.customer_id, c.customer_name, sum(o.total_amount) as total_orders
      from orders o
      join customers c on
      o.customer_id = c.customer_id
      group by o.customer_id, c.customer_name
      having total_orders > 100000
      order by total_orders desc;
    ```

11. **Identify the most popular dish in each city based on the number of orders.**
    ```sql
    select * from 
       (
       select r.city, o.order_item, count(o.order_id) as number_of_orders,
       dense_rank() over(partition by r.city order by count(o.order_id) desc) as rnk
       from restaurants r 
       join orders o on 
       o.restaurant_id = r.restaurant_id
       group by r.city, o.order_item
	   ) as pd
       where rnk = 1;
    ```

12. **Segment customers into 'Gold' or 'Silver' groups based on their total spending compared to the average order value (AOV). 
--     If a customer's total spending exceeds the AOV, label them as 'Gold'; otherwise, label them as 'Silver'.**
    ```sql
    with customer_orders as
       (
	   select c.customer_id, c.customer_name, sum(o.total_amount) as total_spending, count(o.order_id) as order_count,
       round(sum(o.total_amount) / count(o.order_id), 2) as customer_aov
       from customers c
       join orders o on
       c.customer_id = o.customer_id
       group by c.customer_id, c.customer_name
       order by total_spending desc
       ),
       segmentation as
       (
       select co.customer_id, co.total_spending, co.order_count, co.customer_aov,
       case when co.total_spending > customer_aov then 'Gold'
       else 'Silver'
       end as category
       from customer_orders co
       join orders o on
       o.customer_id = co.customer_id
       group by co.customer_id
       order by co.total_spending desc
       )
       select category, sum(total_spending) as total_revenue, sum(order_count) as total_orders
       from segmentation
       group by category;
    ```

13. **Find the number of 5-star, 4-star, and 3-star ratings each rider has. Riders receive ratings based on delivery time.**
    ```sql
    with del_time as
       (
       select rd.rider_id, rd.rider_name, o.order_time, dl.delivery_time,
       case 
       when delivery_time > order_time 
       then concat(round(timestampdiff(minute, order_time, delivery_time), 2), " ", 'mins')
       else concat(round(timestampdiff(minute, order_time, delivery_time + interval 1 day), 2), " ", 'mins')
       end as time_taken
       from riders rd
       join deliveries dl on
       rd.rider_id = dl.rider_id
       join orders o on
       o.order_id = dl.order_id
       where order_status="Completed" and delivery_status="Delivered"
       )
       select del_time.rider_id, del_time.rider_name,
       sum(case when time_taken < 30 then 1 else 0 end) as 'Five_Star',
       sum(case when time_taken between 30 and 45 then 1 else 0 end) as 'Four_Star',
       sum(case when time_taken > 45 then 1 else 0 end) as 'Three_Star'
       from del_time
       group by del_time.rider_id, del_time.rider_name
       order by del_time.rider_id;
    ```

14. **Track the popularity of specific order items over time and identify seasonal demand spikes.**
    ```sql
     with order_items as
       (
       select order_item, count(order_id) as order_count,
       case when month(order_date) between 3 and 5 then "Spring"
       when month(order_date) between 6 and 8 then "Summer"
       when month(Order_date) between 9 and 11 then "Autumn"
       else 'Winter'
       end as seasons
       from orders
       where order_status = 'Completed'
       group by order_item, seasons
       ),
       popular_items as
       (
       select order_item, order_count, seasons,
       dense_rank() over(partition by order_item order by order_count desc) as rnk
       from order_items
       )
       select order_item, order_count, seasons
       from popular_items
       where rnk = 1;
    ```

15. **Rank each city based on the total revenue for the last year (2023).**
    ```sql
       with city_revenue as
       (
       select r.city, sum(o.total_amount) as total_revenue
       from restaurants r
       join orders o on
       r.restaurant_id = o.restaurant_id
       where year(o.order_date) = '2023'
       group by r.city
       ),
       city_rank as 
       (
       select city, total_revenue,
       dense_rank() over(order by total_revenue desc) as rnk
       from city_revenue
       )
       select city, total_revenue, rnk
       from city_rank;
    ```

16. **Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 2 year.**
    ```sql
       select c.customer_id, customer_name, order_item, count(*) as total_orders
      from customers c join orders o
      on c.customer_id = o.customer_id
      where customer_name="Arjun Mehta" and year(order_date) >= year(current_date()) - 2
      group by c.customer_id, customer_name, order_item
      order by total_orders desc 
      limit 5;
    ```

17. **Find the average order value per customer who has placed more than 750 orders.**
    ```sql
      select o.customer_id, c.customer_name, count(o.order_id) as cust_orders, round(avg(o.total_amount),2) as avg_order_value
      from orders o
      join customers c on
      o.customer_id = c.customer_id
      group by o.customer_id, c.customer_name
      having cust_orders > 750
      order by avg_order_value desc;

      create index idx_customer_id on orders(customer_id);
    ```

18. **Write a query to find orders that were placed but not delivered.**
    ```sql
       select o.restaurant_id, r.restaurant_name, r.city, sum(dl.delivery_status = 'Not Delivered') as not_deliverd_orders
      from restaurants r
      join orders o on
      r.restaurant_id = o.restaurant_id
      join deliveries dl on
      o.order_id = dl.order_id
      where order_status = "Completed"
      group by o.restaurant_id, r.restaurant_name, r.city
      order by not_deliverd_orders desc;
    ```

19. **Rank restaurants by their total revenue from the last year.**
    ```sql
       select o.restaurant_id, r.restaurant_name, r.city, sum(o.total_amount) as total_revenue,
      dense_rank() over( partition by city order by sum(total_amount)  desc ) as rnk
      from restaurants r
      join orders o on
      r.restaurant_id = o.restaurant_id
      where year(o.order_date) = year(current_date()) - 1
      group by o.restaurant_id, r.restaurant_name, r.city;

      create index idx_order_date on orders(order_date);
    ```

20. **Find customers who haven’t placed an order in 2024 but did in 2023.**
    ```sql
       select c.customer_id, customer_name
      from customers c
      join orders o on
      o.customer_id = c.customer_id
      where year(o.order_date) = '2023'
	  and c.customer_id not in 
          (select c.customer_id from customers join orders o
            on c.customer_id = o.customer_id where year(o.order_date)='2024')
	  group by c.customer_id, customer_name;
    ```

21. **Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.**
    ```sql
       select rd.rider_id, rd.rider_name, date_format(o.order_date, "%m/%Y") as mnth, 
       sum(o.total_amount) * 8 / 100 as order_amount
       from riders rd
       join deliveries dl on
       rd.rider_id = dl.rider_id
       join orders o on
       dl.order_id = o.order_id
       where order_status = 'Completed' and delivery_status = 'Delivered'
       group by rd.rider_id, rd.rider_name, mnth
       order by rd.rider_id, mnth;
    ```

22. **Analyze order frequency per day of the week and identify the peak day for each restaurant.**
    ```sql
       with order_frequency as
       (
       select r.restaurant_id, r.restaurant_name, dayname(o.order_date) as order_day, count(o.order_id) as order_count
       from restaurants r
       join orders o on
       r.restaurant_id = o.restaurant_id
       group by r.restaurant_id, r.restaurant_name, order_day
       ),
       peak_day as
       (
       select restaurant_id, restaurant_name, order_day, order_count,
       dense_rank() over(partition by restaurant_id order by order_count desc) as rnk
       from order_frequency
       )
       select restaurant_id, restaurant_name, order_day, order_count
       from peak_day 
       where rnk = 1;
       
       create index idx_restaurant_id on restaurants(restaurant_id);
       create index idx_order_date on orders(order_date);
    ```

23. **Identify sales trends by comparing each month's total sales to the previous month.**
    ```sql
       with sales_trend as
       (
       select r.restaurant_id, r.restaurant_name, date_format(o.order_date, "%m/%Y") as mnth,
       sum(o.total_amount) as sale_amount
       from restaurants r
       join orders o on
       r.restaurant_id = o.restaurant_id
       group by r.restaurant_id, r.restaurant_name, mnth
       order by r.restaurant_id, mnth
       ), 
       sales_with_lag as
       (
	   select *,
       lag(sale_amount, 1) over(partition by restaurant_id order by mnth) as previous_month_sale
       from sales_trend
       )
       select restaurant_id, restaurant_name, mnth, sale_amount, previous_month_sale,
       concat(round((sale_amount-previous_month_sale)/previous_month_sale*100,2)," ","%") as Monthly_sales_trend
       from sales_with_lag;
    ```

24. **Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.**
    ```sql
        with avg_time as
       (
       select rd.rider_id, rd.rider_name, 
       round(avg(
       case 
       when delivery_time > order_time 
       then round(timestampdiff(minute, order_time, delivery_time), 2)
       else round(timestampdiff(minute, order_time, delivery_time + interval 1 day), 2)
       end),2) as delivery_time
       from riders rd
       join deliveries dl on
       rd.rider_id = dl.rider_id
       join orders o on
       o.order_id = dl.order_id
       where dl.delivery_time is not null and o.order_time is not null
       group by rd.rider_id, rd.rider_name
       ),
       min_max_avg as
       (
       select min(delivery_time) as low_avg_delivery_time,
       max(delivery_time) as high_avg_delivery_time
       from avg_time
       )
       select agt.*
       from avg_time agt
       join min_max_avg mma on
       agt.delivery_time = mma.high_avg_delivery_time
       or
       agt.delivery_time = mma.low_avg_delivery_time;
    ```

25. **Calculate the total revenue generated by each customer over all their orders.**
    ```sql
       select c.customer_id, c.customer_name, sum(o.total_amount) as total_revenue
       from customers c
       join orders o on
       c.customer_id  = o.customer_id
       group by c.customer_id, c.customer_name
       order by total_revenue desc;
    ```

## Query Performance

- Used Stored Procedure for customers and restaurant tables.
- Indexed order_date, customer_id, order_id, rider_id, restaurant_id columns
- Used cte's and joins.

## Principal Findings

- Significant differences in restaurant reliability and rider efficiency
- Clear ordering trends by time and season
- Scope for improving customer retention using offers and personalization
- Delivery time can be optimized further for specific riders and hours

## Business Recommendations

- **Seasonal Campaigns & Festival Deals** - Launch themed promotions based on festivals and seasons to align with user mood and food preferences (e.g., hot beverages in winter).
- **Feedback Loop Post-Delivery** - Encourage customers to rate not only the food but also delivery, packaging, and restaurant service — enabling better experience scoring.
- **Improve Communication Flow** - Create a support line for restaurants to quickly resolve logistics issues with riders or customers.
- **Dynamic Route Optimization Tools** - Equip riders with in-app tools to avoid high-traffic areas based on real-time data (improves delivery time and customer satisfaction).
- **Bundle Offers & Upselling** - Promote combo meals and add-on items during checkout to increase average order value.
-  **Gamify the Experience** - Create monthly leaderboards or badges for riders to encourage a healthy competitive environment.

## Future Scope

- Incorporate customer feedback sentiment analysis using NLP.
- Add predictive modeling for order volume forecasting.
- Integrate live dashboards for real-time monitoring.
- Expand rider efficiency analysis using GPS-based data.

# Conclusion

**The analysis successfully uncovered key performance indicators and behavioral patterns across Zomato’s ecosystem. These insights can be used to:**

- Boost delivery efficiency
- Improve customer satisfaction
- Drive growth through targeted offers
- Optimize restaurant and rider performance


