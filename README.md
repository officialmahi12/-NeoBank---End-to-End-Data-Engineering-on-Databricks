# -NeoBank---End-to-End-Data-Engineering-on-Databricks
A production-grade, metadata-driven data pipeline for a NeoBank use case, built entirely on the Databricks Free Edition using Medallion Architecture (Bronze → Silver → Gold), with interactive dashboards and an AI-powered Genie interface for self-service analytics.

📌 Table of Contents

Project Overview
Architecture
Source Tables
Metadata Framework
Project Structure
Key Components
Dashboards & Analytics
Tech Stack
Getting Started
Learning Modules


Project Overview
NeoBank generates large volumes of data from core banking systems, payment gateways, and external credit bureaus. This project solves three core challenges:
ProblemSolutionBanking data exists in multiple disconnected sources (data silos)Unified ingestion via Auto Loader + JDBC into Delta LakeManual pipelines are slow, complex, and hard to scaleMetadata-driven framework with automated orchestration via Databricks JobsBusiness users lack self-service analyticsInteractive dashboards + Databricks Genie AI interface

Architecture
┌─────────────────┐     ┌──────────────────────────────────────────────┐     ┌──────────────────────┐
│   Data Sources  │     │              Data Transformation              │     │     Consumption       │
│                 │     │  ┌─────────┐   ┌────────┐   ┌──────────┐    │     │                      │
│  SQL Server  ───┼────▶│  │ Bronze  │──▶│ Silver │──▶│   Gold   │   ─┼────▶│  Databricks Dashboard│
│  Blob Storage───┼────▶│  └─────────┘   └────────┘   └──────────┘    │     │  Databricks Genie AI │
└─────────────────┘     └──────────────────────────────────────────────┘     └──────────────────────┘
         │                          ▲                  ▲
         │              ┌───────────┴──────────────────┴──────────┐
         │              │              Orchestration               │
         │              │    Databricks Jobs  |  SQL DB (Metadata) │
         │              └─────────────────────────────────────────┘
         │                          ▲
         │              ┌───────────┴──────────────────────────────┐
         └─────────────▶│  Data Ingestion: Auto Loader | Spark JDBC│
                        └─────────────────────────────────────────┘
                                           ▲
                        ┌──────────────────┴──────────────────────┐
                        │  Governance: Unity Catalog | Email Alerts│
                        └─────────────────────────────────────────┘
Layers:

Bronze — Raw ingestion, preserving source data as-is
Silver — Cleaned, deduplicated, conformed data
Gold — Business-aggregated, analytics-ready tables


Source Tables
SQL Server
TableKey ColumnsDescriptionBRANCHESbranch_code (PK)Branch details: name, city, state, regionCUSTOMERScustomer_id (PK), branch_code (FK)KYC info, contact details, PANACCOUNTSaccount_id (PK), customer_id (FK)Account type, balance, currency, statusTRANSACTIONStxn_id (PK), account_id (FK)Transaction type, amount, channel, status
Blob Storage
TableKey ColumnsDescriptionCREDIT_BUREAU_REPORTSreport_id (PK), customer_id (FK)Credit score, risk grade, external loansPAYMENT_GATEWAY_LOGStxn_id (PK/FK)Gateway name, response code, device type, geo

Metadata Framework
The pipeline is driven by a SQL DB metadata layer that eliminates hardcoded configurations.
tables (1) ──────────────── (∞) table_parameters
   │                              parameter_name, parameter_value
   │
   ├──────────────────────── (∞) table_watermarks
   │                              last_watermark_value, last_run_id
   │
   └──────────────────────── (∞) pipeline_runs
                                  run_id, layer, start_time, end_time,
                                  status, number_of_records, error_message
The tables entity stores configuration for every source table including source_system, target_layer, bronze_schema, silver_schema, gold_schema, active_flag, and load_order — making it trivial to onboard new tables without code changes.

Project Structure
13_Banking_Project/
├── 00_Source_Files/              # Raw source data files
├── 01_Setup_Metadata/            # Initialize metadata tables in SQL DB
├── 02_Source_to_Silver/          # Ingestion → Bronze → Silver notebooks
├── 03_Silver_to_Gold/            # Silver → Gold aggregation notebooks
├── 04_Email_Notification/        # Customized pipeline status email alerts
└── 05_Dashboard/
    └── NeoBank_Dashboard.lvdash.json   # Databricks dashboard definition
