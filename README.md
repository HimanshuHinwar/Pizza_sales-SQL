# Pizza_sales-SQL 
--Retrieve the total number of orders placed.
select count(*) from orders as total_order
--Calculate the total revenue generated from pizza sales.
select round(sum(price*quantity),2) as total_revenue from pizzas p
join details d on p.pizza_id = d.pizza_id
--Identify the highest-priced pizza.
select top 1 name , price from pizza_types pt
join pizzas p on pt.pizza_type_id = p.pizza_type_id
order by price desc

--Join relevant tables to find the category-wise distribution of pizzas.
select category , count(name) as total_orders from pizza_types
group by category


--List the top 5 most ordered pizza types along with their quantities.
select top 5 name , sum(quantity) as total_order from pizzas p
join details d on p.pizza_id = d.pizza_id
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
group by name
order by sum(quantity) desc

--Join the necessary tables to find the total quantity of each pizza category ordered.
select pt.pizza_type_id, sum(quantity) as total_quantity from pizzas p
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
join details d on p.pizza_id = d.pizza_id
group by pt.pizza_type_id

--Determine the distribution of orders by hour of the day.
select datepart(hour,time) as Hours, count(d.order_id) as total_hours from orders o
join details d on o.order_id = d.order_id
group by datepart(hour,time)
order by datepart(hour,time)

-- Identify the most common pizza size ordered.
select top 1 size , count(quantity) as total_quantity from pizzas p
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
join details d on p.pizza_id = d.pizza_id
group by size
order by count(quantity) desc



--Group the orders by date and calculate the average number of pizzas ordered per day.

with sum_of_orders as (select date , sum(quantity) as quantity from orders o
join details d on o.order_id = d.order_id
group by date)
select avg(quantity) avg_order from sum_of_orders


--Determine the top 3 most ordered pizza types based on revenue.
select top 3  name , sum(price*quantity) as revenue from pizzas p
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
join details d on p.pizza_id = d.pizza_id
group by name 
order by sum(price*quantity) desc

--Calculate the percentage contribution of each pizza type to total revenue.

select category , round(sum(price*quantity)/(select sum(price*quantity) as total_sales from pizzas p
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
join details d on p.pizza_id = d.pizza_id )*100,2) as percentage from pizzas p
join pizza_types pt on p.pizza_type_id = pt.pizza_type_id
join details d on p.pizza_id = d.pizza_id 
group by category


--Analyze the cumulative revenue generated over time.
with cum as (select date, round(sum(price*quantity),0) as revenue from pizzas p
join details d on p.pizza_id = d.pizza_id
join orders o on d.order_id = o.order_id
group by date)
select date , sum(revenue) over (order by revenue) as cummulative from cum
--Determine the top 3 most ordered pizza types based on revenue for each pizza category.

 with pizz as  (select category, name  ,sum(price*quantity)as revenue from pizzas p
join pizza_types pt on pt.pizza_type_id = p.pizza_type_id
join details d on p.pizza_id = d.pizza_id 
group by category , name)
select category , name from (select category , name , rank() over (partition by category order by revenue desc) as ranks from pizz ) as most
where ranks <=3
