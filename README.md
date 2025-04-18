# AMAZON_DATA_ANALYSIS_ADVANCED_SQL

# 📊 Data-Driven Decisions: Amazon Marketplace Analysis with SQL

![Amazon](images/amazon.png)

## 🧠 Overview
This project is a comprehensive SQL-based analysis of Amazon marketplace data. Using PostgreSQL and structured SQL queries, I explored various business metrics such as sales performance, customer behavior, inventory management, shipping delays, and more.

## 👩‍💻 Author
**Sridhar Undavilli**

## 🎯 Project Objectives
- Identify high-performing products, sellers, and regions
- Analyze customer lifetime value and buying trends
- Track shipping and delivery performance
- Generate profit margin and return rate insights
- Automate inventory updates using a stored procedure

## 📂 Datasets Used
All data is derived from simulated Amazon marketplace records:

- `orders.csv`
- `order_items.csv`
- `products.csv`
- `inventory.csv`
- `customers.csv`
- `sellers.csv`
- `payments.csv`
- `shipping.csv`
- `category.csv`

![EDR for amazon_sql](images/edr_for_amazon_sql.png)

## 🛠️ Tools & Technologies
- **SQL (MS SQL SERVER)** – Core querying and analysis
- **VS Code** – Development environment
- **ChatGPT** – For query optimization and explanation

## 🎯 Project Objectives
- Identify high-performing products, sellers, and regions
- Analyze customer lifetime value and buying trends
- Track shipping and delivery performance
- Generate profit margin and return rate insights
- Automate inventory updates using a stored procedure

## 💻 How to Use

1. Set up SQL SERVER MANAGEMENT STUDIO on your machine.
2. Import the CSV datasets into respective tables.
3. Run the `business_insights.sql` file:

```bash
\i business_insights.sql
```
This file runs all queries and creates the stored procedure for inventory automation.

## 📊 Business Questions, Queries & Insights

### 1. Top Selling Products
```sql
select  top 10 p.product_id,
	p.product_name,
	round(sum(ot.quantity * ot.price_per_unit),0) as total_sales,
	sum(ot.quantity) as Quantity_sold
	from products as p
	join order_items as ot
	on p.product_id=ot.product_id
	group by p.product_name,p.product_id
	order by total_sales desc

```
![Top Selling Products](business_query_results/1.top_selling_products.png)

**Insight**: A small number of products contribute significantly to total revenue and order volume. These are key revenue drivers.

### 2. Revenue by Category
```sql
select c.category_id,c.category_name,
	round(sum(ot.quantity * ot.price_per_unit),0) as Revenue_category,
	sum(ot.quantity) as Quantity_sold
	from category as c
	join products as p
	on c.category_id =p.category_id
	join order_items as ot
	on ot.product_id =p.product_id
	group by c.category_id,c.category_name
	order by Revenue_category desc
```
![Revenue by Category](business_query_results/2.revenue_by_category.png)

**Insight**: Electronics and related categories account for the highest revenue contribution across the platform.

### 3. Average Order Value (AOV)
```sql
select c.customer_id, c.first_name as Name_Cx,
	count(o.order_id) as Total_orders,
	(sum(oi.quantity * oi.price_per_unit)/--COUNT(o.order_id)) as AOV
	(select No_of_orders from(select cx.customer_id,COUNT(os.order_id) as No_of_orders from customers as cx join orders as os on os.customer_id=cx.customer_id group by cx.customer_id)as cx_count_table where cx_count_table.customer_id=c.customer_id
	 )) as AOV
	from customers as c
	join orders as o
	on o.customer_id = c.customer_id
	join order_items as oi
	on oi.order_id=o.order_id
	--where count(o.order_id) >5
	group by c.customer_id,c.first_name
	Having count(o.order_id) >5
	order by AOV desc
```
![Average Order Value (AOV)](business_query_results/3.average_order_value.png)

**Insight**: Returning customers have significantly higher AOV, suggesting a loyal and valuable customer segment.

### 4. Monthly Sales Trend
```sql
select * from 
(select DATEPART(YEAR,o.order_date)as Year_order,DATEPART(month,o.order_date) as Month_order,
		Round(sum(ot.quantity * ot.price_per_unit),0) as Current_mnth_sales,
		LAG(Round(sum(ot.quantity * ot.price_per_unit),0),1)over(partition by DATEPART(YEAR,order_date) order by DATEPART(month,order_date) asc) as Prev_Mnth_orders
		from orders as o
		join order_items as ot
		on ot.order_id=o.order_id
		group by DATEPART(month,order_date) ,DATEPART(YEAR,order_date)) as Monthly_sales_trend
		where Year_order= Datepart(Year,SYSDATETIME())-1
```
![Monthly Sales Trend](business_query_results/4.monthly_sales_trend.png)

