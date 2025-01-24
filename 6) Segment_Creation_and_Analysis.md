This exercise is more difficult. Providing less hints and supporting information. 🧠

Our aim it to enrich the orders information by identifying:
- new orders
- order segments based on purchase frequency
- time between orders
- order segments based on the products in the order

# 1. New Orders

1) Create a partitioned table gwz_orders_17_partitioned with a partition on the date_date column and populate it with the data from the gwz_orders_17 table.

You can watch the video tutorial and follow the steps below:

- create an empty table gwz_orders_17_partitioned with one column date_date (don’t forget to specify that it’s a DATE type column in the Schema!)
- under the “partition and cluster settings” choose “partition by field” -> date_date
- choose “partitioning type” -> by day
- write a query that selects all columns from the gwz_orders_17 table. In the query settings, set a destination for the query results to gwz_orders_17_partitioned. In the settings, select “Overwrite table” and run the query

https://wagon-public-assets.s3.eu-west-3.amazonaws.com/03-Data-Transformation/05-Nested-Formats-And-Complex-Aggregations/06-Segment-Creation-And-Analysis-asset-1-CreatePartitionTable.mp4

 
2) Using ROW_NUMBER() and WITH ... AS(), add the following columns to the gwz_orders_17_partitioned:

- orders_number which contains the order number for a given customer from the oldest to the most recent order.
- is_new to identify new orders.

Save the query in a new view gwz_orders_17_segment.


<details>
    <summary> <font color="red"><b>Target</b></font></summary>

![06-Segment-Creation-And-Analysis-asset-2-Untitled](https://github.com/user-attachments/assets/7b75fc89-320a-4c82-baa7-820fc1833f51)


</details>



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH orders_rn AS
     (SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS orders_number
     FROM `course17.gwz_orders_17_partitioned`
     ORDER BY
     customers_id
     ,date_date
     ,orders_number)

 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,orders_number
     ,IF(orders_number=1,1,0) AS is_new
 FROM orders_rn
 ORDER BY
     customers_id
     ,orders_number
```


</details>


# 2. Segment by Order Frequency

1) Create a new global_sgt function that returns the order segment based on the orders_number column. Store the function in the course17. The order segments should be defined as follows:

- “New” if it is the first order.
- “Occasional” if it is the 2nd, 3rd or 4th order.
- “Frequent” if it is the 5th or more order.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.global_sgt`(orders_number INT64) AS (
 CASE
     WHEN orders_number=1 THEN "New"
     WHEN orders_number IN (2,3,4) THEN "Occasional"
     WHEN orders_number >= 5 THEN "Frequent"
     ELSE NULL
     END
 );
```


</details>


2) Use this function to add the global_sgt column to the gwz_orders_17_segment view. Update and save the view.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH orders_rn AS
     (SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS orders_number
     FROM `course17.gwz_orders_17_partitioned`
     ORDER BY
     customers_id
     ,date_date
     ,orders_number)

 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,orders_number
     ,IF(orders_number=1,1,0) AS is_new
     ,`course17.global_sgt`(orders_number) AS global_sgt
 FROM orders_rn
 ORDER BY
     customers_id
     ,orders_number
```


</details>


# 3. Time Between Orders

We would like to monitor the average time between orders from the same customer. If it decreases, it means that the customer is buying more frequently. If it increases, we have to think of a targeted customer re-engagement strategy to potentially not lose the customer altogether.

1) Perform a self-join on the gwz_orders_17_segment view to add columns containing information about other orders that the same customer made.


<details>
    <summary> <font color="red"><b>Help - What is a self-join?</b></font></summary>

```

```


</details>


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     t1.date_date
     ,t1.customers_id
     ### Key ###
     ,t1.orders_id
     ###########
     ,t1.turnover
     ,t1.orders_number
     ,t1.is_new
     -- previous orders
     ,t2.date_date AS previous_date
 FROM `course17.gwz_orders_17_segment` AS t1
 LEFT JOIN `course17.gwz_orders_17_segment` AS t2 ON t1.customers_id = t2.customers_id AND t1.orders_number=t2.orders_number+1
 ORDER BY
     customers_id
     ,orders_number
```


</details>

2) Edit the previous self-join query to add a calculation for a new column named repurchase_duration. This column should contain the date difference in days between the current order and the previous order. The “new” orders will have a NULL value for this column, because there is no previous order.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     t1.date_date
     ,t1.customers_id
     ### Key ###
     ,t1.orders_id
     ###########
 ,t1.turnover
     ,t1.orders_number
     ,t1.is_new
     ,t1.global_sgt
     -- previous orders
     ,t2.date_date AS previous_date
     ,DATE_DIFF(t1.date_date,t2.date_date,DAY) AS repurchase_duration
 FROM `course17.gwz_orders_17_segment` AS t1
 LEFT JOIN `course17.gwz_orders_17_segment` AS t2 ON t1.customers_id = t2.customers_id AND t1.orders_number=t2.orders_number+1
 ORDER BY
     customers_id
     ,orders_number
```


</details>


