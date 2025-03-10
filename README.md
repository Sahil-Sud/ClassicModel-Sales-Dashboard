Problem Statement: A small company, Axon, a retailer selling classic cars, is facing issues in managing and analyzing its sales data. The sales team is struggling to make sense of the data, and it does not have a centralized system to manage and analyze it. The management cannot get accurate and up-to-date sales reports, which is affecting the decision-making process.

Aim: The goal of the capstone project is to design and implement a BI solution using PowerBI and SQL that can help the company manage and analyze their sales data effectively. The action plan includes the following:
1.	Import and integrate the data from MySQL database into PowerBI.
2.	Clean and transform the data to make it ready for analysis.
3.	Build interactive dashboards and reports using PowerBI that can help the sales team and management make sense of the data.
4.	Use SQL to perform advanced analytics on the data and extract insights that can help the company improve its sales.

SQL Queries: - 

 1. Revenue from each product by product-code
```sql
select productcode,sum(quantityOrdered * priceEach) as revenue_generated_per_product 
from orderdetails group by productcode;
```
2. Revenue from each product by product-name
```sql
select products.productcode,products.productname,sum(quantityOrdered * priceEach) as revenue_generated_per_product 
from orderdetails
inner join products using(productcode)
group by productcode;
```
3. Products never sold by Axon
```sql
select products.productCode, products.productname
from products
left join orderdetails using(productcode)
where orderdetails.productCode is null;
```
4. Product wise Average Selling Price, Average Discount, Average Profit and Average Profit Percentage.
```sql
select productname,buyprice as cost_price,msrp as markup_price, 
sum(quantityordered * priceeach) / sum(quantityOrdered) as avg_selling_price, 
msrp - (sum(quantityordered * priceeach) / sum(quantityOrdered))  as avg_discount, 
((sum(quantityordered * priceeach) / sum(quantityOrdered)) - buyprice) as avg_profit,
round(((sum(quantityordered * priceeach) / sum(quantityOrdered)) - buyprice) / buyprice *100) as avg_profit_percentage
from orderdetails 
inner join products using(productcode)
group by productCode
order by avg_profit_percentage desc;
```
5. Quatity sold, Quantity left Product_Wise
```sql
select products.productname, quantityInStock, sum(quantityOrdered) quantity_sold, 
quantityInStock - sum(quantityOrdered) stock_left
from products inner join orderdetails using(productcode)
group by productCode;
```
6. Order Status 
```sql
select 
	count(*) as total_orders,
    sum(case when status = 'shipped' then 1 else 0 end) as shipped,
    sum(case when status = 'cancelled' then 1 else 0 end) as cancelled,
    sum(case when status = 'resolved' then 1 else 0 end) as resolved,
    sum(case when status = 'on hold' then 1 else 0 end) as on_hold,
    sum(case when status = 'in process' then 1 else 0 end) as in_process,
    sum(case when status = 'disputed' then 1 else 0 end) as disputed
from orders;
```
7. Classifing Shipping Speed
```sql
select ordernumber, 
       orderdate, 
       datediff(shippeddate, orderdate) as dispacted_in, 
       case 
           when datediff(shippeddate, orderdate) > 0 and datediff(shippeddate, orderdate) <= 2 then "quick" 
           when datediff(shippeddate, orderdate) > 2 and datediff(shippeddate, orderdate) <= 3 then "medium" 
           else "slow" 
       end as dispatch_category
from orders 
where status = "shipped";
```
8.  Total Orders placed by Customers having status "shipped" arranged highest to lowest 
```sql
select orders.customernumber, customers.customername, sum(case when status = 'shipped' then 1 else 0 end) as order_counts 
from orders join customers using(customernumber)
group by customerNumber
order by 3 desc;
```
9. Customers details regarding total order placed, transaction worth having status "shipped" arranged highest to lowest 
```sql
select orders.customernumber, customers.customername, sum(case when status = 'shipped' then 1 else 0 end) as order_counts, round(sum(quantityordered * priceeach)) worth 
from orders 
join customers using(customernumber)
join orderdetails using(ordernumber)
group by customerNumber
order by 3 desc, 4 desc;
```
 10. Top 3 best-selling products by Revenue
```sql
with cte as(
  select 
  productcode ,sum(quantityordered * priceeach) revenue_generated, row_number() over (order by sum(quantityordered * priceeach) desc) as ranking
  from orderdetails
  group by productcode)
select productcode, productname, revenue_generated 
from cte 
join products using(productcode) 
where ranking<=3
order by ranking;
```
 11. Top 2 Customers who placed the highest number of orders yearly
