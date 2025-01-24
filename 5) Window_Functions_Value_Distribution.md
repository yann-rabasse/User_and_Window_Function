# 1. Operational Margin at a Sales Level

The sales table gwz_sales_17 contains information on products, orders, turnover and margin on the product granularity level. This means each row is a single product in an order. Let’s say you want to enrich your sales table and calculate the operating margin by adding data about operational costs that are stored in the gwz_ship_17 table. The shipping table contains information on an order granularity level. Using window functions, we can alter the granularity of the sales table to do order level enrichment and analysis.

## 1.1. Losing Sales Granularity

The classic transformation to achieve this operation is done in two steps:

- Aggregation of gwz_sales_17 on the orders_id column.
- Join between the aggregated sales table and gwz_ship_17.

If you perform this classic transformation, the final result has granularity on orders. You lose the granularity on sales. You don’t have the product information (product ids, product categories). You cannot analyse the operational_margin on a product or product category level anymore.

![05-Window-Functions-Value-Distribution-asset-1-Untitled](https://github.com/user-attachments/assets/70e8e466-6df2-442f-b203-cbfdbceea9e3)



Aggregating then joining. The sales table has been aggregated to change it's granularity, then it is joined to orders_operational table on the orders_id key. The output table is shown.

## 1.2. Errors when joining Table with different Granularity

If you simply join the gwz_sales_17 and gwz_ship_17 tables, you will duplicate all the operational cost values. If you want to perform aggregation operations on operational costs, the results won’t be accurate, because the operational costs refer to each order, not to each product in an order.

![05-Window-Functions-Value-Distribution-asset-2-Untitled](https://github.com/user-attachments/assets/7c644184-f98e-40f5-9cd8-a8afc6f431f1)


Direct join. The left of the image has two tables, sales and orders_operational on the same granularity. The output of the join when joined on the orders_id key is depicted on the left.

You can solve this by value distribution.

##1.3. Value Distribution

There are two steps to the process:

- Calculate what is the proportion that the more fine grained elements take up in the more coarse element. For example: what is the proportion of order revenue that a specific product in an order has generated?
- Join the tables with different levels of granularity and distribute the value of the metric across the more fine grained elements by using your proportion column.

![05-Window-Functions-Value-Distribution-asset-3-Untitled](https://github.com/user-attachments/assets/7cb8fad3-aca0-4ef3-b9eb-ce8304351b9f)

Window function and join. There are two tables on the left: sales and orders_operational. The sales function has had the granularity changed with a window function and is then joined to orders_operational on the orders_id key. The output table is shown on the right.


1) From the gwz_sales_17 table calculate the percent_turnover column, which contains the percentage of turnover each product has generated in its corresponding order. Save the result in the gwz_sales_17_turnover_percent table.



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



2) Add the operational cost columns from gwz_ship_17 to gwz_sales_17_turnover_percent by multiplying them with the percent_turnover column to distribute the operational costs per order across the different products. Save the results in the gwz_sales_17_operational table.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ### Key ###
     ,orders_id
     ,products_id
     ###########
     -- sales table --
 ,tu.category_1
     ,tu.turnover
     ,tu.margin
     -- ship table --
     ,tu.percent_turnover
     ,ROUND(sh.shipping_fee*tu.percent_turnover,2) AS shipping_fee
     ,ROUND(sh.log_cost*tu.percent_turnover,2) AS log_cost
     ,ROUND(sh.ship_cost*tu.percent_turnover,2) AS ship_cost
 FROM `course17.gwz_sales_17_turnover_percent` AS tu
 INNER JOIN `course17.gwz_ship_17`AS sh USING (orders_id)
 -- WHERE TRUE
 --   AND orders_id IN (974525,974532,975456)
 ORDER BY
     date_date
     ,orders_id
```


</details>


### Test


This is a full join, we need to do a retention test to check that the data is distributed correctly.

1) Create a value conservation test:

- gwz_sales_17_operational_conservation-ship1 in which you sum the shipping_fee column in the source table gwz_ship_17
- gwz_sales_17_operational_conservation-ship2 in which you sum the shipping_fee column in the joined table gwz_sales_17_operational

Compare and make sure you have the same result in both. You could have a value difference of 1 or 2 digits after the decimal place. This is due to FLOAT approximation and isn’t a problem.



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>


![05-Window-Functions-Value-Distribution-asset-4-Untitled](https://github.com/user-attachments/assets/61cf81ab-378f-4294-a9b9-7602af24cc48)


</details>



<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 -- gwz_sales_17_operational_conservation-ship1
 SELECT
     ROUND(SUM(shipping_fee)) AS nb
 FROM `course17.gwz_ship_17`


 -- gwz_sales_17_operational_conservation-ship2
 SELECT
     ROUND(SUM(shipping_fee)) AS nb
 FROM `course17.gwz_sales_17_operational`
```


</details>



!!! You have well-distributed operational statistics in the sales table!!!

## 1.4. Analysis

1) From the gwz_sales_17_operational table display the following metrics aggregated by **category_1**:

- turnover
- margin
- margin percent
- operational margin
- operational margin percent


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     ### Key ###
     op.category_1
     ###########
     ,ROUND(SUM(op.turnover)) AS turnover
     ,ROUND(SUM(op.margin)) AS margin
     ,ROUND(SUM(op.margin)/SUM(t.turnover)*100,1) AS margin_percent
     ,ROUND(SUM(op.margin+shipping_fee-op.ship_cost-op.log_cost)) AS operational_margin
     ,ROUND(SUM(op.margin+shipping_fee-op.ship_cost-op.log_cost)/SUM(op.turnover)*100,1) AS operational_margin
 FROM `course17.gwz_sales_17_operational` AS op
 GROUP BY 1
```

</details>


You can now analyze the operational margin at the category level! If some products in a category have a high margin but are heavy or are purchased as one-offs, this can affect the operational cost significantly. The overall profitability of the product can be different if this is taken into account. Value distribution has allowed us to perform a more in-depth analysis.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

![05-Window-Functions-Value-Distribution-asset-5-Untitled](https://github.com/user-attachments/assets/6c48aa26-4cfd-4169-87f0-1347f5c064c8)



</details>


# 2. Ads Margin at Orders Level


Check that you have the gwz_campaign_17 table in the ‘course17’ dataset of your BigQuery project. If not, copy it from here.

Our aim is to calculate the ads margin for each order. As the gwz_campaign_17 table containing the ads cost represents information on the date granularity, but the gwz_orders_17 table containing information about orders represents data on a different level of granularity, our only option without using value distribution would be to adopt the coarser granularity level, date in this scenario, to calculate and display the ads margin correctly.

But by using value distribution, we can proportionately split the ads cost across different orders by calculating what proportion from the daily turnover does the revenue from each order constitute.

1) Aggregate ads_cost from the gwz_campaign_17 table on the date_date column for dates between ‘2021-04-01’ and ‘2021-09-30’ and save it as a view gwz_campaign_17_date.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,SUM(ads_cost) AS ads_cost
 FROM `course17.gwz_campaign_17`
 WHERE
     date_date BETWEEN "2021-04-01"  and "2021-09-30"
 GROUP BY
     date_date
```


</details>



2) Calculate the percent_turnover_date from the gwz_orders_17 table by giving the proportion of the turnover of an order from the total turnover of the day. Save it in the gwz_orders_17_percent_turnover view.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 WITH orders_turnover AS
     (SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,margin
     ,shipping_fee
     ,operationnal_cost
     ,SUM(turnover) OVER (PARTITION BY date_date) AS turnover_date
     FROM `course17.gwz_orders_17`)

 SELECT
     date_date
     ,customers_id
     ### Key ###
     ,orders_id
     ###########
     ,turnover
     ,margin
     ,shipping_fee
     ,operationnal_cost
     ,turnover/turnover_date AS percent_turnover_date
 FROM orders_turnover
 ORDER BY
     date_date
     ,orders_id