**Insight**: Sales trends fluctuate monthly with noticeable seasonal patterns. Comparing current and previous months helps detect growth or decline.

### 5. Customers with No Purchases
```sql
select (c.customer_id),CONCAT(c.first_name,' ',c.Last_name) as Name_cx
	from customers as c
	where c.customer_id not in (select Distinct customer_id from orders)

```
![Customers with No Purchases](business_query_results/5.customers_with_no_purchases.png)

**Insight**: There is a segment of registered users who have not placed any orders — potential for re-engagement campaigns.

### 6. Best-Selling Categories by State
```sql
select * from 
(select c.state_name,ct.category_name,
	count(o.order_id) as No_of_orders,
	round(sum(ot.quantity * ot.price_per_unit),0) as Sales_state,
	RANK() over (Partition by c.state_name order by count(o.order_id) desc,round(sum(ot.quantity * ot.price_per_unit),0)desc) as Rank_
	from customers as c
	join orders as o
	on c.customer_id=o.customer_id
	join order_items as ot
	on ot.order_id=o.order_id
	join products as p
	on p.product_id=ot.product_id
	join category as ct
	on ct.category_id=p.category_id
	group by c.state_name,ct.category_name) as Selling_category_by_states_table
	where Rank_=1
	order by Sales_state desc
```
![Best-Selling Categories by State](business_query_results/6.bestselling_categories_by_state.png)

**Insight**: Different states show varied preferences, enabling region-specific marketing strategies.

### 7. Customer Lifetime Value (CLTV)
```sql
select c.customer_id,CONCAT(c.first_name,' ',c.Last_name) as Name_Cx,
	Round(sum(ot.quantity * ot.price_per_unit),0) as CLTV,
	Rank() over(order by Round(sum(ot.quantity * ot.price_per_unit),0) desc) as Rank_
	from customers as c
	join orders as o
	on o.customer_id=c.customer_id
	join order_items as ot
	on ot.order_id= o.order_id
	group by c.customer_id,CONCAT(c.first_name,' ',c.Last_name)
```
![Customer Lifetime Value (CLTV)](business_query_results/7.customer_lifetime_value.png)

**Insight**: Top customers contribute significantly to revenue. Retaining them is crucial.

(Continued in next cell...)

### 8. Inventory Stock Alerts
```sql
select i.inventory_id,p.product_name,i.stock as Stock_left,
	 i.laststock_date,i.warhouse_id
	 from inventory as i
	 join products as p
	 on i.product_id=p.product_id
	 where i.stock <10
```
![Inventory Stock Alerts](business_query_results/8.inventory_stock_alerts.png)

**Insight**: Several high-demand products are at risk of going out of stock, requiring urgent restocking.

### 9. Shipping Delays
```sql
select No_of_days_delivery,shipping_providers,count(customer_id) from
(select c.customer_id,o.order_id,o.order_date,s.shipping_date,s.shipping_providers,
	DATEDIFF(DAY,o.order_date,s.shipping_date)as No_of_days_delivery,
	ot.product_id,p.product_name,
	Round(sum(ot.quantity * ot.price_per_unit),0) as Cost_order
	from customers as c
	join orders as o
	on o.customer_id = c.customer_id
	join order_items as ot
	on ot.order_id = o.order_id
	join products as p
	on p.product_id =ot.product_id
	join shipping as s
	on s.order_id=o.order_id
	group by c.customer_id,o.order_id,o.order_date,s.shipping_date,s.shipping_providers,
	DATEDIFF(DAY,o.order_date,s.shipping_date),ot.product_id,p.product_name) as No_of_days_delivery_table
	where No_of_days_delivery >=4
	group by No_of_days_delivery,shipping_providers

```
![Shipping Delays](business_query_results/9.shipping_delays.png)

**Insight**: Many orders experience shipping delays beyond 2 days, indicating potential issues with fulfillment partners.

