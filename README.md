# Retail-Data-Warehouse-Optimization-MySQL
Retail Data Warehouse Optimization, Conducting a Product-Market Fit Analysis for Targeted Discount Strategies, consolidating 11 separated data tables into a data warehouse using MySQL. The analysis was visualized using Tableau dashboards. 

Schema of the database : 

<img width="468" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/6d5c674c-0530-48ec-90e7-4ff5ef007584">

#Code file to create the database warehouse uploaded separately. - retail_analysis.sql

#Datawarehouse now referred as 'datawarehouse' in the next 8 queries analyzing and drawing different insights from the database. 

The data warehouse is designed to understand customer behavior and optimize retail business strategy. It analyzes customer spending, including semi-annual and quarterly subtotals, to evaluate the impact of discounts on sales. The system also tracks the number of orders, a crucial source of revenue, and examines spending on specific product types to gauge customer preferences. 

#Top 10 countries ranked by total contribution to revenue % 

The information is useful when considering how much effort should be put into these individual countries, such as for advertising or marketing/promotions. 

```
SELECT sub.country, sub.country_subtotal_revenue, sub.total_revenue, ROUND((sub.country_subtotal_revenue / sub.total_revenue)*100,2) AS percent_of_revenue
FROM ( SELECT Country, ROUND(SUM(subtotal),2) AS country_subtotal_revenue,
              (SELECT ROUND(SUM(subtotal),2) FROM datawarehouse) AS total_revenue
       FROM datawarehouse
       GROUP BY Country 
       ORDER BY country_subtotal_revenue DESC) AS sub
LIMIT 10;

```
<img width="317" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/f8fcad67-eb69-445c-b869-1dfa99c4272e">


b.	Customers who are offered the discount but didn’t any value 

Ranking the customers based on discounts provided and ranking their total amount spend to understand the discount's effectiveness. Based on this information, decisions such as : 'whether to give up the customers or keep giving them discounts' - can be taken. 

```
SELECT d.CustomerID, d.subtotal, d.total_discount,  ROUND((d.subtotal + d.total_discount),2) AS original_revenue,
       sub2.percent_of_discount,
	   RANK() OVER(ORDER BY subtotal DESC) AS rank_on_subtotal,
       sub2.rank_on_discount
FROM datawarehouse AS d
JOIN (SELECT sub1.CustomerID, sub1.subtotal, sub1.total_discount, sub1.percent_of_discount, 
	  ROW_NUMBER() OVER (ORDER BY percent_of_discount DESC) AS rank_on_discount
      FROM (SELECT CustomerID, subtotal, total_discount,
                   ROUND((total_discount / (subtotal + total_discount)) * 100,2) AS percent_of_discount
	        FROM datawarehouse) AS sub1) AS sub2
USING (CustomerID)
ORDER BY rank_on_discount;

```
<img width="424" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/747cc645-1fc9-4ea0-b70f-3f7960861dfc">

c.	Subtotal percentage on different shippers given by countries

This query is useful to understand the % distributions of shippers used by different countries. For example, we can see that Denmark takes 56% of their services from 'Federal Shipping'. If 'Federal Shipping' runs into some sort of issue, there will be loss in business. Balanced shipper distribution is recommended. 

```
SELECT Country, 
	   ROUND(subtotal_on_Federal_Shipping / total * 100,2) AS subtotal_percentage_on_Federal_Shipping,
       ROUND(subtotal_on_Speedy_Express / total * 100,2) AS subtotal_percentage_on_Speedy_Express,
       ROUND(subtotal_on_United_Package / total * 100,2) AS subtotal_percentage_on_United_Package
FROM ( SELECT Country, ROUND(SUM(subtotal_on_Federal_Shipping),2) AS subtotal_on_Federal_Shipping, 
              ROUND(SUM(subtotal_on_Speedy_Express),2) AS subtotal_on_Speedy_Express,
              ROUND(SUM(subtotal_on_United_Package),2) AS subtotal_on_United_Package,
              ROUND(ROUND(SUM(subtotal_on_Federal_Shipping),2) + ROUND(SUM(subtotal_on_Speedy_Express),2) + ROUND(SUM(subtotal_on_United_Package),2),2) AS total
       FROM datawarehouse
       GROUP BY Country) AS sub;

```
<img width="410" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/9e50287e-b567-4dfb-b45c-74647f0454a7">

d. Revenue structure in different countries

Simple overview of the existing revenue streams and their numbers for region wise strategy focus. (Which products to focus on X country?) 

```

SELECT Country, ROUND(SUM(subtotal_in_Beverages),2) AS total_Beverages, 
                ROUND(SUM(subtotal_in_Condiments),2) AS total_Condiments,
                ROUND(SUM(subtotal_in_Confections),2) AS total_Confections,
                ROUND(SUM(subtotal_in_Dairy_Products),2) AS total_Products,
                ROUND(SUM(subtotal_in_Grains_Cereals),2) AS total_Cereals,
                ROUND(SUM(subtotal_in_Meat_Poultry),2) AS total_Poultry, 
                ROUND(SUM(subtotal_in_Produce),2) AS total_Produce,
                ROUND(SUM(subtotal_in_Seafood),2) AS total_Seafood
FROM datawarehouse
GROUP BY country WITH ROLLUP
ORDER BY country;

```
<img width="322" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/fd09411c-de85-44bf-b440-c6a30db5e8cb">