3) Use subqueries to add this calculation directly into the gwz_orders_17_segment. Update and save the view.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH orders_rn AS
     (SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS orders_number
     FROM `course17.gwz_orders_17_partitioned`
     WHERE TRUE
     ORDER BY
     customers_id
     ,date_date
     ,orders_number)

     ,segment AS (
     SELECT
         date_date
         ,customers_id
         ### Key ###
         ,orders_id
         ###########
         ,turnover
         ,orders_number
         ,IF(orders_number=1,1,0) AS is_new
         ,`course17.global_sgt`(orders_number) AS global_sgt
     FROM orders_rn
     ORDER BY
         customers_id
         ,orders_number)

 SELECT
     t1.date_date
     ,t1.customers_id
     ### Key ###
     ,t1.orders_id
     ###########
     ,t1.turnover
     ,t1.orders_number
     ,t1.is_new
     ,t1.global_sgt
     -- previous orders
     ,t2.date_date AS previous_date
     ,DATE_DIFF(t1.date_date,t2.date_date,DAY) AS repurchase_duration
 FROM segment AS t1
 LEFT JOIN segment AS t2 ON t1.customers_id = t2.customers_id AND t1.orders_number=t2.orders_number+1
 ORDER BY
     customers_id
     ,orders_number
```


</details>


# 4. Category Segmentation

We want to segment the orders according to the categories of products included in the order.

1) Create a partitioned table gwz_sales_17_partitioned with a partition on the date_date column and populate it with the data from the gwz_sales_17 table.

2) Create a sgt_category function which, depending on the value in category_1, returns the corresponding category segment:

category_1	category_sgt
Mode, Sports, Loisirs, Livres	home and lifestyle
Beauté & Hygiène	heatlh, beauty & wealthness
Maison	home and lifestyle
Epicerie salée	daily food
Extérieur	home and lifestyle
Santé & Bien-être	heatlh, beauty & wealthness
Boisson	daily food
Epicerie sucrée	daily food
Bébé & Enfant	Kids and Baby
Frais	fresh


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.sgt_category`(category_1 STRING) AS (
 CASE
     WHEN category_1 IN ("Epicerie salée","Epicerie sucrée","Boisson") THEN "daily food"
     WHEN category_1 IN ("Beauté & Hygiène","Santé & Bien-être") THEN "heatlh, beauty & wealthness"
     WHEN category_1 IN ("Bébé & Enfant") THEN "Kids and Baby"
     WHEN category_1 IN ("Extérieur","Mode, Sports, Loisirs, Livres","Maison") THEN "home and lifestyle"
     WHEN category_1 IN ("Frais") THEN "fresh"
     ELSE NULL
     END
 );
```


</details>


3) Add a category_sgt column to gwz_sales_17_partitioned using the sgt_category function. Create and run the query


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
 *
 ,`course17.sgt_category`(category_1) AS category_sgt
 FROM `course17.gwz_sales_17_partitioned`
```


</details>


4) Edit the previous query and aggregate gwz_sales_17_partitioned over orders_id and category_sgt, and sum the turnover.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     orders_id
     ,`course17.sgt_category`(category_1) AS category_sgt
     ,SUM(turnover) AS turnover
 FROM `course17.gwz_sales_17_partitioned`
 GROUP BY 1,2
```


</details>


5) Put the previous query into a subquery, and by using ROW_NUMBER() rank the orders according to their turnover. Save the query in a new view gwz_sales_17_cat_sgt.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH cat_sgt_turnover AS
     (SELECT
     orders_id
     ,`course17.sgt_category`(category_1) AS category_sgt
     ,SUM(turnover) AS turnover
     FROM `course17.gwz_sales_17_partitioned`
     GROUP BY 1,2)

 SELECT
     orders_id
     ,category_sgt
     ,turnover
     ,ROW_NUMBER() OVER (PARTITION BY orders_id ORDER BY turnover DESC) AS rn
 FROM cat_sgt_turnover
```


</details>


6) Perform a join between gwz_orders_17_segment and gwz_sales_17_cat_sgt to add the main product segment of each order in a new column called category_sgt. Save the results in a new table.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH orders_rn AS
     (SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS orders_number
     FROM `course17.gwz_orders_17_partitioned`
     WHERE TRUE
     ORDER BY
     customers_id
     ,date_date
     ,orders_number)

     ,segment AS (
     SELECT
         o.date_date
         ,o.customers_id
         ### Key ###
         ,o.orders_id
         ###########
         ,o.turnover
         ,o.orders_number
         ,IF(o.orders_number=1,1,0) AS is_new
         ,`course17.global_sgt`(o.orders_number) AS global_sgt
         -- category_sgt
         ,sgt.category_sgt
     FROM orders_rn AS o
     LEFT JOIN `course17.gwz_sales_17_cat_sgt` AS sgt ON o.orders_id = sgt.orders_id AND sgt.rn = 1
     ORDER BY
         customers_id
         ,orders_number)

 SELECT
     t1.date_date
     ,t1.customers_id
     ### Key ###
     ,t1.orders_id
     ###########
     ,t1.turnover
     ,t1.orders_number
     ,t1.is_new
     ,t1.global_sgt
     -- previous orders
     ,t2.date_date AS previous_date
     ,DATE_DIFF(t1.date_date,t2.date_date,DAY) AS repurchase_duration
     -- category_sgt
     ,t1.category_sgt
 FROM segment AS t1
 LEFT JOIN segment AS t2 ON t1.customers_id = t2.customers_id AND t1.orders_number=t2.orders_number+1
 ORDER BY
     customers_id
     ,orders_number
```


</details>


**Congratulations!**, you have the category_sgt column in the orders. It corresponds to the main product category represented in the order. This can be a very interesting additional dimension to add to your analysis. 🎉