### 10. Payment Success Rate
```sql
Select p.payment_status,count(p.payment_id) as Total_payments,
	count(p.payment_id)*100/(select COUNT(payment_id) from payments) as Percentage_
	from payments as p
	join orders as o
	on p.order_id=o.order_id
	group by p.payment_status
	order by Percentage_ desc
```
![Payment Success Rate](business_query_results/10.payment_success_rate.png)

**Insight**: While most payments succeed, a notable percentage are either pending or failed, suggesting a need for payment system improvements.

### 11. Top Performing Sellers
```sql
select  s.seller_id,s.seller_name,s.origin,--o.order_status,
	Round(sum(ot.quantity * ot.price_per_unit),0) as total_sales,
	o.order_status,count(o.order_status) as No_of_orders,
	RANK() over( partition by o.order_status order by Round(sum(ot.quantity * ot.price_per_unit),0) desc) as Rank_
into Ranked_sellers
	from seller as s
	join orders as o
	on o.seller_id =s.seller_id
	join order_items as ot
	on ot.order_id= o.order_id
	where o.order_status not in ('inprogress','Returned')
	group by s.seller_id,s.seller_name,s.origin ,o.order_status
	
	
select * from Ranked_sellers where Rank_ <=5

select Rs.seller_id,Rs.seller_name,Rs.origin,--Rs.total_sales,
	sum(case when Rs.order_status = 'Completed' then Rs.No_of_orders else 0 end) as Completed_orders,
    sum(case when Rs.order_status='Cancelled' then Rs.No_of_orders  else 0 end )as Failed_orders,
	sum(Rs.No_of_orders) as Total_orders,
	Round(sum(case when Rs.order_status = 'Completed' then Rs.No_of_orders else 0 end)*100/Sum(No_of_orders),2) as Percentage_successful_order,
	(100 -(sum(case when Rs.order_status = 'Completed' then Rs.No_of_orders else 0 end)*100/Sum(No_of_orders))) as Percentage_failed_order
	from Ranked_sellers as Rs 
	where Rs.Rank_ <=5
	group by Rs.seller_id,Rs.seller_name,Rs.origin--,Rs.total_sales
```
![Top Performing Sellers](business_query_results/11.top_perfomring_sellers.png)

**Insight**: Top sellers show high order completion rates, but a few face elevated cancellation rates—warranting performance reviews.

### 12. Product Profit Margin
```sql
select product_id,product_name,Profit_Margin,
	Dense_RANK() Over (order by Profit_Margin Desc) as Product_Ranking
	from
(select p.product_id,p.product_name,
	(sum(ot.quantity * (ot.price_per_unit-p.cogs))*100/SUM(ot.quantity*ot.price_per_unit)) as Profit_Margin
	--Dense_RANK() over(order by sum(ot.quantity * (ot.price_per_unit-p.cogs))*100/SUM(ot.quantity*ot.price_per_unit) desc) as Rank_
	from products as p
	join order_items as ot
	on p.product_id=ot.product_id
	group by p.product_id,p.product_name) as Profit_margin_table
```
![Product Profit Margin](business_query_results/12.product_profit_margin.png)

**Insight**: Certain products yield significantly higher margins, guiding profitability-focused inventory and pricing strategies.

### 13. Most Returned Products
```sql
---APPROACH 1
select p.product_id,p.product_name,
	count(o.order_status) as No_Returned_products
into Returned_Products_Table
	from products as p
	join order_items as ot
	on ot.product_id =p.product_id
	join orders as o
	on o.order_id =ot.order_id
	where order_status='Returned'
	group by p.product_id,p.product_name,o.order_status
	--Having order_status='Returned'
	order by No_Returned_products asc

select p.product_id,p.product_name,
	Count(o.order_id) as Total_No_of_Products_order
into Total_products_order
	from products as p
	join order_items as ot
	on ot.product_id =p.product_id
	join orders as o
	on o.order_id =ot.order_id
	group by p.product_id,p.product_name
	order  by Total_No_of_Products_order desc


select Top 10 TP.product_id,TP.product_name,TP.Total_No_of_Products_order,RP.No_Returned_products,
	Round((RP.No_Returned_products *100/TP.Total_No_of_Products_order),2) as Return_percentage
	from Total_products_order as TP
	join Returned_Products_Table as RP
	on RP.product_id=TP.product_id
	order by Return_percentage desc

--APPROACH 2
select top 10 *,
	(No_of_Returned_Products *100/Total_no_of_solds) as Return_Percentage
 from (select p.product_id,p.product_name,
	count(o.order_status) as Total_no_of_solds,
	SUM(case when o.order_status='Returned' then 1 else 0 end) as No_of_Returned_Products
	from products as p
	join order_items as ot
	on ot.product_id =p.product_id
	join orders as o
	on o.order_id =ot.order_id
	group by p.product_id,p.product_name
	--order by Total_no_of_solds desc
	)as Returns_Table
order by Return_Percentage desc

```
![Most Returned Products](business_query_results/13.most_returned_products.png)