```


</details>


3) Distribute ads_cost from gwz_campaign_17_date to the gwz_orders_17_percent_turnover table using the percent_turnover_date column. Calculate the ads_margin for each order. Save the results in the gwz_orders_17_ads view.

🎯 Target

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

![05-Window-Functions-Value-Distribution-asset-5-Untitled](https://github.com/user-attachments/assets/d0c257c3-8c6f-465b-868c-87120f0cd051)


</details>


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
WITH orders_join AS (
     SELECT
     tu.date_date
     ,tu.customers_id
     ### Key ###
     ,tu.orders_id
     ###########
     -- orders table --
     ,tu.turnover
     ,tu.margin
     ,tu.shipping_fee
     ,tu.operationnal_cost
     ,tu.margin+tu.shipping_fee-tu.operationnal_cost AS operationnal_margin
     ,tu.percent_turnover_date
     -- ads table --
     ,c.ads_cost*tu.percent_turnover_date AS ads_cost
     FROM `course17.gwz_orders_17_percent_turnover` AS tu
     INNER JOIN `course17.gwz_campaign_17_date` AS c USING (date_date)
 )

 SELECT
     o.date_date
     ,o.customers_id
     ### Key ###
     ,o.orders_id
     ###########
     -- orders table --
     ,o.turnover
     ,o.margin
     ,o.shipping_fee
     ,o.operationnal_cost
     ,o.operationnal_margin
     ,o.ads_cost
     ,o.operationnal_margin-o.ads_cost AS ads_margin
 FROM orders_join AS o
```


</details>


4) Create a conservation test for the two views: gwz_campaign_17_date and gwz_orders_17_ads.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

The tests do not return equal results. This is not normal. We will solve this in the next step.


- **“gwz_orders_17_ads_conservation-cost1”** where you sum the ads_cost of the join table “gwz_orders_17_ads “ with 0 digits.

```
 SELECT
  ROUND(SUM(ads_cost)) AS nb
 FROM `course17.gwz_orders_17_ads`
```

- **“gwz_orders_17_ads_conservation-cost2”** where you sum the ads_cost from the source table “gwz_campaign_17_date” with 0 digits


```
 SELECT
  ROUND(SUM(ads_cost)) AS nb
 FROM `course17.gwz_campaign_17_date`
```


</details>

(Optional) The following query allows you to identify the unique key (date_date) on which you have a conservation value difference between the source table (gwz_campaign_17_date) and the joined table (gwz_orders_17_ads). Writing and understanding all parts of this query is complex.

```
 WITH t1 AS
     (SELECT
         date_date
         ,ROUND(SUM(ads_cost)) AS nb
     FROM `course17.gwz_orders_17_ads`
     GROUP BY date_date
     )

     ,t2 AS
     (SELECT
         date_date
         ,ROUND(SUM(ads_cost)) AS nb
     FROM `course17.gwz_campaign_17_date`
     GROUP BY date_date
     )

 SELECT
     t1.*
     ,t2.*
 FROM t1
 FULL OUTER JOIN t2 USING (date_date)
 WHERE
     t1.date_date IS NULL
     OR t2.date_date IS NULL
     OR IFNULL(t1.nb,-1)!=IFNULL(t2.nb,-1)
 ORDER BY IFNULL(t1.date_date,t2.date_date) DESC
```

Run it and identify the problem.


<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

The explanation is that in gwz_orders_17_percent_turnover you only have data up to August 2021. We need to update gwz_campaign_17_date to limit the date_range between ‘2021-04-01’ and ‘2021-08-31’.


</details>


6) **(optional)** Update gwz_campaign_17_date table by limiting the date range between ‘2021-04-01’ and ‘2021-08-31’. Rerun the conservation test. It should work fine, if you run into any issues, open a ticket and have a chat with a TA!



