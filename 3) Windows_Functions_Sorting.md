# Count and Rank

Window functions can be used to sort/rank or count lines. In this exercise we’ll use window functions to identify a customers first order.

# 1. Orders

## 1.1. Sources

[Download Here](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_orders_17)


## 1.2. Identifying New Orders
The gwz_orders_17 table contains all the orders of our customers until “2021-08-31”.

We consider an order to be new if it is the first order from a customer. To find the first order of each customer, we need to sort the orders from the oldest date to the most recent date.

1) Use the ROW_NUMBER() function to add a new column called rn (row number) to the gwz_orders_17 table. For each customer, the new column should contain a number that indicates how recent the order was, with 1 being the oldest (or first) order, 2 being the second oldest, etc. You will have to specify which column you want the table to be PARTIONED by for the ROW_NUMBER() function to work. You can check out the documentation [here](https://cloud.google.com/bigquery/docs/reference/standard-sql/numbering_functions#row_number).


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date) AS rn
 FROM `course17.gwz_orders_17`
 ORDER BY
     customers_id
     ,date_date
     ,rn
```


</details>


2) Use the RANK() function to achieve the same thing in another column called rk.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date) AS rn
     ,RANK() OVER (PARTITION BY customers_id ORDER BY date_date) AS rk
 FROM `course17.gwz_orders_17`
 ORDER BY
     customers_id
     ,date_date
     ,rn
```


</details>


3) Create a query to compare the values in the rn and rk columns for the customers with customers_id in (212497,205343,33690). How would you explain the difference between the two values?

Our goal is to:
- Have a single orders_number for each row
- Identify a single order as the first one and flag it as new.

Knowing this, which function seems more relevant to you?


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

Some customers place several orders on the same date. When using ROW_NUMBER(), an arbitrary order is made between orders of the same date, which ensures that there are no rows with the same order number. When using RANK(), the same rank is assigned to orders made on the same day, which is not what we want as we could have several first orders for the same customer. In our scenario, using ROW_NUMBER() is more appropriate.

```
 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date) AS rn
     ,RANK() OVER (PARTITION BY customers_id ORDER BY date_date) AS rk
 FROM `course17.gwz_orders_17`
 WHERE customers_id IN (33690,205343,212497)
 ORDER BY
     customers_id
     ,date_date
     ,rn
```

 Note - orders_id is an incremental id affected to order. The smaller it is, the older the order is. If we want to distinguish same-date orders we could ORDER BY orders_id after date_date

```
SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rn
     ,RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rk
 FROM `course17.gwz_orders_17`
 WHERE customers_id IN (33690,205343,212497)
 ORDER BY
     customers_id
     ,date_date
     ,rn
```

</details>


4) Add a new column is_new which is equal to 1 if the order is the first customer order and 0 otherwise. The calculation is done in 2 steps:

calculate the rn column using the ROW_NUMBER() function.

calculate the column is_new based on the rn column.

Use the WITH AS clause to combine the 2 steps into a single query. Save the results in a new table gwz_orders_17_rn.

**Target**

![03-Window-Functions-Sorting-asset-1-Untitled](https://github.com/user-attachments/assets/c18c7fc5-c8d4-468b-a2c8-11facf4ad91a)



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
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rn
     FROM `course17.gwz_orders_17`
     WHERE TRUE
     -- AND customers_id IN (33690,205343,212497)
     ORDER BY
     customers_id
     ,date_date
     ,rn)

 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,rn
     ,IF(rn=1,1,0) AS is_new
 FROM orders_rn
 ORDER BY
     customers_id
     ,date_date
     ,rn
```


</details>


# 2. Sales

## 2.1. Data Import

Make sure you have the gwz_sales_17 table in the course17 dataset of your Bigquery project. If you don’t, you can copy it from here.

2.2. Identifying New Sales
Like we did in the previous section, our aim is to identify and mark all the rows corresponding to new orders in gwz_sales_17

One way to achieve this is with window functions!

Note - we could also use a join with gwz_orders_17, but our aim is to practice window functions.

1) Use the ROW_NUMBER() function to add a new column called rn (row number) to the gwz_sales_17 table. For each customer, the new column should contain a number that designates the order that the customer made their orders in from the oldest to the newest.

Note - You will need to add orders_id after date_date in the ORDER BY clause.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rn
 FROM `course17.gwz_sales_17`
 ORDER BY
     customers_id
     ,date_date
     ,orders_id
     ,rn
```


</details>
2) Use the RANK() function to achieve the same thing in another column called rk.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rn
     ,RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rk
 FROM `course17.gwz_sales_17`
 ORDER BY
     customers_id
     ,date_date
     ,orders_id
     ,rn
```


</details>

3) Use the DENSE_RANK() function to achieve the same thing in another column called ds_rk.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rn
     ,RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rk
     ,DENSE_RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS ds_rk
 FROM `course17.gwz_sales_17`
 ORDER BY
     customers_id
     ,date_date
     ,orders_id
     ,rn
```


</details>

4) Create a query to compare the values in the rk and ds_rk columns for the customers with customers_id in (98869,217071,268263). How to explain the difference between the two values?

Our goal is to:

- have a single orders_number for each row.
- identify a single order as the first one and flag it as new.

So, which function seems more relevant to you?

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

When using RANK(), the products_id from the same orders have the same values. However, the rank values vary from order to order depending on the number of products included in each order.

DENSE_RANK() looks more appropriate. All products with the same orders_id have the same values. There is no difference based on how many products there are in each order.

```
 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,ROW_NUMBER() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rn
     ,RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS rk
     ,DENSE_RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS ds_rk
 FROM `course17.gwz_sales_17`
 WHERE TRUE
     AND customers_id IN (98869,217071,268263)
 ORDER BY
     customers_id
     ,date_date
     ,orders_id
     ,rn
```


</details>


5) Add a new column is_new which is equal to 1 if the order is the first customer order and 0 otherwise. The calculation is done in 2 steps:

- calculate the ds_rk column using the DENSE_RANK() function.
- calculate the column is_new based on the ds_rk column.

Use a WITH AS clause to combine the 2 steps into a single query. Save the results in a new table called gwz_sales_17_ds_rk.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH sales_ds_rk AS
     (SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,DENSE_RANK() OVER (PARTITION BY customers_id ORDER BY date_date,orders_id) AS ds_rk
     FROM `course17.gwz_sales_17`
     ORDER BY
     customers_id
     ,date_date
     ,orders_id
     ,ds_rk)

 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     ,ds_rk
     ,IF(ds_rk=1,1,0) AS is_new
 FROM sales_ds_rk
 ORDER BY
     customers_id
     ,date_date
     ,orders_id
```


</details>
