Creating a **common vocabulary** for discussing data platforms is critical for alignment between engineers, architects, product managers, and stakeholders. Below is a **structured glossary** of key terms categorized by functional domain. This can serve as a **shared language** to describe and reason about modern data platform components, workflows, and governance.

---

## 🔧 **Infrastructure & Architecture**

| Term               | Definition                                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **Data Platform**  | The end-to-end system for collecting, storing, transforming, and serving data for operational or analytical use.                |
| **Data Lake**      | Centralized storage (usually object-based like S3, ADLS, or GCS) for raw, semi-structured, and structured data.                 |
| **Lakehouse**      | A data architecture combining the flexibility of data lakes with the transactional features of data warehouses.                 |
| **Data Warehouse** | A structured, optimized storage system (e.g., Redshift, BigQuery, Snowflake) for fast analytical queries.                       |
| **Control Plane**  | The layer responsible for orchestrating jobs, managing configurations, and ensuring governance.                                 |
| **Data Mesh**      | A decentralized architectural paradigm where domains own their data products and pipelines, supported by a self-serve platform. |
| **Hub-and-Spoke**  | Architecture where central services (hub) coordinate or standardize access to decentralized data sources (spokes).              |

---

## 🚰 **Data Flow & Processing**

| Term                 | Definition                                                                                                            |
| -------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Ingestion**        | Bringing data into the platform from external or internal sources (batch or streaming).                               |
| **ETL / ELT**        | Extract-Transform-Load (traditional) vs Extract-Load-Transform (modern, often with dbt or SQL-based transformations). |
| **Data Pipeline**    | A chain of tasks that move and transform data from source to destination.                                             |
| **Orchestration**    | The coordination of tasks in a pipeline (e.g., scheduling, dependency management, retries).                           |
| **Streaming**        | Real-time ingestion and processing of event-based data using Kafka, Flink, or Kinesis.                                |
| **Batch Processing** | Processing data in fixed-size, time-based chunks (e.g., hourly, daily).                                               |
| **Backfill**         | Reprocessing historical data to populate or correct datasets.                                                         |
| **Idempotency**      | A task or transformation that can be safely re-run without altering results incorrectly.                              |

---

## 🧠 **Data Modeling & Storage**

| Term                                 | Definition                                                                                                  |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| **Schema**                           | The structure of a dataset (column names, types, nullability, etc.).                                        |
| **Partitioning**                     | Dividing datasets by a field (e.g., `event_date`) to improve query performance and manageability.           |
| **Table Format**                     | A layer over file storage that supports schema evolution and ACID (e.g., Apache Iceberg, Delta Lake, Hudi). |
| **Metadata Store**                   | Stores metadata about tables and datasets (e.g., Glue Catalog, Hive Metastore).                             |
| **Slowly Changing Dimensions (SCD)** | Techniques for tracking changes in dimensional data (types 1, 2, 3…).                                       |
| **Snapshot**                         | A point-in-time version of a dataset, useful for auditing and rollback.                                     |

---

## 🧪 **Transformation & Lineage**