**Insight**: Return rates vary by product. High-return items may need review for quality, expectations, or listing accuracy.

### 14. Identify Returning vs. New Customers
```sql
select c.customer_id,CONCAT(c.first_name,' ',c.Last_name) as Name_of_cx,o.order_id,
	p.product_name,o.order_date,p1.payment_date,s.shipping_date,s.shipping_providers,
	round(sum(ot.quantity * ot.price_per_unit),0)as Price_order,p1.payment_status
	from customers as c
	join orders as o
	on o.customer_id=c.customer_id
	join order_items as ot
	on ot.order_id=o.order_id
	join products as p
	on p.product_id =ot.product_id
	join payments as p1
	on p1.order_id=o.order_id
	join shipping as s
	on o.order_id =s.order_id
	where p1.payment_status ='Payment Successed' and o.order_status='Inprogress'
	group by c.customer_id,CONCAT(c.first_name,' ',c.Last_name),o.order_id,
	p.product_name,o.order_date,p1.payment_date,p1.payment_status,s.shipping_date,
	s.shipping_providers
	order by Price_order desc
```
![Identify Returning vs. New Customers](business_query_results/14.identify_customers.png)

**Insight**: Customer behavior varies based on return activity—this can be useful for segmentation and tailored engagement.

### 15. Top 5 Customers by Orders per State
```sql
SELECT *
FROM (
    SELECT
        c.state,
        CONCAT(c.first_name, ' ', c.last_name) customers,
        COUNT(o.order_id) total_orders,
        SUM(oi.total_sale) total_sale,
        DENSE_RANK() OVER(PARTITION BY c.state ORDER BY COUNT(o.order_id) DESC) rank
    FROM orders o
    JOIN order_items oi ON oi.order_id = o.order_id
    JOIN customers c ON c.customer_id = o.customer_id
    GROUP BY 1, 2
)
WHERE rank <= 5;
```
![Top 5 Customers by Orders per State](business_query_results/1.top_selling_products.png)

**Insight**: Top customers in each state reveal regional power users that drive local revenue.

### 16. Revenue by Shipping Provider
```sql
SELECT
    s.shipping_providers,
    COUNT(o.order_id) orders_handled,
    ROUND(SUM(oi.total_sale)) total_sale,
    ROUND(COALESCE(AVG(s.return_date - s.shipping_date), 0)) average_days
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
JOIN shippings s ON s.order_id = o.order_id
GROUP BY 1;
```
![Revenue by Shipping Provider](business_query_results/16.revenue_by_shipping_provider.png)

**Insight**: Shipping providers vary in speed and volume. Some are slower despite handling fewer orders.

### 17. Products with Revenue Decline (2023 vs 2024)
```sql
WITH last_year_sale AS (
    SELECT p.product_id, p.product_name, SUM(oi.total_sale) revenue
    FROM orders o
    JOIN order_items oi ON oi.order_id = o.order_id
    JOIN products p ON p.product_id = oi.product_id
    WHERE EXTRACT(YEAR FROM o.order_date) = 2023
    GROUP BY 1, 2
),
current_year_sale AS (
    SELECT p.product_id, p.product_name, SUM(oi.total_sale) revenue
    FROM orders o
    JOIN order_items oi ON oi.order_id = o.order_id
    JOIN products p ON p.product_id = oi.product_id
    WHERE EXTRACT(YEAR FROM o.order_date) = 2024
    GROUP BY 1, 2
)
SELECT
    cs.product_id,
    cs.product_name,
    ls.revenue last_year_revenue,
    cs.revenue current_year_revenue,
    ls.revenue - cs.revenue rev_difference,
    ROUND((cs.revenue - ls.revenue)::NUMERIC / ls.revenue::NUMERIC * 100, 2) revenue_decrease_ratio
FROM last_year_sale ls
JOIN current_year_sale cs ON ls.product_id = cs.product_id
WHERE ls.revenue > cs.revenue
ORDER BY 6 DESC
LIMIT 10;
```
![Products with Revenue Decline (2023 vs 2024)](business_query_results/17.top_10_products_with_highest_decreasing_revenue_ratio.png)

