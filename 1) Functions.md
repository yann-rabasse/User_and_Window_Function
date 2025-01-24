# Instructions

The challenge will be considered complete when you submit the URL of the document you have been working on. Make sure that the URL is in share mode and accessible by the teacher.
If there is no submission option on the exercise, click on the “I’m done” button.

In this challenge we’ll create our first user-defined functions. Re-usable SQL script that can be used to keep our SQL queries clean and easy to read.

In this challenge we’ll be using data from Greenweez, the organic e-commerce website:

Mail Campaigns
NPS (Net Promoter Score)

## Sources

[gwz_mail](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_mail_17)


[gwz_nps_17](https://console.cloud.google.com/bigquery?project=data-analytics-bootcamp-363212&ws=!1m5!1m4!4m3!1sdata-analytics-bootcamp-363212!2scourse17!3sgwz_nps_17)


# Greenweez Mail

## 1.1) is_mail_be Function
The following query returns the different Greenweez mail campaigns:

```
SELECT
  ### Key ###
  journey_id
  ###########
  ,journey_name
  ,sent_nb
FROM `course17.gwz_mail_17`
ORDER BY
  sent_nb DESC
```

If the campaigns are sent to Belgian customers, the string in the journey_name column contains “nlbe”.

**Our goal is to create a function that returns 1 if the mail campaign was sent to Belgium or 0 if it was sent somewhere else.**

Let’s break it down into a few steps:

1) Start by extending the query provided above. Create a new column: mail_be, which will equal to 1 if the mail campaign is sent to Belgium. Otherwise it will equal 0.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     ### Key ###
     journey_id
     ###########
     ,journey_name
     ,IF(journey_name LIKE "%_nlbe_%",1,0) AS mail_be
     ,sent_nb
 FROM `course17.gwz_mail_17`
 ORDER BY
     sent_nb DESC
```


</details>
    
2)Create a function is_mail_be in the course17 dataset that takes the journey_name as input and returns 1 if the mail campaign is sent to Belgium and 0 otherwise.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION course17.is_mail_be(journey_name STRING) AS
     (IF(journey_name LIKE "%nlbe%",1,0))
```


</details>
    

[Create User defined function documentation](https://cloud.google.com/bigquery/docs/reference/standard-sql/user-defined-functions)

3) Update your query from step one by using the new is_mail_be function to create the mail_be column. When creating columns remember to give them a useful name using an alias and the AS clause.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     ### Key ###
     journey_id
     ###########
     ,journey_name
     ,course17.is_mail_be(journey_name) AS mail_be
     ,sent_nb
 FROM `course17.gwz_mail_17`
 ORDER BY
     sent_nb DESC
```


</details>



## 1.2. mail_type Function
There are 3 different types of mail:

- newsletter - if the journey_name contains “nl” or “nlbe”.
- abandoned_basket - if the journey_name contains “panier_abandonne” (or “abandoned_basket” in 🇬🇧)
- back_in_stock - if the journey_name contains “back_in_stock”.

The data could be easier to analyse if this column is deconstructed into separate columns.

**Our goal is to create a function that classifies mail type based on journey_name**

1) Create a function called mail_type in the course17 dataset that contains one of the 3 above mentioned mail types depending on the journey_name.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

``` 
CREATE OR REPLACE FUNCTION `course17.mail_type`(journey_name STRING) AS (
 CASE
     WHEN journey_name LIKE "%panier_abandonne%" THEN "abandoned_basket"
     WHEN journey_name LIKE "%back_in_stock%" THEN "back_in_stock"
     WHEN journey_name LIKE "%nl%" THEN "newsletter"
     ELSE NULL
     END
 );
```


</details>

2)Select all the columns from the gwz_mail_17 table and add a mail_type column to the results by using the mail_type function you created.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
SELECT
     ### Key ###
     journey_id
     ###########
     ,journey_name
     ,course17.is_mail_be(journey_name) AS mail_be
     ,course17.mail_type(journey_name) AS mail_type
     ,sent_nb
 FROM `course17.gwz_mail_17`
 ORDER BY
     sent_nb DESC
```

</details>

## 2. NPS and Deliveries


