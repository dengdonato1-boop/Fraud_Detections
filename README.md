# Fraud_Detections
Built Cloud- native banking fraud risk Engine using Big Query machine langauage 

Hands-On Cloud Analytics: Decomposing Financial Fraud Vectors with BigQuery ML
🚀 What I Built (Hands-On)
I engineered and deployed a live, cloud-native risk engine inside Google Cloud Platform (GCP) using BigQuery ML and SQL. Instead of traditional, slow workflows that require pulling millions of sensitive financial rows out into external Python environments, I kept the entire data engineering and machine learning lifecycle directly within the secure cloud data warehouse.

Using a real-world dataset of credit card transactions, I trained a Logistic Regression binary classification model to analyze transaction amounts, timestamps, and anonymized behavioral vectors. The system evaluates these inputs in real time to instantly calculate a fraud probability score and determine whether a transaction should be flagged (Class = 1) or cleared (Class = 0).

🛠️ Practical Challenges I Overcame
A portfolio isn't just about showing code that works perfectly on the first try—it's about engineering solutions when things break. During this hands-on deployment, I solved two critical real-world pipeline issues:

Environment Schema Alignment: My initial pipeline threw an environment error because the destination schema path didn't exist. I manually configured a dedicated data layer (financial_risk_ops) matching regional compliance standards (US multi-region) to allow the machine learning assets to compile properly.

The Imbalanced Data Trap: Financial fraud is incredibly rare. My first training run threw a model error because a basic dataset limit didn't capture a single fraud case. To fix this, I re-engineered the ingestion pipeline using structural sorting (ORDER BY Class DESC) paired with AUTO_CLASS_WEIGHTS = TRUE to force the database to balance the target labels so the model could actually learn criminal patterns.

📈 Core Portfolio Competencies
Cloud Security Architecture: Processing data entirely inside the data warehouse to drastically reduce data exposure risks by eliminating unnecessary network movement.

Behavioral Baseline Analytics: Mapping foundational data science concepts directly to User and Entity Behavior Analytics (UEBA)—the core logic driving modern enterprise SIEMs and automated alert desks.

Production Inference Controls: Structuring live queries designed to deliver real-time risk outputs capable of triggering automated card freezes.

💻 The Production SQL Scripts
1. Engineering the Fraud Classifier
SQL
CREATE OR REPLACE MODEL `financial_risk_ops.banking_fraud_model`
OPTIONS
  (model_type='logistic_reg',
   input_label_cols=['Class'],
   AUTO_CLASS_WEIGHTS = TRUE) AS
SELECT
  Time,
  Amount,
  V1,
  V2,
  V3,
  V4,
  V5,
  Class
FROM
  `bigquery-public-data.ml_datasets.ulb_fraud_detection`
WHERE
  Class IS NOT NULL
  AND Amount > 0
ORDER BY 
  Class DESC
LIMIT 120000;
2. Live Risk Inference & Threat Scoring
SQL
SELECT
  predicted_Class,
  predicted_Class_probs[OFFSET(1)].prob AS fraud_probability_score,
  Amount,
  Time
FROM
  ML.PREDICT(MODEL `financial_risk_ops.banking_fraud_model`,
    (
    SELECT
      CAST(8500.00 AS NUMERIC) AS Amount, -- Simulated high-value anomaly
      CAST(45000 AS INT64) AS Time,       
      CAST(-1.35 AS FLOAT64) AS V1,       -- Anonymized behavioral vectors
      CAST(2.41 AS FLOAT64) AS V2,
      CAST(-3.11 AS FLOAT64) AS V3,
      CAST(0.85 AS FLOAT64) AS V4,
      CAST(-1.12 AS FLOAT64) AS V5
    ));
🎯 The Production Output (Verified Results)
When passing a mock high-value transaction of $8,500 through the final inference framework, the deployed cloud engine successfully flagged the behavior:

Prediction: Class = 1 (Confirmed Fraud Flag)

Risk Score: ~34.8% anomaly probability

In a production banking environment, this sub-second database calculation instantly feeds into automated security playbooks to temporarily freeze the user's account and trigger immediate out-of-band verification alerts.
