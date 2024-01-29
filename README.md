# Retail-Data-Warehouse-Optimization-
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






