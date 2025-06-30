# Syntax BigQuery

Berikut adalah query yang saya gunakan untuk menggabungkan kf_final_transaction, kf_inventory, kf_kantor_cabang, dan kf_product menjadi satu.

QUERY kf_results_1 
```
SELECT
  f.transaction_id,
  f.date,
  COALESCE(f.branch_id,i.branch_id)branch_id,
  f.rating,
  f.customer_name,
  COALESCE(f.product_id,i.product_id)product_id,
  f.price,
  f.discount_percentage
FROM `rakamin-kf-analytics-463714.kimia_farma.kf_final_transaction`AS f 
INNER JOIN `rakamin-kf-analytics-463714.kimia_farma.kf_inventory` 
AS i ON f.branch_id = i.branch_id;
```

QUERY kf_results_2
```
SELECT
  r.transaction_id,
  r.date,
  COALESCE(r.branch_id,k.branch_id)branch_id,
  k.branch_name,
  k.kota,
  k.provinsi,
  COALESCE(k.rating)rating_cabang,
  r.customer_name,
  r.product_id,
  r.price,
  r.discount_percentage,
  r.rating_transaksi,
FROM `rakamin-kf-analytics-463714.kimia_farma.kf_results_1` AS r
INNER JOIN `rakamin-kf-analytics-463714.kimia_farma.kf_kantor_cabang` AS k 
ON r.branch_id = k.branch_id;
```

QUERY kf_results_3 
```
SELECT
  r.transaction_id,
  r.date,
  r.branch_id,
  r.branch_name,
  r.kota,
  r.provinsi,
  r.rating_cabang,
  r.customer_name,
  COALESCE(r.product_id,p.product_id)product_id,
  p.product_name,
  COALESCE(r.price,p.price)actual_price,
  r.discount_percentage,
  r.rating_transaksi
FROM `rakamin-kf-analytics-463714.kimia_farma.kf_results_2` AS r
INNER JOIN `rakamin-kf-analytics-463714.kimia_farma.kf_product` AS p 
ON r.product_id = p.product_id
```

Kode yang di bawah adalah kode yang saya gunakan untuk menghasilkan kolom persentase_gross_laba, nett_sales, dan nett_profit berdasarkan data yang sudah ada.

QUERY persentase_gross_laba
```
CREATE OR REPLACE TABLE `rakamin-kf-analytics-463714.kimia_farma.kf_results_3` AS
SELECT
  *,
  CASE
    WHEN actual_price <= 50000 THEN 0.1
    WHEN actual_price > 50000 AND actual_price <= 100000 THEN 0.15
    WHEN actual_price > 100000 AND actual_price <= 300000 THEN 0.2
    WHEN actual_price > 300000 AND actual_price <= 500000 THEN 0.25
    ELSE 0.3
  END AS persentase_gross_laba
FROM
  `rakamin-kf-analytics-463714.kimia_farma.kf_results_3`;
```

QUERY nett_sales 
```
CREATE OR REPLACE TABLE `rakamin-kf-analytics-463714.kimia_farma.kf_results_3` AS
SELECT
  *,
  actual_price - (actual_price * discount_percentage) AS nett_sales
FROM
  `rakamin-kf-analytics-463714.kimia_farma.kf_results_3`;
```

QUERY nett_profit 
```
CREATE OR REPLACE TABLE `rakamin-kf-analytics-463714.kimia_farma.kf_results_3` AS
SELECT
  *,
  nett_sales * persentase_gross_laba AS nett_profit
FROM
  `rakamin-kf-analytics-463714.kimia_farma.kf_results_3`;
```


