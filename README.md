
# ![Apple Logo](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/Apple_Changsha_RetailTeamMembers_09012021_big.jpg.slideshow-xlarge_2x.jpg) Apple Retail Sales SQL Project - Analyzing Millions of Sales Rows

## Project Overview

This project is designed to showcase advanced SQL querying techniques through the analysis of over 1 million rows of Apple retail sales data. The dataset includes information about products, stores, sales transactions, and warranty claims across various Apple retail locations globally. By tackling a variety of questions, from basic to complex, you'll demonstrate your ability to write sophisticated SQL queries that extract valuable insights from large datasets.

The project is ideal for data analysts looking to enhance their SQL skills by working with a large-scale dataset and solving real-world business questions.

## Entity Relationship Diagram (ERD)

![ERD](https://github.com/najirh/Apple-Retail-Sales-SQL-Project---Analyzing-Millions-of-Sales-Rows/blob/main/erd.png)

Here’s the shortened and improved version of the "What’s Included" and "Why Choose This Project" sections, along with the link:
---

### What’s Included:
- **100 SQL Practice Problems**: Extensive coverage of major SQL topics for mastering concepts with real-world data.
- **20 Advanced SQL Queries**: Step-by-step solutions for complex queries, enhancing your skills in performance tuning and optimization.
- **5 Detailed Tables**: Comprehensive datasets with over 1 million rows, including sales, stores, product categories, products, and warranties.
- **Query Performance Tuning**: Learn to optimize queries for real-world data handling.
- **Portfolio-Ready Project**: Showcase your SQL expertise through large-scale data analysis.

### Why Choose This Project?
- **Hands-on Learning**: Practical experience with complex datasets and advanced business problem-solving.
- **Comprehensive Coverage**: Each table provides new opportunities to explore SQL concepts.
- **Exceptional Value**: For just **$9**, access 100 SQL problems, 20 advanced query solutions, and a real-world project.
- **Limited Offer**: Special price available for the **first 100 students**!

**Get the guided project/datasets here**: [Get the Project Datasets](https://topmate.io/zero_analyst/1237072)

## Database Schema

The project uses five main tables:

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Objectives

The project is split into three tiers of questions to test SQL skills of increasing complexity:

### Easy to Medium (10 Questions)

1. Find the number of stores in each country.
 ```sql
SELECT country,
	count(store_id) as t_store
FROM stores
group by 1
order by 1
```
2. Calculate the total number of units sold by each store.
```sql
SELECT st.store_name,s.store_id,
	sum(s.quantity) as t_q
FROM sales as s
left join 
stores as st
on s.store_id=st.store_id
group by 1,2
order by 3 desc
```

3. Identify how many sales occurred in December 2023.
```sql
SELECT *,to_char(sale_date,'MM-YYYY') 
FROM sales
where  to_char(sale_date,'MM-YYYY')='12-2023'
```

4. Determine how many stores have never had a warranty claim filed.
```sql
SELECT count(*) FROM stores
where store_id NOT IN(SELECT 
							distinct store_id
							from sales as s
							right join warrenty as w
							on s.sale_id=w.sale_id);
```


5. Calculate the percentage of warranty claims marked as "Warranty Void".
```sql
SELECT round(count(claim_id)/(select count(*) from warrenty)::numeric *100,
       2)
FROM warrenty
where repair_status ilike 'pending'
```
6. Identify which store had the highest total units sold in the last year.
```sql
SELECT store_id,
		sum(quantity)
FROM sales
where  sale_date >=(CURRENT_DATE- interval'2 years')
group by 1
order by 2 desc
```
7. Count the number of unique products sold in the last year.
```sql
SELECT count(distinct product_id)
FROM sales
where  sale_date >=(CURRENT_DATE- interval'2 years')
```

8. Find the average price of products in each category.
```
SELECT c.category_name,
	round(avg(p.price::numeric),2)
FROM category as c
join
products as p
ON p.category_id=c.category_id
group by 1
order by 2 desc
```

9. How many warranty claims were filed in 2020?
```SELECT count(*) 
FROM warrenty
where  TO_char(claim_date,'YYYY')='2024'
```
10. For each store, identify the best-selling day based on highest quantity sold.
```
SELECT *
FROM
(SELECT store_id	,
		to_char(sale_date,'Day'),
		sum(quantity),
		rank() over(partition by store_id order by sum(quantity) desc) as ranks
FROM sales
group by 1 ,2
) AS t1
where ranks = 1
```
### Medium to Hard (5 Questions)

11. Identify the least selling product in each country for each year based on total units sold.
```sql
with least_p
as
(SELECT country,
		product_name,
		sum(quantity),
		dense_rank() over(partition by country order by sum(quantity)) as ranks
FROM stores as s
 join
sales as sa
on s.store_id=sa.store_id
join
products as p
ON sa.product_id = p.product_id
group by 1,2
)
select * from least_p
where ranks=1
```

12. Calculate how many warranty claims were filed within 180 days of a product sale.
```sql
SELECT w.claim_id,w.claim_date-s.sale_date as days
 FROM warrenty as w
left join
sales as s
on s.sale_id=w.sale_id
join
products as p
ON s.product_id = p.product_id
where w.claim_date-s.sale_date <= '180' 
```
13. Determine how many warranty claims were filed for products launched in the last two years.
```sql
SELECT 
	p.product_name,
	count(w.claim_id)
 FROM warrenty as w
left join
sales as s
on s.sale_id=w.sale_id
join
products as p
ON s.product_id = p.product_id
where p.launch_date >= current_date -interval'2 years'
group by 1
```
14. List the months in the last three years where sales exceeded 5,000 units in the UAS
```sql
SELECT to_char(sale_date,'MM-YYYY') as months,
		sum(s.quantity) as total_unit_sold
FROM sales as s
join 
stores as st
on s.store_id=st.store_id
where st.country ilike 'UAE' and
s.sale_date >= current_date -interval'3 years'
group by 1
having sum(s.quantity) >= 5000
```
15. Identify the product category with the most warranty claims filed in the last two years.
```sql
SELECT p.category_id,
		count(w.claim_id) as t_c ,
		dense_rank() over(partition by category_id order by count(w.claim_id) desc)
FROM products as p
join 
sales as s 
on p.product_id =s.product_id
join
warrenty as w
on s.sale_id=w.sale_id
where w.claim_date >= current_date -interval'2 years'
group by 1
```

### Complex (5 Questions)

16. Determine the percentage chance of receiving warranty claims after each purchase for each country.
```sql
SELECT st.country
	,sum(s.quantity) as t_units_sold
	,count(w.claim_id) as t_claim
	,round(count(w.claim_id)::numeric/sum(s.quantity)::numeric *100,2) as percentage_of_warrenty
from sales as s
join 
stores as st
on s.store_id=st.store_id
left join
warrenty as w
ON w.sale_id=s.sale_id
group by 1
```

17. Analyze the year-by-year growth ratio for each store.
```sql
with yearly_sales
as
(SELECT 
 s.store_id,
 st.store_name,
 EXTRACT(YEAR FROM sale_date) as year,
 sum(s.quantity*p.price) as t_sales
FROM sales as s
join
products as p
on s.product_id = p.product_id
join
stores as st
ON st.store_id=s.store_id
group by 1,2,3
order by 2,3
),
growth_ratio
as
(select 
	store_name,
	year,
	t_sales as current_year_sale,
	lag(t_sales,1) over(partition by store_name order by year) as last_year_sale
from yearly_sales
)

select
    store_name,
	year,
	current_year_sale,
     last_year_sale,
	 round((current_year_sale-last_year_sale)::numeric/last_year_sale::numeric * 100,2) as growth_ratio
from  growth_ratio
where last_year_sale is not null
```

18. Calculate the correlation between product price and warranty claims for products sold in the last five years, segmented by price range.
```sql
SELECT 
	case 
	  when price < 500 then 'less expensive product'
	  when price between 500 and 1000 then 'mid expensive product'
	  else 'most expensive product'
	  end,
	  count(w.claim_id) as total_claims
FROM warrenty as w
left join
sales as s
on s.sale_id=w.sale_id
join 
products as p
on p.product_id=s.product_id
where claim_date >=CURRENT_DATE - INTERVAL'5 year'
group by 1

```

19. Identify the store with the highest percentage of "Paid Repaired" claims relative to total claims filed.
```sql
with paid_repair
as
(SELECT 
	s.store_id,
	count(w.claim_id) as paid_repaired
FROM sales as s
right join warrenty as w
on w.sale_id=s.sale_id
where w.repair_status='Completed'
group by 1
),
t_repair
as
(
SELECT 
	s.store_id,
	count(w.claim_id) as t_repaired
FROM sales as s
right join warrenty as w
on w.sale_id=s.sale_id
group by 1
)
SELECT t.store_id,
		p.paid_repaired,
		t.t_repaired,
		round(p.paid_repaired::numeric/t.t_repaired::numeric * 100,2) as percentage_paid_repaired
FROM paid_repair as p
join
t_repair as t
on p.store_id=t.store_id
```

20. Write a query to calculate the monthly running total of sales for each store over the past four years and compare trends during this period.
```sql
with monthly
as
(SELECT st.store_name,
		extract(year from sale_date) as year,
		extract(month from sale_date) as month,
		sum(p.price*s.quantity) as t_revenue
FROM sales as s
join 
stores as st
on s.store_id = st.store_id 
join 
products as p
on p.product_id=s.product_id
group by 1,2,3
order by 1,2,3
)
select store_name,
     month,
	 year,
	 t_revenue,
	 sum(t_revenue) over(partition by store_name order by year,month) as running_total
from monthly
```
### Bonus Question

- Analyze product sales trends over time, segmented into key periods: from launch to 6 months, 6-12 months, 12-18 months, and beyond 18 months.
```sql
SELECT 
	--s.sale_date,
	--p.launch_date,
	p.product_name,
	case
	    when s.sale_date between p.launch_date and p.launch_date + interval'6 month' then '0-6 month'
		when s.sale_date between p.launch_date + interval'6 month' and p.launch_date + interval'12 month' then '6-12'
		when s.sale_date between p.launch_date + interval'12 month' and p.launch_date + interval'18 month' then '12-18'
		else '18+'
		end  months,
		sum(s.quantity) as t_qty_sales
FROM sales as s
join products as p 
on s.product_id=p.product_id
group by 1,2
```
## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Conclusion

By completing this project, you will develop advanced SQL querying skills, improve your ability to handle large datasets, and gain practical experience in solving complex data analysis problems that are crucial for business decision-making. This project is an excellent addition to your portfolio and will demonstrate your expertise in SQL to potential employers.

author--Veeresh.D.Badiger

