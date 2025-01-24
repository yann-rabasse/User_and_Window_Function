# 1. Representing Data as a Proportion of Total

To calculate the proportion or a percentage from total for a given value, we can use window functions with aggregation functions like SUM() or COUNT().

In this exercise, we will use window functions to answer the following questions:

- Was the product purchased alone or together with other products?
- What is the revenue share of the product in the specific order?
- Does the order contain several distinct product categories or are all the products in the order from the same category?


## 1.1. Number of Products

We will use the gwz_sales_17 table in your course17 dataset as the data source.

1) To find out if the product was purchased alone or together with other products, we could calculate the number of products in each order and add this information to a new column nb_products. Functions like COUNT() and DISTINCT() coult be useful.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,COUNT(DISTINCT products_id) OVER (PARTITION BY orders_id) AS nb_products
 FROM `course17.gwz_sales_17`
 ORDER BY
     customers_id
     ,orders_id
     ,products_id
```


</details>


2) Add a new column called orders_type which contains the value single_product if there is only one product in the order and multi_product if there are several products in the order.

he calculation is done in 2 steps:
- calculate the new column nb_products.
- calculate a new column orders_type based on the column nb_products.

Use a subquery to combine the 2 subqueries into one query. Run it to check the results and save the query as a view called gwz_sales_17_orders_type.

<details>
    <summary> <font color="red"><b>How to save query as a view</b></font></summary>

  
<img width="773" alt="sdzem6b3m6r0fxsw5ezjkq7ftflq" src="https://github.com/user-attachments/assets/a53a62b8-7323-4311-955f-4a091c769a50" />


<img width="1013" alt="k572jsn6o9v2smdhr0cevyp4cptp" src="https://github.com/user-attachments/assets/0a478c37-39e8-4cbf-aec0-e33db61f5d05" />


</details>



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH sales_orders_type AS (
     SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,COUNT(DISTINCT products_id) OVER (PARTITION BY orders_id) AS nb_products
     FROM `course17.gwz_sales_17`
     ORDER BY
     customers_id
     ,orders_id
     ,products_id
 )
 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,nb_products
     ,IF(nb_products=1,'single products','multi products') AS orders_type
 FROM sales_orders_type
 ORDER BY
     date_date
     ,orders_id
```


</details>


## 1.2. Product value as a Percentage of Turnover

What proportion of the total order turnover does a single product represent? This might help us identify whether it is the main product of the order or a secondary product.

1) Add a new column orders_turnover to the gwz_sales_17 table that contains the total turnover of the associated order. using the SUM() OVER window function.

Note - we could have done a GROUP BY on orders_id and then a join, but it’s quicker to do it in one calculation with the window function.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,SUM(turnover) OVER (PARTITION BY orders_id) AS orders_turnover
 FROM `course17.gwz_sales_17`
 ORDER BY
     customers_id
     ,orders_id
     ,products_id
```

</details>



2) Add a new percent_turnover column which contains the turnover proportion that each product in a given order represents. Express this proportion as a percentage and round it to 2 digits after the decimal place.

The calculation is done in 2 steps:

- calculate the new orders_turnover column.
- calculate a new percent_turnover column based on the orders_turnover column.

Use the WITH AS clause to combine the 2 subqueries into a single query. Run it to check the results and save the query as a view called gwz_sales_17_turnover_percent.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH sales_orders_turnover AS
     (SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,category_1
     ,turnover
     ,turnover-purchase_cost AS margin
     ,SUM(turnover) OVER (PARTITION BY orders_id) AS orders_turnover
     FROM `course17.gwz_sales_17`
     ORDER BY
     customers_id
     ,orders_id
     ,products_id)

 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,category_1
     ,turnover
     ,margin
     ,ROUND(orders_turnover,2) AS orders_turnover
     ,SAFE_DIVIDE(turnover,orders_turnover) AS percent_turnover
 FROM sales_orders_turnover
 ORDER BY
     date_date
     ,orders_id
```


</details>


## 1.3. Distinct “Level 1” Categories
Are there several distinct Level 1 product categories (category_1) present in the order? This can help us identify whether the order contains only a specific product category or whether it is a multi-category order.