| Term                | Definition                                                                       |
| ------------------- | -------------------------------------------------------------------------------- |
| **Transformation**  | Logic to clean, enrich, or restructure data (often using SQL or Spark).          |
| **Model (dbt)**     | A SQL-based transformation logic unit that creates a derived table or view.      |
| **Lineage**         | The trace of data flow from source to final form, including intermediate steps.  |
| **Data Asset**      | Any entity representing data (e.g., tables, views, files) tracked in the system. |
| **Materialization** | Persisting intermediate results (e.g., dbt's table, view, incremental types).    |
| **Expectations**    | Defined assertions about data quality (e.g., no nulls, values within range).     |

---

## 📊 **Analytics & Serving**

| Term                | Definition                                                                                              |
| ------------------- | ------------------------------------------------------------------------------------------------------- |
| **Semantic Layer**  | An abstraction layer that maps technical fields to business-friendly terms and metrics.                 |
| **BI Tool**         | Business Intelligence software for data exploration and dashboards (e.g., Tableau, QuickSight, Looker). |
| **Data Mart**       | A curated subset of data designed for a specific business domain (e.g., sales, marketing).              |
| **Federated Query** | A query that spans multiple data sources (e.g., S3 + Redshift + external DB).                           |

---

## 🔍 **Observability & Quality**

| Term                  | Definition                                                                                     |
| --------------------- | ---------------------------------------------------------------------------------------------- |
| **Data Quality**      | The accuracy, completeness, and reliability of data.                                           |
| **Data Contracts**    | Formal agreements between producers and consumers defining schema and SLAs.                    |
| **SLAs/SLOs**         | Service Level Agreements/Objectives — uptime, freshness, or quality targets for data products. |
| **Anomaly Detection** | Monitoring for unexpected deviations in data patterns (e.g., row count drop).                  |
| **Validation Tests**  | Rules to ensure data meets expectations (e.g., using Great Expectations or custom SQL tests).  |

---

## 🔐 **Governance, Security & Compliance**

| Term                    | Definition                                                                             |
| ----------------------- | -------------------------------------------------------------------------------------- |
| **PII**                 | Personally Identifiable Information — requires protection under GDPR/CCPA/etc.         |
| **WORM**                | Write Once, Read Many — immutable storage model for compliance (e.g., S3 Object Lock). |
| **Data Classification** | Categorizing data by sensitivity (e.g., public, internal, confidential).               |
| **Access Control**      | Defining who can read/write/modify data — managed via IAM, ACLs, or policies.          |
| **Audit Trail**         | Logs capturing access or modification of datasets, used for compliance and debugging.  |
| **Data Residency**      | Legal requirement that data must remain within certain geographic regions.             |

---

## 🧭 **Data Mesh Vocabulary**

| Term                     | Definition                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------ |
| **Domain**               | A business-aligned unit (e.g., Marketing, Sales) that owns its data pipelines.             |
| **Data Product**         | A curated dataset owned and maintained by a domain team, with consumers in mind.           |
| **Product Owner (Data)** | A role responsible for the lifecycle, quality, and usability of a data product.            |
| **Self-Serve Platform**  | Infrastructure and tooling provided by a central team to enable domain autonomy.           |
| **Federated Governance** | A governance model where standards are agreed centrally but enforced locally.              |
| **Discoverability**      | The ease with which a user can find and understand a data product (typically via catalog). |

---

## 🛠 Tools (and Associated Concepts)

| Tool                                | Domain          | Concept                             |
| ----------------------------------- | --------------- | ----------------------------------- |
| **dbt**                             | Transformations | ELT, analytics engineering, lineage |
| **Airflow**                         | Orchestration   | DAGs, scheduling                    |
| **Dagster**                         | Orchestration   | Asset-based pipelines, lineage      |
| **Apache Iceberg**                  | Storage         | Table formats, ACID on S3           |
| **Flink / Spark**                   | Processing      | Streaming and batch transformations |
| **Kafka / Kinesis**                 | Ingestion       | Event streaming, pub/sub            |
| **Trino / Presto**                  | Query           | SQL on data lakes                   |
| **DataHub / Amundsen**              | Catalogs        | Metadata, lineage, discoverability  |
| **Great Expectations**              | Quality         | Validation, tests, documentation    |
| **OpenLineage**                     | Lineage         | Event-based lineage tracking        |
| **Snowflake / Redshift / BigQuery** | Analytics       | Data warehouse                      |

---

## 💬 How to Use This Vocabulary

* **For architecture discussions**:
  *“This domain team owns the ingestion and transformation of the Customer data product, which is published to the lakehouse in Iceberg format and documented in DataHub.”*

* **For platform operations**:
  *“We need lineage visibility from Kafka to dbt to Redshift – let’s integrate OpenLineage and emit events from Airflow.”*

* **For governance conversations**:
  *“All PII fields must be encrypted and tracked with access controls and audit trails to comply with GDPR.”*

---

Would you like a downloadable PDF of this vocabulary or an editable version for internal documentation (e.g. Confluence or Notion)?
