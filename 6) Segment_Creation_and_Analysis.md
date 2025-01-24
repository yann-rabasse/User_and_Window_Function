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

<video width="1080" controls="">
 <source src="https://wagon-public-assets.s3.eu-west-3.amazonaws.com/03-Data-Transformation/05-Nested-Formats-And-Complex-Aggregations/06-Segment-Creation-And-Analysis-asset-1-CreatePartitionTable.mp4" type="video/mp4">
 Your browser does not support the video tag.
 </video>
 
Using ROW_NUMBER() and WITH ... AS(), add the following columns to the gwz_orders_17_partitioned:

orders_number which contains the order number for a given customer from the oldest to the most recent order.

is_new to identify new orders.

Save the query in a new view gwz_orders_17_segment.

🎯 Target
Solution 🔓
2. Segment by Order Frequency
Create a new global_sgt function that returns the order segment based on the orders_number column. Store the function in the course17. The order segments should be defined as follows:

“New” if it is the first order.

“Occasional” if it is the 2nd, 3rd or 4th order.

“Frequent” if it is the 5th or more order.

Solution 🔓
Use this function to add the global_sgt column to the gwz_orders_17_segment view. Update and save the view.

Solution 🔓
3. Time Between Orders
We would like to monitor the average time between orders from the same customer. If it decreases, it means that the customer is buying more frequently. If it increases, we have to think of a targeted customer re-engagement strategy to potentially not lose the customer altogether.

Perform a self-join on the gwz_orders_17_segment view to add columns containing information about other orders that the same customer made.

💡 Help - What is a self-join?
Solution 🔓
Edit the previous self-join query to add a calculation for a new column named repurchase_duration. This column should contain the date difference in days between the current order and the previous order. The “new” orders will have a NULL value for this column, because there is no previous order.

Solution 🔓
Use subqueries to add this calculation directly into the gwz_orders_17_segment. Update and save the view.

Solution 🔓
4. Category Segmentation
We want to segment the orders according to the categories of products included in the order.

Create a partitioned table gwz_sales_17_partitioned with a partition on the date_date column and populate it with the data from the gwz_sales_17 table.

Create a sgt_category function which, depending on the value in category_1, returns the corresponding category segment:

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
Solution 🔓
Add a category_sgt column to gwz_sales_17_partitioned using the sgt_category function. Create and run the query

Solution 🔓
Edit the previous query and aggregate gwz_sales_17_partitioned over orders_id and category_sgt, and sum the turnover.

Solution 🔓
Put the previous query into a subquery, and by using ROW_NUMBER() rank the orders according to their turnover. Save the query in a new view gwz_sales_17_cat_sgt.

Solution 🔓
Perform a join between gwz_orders_17_segment and gwz_sales_17_cat_sgt to add the main product segment of each order in a new column called category_sgt. Save the results in a new table.

Solution 🔓
🎉 Congratulations, you have the category_sgt column in the orders. It corresponds to the main product category represented in the order. This can be a very interesting additional dimension to add to your analysis. 🎉

Have you already done everything?
Create an account on Hacker rank and perform as many challenges as you can.

❗ Indications on how to use Hacker rank ❗
If you prefer, you can also deep-dive into today’s datasets to create a more in-depth analysis report.