1) For each row of gwz_sales_17, calculate the number of distinct products in the order which belong to category_1. Give the column a name nb_cat_1.

Note - we could have done a GROUP BY on orders_id and then a join, but it’s faster to do it in one calculation with the window function.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
-- Option 1: GROUP BY then JOIN
 WITH orders AS (
     SELECT
     ### Key ###
     orders_id
     ###########
     ,COUNT(DISTINCT category_1) AS nb_cat_1
     FROM `course17.gwz_sales_17`
     GROUP BY
     orders_id
 )

 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,nb_cat_1
 FROM `course17.gwz_sales_17`
 LEFT JOIN orders USING (orders_id)
 ORDER BY
     customers_id
     ,orders_id
     ,products_id

 -- Option 2: WINDOW FUNCTION

 SELECT
     date_date,
     ### Key ###
     orders_id,
     products_id,
     customers_id,
     ###########
     COUNT(DISTINCT category_1) OVER (PARTITION BY orders_id) AS nb_cat_1
 FROM `course17.gwz_sales_17`
```


</details>


2) Add a new column orders_cat_type with the value mono-category if nb_cat_1 = 1 and the value multi-category if nb_cat_1 > 1.

The calculation is done in 2 steps:
- calculate the new column nb_cat_1.
- calculate a new column orders_cat_type based on nb_cat_1

Use the WITH AS clause to combine the 2 subqueries into a single query. Run it to check the results and save the query as a view called gwz_sales_17_cat1.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH sales_cat_1 AS
     (SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,category_1
     ,COUNT(DISTINCT category_1) OVER (PARTITION BY orders_id) AS nb_cat_1
     FROM `course17.gwz_sales_17`
     ORDER BY
     customers_id
     ,orders_id
     ,products_id
     ,category_1)

 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,category_1
     ,nb_cat_1
     ,IF(nb_cat_1=1,"mono category","multi category") AS orders_cat_type
 FROM sales_cat_1
 WHERE TRUE
     -- AND orders_id IN (974525,974532,974541)
 ORDER BY
     date_date
     ,orders_id
```


</details>


# 2. Decomposing Window Functions into GROUP BY + JOIN

Window functions with an aggregation function (SUM(), COUNT(), AVG()) are just a combination of a 2-step transformation:

- a GROUP BY on the PARTITION BY stored in a temporary result set.
- a JOIN between the source table and the temporary result set.


## 2.1. Total Turnover

For each row in the gwz_sales_17 table let’s calculate the total turnover of the associated order again, but this time let’s use a GROUP BY + JOIN .

1) Let’s do a two-step calculation:

- GROUP BY orders_id to calculate the total turnover per order and name this column orders_turnover. Write the code as a subquery.
- Make a JOIN between your subquery and the gwz_sales_17 table to add the orders_turnover column to gwz_sales_17.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH orders AS (
     SELECT
     ### Key ###
     orders_id
     ###########
     ,SUM(turnover) AS orders_turnover
     FROM `course17.gwz_sales_17`
     GROUP BY
     orders_id
 )

 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,orders_turnover
 FROM `course17.gwz_sales_17`
 LEFT JOIN orders USING (orders_id)
 ORDER BY
     customers_id
     ,orders_id
     ,products_id
```


</details>


## 2.2. Distinct “Level 1” Categories

Let’s calculate the number of products belonging to a distinct category_1 in a given order and add these values to each row in the gwz_sales_17 table again. But this time let’s use a GROUP BY + JOIN.

For each row of gwz_sales_17 calculate the number of distinct products belonging to a specific category_1 in that order. Use GROUP BY + JOIN instead of window functions.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH orders AS (
     SELECT
         ### Key ###
         orders_id
         ###########
         ,COUNT(DISTINCT category_1) AS nb_cat_1
     FROM `course17.gwz_sales_17`
     GROUP BY
         orders_id
     )

 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,nb_cat_1
 FROM `course17.gwz_sales_17`
 LEFT JOIN orders USING (orders_id)
 ORDER BY
     customers_id
     ,orders_id
     ,products_id
```


</details>



# Finishing Up
º
In this exercise we’ve seen how we can use window functions and aggregations. You may have noticed that there is usually more than one way to write a query to get the result you want. Try to use window functions to keep your queries clean and readable.