```sql
with cte as (
  select 
  year(orderdate) as ordered_year, customernumber, 
  count(*) as no_of_orders_placed, 
  rank() over (partition by year(orderdate) order by count(*) desc) as ranking
  from orders
  group by customernumber, year(orderdate)
  order by 1, 3 desc)
select ordered_year, customernumber, customername, no_of_orders_placed 
from cte join customers using(customernumber)
where ranking <= 2;
```
 12.  Products with increasing sales trends
```sql
with cte as(
  select 
  year(orderdate) year_of_order, 
  productcode, 
  sum(quantityOrdered) present_year_sale,
  lead(sum(quantityOrdered),1) over (partition by productCode order by year(orderdate)) next_year_sale
  from orders join orderdetails using(ordernumber)
  group by productcode, year(orderdate)), 
  cte2 as (
  select * , case when present_year_sale< next_year_sale then "Increasing" else "Decresing" end as trend
  from cte)
select productcode, present_year_sale, next_year_sale 
from cte2 
where trend = "increasing" order by productcode;
```
13. Customers who have never placed an order.
```sql
select customernumber, customername 
from customers 
where customernumber not in 
(select distinct customernumber from orders);
```
14. Orders that took more than the average shipping time 
```sql
select ordernumber, 
datediff(shippeddate,orderdate) as shipping_Time 
from orders 
where datediff(shippeddate,orderdate) > (select avg(datediff(shippeddate,orderdate)) from orders);
```
15. Monthly revenue trend of the company
```sql
with cte as (
    select 
        year(orderdate) as year_of_order, 
        month(orderdate) as month_of_order, 
        sum(quantityordered * priceeach) as revenue
    from orders 
    join orderdetails using(ordernumber)
    group by 1,2
)
select 
    year_of_order, 
    month_of_order, 
    round(revenue) revenue, 
    round(sum(revenue) over (partition by year_of_order order by month_of_order)) as cumulative_revenue
from cte;
```
16. Percentage contribution of each product to total-sales
```sql
select productcode, productname,
sum(quantityOrdered* priceEach) revenue_generated, 
round(sum(quantityOrdered* priceEach)/(select round(sum(quantityOrdered* priceEach)) total_sales from orderdetails) *100,3) percentage_of_total_sales
from orderdetails join products using(productcode)
group by productcode;
```
17.  Finding the top 5 customers contributing the most revenue each year
```sql
with cte as(
  select 
  year(orderdate) transaction_year, 
  customernumber, 
  sum(quantityOrdered * priceEach) revenue,
  row_number() over (partition by year(orderdate) order by sum(quantityOrdered * priceEach) desc ) ranking
  from orders join orderdetails using(ordernumber)
  group by year(orderdate), customernumber
  order by 1 asc, 3 desc)
select customername, transaction_year, revenue , ranking 
from cte join customers using(customernumber)
where ranking <= 5;
```
18. Total Orders and Sales for each productLine 
```sql
select productline, count(*) total_orders, sum(quantityOrdered * priceEach) total_sales
from products join orderdetails using(productcode)
group by productline;
```
19. Orders where the customer ordered more than their average order quantity 
```sql
select ordernumber, customernumber, sum(quantityOrdered) quantity_ordered
from orders o join orderdetails od using(ordernumber)
group by 1, 2
having quantity_ordered > 
(select sum(quantityordered)/count(distinct ordernumber)  avg_quantity
from orders o1 join orderdetails od1 using(ordernumber)
where o1.customernumber = o.customernumber);
```
Insights from Dashboard: -

1. Total sale for the company stood to be $10 Million for the year 2003, 2004, 2005.

2. Total order placed till date stands at 326.

3. Average Order Value is at $29.46K and Average Processing time for the order stands at 4 days.

4. Total Unique customers for Axon company is 98.

5. Out of $10 Million revenue, gross profit stands at $3.83 Million

6. High revenue was recorded for the month of november and December for the year 2003, 2004 which may be a seasonal trend.

7. Highest Revenue for all the three years came from Classic Cars productLine followed by vintage cars.

8. Out of 326 orders 209 orders were placed for products under classic cars category followed by 187 orders for vintage cars.

9. Over the three years, Classic and Vintage Cars accounted for 62% of the total orders, while Trains recorded the fewest orders. 

10. Highest Revenue was recorded from USA approx $3.2 Million followed by Spain, France for all three years combined.

11. Order volume were highest for USA (112) followed by Spain (37) and France(36).

12. Customer id 141 (Euro + Shopping Channel) was the highest revenue generater for Axon for all 3 years consecutively suggestiong priority service to be given to them.
