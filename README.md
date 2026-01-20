# Databricks Lakehouse for Financial Services - Credit Decisioning Workshop

Welcome to this hands-on workshop on building a complete lakehouse solution for credit decisioning using Databricks. This tutorial uses the `lakehouse-fsi-credit` demo as a foundation and includes additional hands-on exercises to deepen your understanding of the Databricks Data Intelligence Platform.

## Workshop Overview

In this workshop, you will:
- Set up a Databricks environment and install the FSI Credit Decisioning demo
- Learn about Spark Declarative Pipelines (SDP) for data ingestion and transformation
- Build your own data pipeline with bronze/silver/gold layers
- Implement data quality expectations
- Explore Unity Catalog governance features
- Train ML models using AutoML and Databricks Assistant
- Deploy models for real-time serving with A/B testing capabilities

**Duration**: Approximately 3-4 hours
**Skill Level**: Intermediate (basic knowledge of Python and SQL helpful)

---

## Prerequisites

- A valid email address for Databricks account registration
- Basic familiarity with Python and SQL
- Web browser (Chrome, Firefox, or Safari recommended)

---

## Part 1: Environment Setup

### Step 1.1: Register for Databricks Community Edition (Free)

1. Navigate to [https://www.databricks.com/try-databricks](https://www.databricks.com/try-databricks)
2. Click on "Get started for free" or "Try Databricks"
3. Select **"Free Edition"** (the free tier)
4. Fill in your registration details:
   - Email address
   - First and last name
   - Create a password
5. Verify your email address by clicking the link sent to your inbox
6. Complete your profile setup
7. You'll be redirected to your Databricks workspace

**Note**: Community Edition has some limitations compared to the full platform, but it's perfect for learning and this workshop.

### Step 1.2: Create a Bootstrap Notebook

1. Once logged into your Databricks workspace, click on **"Workspace"** in the left sidebar
2. Navigate to your user folder (usually `/Users/your-email@domain.com`)
3. Click the dropdown arrow next to your folder name
4. Select **"Create" → "Notebook"**
5. Name the notebook: `setup-dbdemos`
6. Select **Python** as the default language
7. Click **"Create"**

### Step 1.3: Install DBDemos

In the first cell of your new notebook, enter the following command:

```python
%pip install dbdemos
```

**Execute the cell** by pressing `Shift + Enter` or clicking the Run icon (▶) at the top right of the cell.

You should see installation progress messages. Wait until you see "Successfully installed dbdemos" message.

### Step 1.4: Install the Lakehouse FSI Credit Demo

In a new cell below, enter:

```python
import dbdemos
dbdemos.install('lakehouse-fsi-credit')
```

**Execute this cell.** The installation will:
- Create a directory called `lakehouse-fsi-credit` in your workspace
- Set up sample databases and tables in Unity Catalog
- Create Spark Declarative Pipeline (SDP) definitions
- Initialize ML experiments and models
- Set up dashboards and workflows

**Important**: This setup process will take approximately 10-15 minutes to complete. You'll see progress messages in the cell output.

### Step 1.5: Prepare for the Next Steps (During Setup)

While the installation is running, let's prepare a critical notebook modification.

**Important Context**: The `lakehouse-fsi-credit` demo was originally created for Databricks classic workspace environments. Since Databricks Community Edition (Free Edition) uses a serverless architecture, we need to make some compatibility updates to ensure the notebooks run smoothly.

Follow these steps to update the notebook:

1. In the left sidebar, navigate to **Workspace → Users → your-email@domain.com → lakehouse-fsi-credit → 03-Data-Science-ML**
2. Open the notebook: **`03.3-Batch-Scoring-credit-decisioning`**
3. Locate the first cell that contains:
   ```python
   %pip install mlflow==2.19.0
   ```
4. **Update it to**:
   ```python
   %pip install mlflow==2.22.1
   ```
   (We're updating MLflow from 2.19.0 to 2.22.1 for better compatibility with serverless)

5. Scroll down to find the cell that reads from the feature store (usually contains):
   ```python
   .cache()
   ```
   in the context of reading `credit_decisioning_features`

6. **Remove the `.cache()` call** from that line.
   - Before: `.withColumn("prediction", loaded_model(F.struct(*features))).cache()`
   - After: `.withColumn("prediction", loaded_model(F.struct(*features)))`

7. Save the notebook (Cmd+S or Ctrl+S)

**Why these changes?**
- **Classic vs Serverless**: The demo was built for classic compute clusters, but Community Edition uses serverless compute which has different resource management
- **MLflow version**: Newer MLflow 2.22.1 has better serverless compatibility and bug fixes
- **Removing `.cache()`**: Serverless environments handle caching differently; explicit caching can cause memory issues in the Community Edition's resource-constrained environment

---

## Part 2: Understanding the Demo Structure

Once the installation completes, explore the `lakehouse-fsi-credit` directory. You'll see:

```
lakehouse-fsi-credit/
├── 00-Credit-Decisioning.py          # Introduction and overview
├── config.py                         # Configuration (catalog, schema, volume)
├── 01-Data-Ingestion/               # Data pipeline implementations
│   ├── 01.1-sdp-sql/                # SQL-based SDP
│   └── 01.2-sdp-python/             # Python-based SDP
├── 02-Data-Governance/              # Unity Catalog features
├── 03-Data-Science-ML/              # ML pipeline notebooks
├── 04-BI-Data-Warehousing/          # Dashboards and analytics
├── 05-Generative-AI/                # AI functions and agents
├── 06-Workflow-Orchestration/       # Workflow automation
└── _resources/                      # Setup utilities (don't edit)
```

### Understanding the Architecture

This demo implements a **medallion architecture**:
- **Bronze Layer**: Raw data ingestion from various sources
- **Silver Layer**: Cleaned and validated data
- **Gold Layer**: Business-level aggregates ready for analytics and ML

---

## Part 3: Hands-On Exercise 1 - Exploring Spark Declarative Pipelines

### Step 3.1: Review SQL-Based SDP Transformations

1. Navigate to: `lakehouse-fsi-credit/01-Data-Ingestion/01.1-sdp-sql/transformations/`
2. Open **`01-bronze.sql`**
   - Notice how it uses `cloud_files` (Autoloader) to incrementally ingest data
   - See the `CREATE OR REFRESH STREAMING TABLE` statements
   - Observe the source data paths

3. Open **`02-silver.sql`**
   - See how it references bronze tables
   - Notice data cleaning and transformation logic
   - Look for `CONSTRAINT` statements (data quality expectations)

4. Open **`03-gold.sql`**
   - See aggregations and business-level transformations
   - Notice joins between multiple silver tables

### Step 3.2: Review Python-Based SDP Alternative

1. Navigate to: `lakehouse-fsi-credit/01-Data-Ingestion/01.2-sdp-python/transformations/`
2. Open **`01-bronze.py`**, **`02-silver.py`**, and **`03-gold.py`**
3. Compare the Python decorator approach:
   ```python
   @dlt.table(name="table_name")
   def create_table():
       return spark.readStream...
   ```

**Key Takeaway**: Both SQL and Python approaches achieve the same result. Choose based on your team's preference and existing skills.

---

## Part 4: Hands-On Exercise 2 - Build Your Own Data Pipeline

Now you'll create a pipeline to ingest additional sales transaction data into the lakehouse.

### Step 4.1: Download Sample Data

1. Download the file **`sales_transactions.csv`** from the `/data` folder in this repository
2. Save it to your local computer

### Step 4.2: Upload Data to Databricks Volume

1. In your Databricks workspace, click **"Catalog"** in the left sidebar
2. Navigate to: **main → dbdemos_fsi_credit_decisioning → credit_raw_data**
3. You should see the `credit_raw_data` volume
4. Click on the volume name to open it
5. Click **"Create directory"** button
6. Name the directory: **`additional`**
7. Click into the newly created `additional` directory
8. Click **"Upload"** or **"Upload files"**
9. Select `sales_transactions.csv` from your computer
10. Wait for the upload to complete

**Verify**: You should see the file at the path:
```
/Volumes/main/dbdemos_fsi_credit_decisioning/credit_raw_data/additional/sales_transactions.csv
```

### Step 4.3: Create a New SDP Bronze Table

1. Go to **Workspace** and navigate to `lakehouse-fsi-credit/01-Data-Ingestion/01.2-sdp-python/transformations/`
2. Click the dropdown next to `transformations` folder
3. Select **"Create" → "Notebook"**
4. Name it: **`04-bronze-sales`**
5. Select **Python** as the language

In the first cell, add the following code:

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import *

@dp.table(
    name="sales_transactions_bronze",
    comment="Bronze layer: Raw sales transaction data from CSV",
    table_properties={
        "quality": "bronze",
        "pipelines.autoOptimize.zOrderCols": "transaction_date"
    }
)
def sales_transactions_bronze():
    return (
        spark.readStream.format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .option("cloudFiles.inferColumnTypes", "true")
        .option("header", "true")
        .load("/Volumes/main/dbdemos_fsi_credit_decisioning/credit_raw_data/additional/")
        .select(
            col("transaction_id"),
            col("transaction_date").cast("date").alias("transaction_date"),
            col("customer_id"),
            col("product_category"),
            col("product_name"),
            col("quantity").cast("integer").alias("quantity"),
            col("unit_price").cast("decimal(10,2)").alias("unit_price"),
            col("total_amount").cast("decimal(10,2)").alias("total_amount"),
            col("payment_method"),
            col("store_location"),
            col("payment_status"),
            current_timestamp().alias("ingestion_timestamp")
        )
    )
```

Alternatively, you can also ask Databricks Assistant to generate the code by giving the following prompt:
```
use spark declarative pipeline to load data located in 
/Volumes/main/dbdemos_fsi_credit_decisioning/credit_raw_data/additional/ 
into streaming table with the name sales_transaction_bronze
```

if you want to take the vibe data engineering even further you can highlight the code above then use databricks assistant to add data quality check by adding this prompt
```
add data quality check to ensure that total_amount is greater than 0
```

**Save the notebook** (Cmd+S or Ctrl+S).

### Step 4.4: Create Silver Layer with Transformations

Create another notebook in the same directory: **`05-silver-sales`**

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import *

@dp.table(
    name="sales_transactions_silver",
    comment="Silver layer: Cleaned and validated sales transactions",
    table_properties={
        "quality": "silver"
    }
)
@dp.expect_or_drop("valid_transaction_id", "transaction_id IS NOT NULL")
@dp.expect_or_drop("valid_amount", "total_amount > 0")
@dp.expect_or_drop("valid_quantity", "quantity > 0")
def sales_transactions_silver():
    return (
        dp.read_stream("sales_transactions_bronze")
        .filter(col("payment_status").isin(["Completed", "Pending"]))
        .withColumn("transaction_year", year(col("transaction_date")))
        .withColumn("transaction_month", month(col("transaction_date")))
        .withColumn("transaction_day", dayofmonth(col("transaction_date")))
        .withColumn("is_high_value", when(col("total_amount") > 500, True).otherwise(False))
    )
```

**Save the notebook**.

### Step 4.5: Create Gold Layer with Aggregations

Create another notebook: **`06-gold-sales`**

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import *
from pyspark.sql.window import Window

@dp.table(
    name="customer_sales_summary_gold",
    comment="Gold layer: Customer-level sales aggregations",
    table_properties={
        "quality": "gold"
    }
)
def customer_sales_summary_gold():
    return (
        dp.read("sales_transactions_silver")
        .groupBy("customer_id")
        .agg(
            count("transaction_id").alias("total_transactions"),
            sum("total_amount").alias("total_spent"),
            avg("total_amount").alias("avg_transaction_amount"),
            max("total_amount").alias("max_transaction_amount"),
            min("transaction_date").alias("first_transaction_date"),
            max("transaction_date").alias("last_transaction_date"),
            countDistinct("product_category").alias("distinct_categories_purchased"),
            sum(when(col("is_high_value") == True, 1).otherwise(0)).alias("high_value_transactions")
        )
        .withColumn(
            "customer_segment",
            when(col("total_spent") > 1000, "Premium")
            .when(col("total_spent") > 500, "Standard")
            .otherwise("Basic")
        )
    )
```

**Save the notebook**.

---

## Part 5: Hands-On Exercise 3 - Adding Data Quality Expectations

Data quality expectations ensure that only valid data flows through your pipeline. Let's add more sophisticated expectations to our sales pipeline.

### Step 5.1: Understand Expectation Types

SDP supports three types of expectations:
- **`@dlt.expect`**: Track violations but allow data through
- **`@dlt.expect_or_drop`**: Drop rows that violate the expectation
- **`@dlt.expect_or_fail`**: Fail the entire pipeline if any row violates

### Step 5.2: Enhance Silver Table with Advanced Expectations

Edit your **`05-silver-sales`** notebook and update it:

```python
from pyspark import pipelines as dp
from pyspark.sql.functions import *

@dp.table(
    name="sales_transactions_silver",
    comment="Silver layer: Cleaned and validated sales transactions",
    table_properties={
        "quality": "silver"
    }
)
# Critical expectations - drop invalid rows
@dp.expect_or_drop("valid_transaction_id", "transaction_id IS NOT NULL")
@dp.expect_or_drop("valid_customer_id", "customer_id IS NOT NULL AND customer_id != ''")
@dp.expect_or_drop("valid_amount", "total_amount > 0")
@dp.expect_or_drop("valid_quantity", "quantity > 0 AND quantity <= 1000")
@dp.expect_or_drop("valid_unit_price", "unit_price > 0")

# Warning expectations - track but don't drop
@dp.expect("reasonable_amount", "total_amount <= 10000")
@dp.expect("valid_date_range", "transaction_date >= '2024-01-01' AND transaction_date <= current_date()")
@dp.expect("known_payment_method", "payment_method IN ('Credit Card', 'Debit Card', 'Cash', 'PayPal')")

# Validation rule: total_amount should equal quantity * unit_price (with small rounding tolerance)
@dp.expect("amount_matches_calculation", "ABS(total_amount - (quantity * unit_price)) < 0.01")

def sales_transactions_silver():
    return (
        dp.read_stream("sales_transactions_bronze")
        .filter(col("payment_status").isin(["Completed", "Pending"]))
        .withColumn("transaction_year", year(col("transaction_date")))
        .withColumn("transaction_month", month(col("transaction_date")))
        .withColumn("transaction_day", dayofmonth(col("transaction_date")))
        .withColumn("is_high_value", when(col("total_amount") > 500, True).otherwise(False))
        .withColumn("day_of_week", dayofweek(col("transaction_date")))
        .withColumn("is_weekend", when(col("day_of_week").isin([1, 7]), True).otherwise(False))
    )
```

**Save and run** this updated notebook.

### Step 5.3: Monitor Data Quality Metrics

After running your pipeline:
1. Navigate to **Data Engineering → Pipelines** in the left sidebar
2. Find your pipeline (if you created one) or note that expectations are tracked
3. Click on a pipeline run to see the **Data Quality** tab
4. Review metrics showing how many rows passed/failed each expectation

---

## Part 6: Understanding SDP Connectors

Spark Declarative Pipelines support various data sources. Here are the most common:

### Supported Source Connectors

| Connector | Use Case | Example |
|-----------|----------|---------|
| **Cloud Files (Autoloader)** | Incrementally load files from cloud storage | CSV, JSON, Parquet, Avro from S3/ADLS/GCS |
| **Kafka** | Real-time streaming from Kafka topics | Transaction streams, IoT data, clickstreams |
| **Delta Tables** | Read from existing Delta tables | Reference tables, dimension tables |
| **Kinesis** | AWS Kinesis streams | AWS-native event streaming |
| **SQL Databases** | JDBC connections to databases | SQL Server, PostgreSQL, MySQL |

### Example: Kafka Source

While we won't implement this in the workshop, here's how you would ingest from Kafka:

```python
@dp.table(name="transactions_from_kafka")
def kafka_stream():
    return (
        spark.readStream
        .format("kafka")
        .option("kafka.bootstrap.servers", "kafka-broker:9092")
        .option("subscribe", "transactions-topic")
        .option("startingOffsets", "latest")
        .load()
        .select(
            col("key").cast("string"),
            from_json(col("value").cast("string"), schema).alias("data")
        )
        .select("data.*")
    )
```

**Key Point**: SDP abstracts away the complexity of incremental processing, checkpointing, and fault tolerance regardless of the source.

---

## Part 7: (Optional) Data Governance with Unity Catalog

**Note to Facilitator**: Ask workshop participants if they want to explore governance features. This section takes approximately 20-30 minutes.

### Step 7.1: Understanding Unity Catalog

Unity Catalog provides:
- Fine-grained access control (table, column, row level)
- Data discovery and lineage tracking
- Centralized audit logs
- Data sharing across clouds/organizations

### Step 7.2: Column-Level Masking

1. Open the notebook: **`lakehouse-fsi-credit/02-Data-Governance/02-Data-Governance-credit-decisioning`**
2. Review the section on **Dynamic Views** for PII masking

Example of creating a masked view:

```sql
CREATE OR REPLACE VIEW customer_masked AS
SELECT
    customer_id,
    CASE
        WHEN is_member('analysts') THEN email
        ELSE 'REDACTED'
    END as email,
    CASE
        WHEN is_member('analysts') THEN phone
        ELSE 'XXX-XXX-' || RIGHT(phone, 4)
    END as phone,
    name,
    age,
    city
FROM customer_gold;
```

### Step 7.3: Tagging Sensitive Data

1. In Catalog Explorer, navigate to one of your tables
2. Click on a column that contains sensitive information
3. Click **"Add tag"**
4. Create a tag: `PII` with description "Personally Identifiable Information"
5. Apply the tag

### Step 7.4: Creating Access Policies

Example SQL to grant permissions:

```sql
-- Grant read access to analysts group
GRANT SELECT ON TABLE customer_sales_summary_gold TO `analysts`;

-- Grant read access on specific columns only
GRANT SELECT (customer_id, total_transactions, customer_segment)
ON TABLE customer_sales_summary_gold TO `data_viewers`;

-- Revoke access if needed
REVOKE SELECT ON TABLE customer_sales_summary_gold FROM `data_viewers`;
```

### Step 7.5: Exploring Data Lineage

1. In **Catalog Explorer**, select any gold table (e.g., `customer_sales_summary_gold`)
2. Click on the **"Lineage"** tab
3. Explore the visual graph showing:
   - Source tables/files
   - Transformation notebooks
   - Downstream dashboards and models
   - Dependencies between tables

**Key Insight**: Unity Catalog automatically tracks lineage across all transformations, making it easy to understand data flow and perform impact analysis.

---

## Part 8: Machine Learning with AutoML

### Step 8.1: Understanding the AutoML Workflow

1. Open the notebook: **`lakehouse-fsi-credit/03-Data-Science-ML/03.1-Feature-Engineering-credit-decisioning`**
2. Review the notebook cells and understand each section:

   **Section 1: Data Exploration**
   - Uses pandas and plotly for visualization
   - Analyzes feature distributions
   - Identifies correlations

   **Section 2: Feature Engineering**
   - Creates features from raw data
   - Handles missing values
   - Encodes categorical variables

   **Section 3: Feature Store**
   - Stores features in Databricks Feature Store
   - Enables feature reuse across models
   - Tracks feature lineage

3. **Run the notebook** to create the feature tables (this takes 5-10 minutes)

### Step 8.2: AutoML Model Training

1. Open: **`lakehouse-fsi-credit/03-Data-Science-ML/03.2-AutoML-credit-decisioning`**
2. Review the AutoML invocation code:

```python
from databricks import automl

summary = automl.classify(
    dataset=training_df,
    target_col="default",
    primary_metric="f1",
    timeout_minutes=20,
    max_trials=10
)
```

**Important Note for Community Edition**: The AutoML UI is limited in Community Edition, but the Python SDK works. The notebook uses the SDK approach which will work for you.

3. Review the results:
   - AutoML automatically tries multiple algorithms (Random Forest, XGBoost, LightGBM, etc.)
   - Performs hyperparameter tuning
   - Generates notebooks for the best model
   - Registers the model in MLflow

### Step 8.3: Review Model Performance

After AutoML completes:
1. Navigate to **Machine Learning → Experiments** in the left sidebar
2. Find the experiment created by AutoML
3. Click to view all runs
4. Sort by your primary metric (F1 score)
5. Click on the best run to see:
   - Model parameters
   - Metrics (precision, recall, F1, AUC)
   - Confusion matrix
   - Feature importance

---

## Part 9: Hands-On Exercise 4 - Using Databricks Assistant for ML

Now let's explore an alternative approach using Databricks Assistant (AI-powered coding assistant).

### Step 9.1: Create a New Notebook for AI-Assisted Analysis

1. In **Workspace**, navigate to your user folder
2. Create a new notebook: **`AI-Assisted-Sales-Analysis`**
3. Select **Python** as the language

### Step 9.2: Use Databricks Assistant for Exploratory Data Analysis

Click the **Assistant** button (usually a sparkle icon ✨ or "AI" button) in the notebook toolbar.

**Copy and paste this prompt into the Assistant**:

```
Please perform exploratory data analysis on the table
main.dbdemos_fsi_credit_decisioning.sales_transactions_silver

Store the result in a dataframe called 'df'.

1. Load the data and show basic statistics for all columns

2. Create visualizations for:
   - Distribution of transactions by product category
   - Transaction trends over time
   - Payment method distribution
   - Top 5 customers by total spending

3. Identify any patterns or anomalies in the data

4. Check for missing values and data quality issues

Generate executable code cells with explanations.
```

**Execute the generated code** to explore your sales data.

### Step 9.3: Build a Predictive Model with AI Assistance

In a new cell, click the Assistant again and use this prompt:

```
Build a machine learning model to predict payment_status using the dataframe 'df' from the previous step.

The dataframe 'df' contains sales transaction data from the table
main.dbdemos_fsi_credit_decisioning.sales_transactions_silver

Please:
1. Use the existing dataframe 'df' as the input dataset

2. Prepare the data:
   - Handle missing values
   - Encode categorical variables (product_category, payment_method, store_location, etc.)
   - Select relevant features for prediction (exclude transaction_id and other identifiers)
   - Scale numerical features if needed

3. Split data into training (70%) and testing (30%) sets

4. Train a classification model to predict payment_status (Completed, Pending, or Failed)
   - use Random Forest from sklearn library

5. Evaluate the model with these metrics:
   - Accuracy, Precision, Recall, F1-score
   - Confusion matrix
   - Classification report

6. Show feature importance to understand key drivers of payment status

7. Register the model in Unity Catalog with the name "main.dbdemos_fsi_credit_decisioning.sales_payment_pred"
   - don't forget to infer model signature and use the signature when registering the model

Use best practices for ML workflows and explain each step.
```

---

## Part 10: Model Serving and A/B Testing

### Step 10.1: Understanding Model Serving

1. Open: **`lakehouse-fsi-credit/03-Data-Science-ML/03.4-model-serving-BNPL-credit-decisioning`**
2. Review the model serving setup

Model serving allows you to:
- Deploy models as REST APIs
- Auto-scale based on traffic
- Perform A/B testing between model versions
- Monitor performance in production

### Step 10.2: Deploy Model to Serving Endpoint

Review this example code from the notebook:

```python
from databricks.sdk import WorkspaceClient
from databricks.sdk.service.serving import EndpointCoreConfigInput, ServedEntityInput

w = WorkspaceClient()

# Create serving endpoint
w.serving_endpoints.create(
    name="credit-decisioning-endpoint",
    config=EndpointCoreConfigInput(
        served_entities=[
            ServedEntityInput(
                entity_name="main.dbdemos_fsi_credit_decisioning.credit_model",
                entity_version="1",
                scale_to_zero_enabled=True,
                workload_size="Small"
            )
        ]
    )
)
```

### Step 10.3: A/B Testing with Traffic Splitting

To deploy two model versions with traffic splitting:

```python
# Update endpoint with A/B testing
w.serving_endpoints.update_config(
    name="credit-decisioning-endpoint",
    served_entities=[
        ServedEntityInput(
            entity_name="main.dbdemos_fsi_credit_decisioning.credit_model",
            entity_version="1",
            scale_to_zero_enabled=True,
            workload_size="Small",
            traffic_percentage=70  # 70% of traffic to version 1
        ),
        ServedEntityInput(
            entity_name="main.dbdemos_fsi_credit_decisioning.credit_model",
            entity_version="2",
            scale_to_zero_enabled=True,
            workload_size="Small",
            traffic_percentage=30  # 30% of traffic to version 2
        )
    ]
)
```

**Key Points**:
- You can gradually shift traffic from old to new model versions
- Monitor both versions' performance before full rollout
- Instantly rollback by adjusting traffic percentages

### Step 10.4: Test the Endpoint

```python
import requests
import json

# Get workspace URL and token
workspace_url = spark.conf.get("spark.databricks.workspaceUrl")
token = dbutils.notebook.entry_point.getDbutils().notebook().getContext().apiToken().get()

# Prepare test data
test_data = {
    "dataframe_records": [
        {
            "customer_id": "CUST12345",
            "transaction_amount": 450.00,
            "credit_score": 720,
            "debt_to_income": 0.35
        }
    ]
}

# Call the endpoint
response = requests.post(
    f"https://{workspace_url}/serving-endpoints/credit-decisioning-endpoint/invocations",
    headers={"Authorization": f"Bearer {token}"},
    json=test_data
)

print(f"Prediction: {response.json()}")
```

---

## Part 11: (Optional) AI Functions and Agent Framework

**Note**: This section is optional and time-permitting (15-20 minutes).

### Step 11.1: Understanding AI Functions

AI Functions allow you to call LLMs directly from SQL queries.

1. Open: **`lakehouse-fsi-credit/05-Generative-AI/05.1-AI-Functions-Creation`**
2. Review the AI function creation:

```sql
CREATE OR REPLACE FUNCTION analyze_customer_risk(
    customer_profile STRING
)
RETURNS STRING
LANGUAGE SQL
RETURN ai_query(
    'databricks-meta-llama-3-70b-instruct',
    CONCAT('Analyze the following customer profile and provide a risk assessment: ', customer_profile)
);
```

### Step 11.2: Using AI Functions in Queries

```sql
SELECT
    customer_id,
    total_spent,
    analyze_customer_risk(
        CONCAT('Customer ID: ', customer_id,
               ', Total Spent: $', total_spent,
               ', Transaction Count: ', total_transactions,
               ', Segment: ', customer_segment)
    ) as ai_risk_assessment
FROM main.dbdemos_fsi_credit_decisioning.customer_sales_summary_gold
LIMIT 5;
```

### Step 11.3: Agent Framework Overview

If time permits, open: **`05.2-Agent-Creation-Guide`** and review how to:
- Create conversational agents
- Connect agents to your data
- Deploy agents as chatbots
- Use Retrieval Augmented Generation (RAG) with your lakehouse data

---

## Part 12: Workshop Wrap-Up

### What You've Learned

✅ Setting up Databricks Community Edition
✅ Installing and exploring the lakehouse-fsi-credit demo
✅ Building data pipelines with Spark Declarative Pipelines (SQL and Python)
✅ Implementing data quality expectations
✅ Understanding various SDP connectors (Cloud Files, Kafka, etc.)
✅ Applying Unity Catalog governance features
✅ Training ML models with AutoML
✅ Using Databricks Assistant for AI-powered development
✅ Deploying models with serving endpoints
✅ Implementing A/B testing with traffic splitting
✅ (Optional) Using AI Functions for LLM integration

### Next Steps

1. **Explore More Demos**: Run `dbdemos.list()` to see other available demos
2. **Documentation**: Visit [docs.databricks.com](https://docs.databricks.com) for comprehensive guides
3. **Community**: Join the [Databricks Community Forums](https://community.databricks.com)
4. **Certification**: Consider pursuing Databricks certification (Data Engineer, ML Associate, etc.)
5. **Try Commercial Edition**: Sign up for a 14-day trial to access advanced features

### Common Troubleshooting

**Issue**: Pipeline fails with "Permission denied"
**Solution**: Ensure you're using the correct catalog and schema names from `config.py`

**Issue**: AutoML notebook fails
**Solution**: Verify MLflow version is 2.22.1 and restart Python after installation

**Issue**: Can't find uploaded CSV file
**Solution**: Check the volume path is exactly `/Volumes/main/dbdemos_fsi_credit_decisioning/credit_raw_data/additional/`

**Issue**: Model serving not available
**Solution**: Model serving may be limited in Community Edition; focus on batch inference instead

---

## Additional Resources

- [Databricks Lakehouse Platform](https://www.databricks.com/product/data-lakehouse)
- [Spark Declarative Pipelines Documentation](https://docs.databricks.com/en/delta-live-tables/index.html)
- [Unity Catalog Documentation](https://docs.databricks.com/en/data-governance/unity-catalog/index.html)
- [MLflow Documentation](https://mlflow.org/docs/latest/index.html)
- [Databricks Academy (Free Courses)](https://www.databricks.com/learn/training/home)

---

## Feedback

We'd love to hear your feedback on this workshop! Please share:
- What worked well
- What was challenging
- Suggestions for improvement
- Topics you'd like to see covered

---

**Workshop Version**: 1.0
**Last Updated**: January 2026
**Demo Version**: lakehouse-fsi-credit v1

Happy Learning! 🚀