Supporting Learning Modules
01_Notebook_Features_Overview/    # Databricks notebook fundamentals
02_Notebook_Run_from_Notebook/    # %run and dbutils.notebook.run patterns
03_DBUtils_Secrets/               # Secure credential management
04_DBUtils_FileSystem/            # DBFS file operations
05_Jobs/                          # Databricks Jobs orchestration
06_Auto_Loader/                   # Incremental file ingestion with Auto Loader
07_Debug_Code/                    # Debugging techniques
08_AI_Assistant/                  # Chat, Edit, and Agent mode with Databricks AI
09_Cost_Calculation/              # Monitoring DBU costs and job runs
10_Lakeflow_Connect/              # Salesforce CSV ingestion via Lakeflow
11_Lakeflow_Declarative_Pipeline/ # Declarative pipeline with transformations
12_Genie/                         # Databricks Genie AI setup
13_Banking_Project/               # ← Main project (see above)

---Key Components

Data Ingestion

Auto Loader (cloudFiles) for incremental file ingestion from Blob Storage
JDBC connector for SQL Server tables with watermark-based incremental loads
Secrets managed via DBUtils Secrets (no hardcoded credentials)

Data Transformation

Apache Spark for distributed data processing
Delta Lake for ACID transactions, time travel, and schema enforcement
Watermark tracking ensures idempotent, incremental loads

Orchestration

Databricks Jobs for scheduled and trigger-based pipeline execution
SQL DB (metadata layer) tracks pipeline runs, watermarks, and parameters
Email Notifications sent on pipeline completion or failure

Governance

Unity Catalog for centralized data governance, lineage, and access control


--Dashboards & Analytics

The NeoBank_Dashboard in Databricks contains four pages:
PageMetricsExecutive DashboardTotal Customers: 5K · Total Deposit: 1.38B · Total Transactions: 20K · High Risk Customers: 1.5KCustomer InsightsSegment distribution (HIGH/MEDIUM/LOW value) · Risk grade distributionBranch PerformanceTotal deposits and customers by branch · Transaction amounts by branchGateway PerformanceSuccessful vs. failed transactions per gateway (BillDesk, PayU, Razorpay, Stripe) · Device type breakdown
Databricks Genie is also configured to allow business users to query banking and risk data using natural language (see 12_Genie/).

Tech Stack
LayerTechnologyStorageAzure Blob Storage, Delta LakeComputeDatabricks (Serverless + Free Edition)IngestionAuto Loader, Apache Spark, JDBCTransformationPySpark, Spark SQLOrchestrationDatabricks Jobs, Databricks WorkflowsMetadata & AuditingAzure SQL DatabaseGovernanceUnity CatalogVisualizationDatabricks Dashboards (Lakeview)AI / Self-ServiceDatabricks GenieNotificationsCustomized Email via SMTP

Getting Started

Clone this repository into your Databricks workspace via Git integration
Configure secrets — set up a Databricks secret scope with your SQL Server credentials and SMTP details (see 03_DBUtils_Secrets/02_Manage_Secrets.py)
Initialize metadata — run 13_Banking_Project/01_Setup_Metadata/ notebooks to create and populate the metadata tables in your SQL DB
Upload source files — place Blob Storage source files into DBFS (see 00_Source_Files/)
Run the pipeline — execute 02_Source_to_Silver/ then 03_Silver_to_Gold/, or schedule via a Databricks Job
Import the dashboard — upload 05_Dashboard/NeoBank_Dashboard.lvdash.json in Databricks → Dashboards
Explore with Genie — set up the Genie space using the configuration in 12_Genie/


Learning Modules
This repository doubles as a Databricks learning path. Work through the numbered folders in order to master:

Notebook features and inter-notebook execution
DBUtils for secrets and file system operations
Databricks Jobs and scheduling
Auto Loader for streaming file ingestion
Debugging and AI-assisted development
Cost monitoring and optimization
Lakeflow Connect and Declarative Pipelines
Genie AI for natural language data querying
Full end-to-end banking project 