**Insight**: Revenue decline detection helps flag underperforming products for reevaluation in 2024.

---

## 📘 What I Learned

This project was a comprehensive learning experience in data analysis, SQL, and business intelligence. Here's what I learned:

- How to write complex SQL queries using advanced features like CTEs, window functions, conditional logic, and subqueries.
- How to structure a real-world data analysis project with clear objectives, questions, and measurable outcomes.
- How to extract actionable business insights from raw datasets using SQL alone.
- The importance of validating data relationships and using joins correctly to link multiple tables.
- How to implement automation in SQL with stored procedures using PL/pgSQL.
- Practical debugging and optimization of queries using PostgreSQL.
- How to build stored procedures like `add_sales` that automate inventory and order management in a real-world scenario.
- How to present technical findings clearly and professionally through documentation and structure.


## 🛒 Stored Procedure: `add_sales`

As part of this project, I created a PostgreSQL stored procedure named `add_sales` to simulate real-time product transactions and automate inventory updates.

### 🔄 How It Works
This procedure replicates the logic of placing an order on an e-commerce platform:
1. It first checks the inventory to confirm that enough stock is available for the requested product.
2. If the product is available:
   - A new entry is added to the `orders` table.
   - A related entry is created in the `order_items` table including the calculated total sale.
   - The `inventory` table is updated to reflect the reduced stock.
   - A confirmation notice is displayed with the product name.
3. If the product is not in stock, a notice informs the user that the product is unavailable.

### 📜 Procedure Code
```sql
CREATE OR REPLACE PROCEDURE add_sales (
    p_order_id INT,
    p_customer_id INT,
    p_seller_id INT,
    p_order_item_id INT,
    p_product_id INT,
    p_quantity INT
)
LANGUAGE plpgsql
AS $$
DECLARE
    v_count INT;
    v_price FLOAT;
    v_product_name VARCHAR(50);
BEGIN
    SELECT price, product_name
    INTO v_price, v_product_name
    FROM products
    WHERE product_id = p_product_id;

    SELECT COUNT(*)
    INTO v_count
    FROM inventory
    WHERE product_id = p_product_id AND stock >= p_quantity;

    IF v_count > 0 THEN
        INSERT INTO orders (
            order_id, order_date, customer_id, seller_id
        )
        VALUES (
            p_order_id, CURRENT_DATE, p_customer_id, p_seller_id
        );

        INSERT INTO order_items (
            order_item_id, order_id, product_id, quantity, price_per_unit, total_sale
        )
        VALUES (
            p_order_item_id, p_order_id, p_product_id, p_quantity, v_price, v_price * p_quantity
        );

        UPDATE inventory
        SET stock = stock - p_quantity
        WHERE product_id = p_product_id;

        RAISE NOTICE 'Thank you, product "%" has been added and inventory updated.', v_product_name;
    ELSE
        RAISE NOTICE 'Product "%" is not available at the moment.', v_product_name;
    END IF;
END;
$$;
```

### 🧪 Example Usage
```sql
CALL add_sales(25001, 2, 5, 26001, 1, 10);
SELECT product_id, stock FROM inventory WHERE product_id = 1;
```

This procedure simulates a core part of e-commerce operations and demonstrates how SQL can be used not only for analysis but also for backend logic automation.

### 🧠 What I Learned
- How to declare and use variables inside a stored procedure.
- How to perform conditional logic and error handling in `PL/pgSQL`.
- How to use transactions to automate business processes in a database environment.
- How stored procedures can encapsulate complex logic and ensure consistency across related tables.

## ✅ Conclusion

This SQL project demonstrates how data can inform strategic decisions in an e-commerce environment. By analyzing Amazon-style marketplace data, I was able to:

- Uncover best-selling products and top-performing categories.
- Analyze revenue trends, profit margins, and customer value.
- Identify operational inefficiencies such as shipping delays and inventory shortages.
- Use SQL not just for querying, but also for creating database functionality through a stored procedure.

From querying to automation with the `add_sales` procedure, this project showcases how data analysis and backend logic can bring clarity to complex business questions and optimize decision-making.

## 📜 License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).