### 2.1. NPS Function

The following query returns the NPS scores (global_note) for different orders.

```
SELECT
    date_date
    ,orders_id
    ,transporter
    ,global_note
FROM `course17.gwz_nps_17`
A quick reminder of the NPS function:
```

NPS = % promoters - % detractors, where:
- Detractors give an overall score between 0 and 6
- Passives give an overall score between 7 and 8
- Promoters give an overall score between 9 and 10 🟢

![01-Functions-asset-1-Untitled](https://github.com/user-attachments/assets/7a44f9bf-52eb-49de-a1f6-3e49c65a57bf)


We can cleanly classify a customer as a detractor, passive, or promoter - a perfect place to use a function!

**Our goal is to create a function that returns -1 if the NPS (global_note) for an order is a detractor, 0 if the NPS is passive, or 1 if the NPS is a promoter.**

Like before, let’s break it down into steps:

    1) Extend the previous query by adding an nps column that equals to -1 if they are a detractor, 0 if they are passive, and 1 if the customer is a promoter.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,transporter
     ,global_note
     ,CASE
     WHEN global_note IN (9,10) THEN 1
     WHEN global_note IN (7,8) THEN 0
     WHEN global_note BETWEEN 0 and 6 THEN -1
     ELSE NULL
     END AS nps
 FROM `course17.gwz_nps_17`
```


</details>

2) Create an nps function in the course17 dataset that takes global_note as an input and returns the same values as the nps column created in the step above.

[Create User defined function documentation](https://cloud.google.com/bigquery/docs/reference/standard-sql/user-defined-functions)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.nps`(global_note INT64) AS (
 CASE
     WHEN global_note IN (9,10) THEN 1
     WHEN global_note IN (7,8) THEN 0
     WHEN global_note BETWEEN 0 and 6 THEN -1
     ELSE NULL
     END
 );
```


</details>


3) Select all the columns from the gwz_nps_17 table and add an nps column to the results by using the nps function you created.

**Target**:

![01-Functions-asset-2-Untitled](https://github.com/user-attachments/assets/fae65bd1-4f7a-4c3e-b26d-8c452d59f2d7)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,transporter
     ,global_note
     ,course17.nps(global_note) AS nps
 FROM `course17.gwz_nps_17`
```


</details>


### 2.2. Transporter (carrier) Function

There are 2 different companies completing the deliveries:

- Chrono
- DPD

There are 2 kinds of delivery modes:

- Pickup point delivery
- Home delivery

We might be able to derive some insights between NPS and the logistics company or delivery mode.

**From the transporter column, create two new columns: one with of the delivery company, and one with the delivery mode.**

1) Create a transporter_brand function in the course17 dataset that takes the transporter column as an input and returns one of the 2 above-mentioned transporter brands.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.transporter_brand`(transporter STRING) AS (
 CASE
     WHEN transporter LIKE "%Chrono%" THEN "Chrono"
     WHEN transporter LIKE "%DPD%" THEN "Dpd"
     ELSE NULL
     END
 );
```


</details>

2) Create a delivery_mode function in the course17 dataset that takes the transporter column as an input and returns one of the 2 above-mentioned delivery modes.

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 CREATE OR REPLACE FUNCTION `course17.delivery_mode`(transporter STRING) AS (
 CASE
     WHEN transporter LIKE "%Pickup%" THEN "Pickup"
     WHEN transporter LIKE "%Home%" THEN "Home"
     ELSE NULL
     END
 );
```


</details>

3) Select all the columns from the gwz_nps_17 table and add the nps, delivery_mode, and transporter_brand columns to it using the new functions you created.


**Target:**

![01-Functions-asset-3-Untitled](https://github.com/user-attachments/assets/8ba18797-6b29-452a-8ab8-804b77d98fe6)

<details>
    <summary> <font color="red"><b>Answer</b></font></summary>

```
 SELECT
     date_date
     ,orders_id
     ,course17.delivery_mode(transporter) AS delivery_mode
     ,course17.transporter_brand(transporter) AS transporter_brand
     ,transporter
     ,global_note
     ,course17.nps(global_note) AS nps
 FROM `course17.gwz_nps_17`
```


</details>
