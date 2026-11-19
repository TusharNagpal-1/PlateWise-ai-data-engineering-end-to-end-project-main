# PlateWise — AI-Powered Food Delivery Analytics Platform

**An End-to-End Data Engineering Project**

PlateWise is an end-to-end batch data engineering platform designed to transform raw food delivery data into reliable business insights and AI-powered analytics.

The platform combines cloud storage, data warehousing, data transformation, workflow orchestration, and generative AI to build a scalable analytics pipeline.

### Architecture

**Food Delivery Data → Amazon S3 → Snowflake → dbt → Apache Airflow → AI Analytics**

The platform follows a medallion architecture to process data across three layers:

- **Bronze:** Ingests raw CSV datasets into Snowflake.
- **Silver:** Cleans, validates, and standardizes source data using dbt.
- **Gold:** Builds analytical dimensions, fact tables, and business-focused data marts.

An AI layer extends the warehouse with automated review analysis, retrieval-augmented generation (RAG), and natural-language-to-SQL capabilities. Streamlit provides interactive interfaces for analytics and AI applications.

---

## Project Highlights

| Layer | Technology | Implementation |
|---|---|---|
| Data Sources | CSV, Python, Pandas | Restaurant, customer, food, menu, order, order item, and review datasets |
| Data Lake | Amazon S3 | Organized raw data storage |
| Data Warehouse | Snowflake | Centralized storage and analytical processing |
| Bronze | Snowflake SQL | Raw ingestion using `COPY INTO` |
| Silver | dbt | Data cleaning, standardization, and validation |
| Gold | dbt + Snowflake | Dimensions, incremental facts, and analytical marts |
| AI Analytics | OpenAI | Review enrichment, RAG, and text-to-SQL |
| Orchestration | Apache Airflow | Scheduled end-to-end pipeline execution |
| Applications | Streamlit | Interactive analytics and AI interfaces |

## Technology Stack

- **Programming:** Python, SQL
- **Data Processing:** Pandas
- **Cloud Storage:** Amazon S3
- **Data Warehouse:** Snowflake
- **Transformation:** dbt Core, dbt-snowflake
- **Orchestration:** Apache Airflow 3, Docker
- **AI Integration:** OpenAI API, GPT-4o-mini, text-embedding-3-small
- **Application Layer:** Streamlit
- **Cloud Security:** AWS IAM, Snowflake Storage Integration

---

## Repository Structure

```text
PlateWise/
│
├── airflow/
│   ├── Dockerfile
│   ├── docker-compose.yaml
│   ├── example.env
│   └── dags/
│       └── platewise_pipeline.py
│
├── platewise/
│   ├── models/
│   │   ├── staging/
│   │   └── marts/
│   ├── snapshots/
│   ├── macros/
│   └── dbt_project.yml
│
├── ai/
│   ├── enrich_reviews.py
│   ├── rag_chat.py
│   ├── text_to_sql.py
│   └── example.env
│
├── snowflake/
│   ├── 01_setup.sql
│   ├── 02_storage_integration.sql
│   ├── 03_stage_and_formats.sql
│   ├── 04_raw_tables.sql
│   └── 05_copy_into.sql
│
├── aws/
│   └── iam/
│       ├── s3-read-policy.json
│       ├── snowflake-role-trust-policy-initial.json
│       └── snowflake-role-trust-policy-final.json
│
├── docs/
│   └── architecture.png
│
├── data/
└── README.md
```

*Note: The `data/` directory contains large source CSV files and is excluded from version control.*

---

## Data Engineering Workflow

### 1. Data Ingestion and Cloud Storage

The pipeline uses seven datasets representing the core operations of a food delivery platform:

- Restaurants
- Customers
- Food items
- Menus
- Orders
- Order items
- Customer reviews

The source datasets are uploaded to Amazon S3, organized into separate folders for easier ingestion and management.

Example:

```text
s3://<BUCKET>/raw/
    restaurants/
    users/
    food/
    menu/
    orders/
    order_items/
    reviews/
```

The dataset includes approximately:

- 10 million order records
- 23 million order-item records
- 300,000 customer reviews

### 2. Secure S3–Snowflake Integration

Snowflake accesses the S3 data lake through an AWS IAM role and Snowflake Storage Integration.

This approach avoids storing long-lived AWS access keys in the pipeline.

The integration process involves:

1. Creating an AWS IAM read policy.
2. Configuring an IAM role with the appropriate trust relationship.
3. Creating a Snowflake storage integration.
4. Retrieving the Snowflake IAM user ARN and external ID.
5. Updating the IAM role trust policy.
6. Creating an external stage in Snowflake.

### 3. Bronze Layer — Raw Data Ingestion

Raw CSV files are loaded into the `PLATEWISE.RAW` schema using Snowflake's `COPY INTO` command.

The raw layer preserves source data and provides a consistent starting point for downstream transformations.

### 4. Silver Layer — Data Transformation

The Silver layer is implemented using dbt staging models.

Key transformations include:

- Handling missing and inconsistent values.
- Standardizing column names and data types.
- Converting currency fields into numeric values.
- Normalizing customer information.
- Deriving useful business attributes.
- Preparing consistent source models for analytical processing.

