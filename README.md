# Automated-ETL-Pipeline-for-Music-Streaming-Analytics-Using-Airflow
# Project Overview
Sparkify, a digital music streaming platform, aims to enhance automation and monitoring of their data warehouse ETL workflows. To achieve this, they selected Apache Airflow as the orchestration tool to build robust, maintainable, and testable data pipelines that support monitoring, backfilling, and modular design.

# Problem Statement
Sparkify needs to construct high-quality, dynamic data pipelines built from reusable tasks. These pipelines must support monitoring, error handling, and backfilling. Given the importance of data quality in downstream analytics, data validation tests are required post-ETL. The raw data resides in Amazon S3 and includes:
1) CSV-formatted user activity logs
2) JSON-formatted song metadata

The processed data must be loaded into Amazon Redshift for analytics.

# ETL Design Principles
1) Partition Data Tables:
Efficient handling of historical data through date-based partitioning enables parallel backfills and faster processing.

2) Load Data Incrementally:
Only new data is appended during each ETL run. This approach optimizes load performance and avoids scanning entire datasets.

3) Enforce Idempotency:
Ensures that rerunning the same query on identical inputs consistently produces the same results—crucial for historical analytics.

4) Parameterize Workflows with Jinja:
Jinja templating is integrated into SQL logic for flexible and reusable queries, particularly useful for backfill operations.

5) Perform Data Checks Frequently:
Data is first loaded into a staging table, validated for quality, and only then moved to the production table. Validation includes checks such as:
- Record count > 0
- No NULL values in mandatory fields
- Anomaly detection (e.g., category drift or outliers)

6) Implement Alerts and Monitoring:
Alerts (via EmailOperator) notify teams when jobs fail SLA thresholds. This eliminates the need for manual DAG monitoring.

# Pipeline Implementation with Airflow
Airflow represents workflows as DAGs (Directed Acyclic Graphs) where:
Nodes = individual tasks
Edges = dependencies between tasks
Tasks are defined in a DAG definition file. Airflow’s UI provides a graphical view of this DAG, making the pipeline easier to understand and monitor.
Operators used fall into three broad categories:
- Sensors: Wait for events (e.g., file arrival, time triggers)

- Operators: Execute specific logic (e.g., Bash, Python, SQL)

- Transfers: Move data from source to destination

# Custom Operators Created:
1) StageToRedshiftOperator:
-Loads JSON/CSV files from S3 into Redshift using a templated SQL COPY command
-Supports dynamic file path resolution using execution timestamp
- Differentiates between JSON and CSV formats via parameters
  
2) LoadFactOperator:
- Executes transformation SQL for loading fact tables
- Accepts SQL input and Redshift connection parameters
- Fact tables support append-only strategy

3) LoadDimensionOperator:
- Loads dimension tables using either truncate-insert or append mode
-Allows flexibility in table loading strategy

4) DataQualityOperator:
-Runs SQL-based validation checks (e.g., count NULLs, record presence)
-Compares actual results with expected outcomes
-Raises exceptions on mismatches to trigger retries/failures

# How to Run the Project
1) Launch Redshift Cluster
Open create_cluster.ipynb
Fill AWS credentials in dwh.cfg
Use notebook to create the cluster and IAM role with S3 access

2) Update Configuration
Add Redshift host, database, user, password, port, and IAM ARN to dwh.cfg

3) Create Tables & Run ETL

python create_tables.py
python etl.py
 
4) Perform Analysis
-Use analysis.ipynb to run analytical queries
-Alternatively, connect tools like Power BI, Amazon QuickSight, or Tableau to Redshift for interactive dashboards

# Final Result / Analysis Capabilities
With the data pipeline in place, Sparkify’s analytics team can now explore and answer key business questions such as:

- How many users are currently listening to music?

- What is the geographical distribution of users?

- Which songs are being played the most?

The processed data enables slicing and dicing by various dimensions and supports advanced what-if analysis using BI tools.

# Software & Tools Used
Programming Language: Python 3

Libraries: psycopg2

Cloud Services: Amazon S3, Amazon Redshift

Orchestration: Apache Airflow

Notebook: Jupyter

If Python is not yet installed, it is recommended to use the Anaconda distribution which includes all required dependencies.

- Anjali Tavhare

Acknowledgement
This project is developed as part of the Udacity Data Engineering Nanodegree. Credit goes to Udacity for providing the starter templates and dataset.
Note: This repository cannot be reused for Udacity’s capstone submission, but is open for personal experimentation and learning.

