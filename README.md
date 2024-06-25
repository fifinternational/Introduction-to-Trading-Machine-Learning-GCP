# Building a Regression Model in BigQuery for AAPL Stock Data

This project demonstrates how to build a regression model in Google BigQuery to predict the closing price of AAPL (Apple Inc.) stock. We will use historical stock data, preprocess it, and train a machine learning model using BigQuery ML.

## Setup and Requirements

### Prerequisites

- Google Cloud Platform (GCP) account
- Access to BigQuery
- Basic knowledge of SQL

### Setup

1. **Create a Google Cloud Project:**
   - Go to the [Google Cloud Console](https://console.cloud.google.com/).
   - Create a new project.

2. **Enable BigQuery API:**
   - In the Google Cloud Console, go to APIs & Services > Library.
   - Enable the BigQuery API.

3. **Create a Dataset in BigQuery:**
   - Go to the BigQuery console.
   - Create a new dataset, for example, `{{DATASET_NAME}}`.

4. **Upload AAPL Stock Data:**
   - Obtain historical AAPL stock data (CSV format).
   - Upload the CSV file to a new table in your dataset.

## Data Preparation

### SQL Query for Data Preparation

```sql
WITH raw AS (
  SELECT
    date,
    close,
    LAG(close, 1) OVER(ORDER BY date) AS min_1_close,
    LAG(close, 2) OVER(ORDER BY date) AS min_2_close,
    LAG(close, 3) OVER(ORDER BY date) AS min_3_close,
    LAG(close, 4) OVER(ORDER BY date) AS min_4_close
  FROM
   `{{PROJECT_ID}}.{{DATASET_NAME}}.{{TABLE_NAME}}`
  ORDER BY 
    date DESC
),
raw_plus_trend AS (
  SELECT 
    date,
    close,
    IF (min_1_close = min_2_close > 0, 1, -1) AS min_1_trend,
    IF (min_2_close = min_3_close > 0, 1, -1) AS min_2_trend,
    IF (min_3_close = min_4_close > 0, 1, -1) AS min_3_trend
  FROM 
    raw
),
ml_data AS (
  SELECT
    date,
    close,
    min_1_close AS day_prev_close,
    IF (min_1_trend + min_2_trend + min_3_trend > 0, 1, -1) AS trend_3_day
  FROM
    raw_plus_trend
)
SELECT
  *
FROM 
  ml_data;
```

## Building the Regression Model
## SQL Query to Create the Model
```sql
CREATE OR REPLACE MODEL `your_project_id.stock_data.aapl_model`
OPTIONS(
  model_type='linear_reg',
  input_label_cols=['close'],
  data_split_method='seq',
  data_split_eval_fraction=0.3,
  data_split_col='date'
) AS
SELECT
  date,
  close,
  day_prev_close,
  trend_3_day
FROM
  `your_project_id.stock_data.ml_data`;

## Evaluating the Model
## SQL Query to Evaluate the Model
```sql
SELECT
  *
FROM
  ML.EVALUATE(MODEL `your_project_id.stock_data.aapl_model`);
```

## Making Predictions
## SQL Query to Make Predictions
``sql
SELECT
  *
FROM
  ML.PREDICT(MODEL `your_project_id.stock_data.aapl_model`,
    (
    SELECT
      *
    FROM
      `your_project_id.stock_data.ml_data`
    WHERE
      date >= '2019-01-01'
    )
  );
```

## Data Sources
cloud-training/ai4f/AAPL10Y.csv

## Conclusion
{{In this project, we demonstrated how to build, train, evaluate, and use a regression model in BigQuery to predict AAPL stock closing prices. By leveraging BigQuery ML, we can seamlessly integrate machine learning capabilities into our SQL workflows, enabling efficient and scalable data analysis and prediction.}}

## Repository Structure
{{`README.md`: This file, containing the project overview and instructions.
`sql/`: Directory containing SQL scripts used in this project.
`data_preparation.sql`: SQL script for data preparation.
`create_model.sql`: SQL script to create the regression model.
`evaluate_model.sql`: SQL script to evaluate the model.
`make_predictions.sql`: SQL script to make predictions.}}
