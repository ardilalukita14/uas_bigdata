------------------------ FILTERING DATA SALES RECEIPTS -------------------------

SELECT transaction_id,transaction_date, transaction_time, sales_outlet_id, staff_id, customer_id, product_id, quantity, line_item_amount
FROM `uas-bigdata.coffeeshop. sales reciepts`
WHERE transaction_date < '2019-04-15'
ORDER BY transaction_date, transaction_time ASC;
--------------------------------------------------------------------------------

------------------------ REDUCING DATA SALES RECEIPTS --------------------------

SELECT s.transaction_id, s.transaction_date, s.customer_id, s.line_item_amount, c.customer_first_name, c.customer_email, c.loyalty_card_number, c.customer_since, c.gender
FROM `uas-bigdata.coffeeshop. sales reciepts` AS s
INNER JOIN `uas-bigdata.coffeeshop.customer` AS c
ON s.customer_id = c.customer_id
WHERE c.birth_year >= 1999 AND s.line_item_amount <= 5.0
ORDER BY s.transaction_date ASC
LIMIT 500;
--------------------------------------------------------------------------------

---------------------- TRANSFORMATION DATA SALES OUTLET DAN SALES TARGET ------------------------
SELECT so.sales_outlet_id, so.sales_outlet_type, st.year_month, st.beverage_goal, st.food_goal,
CONCAT (so.store_address, ', ', so. store_city ) AS address_full
FROM `uas-bigdata.coffeeshop.sales_outlet` AS so
JOIN `uas-bigdata.coffeeshop.sales targets` AS st
ON so.sales_outlet_id = st.sales_outlet_id
ORDER BY so.sales_outlet_id ASC;
--------------------------------------------------------------------------------

---------------- CREATE MACHINE LEARNING MODELS IN BIGQUERY ML -----------------
CREATE OR REPLACE MODEL `uas-bigdata.coffeeshop.model_reciepts`
OPTIONS (input_label_cols=['line_item_amount'], model_type='linear_reg') AS
SELECT
sr.unit_price,
sr.quantity,
sr.line_item_amount,
p.product
FROM `uas-bigdata.coffeeshop. sales reciepts` AS sr
INNER JOIN `uas-bigdata.coffeeshop.product` AS p
ON sr.product_id = p.product_id
WHERE sr.unit_price BETWEEN 2.0 AND 10.0
AND sr.quantity >= 1
AND sr.line_item_amount >= 2.0
AND p.product IS NOT NULL;
--------------------------------------------------------------------------------

---------------- EVALUATION MODELS IN BIGQUERY ML ------------------------------
SELECT * FROM ML.EVALUATE(MODEL `uas-bigdata.coffeeshop.model_reciepts`);
--------------------------------------------------------------------------------

------------------ Predict MODEL IN BigQuery ML --------------------------------
SELECT * FROM ML.PREDICT(MODEL `uas-bigdata.coffeeshop.model_reciepts`, 
  (SELECT 3.0 as unit_price, 2 AS quantity, 'Ouro Brasileiro shot' AS product) 
);
--------------------------------------------------------------------------------

---------------- CREATE MACHINE LEARNING (CLASSIFICATION) MODELS IN BIGQUERY ML -----------------
CREATE OR REPLACE MODEL `uas-bigdata.coffeeshop.morningtime`
OPTIONS (input_label_cols=['morning_time'], model_type='logistic_reg') AS
SELECT IF(sr.transaction_time  < '12:00:00', 1, 0) AS morning_time,
c.customer_first_name,
g.generation,
sr.line_item_amount
FROM `uas-bigdata.coffeeshop. sales reciepts` AS sr
JOIN `uas-bigdata.coffeeshop.customer` AS c
ON sr.customer_id = c.customer_id
JOIN `uas-bigdata.coffeeshop.generations` AS g
ON c.birth_year = g.birth_year
WHERE sr.transaction_time IS NOT NULL;
--------------------------------------------------------------------------------

---------------- EVALUATION MODELS IN BIGQUERY ML ------------------------------
SELECT * FROM ML.EVALUATE(MODEL `uas-bigdata.coffeeshop.morningtime`);
--------------------------------------------------------------------------------

------------------ Predict MODEL IN BigQuery ML --------------------------------
SELECT * FROM ML.PREDICT(MODEL `uas-bigdata.coffeeshop.morningtime`, 
  (SELECT sr.transaction_time,
  c.customer_first_name,
  g.generation,
  sr.line_item_amount
FROM `uas-bigdata.coffeeshop. sales reciepts` AS sr
JOIN `uas-bigdata.coffeeshop.customer` AS c
ON sr.customer_id = c.customer_id
JOIN `uas-bigdata.coffeeshop.generations` AS g
ON c.birth_year = g.birth_year
WHERE sr.transaction_time IS NOT NULL
LIMIT 200));
--------------------------------------------------------------------------------




