# 1. Sales

## 1.1. Data Import

## Sources: 
[Download Here](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_sales_17)

[creating user-defined functions](https://cloud.google.com/bigquery/docs/reference/standard-sql/user-defined-functions)

## 1.2. Margin Percent

The following query displays sales data:

```
SELECT
    date_date
    ,orders_id
    ,products_id
    ,promo_name
    ,turnover_before_promo
    ,turnover
    ,purchase_cost
FROM `course17.gwz_sales_17`
```

1) We can enrich this data with a margin percent, the amount of profit we’re making on each product sold.

Extend the above query by creating a margin_percent column. Round the value to a single digit after the decimal place.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,products_id
     ,promo_name
     ,turnover_before_promo
     ,turnover
     ,purchase_cost
     ,ROUND(SAFE_DIVIDE(turnover-purchase_cost,turnover)*100,1) AS margin_percent
 FROM `course17.gwz_sales_17`
```


</details>

2) Create a function margin_percent in the course17 dataset that takes the purchase_cost and turnover as inputs and returns the percent margin.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.margin_percent`(turnover FLOAT64, purchase_cost FLOAT64) AS (
     ROUND(SAFE_DIVIDE(turnover-purchase_cost,turnover)*100,1)
 );
```


</details>

3) Update the query from the first step by using the new margin_percent function to create the margin_percent column.

**Target:**

![02-Functions-Sales-asset-1-Untitled](https://github.com/user-attachments/assets/20f564b4-2971-41cf-9975-0f48acd9de28)


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,products_id
     ,promo_name
     ,turnover_before_promo
     ,turnover
     ,purchase_cost
     ,course17.margin_percent(turnover, purchase_cost) AS margin_percent
 FROM `course17.gwz_sales_17`
```


</details>

## 1.3. Promotion percent

1) Add a promo_percent column to the previous query ((turnover_before_promo - turnover) / (turnover_before_promo). Round the promo_percent value to a whole number.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,products_id
     ,promo_name
     ,turnover_before_promo
     ,turnover
     ,purchase_cost
     ,course17.margin_percent(turnover, purchase_cost) AS margin_percent
     ,ROUND(SAFE_DIVIDE(turnover_before_promo-turnover,turnover_before_promo)*100,0) AS promo_percent
 FROM `course17.gwz_sales_17`
```


</details>

2) Create a function promo_percent in the course17 dataset that takes the turnover and turnover_before_promo as inputs and returns the promo_percent as a percentage as a whole number.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.promo_percent`(turnover FLOAT64, turnover_before_promo FLOAT64) AS (
     ROUND(SAFE_DIVIDE(turnover_before_promo-turnover,turnover_before_promo)*100,0)
 );
```


</details>

3) Query the gwz_sales_17 table to select all the columns and add the margin_percent and promo_percent columns by using the newly created functions.

**Target**

![02-Functions-Sales-asset-2-Untitled](https://github.com/user-attachments/assets/70797b88-5a76-44f8-9227-e761ed685071)


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,products_id
     ,promo_name
     ,turnover_before_promo
     ,turnover
     ,purchase_cost
     ,course17.margin_percent(turnover, purchase_cost) AS margin_percent
     ,course17.promo_percent(turnover, turnover_before_promo) AS promo_percent
 FROM `course17.gwz_sales_17`
```


</details>