These transformations create clean, reliable datasets for downstream models.

### 5. Gold Layer — Analytics and Data Modeling

The Gold layer contains business-ready analytical models.

**Dimension tables**

- `dim_restaurants`
- `dim_customer`
- `dim_food`
- `dim_date`

**Fact tables**

- `fct_orders`
- `fact_order_items`

Incremental dbt models with a MERGE strategy process new records without rebuilding the entire dataset on every run.

**Business intelligence marts**

- Daily revenue and GMV by city.
- Average order value (AOV).
- Order cancellation analysis.
- Restaurant performance metrics.
- Delivery SLA analysis using p50 and p90.
- Customer review insights.

**Data quality**

The project uses dbt tests to validate:

- Uniqueness
- Non-null constraints
- Referential integrity
- Accepted values
- Data reconciliation

An SCD Type 2 snapshot is also included to track historical changes in dimension data.

### 6. Airflow — Pipeline Orchestration

Apache Airflow orchestrates the complete data workflow through a scheduled DAG.

The pipeline consists of four major stages:

```text
load_raw_data
      |
      v
dbt_build_core
      |
      v
enrich_customer_reviews
      |
      v
dbt_build_ai_marts
```

The workflow handles raw ingestion, transformation, AI enrichment, and final analytical model generation.

Docker Compose manages the Airflow environment, while environment variables are used to configure Snowflake and OpenAI credentials.

### 7. AI-Powered Analytics

PlateWise introduces three AI capabilities on top of the analytical warehouse.

**A. Automated Review Enrichment**

The review enrichment module processes customer feedback using GPT-4o-mini.

It extracts structured information such as:

- Sentiment classification
- Review topics
- Key feedback categories

The results are stored in `PLATEWISE.AI.REVIEW_ENRICHED` and made available for further dbt transformations.

Processing is designed to avoid unnecessary repeated API calls and supports sample-based execution.

**B. Retrieval-Augmented Generation (RAG)**

The RAG application enables users to interact with customer reviews using natural language.

Workflow:

1. Generate embeddings for review text.
2. Retrieve semantically relevant reviews.
3. Provide retrieved context to the language model.
4. Generate answers grounded in the retrieved customer feedback.

The application also provides supporting review sources for its responses.

**C. Natural Language to SQL**

The text-to-SQL application allows users to query warehouse analytics using ordinary English questions.

For example:

- Which city generated the highest revenue?
- Which restaurants have the highest cancellation rates?
- What is the average order value by month?
- Which food categories receive the most positive reviews?

The application uses the available Gold-layer schema to generate Snowflake SQL. A SELECT-only validation layer restricts execution to read-only analytical queries.

---

## Getting Started

### Prerequisites

- Python
- Docker and Docker Compose
- AWS account with S3 access
- Snowflake account
- OpenAI API key
- dbt Core with the Snowflake adapter

### 1. Configure Snowflake

Execute the SQL scripts in the following order using Snowflake Snowsight:

```text
snowflake/01_setup.sql
snowflake/02_storage_integration.sql
snowflake/03_stage_and_formats.sql
snowflake/04_raw_tables.sql
snowflake/05_copy_into.sql
```

Configure the corresponding AWS IAM policy and role using the files in `aws/iam/`.

### 2. Configure and Run dbt

```bash
cd platewise

export SNOWFLAKE_ACCOUNT=...
export SNOWFLAKE_USER=...
export SNOWFLAKE_PASSWORD=...

dbt debug
dbt build --exclude tag:ai
```

### 3. Start Airflow

```bash
cd airflow

cp example.env .env
```

Add the required Snowflake and OpenAI credentials to `.env`.

Start the services:

```bash
docker compose build
docker compose up -d
```

Open the Airflow interface at:

```text
http://localhost:8080
```

Enable and trigger the `platewise_pipeline` DAG.

### 4. Run AI Applications

Configure the OpenAI API key:

```bash
export OPENAI_API_KEY=...
```

Run review enrichment:

```bash
python ai/enrich_reviews.py
```

Launch the RAG application:

```bash
streamlit run ai/rag_chat.py
```

Launch the text-to-SQL application:

```bash
streamlit run ai/text_to_sql.py
```

---

## Engineering Concepts Demonstrated

- End-to-end batch data pipeline development.
- Cloud data lake and warehouse integration.
- AWS IAM role-based authentication.
- Snowflake data loading and SQL transformations.
- Medallion architecture (Bronze, Silver, Gold).
- dbt incremental models and MERGE strategies.
- Dimensional data modeling.
- SCD Type 2 historical tracking.
- Data quality testing and reconciliation.
- Airflow DAG orchestration.
- Docker-based deployment.
- LLM-based data enrichment.
- Vector embeddings and RAG.
- Natural-language-to-SQL applications.

## Dataset and Documentation

The source CSV datasets and supporting project material are maintained separately because of their size.

**Project resources:** Add your dataset and documentation links here.

---

**PlateWise — From Raw Food Delivery Data to Intelligent Business Insights.**