e. Number of orders in different quarters

Overview of order count based on different countries accross different quarters. This information is useful to predict future order trends based on different quarters. 

```

SELECT Country, SUM(NumOrder_1996Q3) AS total_NumOrder_1996Q3, 
                SUM(NumOrder_1996Q4) AS total_NumOrder_1996Q4,
                SUM(NumOrder_1997Q1) AS total_NumOrder_1997Q1,
                SUM(NumOrder_1997Q2) AS total_NumOrder_1997Q2,
                SUM(NumOrder_1997Q3) AS total_NumOrder_1997Q3,
                SUM(NumOrder_1997Q4) AS total_NumOrder_1997Q4,
                SUM(NumOrder_1998Q1) AS total_NumOrder_1998Q1,
                SUM(NumOrder_1998Q2) AS total_NumOrder_1998Q2,
                SUM(NumOrder_1996Q3) + SUM(NumOrder_1996Q4) + SUM(NumOrder_1997Q1) + SUM(NumOrder_1997Q2) + SUM(NumOrder_1997Q3) + SUM(NumOrder_1997Q4) + SUM(NumOrder_1998Q1) + SUM(NumOrder_1998Q2) AS total_order
FROM (SELECT CustomerID, Country,
			 CASE WHEN NumOrder_1996Q3 IS NULL THEN 0 ELSE NumOrder_1996Q3 END AS NumOrder_1996Q3,
			 CASE WHEN NumOrder_1996Q4 IS NULL THEN 0 ELSE NumOrder_1996Q4 END AS NumOrder_1996Q4,
			 CASE WHEN NumOrder_1997Q1 IS NULL THEN 0 ELSE NumOrder_1997Q1 END AS NumOrder_1997Q1,
			 CASE WHEN NumOrder_1997Q2 IS NULL THEN 0 ELSE NumOrder_1997Q2 END AS NumOrder_1997Q2,
			 CASE WHEN NumOrder_1997Q3 IS NULL THEN 0 ELSE NumOrder_1997Q3 END AS NumOrder_1997Q3,
			 CASE WHEN NumOrder_1997Q4 IS NULL THEN 0 ELSE NumOrder_1997Q4 END AS NumOrder_1997Q4,
			 CASE WHEN NumOrder_1998Q1 IS NULL THEN 0 ELSE NumOrder_1998Q1 END AS NumOrder_1998Q1,
			 CASE WHEN NumOrder_1998Q2 IS NULL THEN 0 ELSE NumOrder_1998Q2 END AS NumOrder_1998Q2
      FROM datawarehouse) AS sub
GROUP BY country;

```

<img width="444" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/03971ff7-033c-49b6-a62b-1eeafbff5594">

h. The rank of subtotal and average subtotal on order

Ranking of a customer's subtotal spending can differ from their ranking of average total spending per order. For example, SIMOB's subtotal is ranked as 23, but average subtotal per order is ranked as 6. Meaning, if the focus is shifted more towards getting SIMOB to order more, it will be beneficial because of large order amount. 

```
SELECT d.CustomerID, Total_orders, d.subtotal, ROUND((d.subtotal / d.Total_orders),2) AS per_revenue_on_order,
       ROW_NUMBER() OVER(ORDER BY ROUND((d.subtotal / d.Total_orders),2) DESC) AS rank_on_avg_revenue_on_order,
       sub1.rank_on_subtotal
FROM datawarehouse AS d
JOIN ( SELECT CustomerID, subtotal,
       RANK() OVER(ORDER BY subtotal DESC) AS rank_on_subtotal
       FROM datawarehouse) AS sub1
USING (CustomerID);

```
<img width="443" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/c0fea412-2de9-4f4e-aacb-4a2803dfa8ae">

------ Visualizing the dataset using Tableau ----------

I. Subtotal by different countries : 

The visualization shows the subtotal contributed by different countries. Countries with darker green are the countries which buying more products. Great overview to decide where to expland, where to put more effort, and getting an overall high level understanding of target market. 

<img width="500" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/a7174185-9933-4796-a1b5-5f0c28d71e33">

II. Number of customers in different countries

It gives an idea about the subtotal for different shippers by country. This information will be useful when considering the risk distribution. As can be seen in Norway, lion’s share of the total heavily relies on United Package. Although there may be few shippers to choose from, it should be noted that this kind of structure will be a huge risk if, for some reason, United Package is unable to perform its duties. 

<img width="500" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/80ea94df-5303-442a-96b6-569543f4d2b3">


III. Subtotal of different product categories on different countries

Through the information, the market preference based on product type can be learned. This information will be useful when deciding what kinds of products to promote by countries. By properly assessing the market preference, it is possible to offer product bundles to attract & retain customers.

Grain cereal market distribution 

<img width="400" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/3119c44b-d9d6-420c-8f15-e42c2df065b6">

IV. Subtotal against discount visualized 

This shows the difference between the subtotal and the discount provided to the customers. Ideally, bigger discounts are encouraged for customers who bring in a bigger subtotal. In the graph, it can also be noticed that there are customers who don't bring much value against the discount provided. 

<img width="500" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/33253b5e-18d5-4de7-aa64-6cbc9f9bf6fb">

V. Dashboard overview 

<img width="600" alt="image" src="https://github.com/saheelchowdhury/Retail-Data-Warehouse-Optimization-/assets/153671296/28dcebf0-0553-419a-882a-6fa30f3f83dd">
















