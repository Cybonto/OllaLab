# OllaLab - System Engineering
OllaLab is a system for accelerating data-centric R&D. The system can be used for both development and production environments all of which may involve hybrid infrastructure where some services will be on premise and some will be on multiple clouds. 

## Key Points
The OllaLab system is designed as a comprehensive, scalable, and secure platform for data-centric R&D, addressing the entire data lifecycle from ingestion to application. Here are its core principles and design choices:

1.  **Data-Centricity:** The entire system is built around the flow and management of data, emphasizing data quality, accessibility, and governance.

2.  **Microservices Architecture:** OllaLab is composed of loosely coupled microservices, promoting flexibility, independent scaling, and fault isolation. This allows individual components to be updated, replaced, or scaled without impacting the entire system.

3.  **Hybrid and Multi-Cloud Ready:** The architecture explicitly supports deployments spanning on-premise infrastructure, single cloud providers, and multiple cloud providers. This flexibility allows OllaLab to adapt to different organizational needs and avoid vendor lock-in.

4.  **Open-Source Focus:** The system heavily relies on open-source technologies (Pulsar, NiFi, Hudi, Airflow, dbt, Great Expectations, Keycloak, etc.). This promotes transparency, community support, and cost-effectiveness.  It also reduces vendor lock-in.

5.  **Data Lakehouse Paradigm:** OllaLab adopts the data lakehouse architecture, combining the scalability of data lakes with the data management features of data warehouses. Apache Hudi is the core of this, enabling ACID transactions, schema evolution, and efficient querying on data stored in object storage (MinIO or cloud equivalents).

6.  **Data Quality as a Priority:** Data quality is emphasized throughout the system, with multiple layers of checks and enforcement:
    *   **Schema Registry (Apicurio):** Centralized schema management and versioning ensure data consistency.
    *   **Data Contracts:** Formal agreements between data producers and consumers, specifying schema, quality rules, SLAs, and ownership.
    *   **Pre-ingestion Scripts:** Initial data cleaning and validation before ingestion.
    *   **Pulsar Functions:** Real-time data validation and transformation during stream processing.
    *   **Great Expectations:** Comprehensive data quality checks and reporting, integrated into both ingestion and processing.
    *   **Metadata Catalog:** Keeps the data quality metrics

7.  **Real-time and Batch Processing:** The system handles both real-time (streaming) and batch data ingestion and processing, catering to diverse data sources and use cases. Apache Pulsar and Debezium handle streaming, while Apache NiFi handles batch.

8.  **Orchestration and Automation:** Apache Airflow (with the option of `systemd` timers) orchestrates complex data pipelines, managing dependencies, scheduling, retries, and monitoring. Infrastructure as Code (Terraform) and configuration management tools are recommended for automation.

9.  **Security by Design:** Security is integrated into every layer:
    *   **Centralized Authentication and Authorization (Keycloak):** Single Sign-On (SSO) and Role-Based Access Control (RBAC) are enforced through Keycloak, ensuring consistent access control across all components.
    *   **Secrets Management (HashiCorp Vault):** Sensitive information (passwords, API keys, etc.) is stored securely in Vault and accessed dynamically by applications.
    *   **Network Security:** Network policies, firewalls, and VPCs/VNets restrict access to components.  TLS/SSL encryption is used for all communication.
    *   **Data Encryption:** Encryption at rest (disk encryption, Hudi encryption, MinIO server-side encryption) and in transit (TLS) protects data.
    *   **Data Masking and Anonymization:** Techniques like dynamic data masking and pseudonymization are used to protect sensitive data while still allowing for analysis.

10. **Observability and Monitoring:** Comprehensive monitoring is built-in, using Prometheus, Grafana, Loki, and Promtail.  Metrics, logs, and alerts are used to track system health, performance, and data quality. Langfuse is used for LLM observability.

11. **Metadata Management:** A metadata catalog (PostgreSQL as a basic implementation, DataHub as a more advanced option) is used to store and manage metadata, improving data discoverability, governance, and collaboration.

12. **Extensibility and Flexibility:**  The system is designed to be extensible.  Open-source components and well-defined APIs allow for customization and integration with other tools.  The use of microservices facilitates adding new features and functionalities.

13. **LLM Capabilities**: Ollama and Langfuse enable local LLM operation, and monitoring.

14. **Cost optimization**: several strategies and tools (like auto-scaling) are in place to reduce the cost of operations.

## Table of Contents:
[System Layers](#system-layers)
- [1. The Ingestion layer](#1-the-ingestion-layer)
  - [1a. Schema Registry](#1a-schema-registry)
  - [1b. Enforcement of Data Contracts](#1b-enforcement-of-data-contracts)
  - [1c. Pre-Ingestion Scripts](#1c-pre-ingestion-scripts)
  - [1d. Streaming Data Ingestion with Apache Pulsar](#1d-streaming-data-ingestion-with-apache-pulsar)
  - [1e. Change Data Capture (CDC) with Debezium](#1e-change-data-capture-cdc-with-debezium)
  - [1f. Batch Data Ingestion with Apache NiFi](#1f-batch-data-ingestion-with-apache-nifi)
- [2. The Orchestration layer](#2-the-orchestration-layer)
  - [Apache Airflow](#apache-airflow)
  - [Airflow Backfilling](#airflow-backfilling)
  - [Airflow Sensors](#airflow-sensors)
  - [Airflow SLAs (Service Level Agreements)](#airflow-slas-service-level-agreements)
  - [Airflow Integration with Keycloak and Other Components](#airflow-integration-with-keycloak-and-other-components)
    - [Systemd Timers](#systemd-timers)
- [3. The Data Lakehouse layer](#3-the-data-lakehouse-layer)
    - [MiniIO](#miniio)
    - [Apache Hudi](#apache-hudi)
  - [Hudi features of interest](#hudi-features-of-interest)
  - [Hudi Table Types: CoW vs. MoR](#hudi-table-types-cow-vs-mor)
  - [Hudi Clustering and Compaction](#hudi-clustering-and-compaction)
  - [Hudi Indexing](#hudi-indexing)
  - [Hudi Time Travel and Rollback](#hudi-time-travel-and-rollback)
  - [Hudi vs. Delta Lake vs. Iceberg](#hudi-vs-delta-lake-vs-iceberg)
  - [Hudi Integration with Keycloak (IdP)](#hudi-integration-with-keycloak-idp)
    - [DuckDB](#duckdb)
- [4. The Batch/Stream Processing layer](#4-the-batchstream-processing-layer)
    - [Pulsar Functions](#pulsar-functions)
    - [Data Build Tool (DBT)](#data-build-tool-dbt)
    - [Great Expectations (GE)](#great-expectations-ge)
- [5. The Metadata Catalog layer](#5-the-metadata-catalog-layer)
  - [PostgreSQL](#postgresql)
  - [DataHub and other options](#datahub-and-other-options)
    - [Automated Metadata Extraction](#automated-metadata-extraction)
- [6. The Database layer](#6-the-database-layer)
  - [General Considerations](#general-considerations)
  - [PostgreSQL](#postgresql-1)
  - [MongoDB (Community Edition)](#mongodb-community-edition)
  - [Neo4j (Community Edition)](#neo4j-community-edition)
- [7. The Analytics & ML layer](#7-the-analytics--ml-layer)
    - [Query Engine / Interactive: duckDB](#query-engine--interactive-duckdb)
    - [Apache Superset](#apache-superset)
    - [MLflow](#mlflow)
    - [MLJAR AutoML](#mljar-automl)
- [8. The Application layer](#8-the-application-layer)
    - [Python & Streamlit](#python--streamlit)
    - [NodeJS & SvelteKit](#nodejs--sveltekit)
    - [FastAPI](#fastapi)
    - [Kong Gateway](#kong-gateway)
    - [Logging](#logging)
- [9. The LLM layer](#9-the-llm-layer)
    - [Ollama (LLM Management)](#ollama-llm-management)
    - [Langfuse (LLM Monitoring)](#langfuse-llm-monitoring)
- [10. The Monitoring layer](#10-the-monitoring-layer)
  - [Metrics Monitoring](#metrics-monitoring)
    - [Prometheus](#prometheus)
    - [Grafana](#grafana)
  - [Log Aggregation and Monitoring)](#log-aggregation-and-monitoring)
    - [Loki](#loki)
    - [Promtail](#promtail)
    - [Standardized Log Format](#standardized-log-format)
- [11. The Authentication and Authorization layer](#11-the-authentication-and-authorization-layer)
    - [KeyCloak](#keycloak)
- [Networking](#networking)
- [Data Governance](#data-governance)
    - [Data Ownership and Stewardship](#data-ownership-and-stewardship)
    - [Data Quality Metrics](#data-quality-metrics)
    - [Data Classification](#data-classification)
    - [Data Retention Policy](#data-retention-policy)
    - [Data Lineage (Beyond Catalog)](#data-lineage-beyond-catalog)
    - [Privacy](#privacy)
- [Backup and Monitoring](#backup-and-monitoring)
    - [Databases](#databases)
    - [PostgreSQL](#postgresql-2)
    - [MongoDB (Community Edition)](#mongodb-community-edition-1)
    - [Neo4j (Community Edition)](#neo4j-community-edition-1)
- [Security](#security)
    - [Secrets Management (HashiCorp Vault)](#secrets-management-hashicorp-vault)
    - [Vulnerability Scanning](#vulnerability-scanning)
- [Scalability](#scalability)
- [Cost Optimization](#cost-optimization)

## System Layers
### 1. The Ingestion layer
The Ingestion layer captures data from a variety of sources (3rd party APIs, relational DBs, NoSQL DBs, etc.) in both real-time (stream) and batch (bulk) modes.
#### 1a. Schema Registry
Instead of relying solely on ad-hoc schema definitions, OllaLab will utilize a dedicated schema registry. This provides a robust and scalable solution for schema management. Apicurio Registry is selected due to its fully open-source nature (Apache 2.0 license) and light resource footprint.
- **Apicurio Registry features of interest**
  - Centralized Schema Management: All schemas are stored and versioned in a single, authoritative repository. This eliminates inconsistencies and promotes reuse.
  - Schema Evolution: Apicurio supports various compatibility rules (backward, forward, full, etc.).  Before updating a schema, the registry checks if the new version is compatible with existing consumers, preventing breaking changes.
  - Serialization/Deserialization: The schema registry provides client libraries (for Java, Python, etc.) that seamlessly integrate with data ingestion tools. These libraries handle serialization and deserialization of data based on the registered schemas.  This eliminates the need for manual schema handling within each component.
  - Versioning: Every schema change creates a new version in the registry.  Consumers can specify which version they need, allowing for smooth transitions and backward compatibility.
  - Web UI: Apicurio provides a user-friendly web interface for browsing, managing, and searching for schemas.
- **Apicurio Registry Integration**
  - Pulsar: Pulsar has built-in support for schema registries and can be configured to use Apicurio Registry for schema validation and retrieval.  This ensures that all messages published to Pulsar topics conform to the defined schemas.
  - NiFi:  NiFi's `ValidateRecord` processor can integrate with a schema registry (using the `AvroSchemaRegistry` controller service, for example). This enables NiFi to validate incoming data against the registered schemas before further processing.
  - Debezium: Debezium can be configured to fetch schemas from a registry for serialization.
#### 1b. Enforcement of Data Contracts**
  - OllaLab support a data contract approach to formalize agreements between data producers (source systems) and consumers (OllaLab and its downstream applications).
  - A data contract is a formal, machine-readable agreement that specifies:
    - Schema: The structure and data types of the data (managed in the schema registry).
    - Data Quality Rules: Specific validation rules beyond basic schema constraints (e.g., ranges, allowed values, relationships between fields). These are often expressed using a combination of declarative rules (e.g., in a YAML file) and custom code (e.g., within Pulsar Functions).
    - Service Level Agreements (SLAs): Expectations around data freshness (latency), availability, and completeness.
    - Ownership: Clear responsibility for data quality and maintenance.
    - Versioning: How changes to the contract will be managed and communicated.
    - Metadata: Additional descriptive information about the data (e.g., purpose, source, sensitivity).
  - Enforcement:
    - Schema Registry: The foundation of the data contract is the schema stored in the registry.
    - Metadata Enrichment: Extend the schema registry (or use a separate metadata store) to store additional contract information (SLAs, ownership, data quality rules).
    - Pre-ingestion Scripts: Enforce basic data quality rules before data enters the pipeline.
    - Pulsar Functions: Implement more complex validation logic based on the data contract.
    - Great Expectations (Processing Layer): Used for more comprehensive data quality checks and reporting.
    - Monitoring and Alerting: Track adherence to SLAs and data quality rules.  Alert relevant teams when violations occur.
    - Communication: Establish clear communication channels between data producers and consumers to discuss changes, resolve issues, and ensure alignment.
#### 1c. Pre-Ingestion Scripts**
  - Pre-ingestion scripts are small programs run before data formally enters a data pipeline. Their purpose is to perform initial checks, cleaning, and transformations on raw data, ensuring only valid and properly formatted data proceeds. The need arises because data sources are often messy, inconsistent, or incomplete, and allowing flawed data into the main system can cause errors, inaccurate results, and wasted resources. They act as a first line of defense for data quality.
  - For batch uploads, custom scripts (Python, etc.) can run before NiFi or other tools even see the data. These scripts can perform initial validation.
  - While Great Expectations is in the processing layer, it can contribute to the Ingestion layer's pre-ingestion checks via a NiFi processor or a Pulsar Function.
#### 1d. Streaming Data Ingestion with Apache Pulsar
Streaming data ingestion is about capturing and processing data continuously as it's generated, rather than in large, delayed batches. Its purpose is to enable real-time reactions to data, like updating dashboards, triggering alerts, or feeding machine learning models that require up-to-the-second information. If streaming requirements are modest, this component can be simplified by using Debezium's embedded engine and writing directly to the data lakehouse (Hudi). Kafka is also a good alternative to Pulsar.
- **Apache Pulsar Features of Interest**
  - Apache Pulsar Functions are lightweight compute processes that can operate on messages in transit. Pulsar Function can be written in Java, Python, or Go to perform validation. Invalid messages can be routed to a dead letter topic.
  - Pulsar has built-in support for authentication (using JAAS, Kerberos, or JWT tokens) and authorization (managing access to topics and namespaces).
  - Configure Pulsar to use TLS for both client-broker and broker-broker communication.
  - Pulsar can log administrative actions and (with some configuration) potentially track message consumption.
- **Access Control for Pulsar**
  - Network Isolation: Pulsar Functions run in a sandboxed environment with restricted network access. By default, they cannot access arbitrary external services.
  - Whitelisting: Explicitly configure network access for Pulsar Functions. This typically involves defining allowed hostnames, IP addresses, and ports.
  - Secrets Management: Sensitive information (e.g., database credentials, API keys) should *never* be hardcoded in the function code. Use a secure secrets management system (e.g., HashiCorp Vault, Kubernetes Secrets, or the cloud provider's secrets manager) to inject secrets into the function's environment.
  - Pulsar Resources: Functions can readily access Pulsar topics, schemas (via the schema registry), and other Pulsar resources within the same cluster.
  - Limited System Resources: Resource limits (CPU, memory) can be configured for Pulsar Functions to prevent them from consuming excessive resources.
- **Security Best Practices**
    - Grant Pulsar Functions only the minimum necessary permissions.
    - Monitor the resource usage and network activity of Pulsar Functions.
    - Thoroughly review Pulsar Function code for security vulnerabilities.
    - Keep the function's dependencies up-to-date to address security patches.
#### 1e. Change Data Capture (CDC) with Debezium
  - CDC primary purpose is to capture every change (insert, update, delete) made to a database table as it happens and make that change data available to other systems. CDC avoids repeatedly querying the entire database (which is slow and resource-intensive). Instead, it taps into the database's internal transaction logs (like the PostgreSQL WAL or MySQL binlog).
  - Sample use cases for CDC:
    - Replicating data from a transactional database (OLTP) to a data warehouse (OLAP) for analytics.
    - Keeping a search index (like Elasticsearch) up-to-date with changes in primary database.
    - Streaming data to a data lakehouse (like Hudi) for immediate querying.
    - Triggering real-time dashboards and alerts.
    - Maintaining a complete audit trail of all database changes.  This is crucial for regulatory compliance and security.
    - CDC provides the "events" that drive event-driven systems. 
  - Debezium provides several single message transformations (SMTs) that can be used to modify records before they are sent downstream. One could write custom Java code (or use existing transformations) to filter or modify events.  However, this is more for basic transformations/filtering than complex validation.  Consider rejecting invalid events to a "dead letter queue" (a separate Pulsar topic, for example) for later inspection.
#### 1f. Batch Data Ingestion with Apache NiFi
  - NiFi is powerful but can have a steeper learning curve.  If batch ingestion needs are simple, one might be able to use simpler tools (e.g., custom scripts, Airflow operators). However, NiFi excels at complex dataflows and error handling.
  - Data quality check with Apache NiFi:
    - `ValidateRecord`:  Use a schema registry (like Avro or a custom one) to validate the structure and data types of incoming records.
    - `RouteOnAttribute` / `RouteOnContent`:  Route data based on specific criteria.  Send invalid data to a separate flow for handling.
    - `UpdateAttribute`: Add attributes indicating validation status (e.g., `validation.status = "failed"`).
    - `ReplaceText`:  Use regular expressions to validate specific fields.
  - NiFi supports various authentication mechanisms (LDAP, Kerberos, certificates) and fine-grained authorization (per-processor group or even per-processor).
  - Configure NiFi to use HTTPS for its web UI and TLS for communication between nodes.
  - NiFi's provenance events provide a detailed record of data flow and transformations, which can be used for auditing.

### 2. The Orchestration layer
The Orchestration layer manages and coordinates the execution of all the tasks in a data pipeline, ensuring they run in the correct order, at the right time, and handles dependencies and failures gracefully. This layer is required when there are multiple, interconnected data processing steps that must be automated, scheduled, and monitored reliably. This layer is essential for complex workflows. Sample use cases for the Orchestration layer are:
- Scheduling daily ETL jobs.
- Triggering data processing pipelines when new data arrives.
- Handling dependencies between tasks (e.g., Task B runs only after Task A completes successfully).
- Retrying failed tasks.
- Managing complex workflows with branching and conditional logic.
#### Apache Airflow
- **Apache Airflow Features of Interest**
  - Directed Acyclic Graphs (DAGs): Airflow workflows are defined as DAGs each of which is a collection of tasks with dependencies defined, showing the order in which they should run.  This is fundamental to Airflow.  The "acyclic" part means there are no circular dependencies.
  - Operators: Operators are the building blocks of DAGs. Each operator represents a single task. Airflow comes with many built-in operators for common tasks such as:
    - `PythonOperator`: Executes a Python callable.
    - `BashOperator`: Executes a Bash command or script.  Useful for interacting with the command line.
    - `PostgresOperator`: Executes SQL in a PostgreSQL database.
    - `S3KeySensor` (and similar for MinIO/other storage): Waits for a file to appear in a storage location. (This is a *Sensor*, explained in more detail below).
    - `DebeziumOperator`: Integrate Debezium to capture data and ingest it in the data lakehouse.
    - `NiFiOperator`: Control NiFi flows from within Airflow.
    - `ExternalTaskSensor`: Waits for a task in *another* DAG to complete.  Essential for cross-DAG dependencies.
    - `BranchPythonOperator`: Allows conditional branching in your DAG based on the result of a Python function.
  - Tasks: A task is a specific *instance* of an operator within a DAG.
  - Scheduling: Airflow has a powerful scheduler that uses cron expressions (e.g., `0 0 * * *` for daily at midnight) to determine when DAGs should run.  One can also trigger DAGs manually.
  - Web UI: Airflow provides a web-based UI for:
    - Monitoring the status of DAGs and tasks (running, success, failed, retrying).
    - Visualizing DAGs and their dependencies.
    - Viewing logs for individual task runs.
    - Manually triggering DAGs.
    - Managing connections, variables, and users (with appropriate permissions).
  - Variables and Connections: Airflow allows storing configuration information (like database credentials, API keys, file paths) as variables and connections. This avoids hardcoding sensitive information in DAG code.
  - Dynamic DAG Generation: One can write Python code to *dynamically* generate DAGs. This is useful in dealing with many similar pipelines (e.g., one pipeline per customer, where each customer has slightly different requirements).
- **Airflow Backfilling:**
  - Backfilling is the process of running a DAG for a past date range.  This is essential in several scenarios such as bug fixes, new features, data recovery, initial load.
  - Airflow's scheduler can be instructed to run a DAG for a specific date range, either through the UI or via the command-line interface (CLI).  Airflow will execute the DAG for each date in the range, respecting dependencies and concurrency limits.
  - Considerations:
      - Idempotency: Tasks should be *idempotent*, meaning they can be run multiple times with the same input without causing unintended side effects.  This is crucial for backfilling and retries.
      - Resource Usage: Backfilling can be resource-intensive, especially for large datasets.
      - Data Availability: Ensure that the historical data needed to process is still available.
- **Airflow Sensors:**
  - Sensors are a special type of operator that *waits* for a certain condition to be true before allowing downstream tasks to run.  They are used to trigger pipelines based on external events.
  - Sensors "poll" for the condition at a specified interval.  When the condition is met, the sensor succeeds, and the next task in the DAG can run.
- **Airflow SLAs (Service Level Agreements):**
  - SLAs define the expected performance of your data pipelines.  They are a way to quantify and track the timeliness of your data processing.
  - One can set an `sla` parameter on individual tasks or on the entire DAG.  The SLA specifies a `timedelta` object representing the maximum acceptable time for the task/DAG to complete.
- **Monitoring and Alerting**  If a task or DAG misses its SLA, Airflow will:
  - Mark the task/DAG run as "SLA missed" in the UI.
  - Send an email alert (if configured).  You can customize the email template.
  - You can also integrate with other alerting systems.

#### Airflow Integration with Keycloak and Other Components
The most critical integration is with Keycloak for authentication and authorization.
- **Keycloak Setup:**
  - Create a client in Keycloak specifically for Airflow.
  - Configure the client to use the "OpenID Connect" protocol.
  - Note the client ID, client secret, and Keycloak's discovery URL (usually something like `https://<keycloak-host>/auth/realms/<your-realm>/.well-known/openid-configuration`).
- **Airflow Configuration (airflow.cfg):**
  - Set `[webserver] authenticate = True`.
  - Set `[webserver] auth_backend = airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager`.
  - Configure FAB (Flask AppBuilder, which Airflow's UI is built on) to use OIDC:
      ```
      [fab]
      auth_type = AUTH_OAUTH
      oauth_providers = [
          {
              'name': 'keycloak',
              'icon': 'fa-key',  # Optional: icon for the login button
              'token_key': 'access_token',
              'remote_app': {
                  'client_id': 'your-airflow-client-id',
                  'client_secret': 'your-airflow-client-secret',
                  'api_base_url': 'https://<keycloak-host>/auth/realms/<your-realm>',
                  'client_kwargs': {
                      'scope': 'openid email profile roles'  # Request roles!
                  },
                  'access_token_url': 'https://<keycloak-host>/auth/realms/<your-realm>/protocol/openid-connect/token',
                  'authorize_url': 'https://<keycloak-host>/auth/realms/<your-realm>/protocol/openid-connect/auth',
                  'userinfo_endpoint': 'https://<keycloak-host>/auth/realms/<your-realm>/protocol/openid-connect/userinfo',
                  'jwks_uri': 'https://<keycloak-host>/auth/realms/<your-realm>/protocol/openid-connect/certs'
              }
          }
      ]

      # Role-based access control (RBAC) within Airflow UI
      [webserver]
      rbac = True
      ```
- **Role Mapping (Airflow UI Roles):**
  - Airflow has its own built-in roles (Admin, User, Op, Viewer, Public). The roles from Keycloak (in the `roles` claim of the JWT) need to be mapped to these Airflow roles.  This is done through the FAB security manager.
  - One can also create custom roles in Airflow's UI.
  - The `AUTH_ROLES_MAPPING` setting in the Airflow configuration might be helpful.
- **API Authentication (for programmatic access to Airflow):**
  - To use the Airflow API (e.g., to trigger DAGs from other applications), one will need to obtain an access token from Keycloak and include it in the `Authorization` header of API requests (as a Bearer token).  The API will then validate the token with Keycloak and enforce permissions based on the roles in the token.
  -  Example using `curl`:
      ```bash
      # 1. Get an access token from Keycloak (replace with your actual values)
      TOKEN=$(curl -X POST \
        'https://<keycloak-host>/auth/realms/<your-realm>/protocol/openid-connect/token' \
      -H 'Content-Type: application/x-www-form-urlencoded' \
      -d 'grant_type=password' \
      -d 'client_id=your-airflow-client-id' \
      -d 'client_secret=your-airflow-client-secret' \
      -d 'username=your-username' \
      -d 'password=your-password' \
      -d 'scope=openid' \
        | jq -r '.access_token')

      # 2. Use the token to trigger a DAG run
      curl -X POST \
        'http://<airflow-host>/api/v1/dags/my_dag/dagRuns' \
      -H "Authorization: Bearer $TOKEN" \
      -H 'Content-Type: application/json' \
      -d '{
          "conf": {},
          "execution_date": "2023-10-27T10:00:00+00:00"
        }'
      ```
- **Integration with other components:**
  - Database Operators (PostgresOperator, etc.): Use Airflow *connections* to store database credentials.  Do *not* put credentials directly in DAG code. When Airflow runs as a user authenticated by Keycloak it can access different resources (databases, storage, etc) based on the permissions associated with the Keycloak roles.
  - MinIO (and other storage): Similarly, use Airflow connections to store MinIO access keys and secrets.  The MinIO client within custom sensor (or other operators) will retrieve these credentials from the connection.
  - Custom Operators/Hooks: If custom operators need to interact with other services (e.g., APIs), they must accept credentials via Airflow connections or to obtain access tokens from Keycloak.
  - DebeziumOperator/NiFiOperator: Use Airflow to start, stop, and monitor the Debezium and NiFi jobs. Configure the credentials in the connections and access them within the operators.

#### Systemd Timers
Systemd timers are a powerful, modern alternative to Cron, deeply integrated into the `systemd` init system (which is standard on most modern Linux distributions). They offer several advantages in the context of a complex system like OllaLab.
- **Features of Interest**
  - Tight `systemd` Integration: Timers are managed as `systemd` units, just like services. This provides a unified management interface (`systemctl`) for starting, stopping, enabling, disabling, and monitoring both services and scheduled tasks. This is a significant advantage over Cron, which has its own separate configuration and management tools.
  - Precise Time Control: Timers offer more precise time control than Cron's minute-level granularity. You can specify:
    - Calendar Events: Similar to Cron expressions (`OnCalendar=*-*-* 12:00:00`), but with greater flexibility.  You can specify specific dates, days of the week, and even use more complex expressions (e.g., "Mon,Wed *-*-1,5 12:00:00" for noon on Mondays and Wednesdays in January and May).
    - Monotonic Timers: Trigger events relative to system boot or service activation (`OnBootSec=`, `OnActiveSec=`, `OnUnitActiveSec=`, `OnUnitInactiveSec=`).  This is invaluable for tasks that need to run a certain time *after* a service starts, regardless of the absolute time.
    - Real-time Timers: Trigger events at specific, wall-clock times (`OnCalendar=`). This is the equivalent of Cron's behavior.
    - Accuracy and Randomized Delay: Timers can be configured with `AccuracySec=` to specify the acceptable delay for a task.  You can also use `RandomizedDelaySec=` to add a random delay, which is *very* useful for avoiding "thundering herd" problems where many tasks all try to run at exactly the same time (e.g., at midnight).
  - Dependencies: Like other `systemd` units, timers can have dependencies (`Requires=`, `After=`, `Before=`, `Wants=`). This allows you to ensure that a timer only runs *after* a specific service is running, or that a service is started *before* the timer triggers. This is a *major* advantage over Cron for managing complex workflows.
  - Resource Control (cgroups): `systemd` timers, like services, can be placed in specific control groups (cgroups). This allows you to limit the CPU, memory, and I/O resources consumed by the scheduled tasks.  This is vital for preventing a runaway task from impacting other services on the system. This is *far* more robust than Cron's simple `nice` levels.
  - Logging and Monitoring: Timer events are logged to the `systemd` journal (`journalctl`). This provides a unified log for all system events, including scheduled tasks.  You can easily filter and search the journal to see when timers ran, whether they succeeded or failed, and any output they produced.  This integrates seamlessly with the OllaLab's Loki/Promtail/Grafana monitoring stack.
  - Persistent Timers: Timers can be configured to be persistent (`Persistent=true`). This means that if a timer was missed (e.g., because the system was down), it will be run as soon as possible after the system recovers. This is important for ensuring that critical tasks are not skipped.
  - Sandboxing and Security: `systemd` offers a range of sandboxing options that can be applied to services triggered by timers. These can restrict the capabilities of the executed processes, limiting their access to the filesystem, network, and other system resources.
  - Automatic Restart: Like services, timers can be configured to automatically restart on failure.
- **Integration**
  - The key to integrating `systemd` timers is that they trigger *services*.  The timer itself doesn't *do* the work; it starts a `.service` unit that performs the actual task.  This is a different mindset from Cron, where the cron job directly executes a command.
  - Airflow alternative in some cases: `systemd` timers would *replace* Airflow's scheduler. Instead of defining DAGs in Airflow, you would define `systemd` timer units and corresponding service units for each task.  The dependencies between tasks would be managed using `systemd` dependencies (`Requires=`, `After=`, etc.).
  - Debezium, NiFi, dbt, Great Expectations (Triggering): You would create `systemd` service units for each of these components.  For example:
    - `debezium-connector-postgres.service`:  A service that runs the Debezium connector for PostgreSQL.  This service might be started by a timer (`debezium-connector-postgres.timer`) that runs it periodically, or it might be a long-running service managed directly by `systemd`.
    - `nifi-dataflow-customer.service`: A service that executes a specific NiFi dataflow (perhaps using the `nifi-toolkit` CLI or a custom script).  This service would be triggered by a timer (`nifi-dataflow-customer.timer`).
    - `dbt-run-daily.service`: A service that executes a `dbt run` command.  This would be triggered by a timer (`dbt-run-daily.timer`).
    - `great-expectations-validation.service`: A service that runs a Great Expectations checkpoint (likely using the GE CLI).  This could be triggered by a timer or by another service (e.g., after a `dbt run`).
  - Databases (PostgreSQL, MongoDB, Neo4j): The service units triggered by the timers would interact with the databases using appropriate client libraries (e.g., `psycopg2` for PostgreSQL, `pymongo` for MongoDB, `neo4j-driver` for Neo4j).  The service units would be configured with the necessary database credentials (obtained from a secrets management system, *not* hardcoded).
  - Hudi, MinIO: Similar to databases, service units would interact with Hudi and MinIO using appropriate libraries (e.g., `pyarrow` for Hudi, `boto3` for MinIO/S3).
  - Keycloak: `systemd` itself *doesn't* directly integrate with Keycloak for authentication. The *services* that are triggered by the timers would be responsible for authenticating with Keycloak (if necessary) using the methods described previously (e.g., obtaining a JWT using the client credentials flow). The service unit could obtain a token *before* executing the main task and pass it as an environment variable or command-line argument.
  - Monitoring (Prometheus, Grafana, Loki, Promtail): `systemd`'s integration with the journal (`journalctl`) makes monitoring straightforward.
    - Loki/Promtail: Promtail is already configured to collect logs from the `systemd` journal, so it will automatically pick up logs from services triggered by timers.
    - Prometheus: You would typically use a Prometheus exporter (like the `node_exporter`) to collect system-level metrics.  For application-specific metrics, the services triggered by the timers would need to expose them (e.g., a Python service using the Prometheus client library).
    - Grafana: Grafana can visualize both system-level metrics (from Prometheus) and logs (from Loki), providing a unified view of the system's health and the status of scheduled tasks.
  - Kong: If a service unit needs to expose an API, that API would still be managed by Kong Gateway. The service unit runs, potentially listening on a port, and Kong is configured to proxy requests to that service. The `systemd` timer only controls *when* the service runs, not how external clients access it.

### 3. The Data Lakehouse layer
The Data Lakehouse layer provides a single, unified storage layer that combines the benefits of data lakes (cost-effective, schema-on-read) and data warehouses (ACID transactions, schema enforcement). This layer is used to store both raw and processed data in a scalable, reliable way, and require data consistency (ACID properties) for analytical workloads.
#### MiniIO
  - Before writing data to Hudi, use processing layer (Pandas, dbt, etc.) to mask or anonymize sensitive fields.  This is the *preferred* approach. The transformed data instead of the raw data should be stored.
  - MinIO supports IAM policies similar to AWS, allowing fine-grained control over buckets and objects.  It integrates with external IdPs.
  - MinIO supports server-side encryption (SSE) with various key management options (MinIO-managed keys, external KMS like AWS KMS or HashiCorp Vault).  This is *essential* for protecting data at rest.
  - MinIO can log all object access events (GET, PUT, DELETE).
  - MinIO is a good choice for on-premise object storage.  For cloud deployments, use the native object storage services (AWS S3, Azure Blob Storage, Google Cloud Storage).  Hudi integrates seamlessly with these.
#### Apache Hudi
- **Hudi features of interest**
  - ACID Transactions: This is the *core* differentiator. Hudi brings transactional guarantees (Atomicity, Consistency, Isolation, Durability) to data lakes. This is essential for data quality and reliability, especially when multiple processes are writing and reading data concurrently.  It prevents the "dirty reads," "non-repeatable reads," and "phantom reads" problems that can occur in traditional data lakes.
  - Upserts and Deletes: Hudi allows *update* and *delete* of individual records in data lakehouse, not just append new data. This is crucial for:
    - CDC: Handling updates and deletes from source databases (via Debezium).  Without upserts/deletes, one would have to rewrite entire partitions, which is incredibly inefficient.
    - Data Corrections: Fixing errors in data without reprocessing everything.
    - GDPR Compliance: Enabling "right to be forgotten" requests by deleting specific user data.
  - Schema Evolution: Hudi can handle changes to data schema over time (adding columns, changing data types).  This is inevitable in real-world projects. Hudi provides mechanisms to manage these changes gracefully, preventing pipelines from breaking.
  - Incremental Queries: Hudi allows query only the *changes* since a specific point in time.  This is far more efficient than scanning the entire dataset for every query, especially for large tables.  This is related to, but distinct from, CDC.  CDC *captures* changes; Hudi's incremental queries *consume* those changes efficiently.
  - Built-in Metadata Management: Hudi tracks metadata about data (commits, file sizes, schemas) within the table itself.  This simplifies data management and enables features like time travel.
  - Concurrency Control: Hudi uses optimistic concurrency control (OCC) to manage multiple writers. This assumes conflicts are rare and checks for them at commit time. If a conflict is detected (two writers modifying the same data), one of the writes will be rejected.
  - Snapshot Isolation: Provides a consistent view of the data at a specific point in time, crucial for consistent reads even during writes.
- **Hudi Table Types: CoW vs. MoR**
  - This is a critical choice that impacts write and read performance.
  - Copy-on-Write (CoW):**
    - Mechanism: On every update, the *entire* file containing the record being updated is rewritten with the new data.
    - Pros:
      - Simpler: Easier to understand and manage.
      - Faster Reads: Data is stored in columnar format (Parquet) directly, optimized for read performance.
    - Cons:
      - Slower Writes: Rewriting entire files is expensive, especially for frequent, small updates.
      - Higher Write Amplification:  A small update can cause a large amount of data to be written.
    - Best For: Read-heavy workloads with infrequent updates or batch updates.
  - Merge-on-Read (MoR):
    - Mechanism: Updates are written to smaller "delta files" (log files, usually in Avro format).  Reads combine the base file (Parquet) with the delta files to reconstruct the latest version of the data.  Compaction (see below) merges delta files into new base files periodically.
    - Pros:
      - Faster Writes:  Writing small delta files is much faster than rewriting entire files.
      - Lower Write Amplification:**  Only the changed data is written initially.
    - Cons:
      - Slower Reads:  Reading requires merging base and delta files, which adds overhead.
      - More Complex:  Requires understanding and managing compaction.
    - Best For: Write-heavy workloads with frequent updates (e.g., CDC streams).
  - Choosing the Right Type for OllaLab:
    - CDC Data (from Debezium): MoR is strongly recommended.  CDC streams generate frequent updates and deletes, making CoW extremely inefficient.
    - Batch Data (from NiFi): CoW might be suitable *if* the batches are large and infrequent. If the batch process involves updates/deletes to existing records, then MoR is a good choice even for batch processing.
    - Processed Data (from Pandas/dbt): This depends on *how* the processing is done. If Pandas/dbt are producing complete snapshots (overwriting the data), CoW is fine. If they are making incremental updates, MoR is better.
- **Hudi Clustering and Compaction**
  - These are essential for optimizing query performance, especially for MoR tables.
  - Compaction (MoR Only):**
    - Purpose:**  Merges the small delta files (Avro) into larger base files (Parquet).  This reduces the overhead of merging files during reads.
    - How it Works:**  Hudi provides a "compactor" process that runs asynchronously. It reads the delta files and rewrites the base files with the merged data.
    - Configuration:**  You can control when compaction runs (inline or scheduled), how many delta commits trigger compaction, and other parameters.  The example above shows inline compaction.
  - Inline vs. Scheduled Compaction:
    - Inline Compaction: Compaction happens as part of the write operation.  This is simpler but can slow down writes.
    - Scheduled Compaction: Compaction runs as a separate, asynchronous process.  This is better for production workloads, as it decouples compaction from write latency.
  - Clustering (CoW and MoR):
    - Purpose: Physically co-locates data within files based on the values of one or more columns. This improves query performance by reducing the amount of data that needs to be scanned.  Think of it as "sorting within partitions."
    - How it Works:  During writes, Hudi can reorder data within files based on a "clustering key."  This is different from partitioning, which divides data into separate directories.
    - Example: If users frequently query data by `customer_id`, one might choose `customer_id` as the clustering key. Hudi will try to store records with the same `customer_id` close together within the Parquet files.
    - Configuration: specify the clustering key when creating the table. One can also configure how often clustering runs (inline or scheduled, similar to compaction).  Hudi provides different clustering strategies (e.g., Z-order clustering, Hilbert space-filling curves).
    - Inline vs. Scheduled Clustering: Similar to compaction, inline clustering happens during writes, while scheduled clustering runs as a separate process. Scheduled clustering is generally preferred for production.
  - Integration with Airflow
    - In Airflow, one would schedule both compaction and clustering as separate tasks.
    - One would create an Airflow DAG to periodically trigger the `hudi-cli` tool to perform compaction on MoR tables.
- **Hudi Indexing**: Hudi provides several indexing options to speed up lookups:
  - Bloom Filters (Default):**
    - How it Works: A Bloom filter is a probabilistic data structure that can tell you if a value is *definitely not* present in a file, or *possibly* present.  Hudi stores Bloom filters for the key columns in the file metadata.
    - Pros: Space-efficient, fast to check.
    - Cons: Can have false positives (it might say a value is present when it's not). Hudi handles this by reading the file if the Bloom filter returns a "maybe."
    - Best For: Most workloads.  It's a good default choice.
  - HBase Index:
    - How it works: Uses Apache HBase as an external index to store the mapping between record keys and file locations.
    - Pros: Fast lookups.
    - Cons: Requires managing an HBase cluster. Adds complexity.
  - Record-Level Indexes (SIMPLE, BUCKET):
    - How it works: Store index within the Hudi metadata itself
    - Pros: Fast lookups and updates
    - Cons:  Can increase the size of your Hudi metadata. Not recommended for very large data sets
  - Choosing the Right Index:
    - For OllaLab, start with **Bloom filters**. They're usually sufficient and don't add much overhead.
    - If you have *very* strict latency requirements for point lookups (finding a specific record by its key), and Bloom filters aren't fast enough, *then* consider record-level indexes, or HBase Index.
- **Hudi Time Travel and Rollback**
  - Time Travel: Hudi keeps track of all commits to a table. You can query the table "as of" a specific timestamp or commit ID.  This is incredibly useful for:
    - Debugging: "What did the data look like yesterday before this pipeline ran?"
    - Auditing: Reproducing past reports.
    - Reproducibility: Ensuring that your ML models can be trained on the same data used in the past.
  - Rollback: If a write operation fails or introduces errors, you can roll back the table to a previous, known-good state.
- **Hudi vs. Delta Lake vs. Iceberg**
  - Hudi:** Strong focus on upserts/deletes and incremental processing. Excellent CDC support. Good integration with Spark and other processing engines.
  - Delta Lake:** Developed by Databricks.  Also provides ACID transactions and schema evolution.  Tightly integrated with Spark.  Historically, Delta Lake had stronger support for schema enforcement and evolution, but Hudi has caught up.
  - Apache Iceberg:**  Another open-source table format.  Focuses on performance and scalability for very large datasets.  Good support for schema evolution.  Increasingly popular.
#### Hudi Integration with Keycloak (IdP)
Hudi itself doesn't have *direct* integration with Keycloak.  Hudi focuses on storage and data management, not authentication/authorization. The integration happens at the *access* layer – how users and services *access* the Hudi data. Here's how it works in the OllaLab context:
- **Data Access via Processing Engines:**
  - Users and applications don't interact with Hudi files directly. They use processing engines like DuckDB, or dbt.  These engines are responsible for enforcing authorization.
- **Service Accounts:**
  - Create service accounts in Keycloak for each component that needs to access Hudi data (dbt, Airflow, etc.).
  - Assign appropriate roles to these service accounts (e.g., `hudi_writer`, `hudi_reader`).  These roles are defined in Keycloak.
- **Authentication:**
  - When a component (e.g., a Spark job) starts, it authenticates with Keycloak using its service account credentials (client ID and secret).
  - Keycloak issues a JWT (JSON Web Token) access token containing the service account's roles.
- **Authorization**
    - The Hudi related job uses the Hudi libraries to read/write data.
    - The Hudi libraries themselves *don't* handle authorization.  The processing engine needs to be configured to use the JWT token and enforce authorization based on the roles in the token.
#### DuckDB
- direct querying of Hudi/Parquet files
- tba

### 4. The Batch/Stream Processing layer
The Batch/Stream Processing is used to clean, transform, enrich, and aggregate data from the data lakehouse (or directly from ingestion) into a format suitable for analysis and other downstream uses. It is ecessary whenever raw data needs to be prepared before it can be used for reporting, machine learning, or other applications.
#### Pulsar Functions
The primary goal of stream processing within this layer is to perform real-time or near real-time transformations, enrichments, and aggregations on data ingested via Apache Pulsar (and potentially Debezium CDC streams, which ultimately feed into Pulsar).  This processed stream data can then be:
  - Written to the Data Lakehouse (Apache Hudi) for historical analysis.
  - Sent to a database (PostgreSQL, MongoDB) for operational dashboards or applications.
  - Used to trigger immediate actions (alerts, notifications).
  - Fed into the Analytics/ML layer (e.g., real-time feature engineering).
- **Mechanism:**  Pulsar Functions are the most natural and efficient way to implement stream processing within OllaLab.  They are lightweight, serverless compute functions that operate directly on messages flowing through Pulsar topics.
- **How it Works:**
  1.  **Data Input:**  A Pulsar Function consumes messages from one or more Pulsar topics (e.g., the raw data streams from Debezium or NiFi).
  2.  **Processing Logic:**  The function executes custom code (Java, Python, or Go) on each message.  This code can perform:
      - Filtering: Discard irrelevant messages.
      - Transformation: Modify the message structure, data types, or values (e.g., converting timestamps, cleaning strings, adding calculated fields).
      - Enrichment: Add data from external sources (e.g., looking up customer information from a database based on a customer ID in the message).
      - Aggregation: Calculate rolling statistics (e.g., average transaction value over the last 5 minutes).  This typically involves using Pulsar's stateful processing capabilities (more on this below).
      - Data Quality Checks: Integrate with the data contract enforcement mechanisms (schema validation, custom rules).  Invalid messages can be routed to a dead-letter topic.
      - Integration with GE: A Pulsar Function can call Great Expectations for data quality validation, leveraging GE's Expectations and Checkpoints.
  3.  **Data Output:**  The function produces output messages to one or more Pulsar topics. These output topics can be:
      - The input topic for another Pulsar Function (creating a chain of processing steps).
      - A topic that feeds into Hudi (via Hudi's Pulsar connector).
      - A topic consumed by another application or service.
- **Advantages:**
  - Tight Integration: Pulsar Functions are deeply integrated with Pulsar, providing low latency and high throughput.
  - Scalability: Pulsar automatically scales the number of function instances to handle the message load.
  - Fault Tolerance: Pulsar Functions are inherently fault-tolerant. If a function instance fails, Pulsar automatically restarts it.
  - Stateful Processing: Pulsar Functions can maintain state (e.g., for aggregations). This state is managed by Pulsar and is fault-tolerant.
  - Exactly-Once Processing (with Idempotency): Pulsar Functions can be configured to provide exactly-once processing semantics, ensuring that each message is processed exactly once, even in the presence of failures.  This is crucial for many data processing tasks.
  - Ease of Development and Deployment: Functions can be written in familiar languages and deployed easily using the Pulsar CLI or API.
  - Cost-Effective: Pulsar Functions are serverless, so you only pay for the compute resources they consume.
- **Stateful Processing Details:**
  - Pulsar uses BookKeeper (its underlying storage layer) to store the state of Pulsar Functions.
  - The state is stored in a fault-tolerant and replicated manner.
  - The state is accessible to the function code through the Pulsar Function's context object.
  - The state can be updated atomically (all-or-nothing updates).
#### Data Build Tool (DBT)
- **DBT Features of Interest**
  - SQL-based Transformations: dbt allows writing data transformations using SQL, making it accessible to analysts and engineers familiar with SQL.  This is its core strength.
  - Modularity: dbt projects are organized into models (SQL `SELECT` statements) that represent individual transformations.  Models can be referenced and reused, promoting DRY (Don't Repeat Yourself) principles.
  - Templating (Jinja): dbt uses Jinja templating, which allows writing dynamic SQL.  This is crucial for:
    - Referencing other models (`{{ ref('model_name') }}`).
    - Using variables (`{{ var('my_variable') }}`).
    - Conditional logic (if/else statements).
    - Loops (for generating repetitive SQL).
    - Configuring environments (development, staging, production).
  - Testing: dbt allows defining tests (assertions) on models and sources to ensure data quality. This is *critical* for OllaLab.
  - Documentation: dbt can generate documentation from project's code and metadata, making it easier to understand data pipelines.
  - Source Freshness Checks: dbt can check if source data is up-to-date, preventing stale data from being processed.
  - Snapshots: dbt snapshots capture the state of mutable data over time, allowing track changes. This is a form of slowly changing dimension (SCD) management.
  - Packages: dbt allows install and use pre-built packages of models and macros, accelerating development.
  - Macros: Reusable SQL snippets that can be called from multiple models.
  - Hooks: Scripts that can be executed before or after dbt commands (e.g., `pre-commit` hooks for code formatting).
  - Exposure: Defines and documents downstream uses of dbt models, providing a basic form of data lineage.
- **dbt + DuckDB:**
  - Local Development & Testing: DuckDB's in-process nature makes it ideal for developing and testing dbt models locally.
  - Performance: DuckDB's columnar storage and vectorized query engine provide excellent performance for analytical queries, making dbt transformations fast.
  - Integration: The `dbt-duckdb` adapter provides seamless integration between dbt and DuckDB.
- **Integration**
  - No Direct Keycloak Integration:** dbt itself doesn't directly interact with Keycloak. dbt operates on *data* within a database.
  - Database Connection:** The database that dbt connects to (DuckDB, PostgreSQL, etc.) should be configured to use service accounts managed by Keycloak. dbt connects to the database using credentials configured in its `profiles.yml` file. These credentials should correspond to a service account, not an individual user.
  - Airflow, which *runs* dbt, would authenticate with Keycloak to obtain a token, and that token's associated roles would determine what databases Airflow can access. The service account Airflow uses would need permissions to connect to the target database (DuckDB/Postgres) and execute queries.
#### Great Expectations (GE)
- **GE Features of Interest**
  - Expectations:  Assertions about data (e.g., "column X should be unique," "column Y should be between 0 and 100").  These are the core building blocks.
  - Expectation Suites:  Collections of Expectations that are grouped together logically (e.g., by table, data source, or business process).
  - Data Contexts:  Configurations that define data sources, expectation suites, and other GE settings.
  - Checkpoints: The mechanism for running validations.  A Checkpoint bundles together:
    - A Data Context.
    - One or more Expectation Suites.
    - One or more Batches of data to validate.
    - Actions to perform after validation (e.g., update Data Docs, send notifications).
  - Data Docs: Human-readable HTML documentation that shows expectations, validation results, and data quality metrics.  These are invaluable for collaboration and communication.
  - Profilers: Automated tools that can analyze data and suggest Expectations.  This can help quickly create a baseline set of Expectations.
  - Validation Operators/Actions: Components that perform actions after validation, such as sending notifications, updating Data Docs, or stopping a pipeline.
  - Data Sources: GE supports a wide range of data sources, including databases (PostgreSQL, DuckDB, etc.), file systems (CSV, Parquet), and cloud storage (S3, GCS, Azure Blob Storage).
- **Expectation Suites Organization:**
  - By Data Source: Create a separate suite for each data source (e.g., "PostgreSQL-orders," "MongoDB-users," "S3-logs").  This is a good starting point.
  - By Table/Entity: Create a suite for each table or entity (e.g., "orders," "customers," "products"). This is more granular.
  - By Business Process: Group expectations related to a specific business process (e.g., "order fulfillment," "user registration").
  - Hybrid Approach: Combine the above approaches. For example, you might have a suite for each data source *and* separate suites for critical tables.
  - The approach used depends on how the team is organized and the complexity of the data assets.
- **Data Docs:**
  - Centralized Documentation: Data Docs provide a single source of truth for data quality expectations and validation results.
  - Collaboration: Data Docs make it easy for different teams (data engineers, analysts, scientists) to understand the quality of the data.
  - Version Control: Store Data Docs in a version control system (Git) to track changes over time.
  - Hosting: One can host Data Docs on a static web server (e.g., MinIO, S3, Netlify) for easy access.
- **Integration with Airflow (GreatExpectationsOperator):**
    ```python
    from airflow import DAG
    from airflow.providers.great_expectations.operators.great_expectations import GreatExpectationsOperator
    from datetime import datetime

    with DAG(
        dag_id='great_expectations_example',
        start_date=datetime(2023, 1, 1),
        schedule_interval=None,  # Or your desired schedule
        catchup=False
    ) as dag:
        ge_checkpoint_task = GreatExpectationsOperator(
            task_id='ge_validate_orders',
            checkpoint_name='orders_checkpoint',  # Name of your GE Checkpoint
            data_context_root_dir='/path/to/your/ge/project',  # Path to GE project
        )
    ```
  - Explanation:**
    - `GreatExpectationsOperator`:  The Airflow operator that runs GE Checkpoints.
    - `checkpoint_name`:  The name of the Checkpoint you defined in your GE project.
    - `data_context_root_dir`:  The path to your GE project directory.
    - The operator will execute the Checkpoint, which will:
      1.  Load the specified Expectation Suite.
      2.  Connect to the data source defined in your Data Context.
      3.  Validate the data against the Expectations.
      4.  Perform any configured actions (e.g., update Data Docs).
    - Failure Handling:**  If any Expectations fail, the `GreatExpectationsOperator` will fail the Airflow task, preventing downstream tasks from running.

        ```
- **Integration**
  - No Direct Keycloak Integration: Similar to dbt, GE primarily interacts with data sources.
  - Data Source Credentials: The credentials used by GE to connect to data sources (databases, file systems) should be managed securely, ideally using service accounts authenticated via Keycloak.
  - Airflow Integration: The `GreatExpectationsOperator` in Airflow runs within the Airflow context. Airflow's connection to Keycloak ensures that GE runs with appropriate permissions.
  - Data Docs Hosting: If you host Data Docs on a web server that requires authentication, you can configure it to use Keycloak for access control. This would allow only authorized users to view the Data Docs.

### 5. The Metadata Catalog layer
The Metadata Catalog layer stores and manages metadata about data assets (schemas, tables, columns, data lineage, ownership, etc.), making it easier to discover, understand, and govern data. This layer is essential for large organizations with many data sources and users. This layer iomproves data discoverability, collaboration, and data governance.
#### PostgreSQL
PostgreSQL, being a robust and familiar relational database, can effectively serve as a basic, yet functional, metastore, particularly as a starting point. Here's how you'd structure it:
- **Tables for Metadata Entities:** Create PostgreSQL tables to represent the different types of metadata you want to manage.  Here's a conceptual example, which you'd adapt to your specific needs:
  - `data_sources`:
    - `source_id` (SERIAL PRIMARY KEY)
    - `source_name` (VARCHAR, UNIQUE) - e.g., "PostgreSQL-Orders", "S3-Logs", "MongoDB-Users"
    - `source_type` (VARCHAR) - e.g., "PostgreSQL", "S3", "MongoDB", "Hudi"
    - `connection_details` (JSONB) - Store connection parameters (host, port, database, credentials) securely.  Ideally, reference a secret stored in a secrets manager.
    - `description` (TEXT)
    - `owner` (VARCHAR) -  Could be a user ID from Keycloak.
    - `created_at` (TIMESTAMP)
    - `updated_at` (TIMESTAMP)
  - `datasets`:
    - `dataset_id` (SERIAL PRIMARY KEY)
    - `source_id` (INTEGER REFERENCES data_sources)
    - `dataset_name` (VARCHAR) - e.g., "orders", "customers", "products", "log_events"
    - `dataset_path` (VARCHAR) -  e.g., table name for databases, file path for S3, Hudi path.
    - `schema_version` (VARCHAR) -  Link to the schema registry (Apicurio) version.
    - `description` (TEXT)
    - `owner` (VARCHAR) -  Could be a user ID from Keycloak.
    - `created_at` (TIMESTAMP)
    - `updated_at` (TIMESTAMP)
  - `columns`:
    - `column_id` (SERIAL PRIMARY KEY)
    - `dataset_id` (INTEGER REFERENCES datasets)
    - `column_name` (VARCHAR)
    - `data_type` (VARCHAR)
    - `description` (TEXT)
    - `is_primary_key` (BOOLEAN)
    - `is_nullable` (BOOLEAN)
    - `data_quality_rules` (JSONB) - Store simple, declarative rules (e.g., `{"min": 0, "max": 100}`). More complex rules would likely be handled externally.
    - `sensitivity_level` (VARCHAR) - e.g., "public", "internal", "confidential".  Relates to data classification.
  - `data_lineage`: (This is a simplified representation. True lineage can get very complex.)
    - `lineage_id` (SERIAL PRIMARY KEY)
    - `source_dataset_id` (INTEGER REFERENCES datasets)
    - `target_dataset_id` (INTEGER REFERENCES datasets)
    - `transformation_description` (TEXT) -  e.g., "Filtered and aggregated orders data."
    - `transformation_script` (TEXT) - Could store the SQL query (from dbt, for example) or a reference to the Airflow DAG/task.
    - `created_at` (TIMESTAMP)
  - `tags`: (Optional, for easier searching and categorization)
    - `tag_id` (SERIAL PRIMARY KEY)
    - `tag_name` (VARCHAR, UNIQUE)
  - `dataset_tags`: (Many-to-many relationship between datasets and tags)
    - `dataset_id` (INTEGER REFERENCES datasets)
    - `tag_id` (INTEGER REFERENCES tags)
    - `PRIMARY KEY (dataset_id, tag_id)`
  - `column_tags`: (Optional)
  - `business_glossary`: (Optional)
- **Data Population:**
  - Manual Entry: Initially, you might populate some metadata manually (e.g., defining data sources).
  - Automated Extraction: The *key* to a successful metastore is automation.  You'll want to write scripts (likely in Python) that:
    - Connect to your various data sources (databases, schema registry, file systems).
    - Extract metadata (schemas, table names, column names, data types).
    - Insert or update the metadata in the PostgreSQL tables.
    - These scripts would ideally be run as part of your data ingestion pipelines (e.g., as Airflow tasks).
    - Extraction from source databases can leverage existing database introspection capabilities.
    -  Extraction from Hudi tables can use Hudi's metadata.
    - Extraction from Apicurio can be done via Apicurio's REST API.
- **Data Governance Considerations:**
  - Ownership: The `owner` field in the `data_sources` and `datasets` tables is crucial.  This should be linked to a user or group in Keycloak.  This establishes responsibility for the data asset.
  - Data Quality: The `data_quality_rules` field in the `columns` table can store basic, declarative rules.  More complex rules, and the actual *enforcement* of those rules, would be handled by other components (Great Expectations, pre-ingestion scripts, Pulsar Functions). The metastore simply *stores* the rules.
  - Data Classification: The `sensitivity_level` field is key for data classification.  This informs access control policies.
- **Integration with Other Components**: This is where the metastore becomes truly valuable.  It's not just a passive repository; it actively *enables* other parts of the system.
  1.  **Ingestion Layer:**
    - Schema Registry (Apicurio):** The `schema_version` field in the `datasets` table links to the specific schema version in Apicurio.  Ingestion components (NiFi, Debezium, Pulsar Functions) can query the metastore to get the schema information they need for validation and serialization/deserialization.
    - Data Contracts:** The metastore stores information related to data contracts (ownership, data quality rules).  Ingestion components can use this information to enforce contracts.
    - Pre-ingestion Scripts:** These scripts can query the metastore to get data quality rules and apply them *before* data enters the pipeline.
  2.  **Orchestration Layer (Airflow):**
    - Dynamic DAG Generation:** Airflow DAGs can query the metastore to dynamically generate tasks based on the available data sources and datasets.  For example, you could have a DAG that automatically creates tasks to process new tables added to a database.
    - Data Lineage Tracking:** Airflow tasks can update the `data_lineage` table in the metastore as they run, recording which datasets were used as input and which were produced as output. This is how you build automated lineage tracking. The Airflow Operators that interact with the data sources would be responsible for also updating the metadata.
    - Dependency Management:** Airflow sensors can query the metastore to check for the existence or readiness of specific datasets before triggering downstream tasks.
  3.  **Data Lakehouse Layer (Hudi):**
    - Schema Evolution:** When the schema of a Hudi table changes, the metastore should be updated to reflect the new schema version.  This could be done by an Airflow task that runs after the Hudi write operation.
    - Data Discovery:**  Users can query the metastore to find Hudi tables based on various criteria (name, description, owner, tags).
  4.  **Processing Layer (dbt, Great Expectations):**
    - dbt:** dbt models can reference metadata from the metastore (e.g., source table names, column descriptions). dbt can *also* be used to *populate* the metastore. dbt's `sources.yml` and `schema.yml` files can be parsed and their contents inserted into the PostgreSQL tables.
    - Great Expectations:** GE can query the metastore to get information about data sources and datasets, including data quality rules.  This allows you to centralize the definition of data quality rules and reuse them across different GE checkpoints.  The results of GE validations can *also* be stored back in the metastore (e.g., in a separate table like `validation_results`).
  5.  **Analytics & ML Layer (Superset, MLflow, MLJAR):**
    - Superset:** Superset can query the metastore to get a list of available data sources and datasets, making it easier for users to create dashboards and visualizations. The metastore could store information like which dashboards use which datasets.
    - MLflow/MLJAR:**  These tools can use the metastore to find datasets for training models, and to record metadata about the models themselves (e.g., which dataset was used for training, which features were used).
  6.  **Application Layer (FastAPI, Streamlit, SvelteKit):**
    - Data Discovery:** Applications can query the metastore to provide users with a searchable catalog of available data.
    - Data Access Control:** Applications can use metadata (e.g., sensitivity level) to enforce access control policies.
  7.  **Authentication and Authorization layer (Keycloak):**
      * The PostgreSQL database users/roles that components use to access the metastore should be linked to service accounts managed by Keycloak.  This ensures consistent access control.
      *  The `owner` fields in the metastore tables should store Keycloak user IDs or group IDs.
  8. **Monitoring Layer:**
      * The metastore itself (the PostgreSQL database) should be monitored like any other critical component (CPU, memory, disk space, query performance).
      *  You could potentially store metrics about the metadata itself (e.g., the number of datasets, the number of schema changes) in the metastore and visualize them in Grafana.
- **Limitations and When to Consider a Dedicated Catalog:**
  - Scalability: While PostgreSQL can handle a significant amount of metadata, it might not scale as well as dedicated metadata catalog solutions for *extremely* large and complex environments (thousands of datasets, petabytes of data, hundreds of users).
  - Advanced Features: PostgreSQL lacks some of the advanced features of dedicated catalogs, such as:
  - Automated data lineage discovery (especially across different systems).
  - Sophisticated search capabilities (faceted search, semantic search).
  - Collaboration features (comments, discussions, data quality ratings).
  - Business glossary management.
  - Impact analysis (understanding how changes to one dataset might affect others).
  - UI/UX: You'd need to build a custom UI (e.g., using Streamlit or SvelteKit) to provide a user-friendly way to interact with the metadata stored in PostgreSQL. Dedicated catalogs typically have a built-in web interface.
#### DataHub and other options
To better scale this layer, the options are: DataHub, Apache Atlas, and Amundsen.

| Feature       | Apache Atlas                 | Amundsen                    | DataHub                     |
| ------------- | ---------------------------- | --------------------------- | --------------------------- |
| Lineage       | Strong (Hadoop focus)       | Basic                       | Strong                      |
| Search        | Good (Technical & Business) | Excellent (User-Friendly)    | Good (User-Friendly)        |
| Collaboration | Basic                        | Strong                      | Good                        |
| Integration   | Custom Development Needed     | Custom Development Needed   | Growing Ecosystem, Some Built-in |
| Recommendation | Best for Governance        | Best for Discovery        | Best Overall Balance       |

Given OllaLab's diverse technology stack (Pulsar, NiFi, Hudi, dbt, etc.) and the need for both strong data lineage and good user discoverability, **DataHub** appears to be the best overall choice. While Atlas excels in lineage within the Hadoop ecosystem, DataHub's broader integration capabilities and real-time metadata focus are better suited to OllaLab's architecture. Amundsen's user experience is excellent for discovery, but its lineage support is less robust.

- **DataHub and Keycloak:** DataHub supports authentication and authorization via OIDC. You'll configure DataHub to use Keycloak as your OIDC provider, similar to how you configured Grafana. This allows users to log in to DataHub using their Keycloak credentials.
- **Service Accounts:** For automated ingestion (e.g., from Airflow), you'll create service accounts in Keycloak and grant them appropriate permissions within DataHub. These service accounts will be used by the ingestion processes.
- **RBAC in DataHub:** DataHub has its own RBAC system. You'll map Keycloak roles to DataHub roles to control access to metadata and features within DataHub.

#### Automated Metadata Extraction
Automated metadata extraction is crucial for keeping the metadata catalog up-to-date and minimizing manual effort. Here's a breakdown of how to automate metadata extraction for OllaLab's key components, focusing on DataHub as the chosen catalog:
- **Apache Pulsar:**
  - DataHub Pulsar Source (Pull): DataHub has a built-in source for extracting metadata from Pulsar. This connector can discover topics, schemas (from the schema registry), and basic statistics.
  - Configuration: You'll configure the Pulsar source with connection details (Pulsar service URL, authentication credentials).
  - Scheduling: The DataHub ingestion framework can be scheduled to run periodically (e.g., every hour, every day) to pull updated metadata.
- **Apache NiFi:**
  - Custom Processor (Push): The most robust approach is to develop a custom NiFi processor that extracts metadata from NiFi flows and pushes it to DataHub's API.
    - Metadata to Extract:
      - Flow Name, Description, Version
      - Processor Types, Configurations, and Connections
      - Input and Output Datasets (including schemas, if available)
      - Data Lineage (connections between processors)
    - Implementation: The processor would use NiFi's Java API to access flow information and DataHub's Python or Java client library to send metadata to DataHub.
    - Triggering: The processor could be triggered:
      - On flow deployment/update.
      - Periodically (e.g., every few minutes).
      - At the end of a flow's execution (to capture lineage).
  - Alternative (Less Robust): You could potentially use NiFi's provenance events (which contain information about data flow) and process them with a separate tool to extract metadata and push it to DataHub. However, this is less reliable than a custom processor.
- **Apache Hudi:**
  - DataHub Hudi Source (Pull): DataHub has a source for extracting metadata from Hudi tables. This connector can discover tables, schemas, partitions, and statistics.
  - Configuration: You'll configure the Hudi source with connection details (storage path, file system type).
  - Scheduling: Schedule the ingestion framework to run periodically.
- **Data Build Tool (dbt):**
  - DataHub dbt Source (Pull): DataHub has excellent integration with dbt. The `dbt` source can extract:
    - Models (tables and views)
    - Schemas
    - Tests
    - Documentation
    - Lineage (dependencies between models)
  - Configuration: Configure the source with details about your dbt project (e.g., the path to your `dbt_project.yml` file).
  - Integration with dbt runs: The ideal approach is to integrate DataHub ingestion into your dbt workflow. After each `dbt run` or `dbt test`, trigger the DataHub ingestion to update the metadata catalog.  This can be done using an Airflow operator or a shell script.
- **PostgreSQL, MongoDB, Neo4j:**
  - DataHub Database Sources (Pull): DataHub has built-in sources for these databases.  These connectors can extract:
    - Tables/Collections
    - Schemas (columns, data types)
    - Views
    - Stored Procedures (for PostgreSQL)
    - Relationships (for Neo4j)
  - Configuration: Configure each source with connection details (host, port, username, password, database name).
  - Scheduling: Schedule periodic ingestion.
- **Apache Airflow:**
  - DataHub Airflow Plugin (Push): DataHub provides an Airflow plugin that automatically captures lineage information from Airflow DAGs and tasks and pushes it to DataHub. This is the *recommended* approach for Airflow integration.
  - Installation: Install the plugin in your Airflow environment.
  - Configuration: Configure the plugin with connection details to your DataHub instance.
  - Benefits:
      - Automatic Lineage Tracking: Captures dependencies between tasks and datasets.
      - Operator-Level Granularity: Can track lineage at the operator level (e.g., which specific dbt model or Great Expectations checkpoint was executed).
- **Great Expectations:**
  - Custom Integration (Push): DataHub does not have a first-party connector for automatically ingesting metadata and data quality metrics from Great Expectations.
  - Use the Great Expectations API within the Airflow DAG to extract expectation results and push them to DataHub's API.
- **Handling Schema Evolution:**
  - Schema Registry (Apicurio): The schema registry (Apicurio) plays a crucial role in managing schema evolution. When a schema changes, the registry will track the new version.
  - DataHub Integration: DataHub's connectors (for Pulsar, Hudi, etc.) should be able to detect schema changes and update the metadata catalog accordingly.
  - Alerting: Configure alerts (in DataHub or your monitoring system) to notify you of schema changes, especially breaking changes.

### 6. The Database layer
The Database layer provides structured storage and efficient retrieval of data for various purposes. Different database types serve different needs. Relational database (i.e. PostgreSQL) is for transactional data, strong consistency, and complex queries. The NoSQL database (i.e. MongoDB) is for flexible schemas, high scalability, and handling semi-structured data. The Vector/Graph database (i.e. Neo4j) is for storing and querying relationships between data points, and performant vector similarity search.
Okay, let's break down the database layer, focusing on integration with Keycloak and highlighting Features of Interest. I'll address PostgreSQL, MongoDB (Community Edition), and Neo4j (Community Edition) individually.

#### General Considerations
- **Workload Characteristics:** The most crucial factor is understanding the *workload*. Are you primarily doing:
  - OLTP (Online Transaction Processing):** Many small, frequent read/write operations (e.g., user logins, order placements). Requires high concurrency and low latency.
  - OLAP (Online Analytical Processing):** Fewer, complex queries that analyze large amounts of data (e.g., reporting, business intelligence). Requires high throughput.
  - Mixed Workload:** A combination of OLTP and OLAP.
- **Data Volume and Growth:** Estimate the initial data volume and the expected growth rate. This significantly impacts storage requirements and scaling needs.
- **Performance Requirements:** Define specific performance targets:
  - Latency: How quickly do queries need to return results?
  - Throughput: How many queries/transactions per second must the database handle?
  - Concurrency: How many simultaneous users/connections will the database support?
- **High Availability (HA) and Disaster Recovery (DR):** Consider HA and DR requirements *from the start*. These are *not* afterthoughts.  HA minimizes downtime within a single environment, while DR protects against major outages.
- **Infrastructure:** Are you deploying on-premise, in the cloud (AWS, GCP, Azure), or a hybrid approach? Cloud providers offer managed database services (e.g., AWS RDS, Azure Database for PostgreSQL, Google Cloud SQL) that simplify management, but can be more expensive.
- **Cost:**  Cloud database costs can vary significantly based on instance type, storage, and data transfer.  Carefully consider your budget.
- **Iterative Approach:** Start with a reasonable initial configuration and *monitor* performance closely.  Scale up or out as needed, based on actual usage patterns.  Don't over-provision upfront.
- **Service Accounts, Not User Credentials:** Applications *never* access databases using individual user credentials.  Instead, applications use *service accounts*. These service accounts are managed within Keycloak and are granted specific, limited permissions within the database. This is a fundamental principle of least privilege and separation of concerns.
- **Keycloak as the Source of Truth:**  User management (creating, deleting, modifying user attributes, assigning roles) happens *exclusively* within Keycloak. The databases should *not* be used to manage end-user accounts.
- **Role Mapping:** The core challenge is mapping roles defined in Keycloak to database-specific permissions. This is where the integration details vary significantly between database systems.
- **Connection Pooling:**  This is crucial for performance and scalability, as it avoids the overhead of repeatedly establishing new database connections.  Most database drivers and ORMs support connection pooling.
* **Network Isolation**: Enforce network-level restrictions. Ensure that only authorized applications/services (running on specific hosts or within specific network segments) can connect to the databases. Leverage Security Groups (AWS/GCP/Azure) or equivalent firewall rules for on-premise environment.

#### PostgreSQL
- **Features of Interest**
  - ACID Compliance: Strong consistency guarantees (Atomicity, Consistency, Isolation, Durability), making it suitable for transactional workloads where data integrity is paramount.
  - Extensibility: Supports custom data types, functions, operators, and extensions (e.g., PostGIS for geospatial data).
  - Row-Level Security (RLS): Allows fine-grained control over which rows users can see or modify *based on their attributes*. This is extremely powerful for implementing complex authorization logic.
  - Views: Can be used to create simplified, restricted views of data, masking sensitive columns.
  - Triggers and Functions:Can be used to enforce complex business rules and data validation.
  - Foreign Data Wrappers (FDW): Can connect to external data sources, including other databases.  Less relevant for this specific integration, but good to know.
  - JSONB Data Type: Allows storing and querying JSON data efficiently.
  - pgAudit: A widely-used extension for detailed audit logging.
- **Integration with Keycloak** - The most robust and recommended approach for integrating PostgreSQL with Keycloak involves a combination of:
  - External Authentication (PAM or GSSAPI): PostgreSQL can delegate authentication to an external system.  The two primary options are:
    - PAM (Pluggable Authentication Modules):  A standard Linux mechanism for authentication. Need a PAM module that can interact with Keycloak.  `libpam-krb5` (using Kerberos) or a custom-built PAM module that understands OIDC/JWTs are possibilities. This is the most complex but potentially most flexible approach.
    - GSSAPI (Generic Security Services API): Typically used with Kerberos. If already using Kerberos for authentication, this is a good option.
  - Role Mapping (using `pg_ident.conf` or a Custom Script):  After authentication, PostgreSQL needs to map the authenticated *principal* (from Kerberos or the information passed by PAM) to a PostgreSQL role.
    - `pg_ident.conf`:  This file allows mapping external usernames (e.g., Kerberos principals) to PostgreSQL roles.  This is suitable for simple mappings.
    - Custom Script (for JWT Claims): If using a custom PAM module that can extract claims from a JWT, need a script to parse those claims and map the Keycloak roles to PostgreSQL roles. This is the most flexible approach for fine-grained role mapping based on Keycloak's role structure.
  - **Service Account Approach (Recommended):**
    - Create Service Accounts in Keycloak: Create a client in Keycloak specifically for application that needs to access PostgreSQL.  Within that client, create service accounts (one or more, depending on needs).  Assign Keycloak roles to these service accounts.
    - Configure Client Credentials Flow:  The application will use the "Client Credentials Grant" flow in OAuth 2.0.  It will present its client ID and client secret to Keycloak and receive an access token.
    - Application-Side Token Handling:
      1.  Obtain a JWT access token from Keycloak using the client credentials flow.
      2.  Parse the JWT (using a library like `python-jose`).  Extract the roles from the `realm_access.roles` claim (or a custom claim).
      3.  Connect to PostgreSQL using a dedicated PostgreSQL role that corresponds to the *most privileged* role found in the JWT.  This PostgreSQL role should have *minimal* permissions – ideally, only the ability to `SET ROLE`.
      4.  Immediately after connecting, issue the `SET ROLE` command to switch to a PostgreSQL role that matches the application's specific needs (derived from the JWT claims). This "least privilege" approach ensures that the initial connection has minimal permissions, and the application only gains the necessary privileges *after* validating the JWT.
  - **Row-Level Security (RLS) (Optional, but Powerful):**
    - If you have very fine-grained access control requirements (e.g., users can only see rows that belong to their organization), RLS is a powerful tool.
    - RLS policies are defined in PostgreSQL, but they can use information from the current user's session (which, in this case, is derived from the JWT).  You might use a PostgreSQL function to extract information from the `current_setting` (which you'd set based on the JWT) to determine row visibility.

#### MongoDB (Community Edition)
- **Features of Interest**
  - Document-Oriented: Stores data in flexible, JSON-like documents.  This allows for schema flexibility and easy adaptation to changing requirements.
  - Scalability:  Designed for horizontal scaling (sharding) to handle large datasets and high throughput.
  - Aggregation Framework: Powerful framework for data processing and analysis within the database.
  - Indexes: Supports various index types (single field, compound, text, geospatial) to optimize query performance.
  - Change Streams:  Allows applications to subscribe to real-time changes in the database (similar in concept to CDC).
  - Role-Based Access Control (RBAC) is supported
- **Integration with Keycloak:**
  - MongoDB Community Edition's native authentication mechanisms are *not* as directly compatible with OIDC/OAuth 2.0 as PostgreSQL's PAM/GSSAPI approach.  MongoDB Enterprise offers more advanced integration options (like LDAP and Kerberos), but for the Community Edition, the best approach is to handle authentication and role mapping *entirely within application logic* and use MongoDB's built-in RBAC system.
  - Keycloak Setup (Same as PostgreSQL):
    - Create a client in Keycloak for application.
    - Create service accounts within that client.
    - Assign Keycloak roles to the service accounts.
  - MongoDB Roles:
    - Create MongoDB roles that correspond to Keycloak roles (e.g., `mongo_data_engineer`, `mongo_analyst`, `mongo_viewer`).
    - Grant appropriate permissions to these MongoDB roles.  For example, `mongo_analyst` might have `read` access to specific collections, while `mongo_data_engineer` might have `readWrite` access.
  - Application-Side Logic (Similar to the PostgreSQL example, but simpler)
    - Obtain a JWT access token from Keycloak using the Client Credentials Grant flow.
    - Decode the JWT and extract the roles.
    - Connect to MongoDB using a dedicated MongoDB user that has permissions to connect, and to assume other roles.
    - The application should then use the `runCommand` to assume the appropriate roles based on the JWT information. This is the *key difference* from PostgreSQL.
- **Key Differences from PostgreSQL:**
  - No External Authentication: MongoDB Community Edition doesn't directly delegate authentication to Keycloak.  The application handles the JWT validation.
  - Role Assumption: The application connects with an initial user that has the ability to perform check on users and roles, then uses `usersInfo` command with roles extracted from Keycloak JWT.
  - Simplified Connection: The `pymongo` code is generally simpler because you're not dealing with PAM or GSSAPI.

#### Neo4j (Community Edition)
- **Features of Interest**
  - Graph Database: Data is stored as nodes, relationships, and properties.  Optimized for traversing relationships between data points.
  - Cypher Query Language: Declarative language specifically designed for querying graph data.
  - High Performance for Connected Data: Excels at queries that involve traversing complex relationships (e.g., finding all friends of friends who like a particular product).
  - ACID Transactions: Supports transactional operations, ensuring data consistency.
  - Schema Constraints (Optional): Can enforce constraints on node properties and relationship types.
- **Integration with Keycloak:**
  - Similar to MongoDB Community Edition, Neo4j Community Edition has limited built-in support for external authentication providers like Keycloak. Enterprise Edition offers more options (LDAP
  - Transparent Data Encryption (TDE) or similar mechanisms can be used to encrypt data files on disk.
  - Built-in audit logging capabilities can be configured to log data access events (SELECT, INSERT, UPDATE, DELETE) and administrative actions. Aggregate these logs to Loki + Promtail.

### 7. The Analytics & ML layer
The Analytics & ML layer enables data analysis, reporting, business intelligence, and machine learning. This layer is required for extracting insights from data, building predictive models, and automating decisions.
#### Query Engine / Interactive: duckDB
  - DuckDB is excellent for interactive queries on local computing resources and smaller datasets.  For larger datasets and more demanding workloads, consider Presto/Trino (for interactive SQL) or Spark (for batch processing).
  - tba
Okay, let's expand on the Analytics & ML layer, focusing on AutoML, and then dive into the specifics of each component in that layer.
#### Apache Superset
- **Features of Interest**
  - Interactive Dashboards: Create and share interactive dashboards with a variety of visualizations (charts, tables, maps, etc.).
  - SQL IDE: A built-in SQL editor for querying data sources and creating virtual datasets.
  - Wide Range of Data Sources: Connects to many databases (PostgreSQL, MySQL, Snowflake, BigQuery, etc.) and data lakehouse solutions (DuckDB, potentially Hudi through Presto/Trino).
  - Data Exploration: Allows users to slice and dice data, apply filters, and drill down into details.
  - Security and Access Control: Fine-grained permissions to control who can access and modify dashboards and data sources.
  - Caching: Supports caching to improve dashboard performance.
  - Alerts and Reports: Sending email and Slack notifications based on data changes or conditions.
  - Customizable Visualizations: Uses a pluggable architecture, so you can add custom visualizations if needed.
- **Keycloak Integration:**
  - Superset uses Flask AppBuilder (FAB) which provides authentication and authorization capabilities.
  - OIDC/OAuth 2.0:  Configure Superset to use Keycloak as an OpenID Connect (OIDC) provider. This is the standard and recommended approach.
  - Configuration Steps:
    1.  **Register Superset as a Client in Keycloak:** Create a new client in your Keycloak realm specifically for Superset. You'll need to configure:
      - Client ID: A unique identifier for Superset.
      - Client Secret: A secret key used for secure communication.
      - Valid Redirect URIs:  The URLs to which Keycloak can redirect the user after authentication (e.g., `https://your-superset-domain.com/oauth-authorized/keycloak`).
    2.  **Configure Superset (superset_config.py):**
      - Set `AUTH_TYPE = AUTH_OAUTH`.
      - Provide details about your Keycloak instance:
        - `OAUTH_PROVIDERS`:  Create a list of dictionaries, one for Keycloak.  Include:
            - `name`:  (e.g., "Keycloak")
            - `icon`: (optional, for the login button)
            - `token_key`:  (usually "access_token")
            - `remote_app`: Configure the Flask-AppBuilder-Security `OAuthRemoteApp` parameters:
                - `client_id`: Your Keycloak client ID.
                - `client_secret`: Your Keycloak client secret.
                - `api_base_url`:  The base URL of your Keycloak instance (e.g., `https://your-keycloak-domain.com/auth/realms/your-realm`).
                - `access_token_url`:  The URL for obtaining tokens (e.g., `/protocol/openid-connect/token`).
                - `authorize_url`:  The URL for initiating the authorization flow (e.g., `/protocol/openid-connect/auth`).
                - `request_token_params`: `{'scope': 'openid profile email roles'}` (request the necessary scopes).
      - `AUTH_ROLES_MAPPING`:  Map Keycloak roles (from the `roles` claim in the access token) to Superset roles.  For example:
            ```python
            AUTH_ROLES_MAPPING = {
                "data_engineer": ["Admin"],
                "analyst": ["Gamma"],  # Or a custom role you create
                "viewer": ["Public"],
            }
            ```
      - `AUTH_USER_REGISTRATION`:  Set to `True` to allow automatic user creation on first login.  You'll likely want this.
      - `AUTH_USER_REGISTRATION_ROLE`: The default role to assign to new users (e.g., "Public" or a custom "Viewer" role).

    3.  **Restart Superset:** After making these changes, restart Superset.
  - User Experience: Users will see a "Login with Keycloak" button.  When they click it, they'll be redirected to Keycloak for authentication.  After successful login, Keycloak will redirect them back to Superset, and Superset will automatically create a user account (if enabled) and assign roles based on the mapping.

#### MLflow
- **Features of Interest:**
  - Tracking Experiments: Log parameters, metrics, code versions, and artifacts (models, datasets) for each training run.
  - Model Registry: Manage model versions, stage them (Staging, Production, Archived), and add descriptions and tags.
  - Model Serving: Deploy models as REST APIs or batch inference jobs.
  - Projects: Package ML code in a reproducible format, making it easy to share and rerun experiments.
  - Pluggable Architecture: Integrates with many ML libraries (scikit-learn, TensorFlow, PyTorch, etc.).

- **Keycloak Integration:**
  - MLflow has *limited* built-in authentication. It primarily relies on external mechanisms for production deployments.
  - Reverse Proxy (Recommended Approach): The best practice is to put MLflow behind a reverse proxy like Nginx or Apache, and configure the proxy to handle authentication with Keycloak.
  - Direct Integration (Less Common, More Complex): It's *theoretically* possible to modify MLflow's Python code to directly integrate with Keycloak using a library like `python-keycloak`, but this is *not recommended* for production use. It would require significant code changes and would make upgrading MLflow difficult.  The reverse proxy approach is much cleaner and more maintainable.

#### MLJAR AutoML
- **Features of Interest:**
  - Automated Model Training:  Automates the entire model building process: data preprocessing, feature engineering, algorithm selection, hyperparameter optimization, and ensembling.
  - Explainable AI (XAI): Provides feature importance plots, SHAP values, and decision tree visualizations to explain model predictions.
  - Markdown Reports: Generates detailed reports for each experiment, summarizing the results and insights.
  - REST API Deployment:  Easily deploy trained models as REST APIs.
  - Python API: Simple and intuitive Python API for integrating into your workflows.
  - Resource Control: Set time limits, CPU/GPU constraints, and other parameters to manage resource usage.
- **Keycloak Integration:**
  - API Focus: MLJAR AutoML is primarily used through its Python API.  The API itself doesn't typically require authentication.  The *user* of the API (Python scripts) will need to be authenticated.
  - Integration within the Application Layer: The most common and recommended integration point is within the application layer (e.g., FastAPI, Streamlit, or Node.js application that uses MLJAR).
  - Workflow:
    1.  **User Authenticates to Your Application:** Your application (e.g., a Streamlit app) uses Keycloak (via a library like `python-keycloak` or `streamlit-keycloak`) to authenticate the user.
    2.  **Application Obtains Access Token:** After successful authentication, your application receives an access token (JWT) from Keycloak.
    3.  **Application Calls MLJAR API:**  Your application code then uses the MLJAR AutoML Python API to perform tasks (train models, make predictions, etc.).  The MLJAR API *itself* doesn't need the access token.
    4.  **Control Access to MLJAR *Results*:**  Your application controls access to the *results* of MLJAR AutoML (models, reports, predictions) based on the user's roles (obtained from the access token).  For example:
      - Only users with the `data_engineer` role can train new models.
      - Only users with the `analyst` role can view reports.
      - All authenticated users can make predictions using a pre-trained model.
    5. **MLJAR Rest API Authentication** If one deploy MLJAR models as Rest APIs, one can configure API Key authentication on MLJAR side. For a more secure scenario, the application should be placed behind Kong, and secured via OIDC as previously described.

### 8. The Application layer
The Application layer is used to build user-facing applications and services that leverage the data and insights from the underlying layers. It is driven by specific business requirements and is where data is turned into actionable value for end-users.
Okay, let's break down the Application Layer components, their features, and how they integrate with Keycloak for authentication and authorization.
#### Python & Streamlit
- **Features of Interest:**
  - Rapid Prototyping: Streamlit excels at quickly building interactive data apps and dashboards with minimal code.  It's designed for data scientists and engineers, not necessarily web developers.
  - Interactive Widgets: Provides a wide range of built-in widgets (sliders, text inputs, dropdowns, charts, etc.) that automatically update the app's output when their values change.  This makes it ideal for exploratory data analysis and creating user interfaces for models.
  - Simple API: Streamlit's API is very Pythonic and intuitive.  No need to write HTML, CSS, or JavaScript.
  - Caching: Streamlit has built-in caching mechanisms (`@st.cache_data` and `@st.cache_resource`) to improve performance by avoiding redundant computations.
  - Theming and Customization: While primarily focused on simplicity, Streamlit allows for some degree of customization through themes and custom components (though custom components require more web development knowledge).
  - Data Visualization: Integrates well with popular Python plotting libraries like Matplotlib, Plotly, Altair, and Bokeh.
  - Session State: Allows storing data that persists across interactions within a single user session.  This is crucial for building multi-step applications.
  - Deployment: Streamlit apps can be deployed easily on various platforms, including Streamlit Community Cloud, Docker containers, and cloud providers.
- **Integration with Keycloak (and other components):**
  - Authentication: Streamlit itself doesn't have built-in user management. The recommended approach is to use a library like `streamlit-keycloak` or `python-keycloak`. These libraries handle the OpenID Connect (OIDC) flow with Keycloak.  The process generally involves:
    1.  **Redirect to Keycloak:** When a user accesses the Streamlit app, they are redirected to Keycloak's login page.
    2.  **Keycloak Authentication:** The user authenticates with Keycloak (username/password, social login, etc.).
    3.  **Token Issuance:** Keycloak issues an ID token (and potentially an access token) to the Streamlit app.
    4.  **Token Validation:** The Streamlit app (using the library) validates the token's signature and expiration.
    5.  **User Information:** The app extracts user information (username, roles, etc.) from the ID token's claims.
    6.  Session Management: The application uses the acquired token or session information to maintain user sessions.
  - Authorization: Once authenticated, the Streamlit app can use the user's roles (from the ID token) to control access to features or data.  This is typically done with conditional logic in the Streamlit code (e.g., `if "admin" in user_roles: ...`).
  - Accessing Other Services: If the Streamlit app needs to access other protected resources (like data from a database or results from an ML model served by FastAPI), it can use the access token obtained from Keycloak.  The access token is passed in the `Authorization: Bearer <token>` header of the request to the protected service. The other services would then verify with Keycloak the validity of this token.
  - Database layer: If the Streamlit app needs to access, for example, data from PostgreSQL, it will send an authorization request through the Kong Gateway.
#### NodeJS & SvelteKit
- **Features of Interest:**
  - Component-Based Architecture: SvelteKit (and Svelte) use a component-based architecture, making it easy to build reusable UI elements.
  - Compiler-Based: Svelte is a compiler, not a runtime framework.  This means that the framework code is largely eliminated during the build process, resulting in smaller bundle sizes and faster performance.
  - Reactivity: Svelte's reactivity system automatically updates the DOM when data changes, without the need for a virtual DOM.
  - Server-Side Rendering (SSR) and Static Site Generation (SSG): SvelteKit supports both SSR (rendering pages on the server for each request) and SSG (rendering pages at build time).  This improves SEO and initial load times.
  - Routing: SvelteKit provides a file-system-based router, making it easy to define routes for applications.
  - API Routes:** SvelteKit allows creation of API endpoints within applications, making it suitable for full-stack development.
  - TypeScript Support: SvelteKit has excellent TypeScript support.
  - Hot Module Replacement (HMR):  HMR allows changes in code reflected in the browser without a full page reload, speeding up development.

- **Integration with Keycloak (and other components):**
  - Authentication: For a SvelteKit application, you'd typically use a library like `oidc-client-ts` or a SvelteKit-specific library for OIDC integration. The flow is very similar to the Streamlit case:
    1.  **Initiate Login:**  A button or link in the SvelteKit app initiates the OIDC flow.
    2.  **Redirect to Keycloak:** The user is redirected to Keycloak for authentication.
    3.  **Callback:** After successful authentication, Keycloak redirects back to a designated callback URL in the SvelteKit app.
    4.  **Token Handling:** The app receives the ID token (and potentially an access token) and validates it.
    5.  **User Session:**  The app creates a user session, typically storing the token in a secure HTTP-only cookie or local storage.
    6.  Protect routes using the session information.
  - Authorization:  Like with Streamlit, you'd use the user's roles (from the ID token claims) within your SvelteKit components and API routes to control access.  You can use Svelte's conditional rendering (`{#if ...}`) to show/hide UI elements based on roles.
  - Accessing Other Services:  When the SvelteKit app (either on the client-side or in a server-side API route) needs to access protected resources, it includes the access token in the `Authorization` header of the request.
  - Backend API Routes: If you're using SvelteKit's API routes, you can protect them using middleware that validates the access token.
  - Database layer: If the SvelteKit app needs to access, for example, data from PostgreSQL, it will send an authorization request through the Kong Gateway.
#### FastAPI
- **Features of Interest:**
  - High Performance: FastAPI is built on Starlette and Pydantic, making it one of the fastest Python web frameworks.
  - Automatic Data Validation: Uses Pydantic for data validation and serialization. This means you define your data models (request and response bodies) using Pydantic, and FastAPI automatically handles validation, type checking, and data conversion.
  - Automatic Documentation: Generates interactive API documentation (Swagger UI and ReDoc) automatically from your code and Pydantic models.
  - Dependency Injection: Has a powerful and easy-to-use dependency injection system.  This makes it easy to manage dependencies, like database connections or authentication logic.
  - Asynchronous Support: Built with `async` and `await`, allowing you to write asynchronous code that can handle many concurrent requests without blocking.
  - Type Hints: Leverages Python type hints to improve code readability, maintainability, and to enable automatic validation and documentation.
  - Security Features: Provides built-in support for common security features like OAuth2, JWT, and CORS.
- **Integration with Keycloak (and other components):**
  - Authentication: FastAPI makes integrating with Keycloak (or any OIDC provider) straightforward using libraries like `fastapi-users` or `python-keycloak`.  A common approach:
    1.  **Dependency Injection:** Create a dependency that handles authentication.  This dependency will:
      - Extract the `Authorization` header from the request.
      - Verify the JWT (using the Keycloak public key or JWKS endpoint).
      - Extract user information and roles from the token claims.
      - Optionally, fetch additional user details from Keycloak.
    2.  **Protect Endpoints:**  Use the authentication dependency in your FastAPI routes to protect them.  Only authenticated users with valid tokens will be able to access these routes.
  - Authorization:
      1.  **Role-Based Access Control:**  Within your protected routes, you can access the user's roles (from the token claims) and use them to control access to specific resources or operations.  This is often done with conditional logic or custom dependencies that check for specific roles.
      2.  **Scope-Based Access Control:**  OAuth2 scopes can also be used for authorization.  Scopes define specific permissions (e.g., `read:data`, `write:data`).  Your FastAPI endpoints can check for required scopes in the access token.
  - Accessing Other Services: If your FastAPI service needs to access *other* protected services, it can:
      - Client Credentials Flow: Use the client credentials flow (if the service acts on its own behalf) to obtain an access token from Keycloak.
      - Token Exchange: In some cases, you might use token exchange (if supported by Keycloak and the other service) to exchange the user's token for a token with appropriate scopes for the downstream service.
  - Database layer: If the FastAPI app needs to access, for example, data from PostgreSQL, it will send an authorization request through the Kong Gateway.
#### Kong Gateway
- **Features of Interest:**
  - API Gateway: Acts as a central point of entry for all API requests, routing them to the appropriate backend services.
  - Authentication and Authorization: Handles authentication and authorization using plugins like the JWT plugin and the ACL plugin.
  - Request Transformation: Can modify requests and responses (add headers, transform bodies, etc.).
  - Rate Limiting: Protects backend services from overload by limiting the number of requests from a client.
  - Traffic Control: Provides features like load balancing, health checks, and circuit breaking.
  - Logging and Monitoring: Logs API traffic and provides metrics for monitoring.
  - Plugin Architecture: Extremely extensible through a plugin architecture.  Many plugins are available, and you can write your own.
- **Integration with Keycloak (and other components):**
  - Authentication: The Kong JWT plugin is central to this integration.
    1.  **Configuration:** Configure the JWT plugin with the Keycloak public key or JWKS URL.  This allows Kong to verify the signature of JWTs issued by Keycloak.
    2.  **Request Interception:**  Kong intercepts all incoming API requests.
    3.  **Token Validation:**  If a request contains an `Authorization: Bearer <token>` header, Kong validates the token:
      - Signature verification.
      - Expiration check.
      - Issuer check (ensures the token was issued by Keycloak).
      - Audience check (ensures the token is intended for Kong).
    4.  **Upstream Headers (Optional):** Kong can be configured to pass user information (e.g., user ID, roles) to the backend services in custom headers.  This is *less secure* than having the backend services validate the token themselves.  It's generally recommended that backend services *also* validate the JWT.
  - Authorization:
    1.  **ACL Plugin:** The Kong ACL plugin can be used to implement role-based access control. You can map roles (from the JWT claims) to specific routes and HTTP methods.
    2.  **Custom Plugins:** For more complex authorization logic, you can write custom Kong plugins (in Lua).
    3.  **Delegation to Backend Services:** Kong can handle basic authorization (e.g., checking for the presence of a valid token), and then delegate finer-grained authorization to the backend services themselves. This is often the best approach for complex authorization rules.
  - Routing: After authentication and authorization, Kong routes the request to the appropriate backend service (FastAPI, SvelteKit API routes, etc.) based on the request path, headers, or other criteria.
  - Other components: Kong gateway protects any other components that are exposed through APIs.
#### Logging
- Applications should log significant events, including user actions and data access.
- Logging guidance - tba

### 9. The LLM layer
This layer is designed to facilitate the deployment, management, and monitoring of Large Language Models (LLMs).  It emphasizes open-source tools for greater control and flexibility, aligning with the overall architecture.
#### Ollama (LLM Management)
Ollama simplifies the process of running LLMs locally or on-premises. It's analogous to a "Docker for LLMs," providing a consistent and user-friendly way to manage and interact with various models.
- **Features of Interest:**
  - Simplified Model Management: Download, run, and switch between different LLMs (like Llama 2, Mistral, etc.) with simple commands.  Ollama handles the complexities of downloading model weights, setting up environments, and managing dependencies.
  - Local Execution:  Run LLMs *without* sending data to external APIs. This is crucial for privacy, security, and cost control, especially when dealing with sensitive data.  It aligns with the system's emphasis on on-premise and private cloud deployment.
  - API Endpoint: Ollama exposes a REST API, making it easy to integrate LLMs into applications. This API is similar in structure to OpenAI's API, simplifying the transition or enabling hybrid approaches.
  - Model Customization: Ollama allows users to create custom models by modifying existing ones (changing prompts, system messages, etc.). This is done through "Modelfiles," which are like Dockerfiles for LLMs.
  - GPU Support: Ollama automatically utilizes available GPUs for accelerated inference, significantly improving performance.
  - Multi-modal capabilities: supports models capable of working with multiple data types, such as images and text.
  - Built-in Models Registry: Ollama simplifies the process of discovering and obtaining LLM models.
- **Integration with Other Components:**
  - Keycloak (Authentication/Authorization):
    - API Authentication: Ollama itself *does not* natively support authentication or authorization. To secure the Ollama API, you'll put it behind Kong API Gateway. Kong will handle the Keycloak integration.
    - Kong Integration: Configure Kong with the Keycloak OIDC plugin.  API requests to Ollama must include a valid JWT access token issued by Keycloak. Kong will validate the token, and, if valid, forward the request to Ollama.  If invalid, Kong will reject the request.
    - RBAC:** Role-Based Access Control is managed at the Kong level.  For example, you might define roles like `llm-user` (can use certain models) and `llm-admin` (can manage models, configurations, etc.).  These roles are defined in Keycloak and included in the JWT claims. Kong's ACL plugin or custom logic will map these roles to specific routes/methods on the Ollama API.
  - Application Layer (FastAPI, Streamlit, etc.): Applications interact with LLMs through the Ollama API (proxied by Kong).  Applications will need to obtain a JWT from Keycloak and include it in the `Authorization` header of their requests.
  - Database Layer (Vector DB - Neo4j): Ollama can be used to generate embeddings (vector representations of text) that are then stored in Neo4j for similarity searches or other graph-based operations.  The application would use Ollama to generate the embedding and then use the Neo4j driver to store it.
  - Monitoring (Prometheus/Grafana): Ollama doesn't natively expose Prometheus metrics. However, you can use a third-party exporter or create custom scripts to scrape Ollama's API and expose relevant metrics (e.g., request latency, error rates, model usage).  These metrics can then be visualized in Grafana.  You would likely monitor the Kong gateway metrics (request counts, latency, error rates) related to the Ollama API endpoints.
  - Networking: Ollama should be deployed in a private subnet and should not be accessible from the public internet. Only Kong API Gateway should be able to directly interact with the Ollama instance.
#### Langfuse (LLM Monitoring)
Langfuse provides observability for LLM-based applications.  It helps track and analyze the performance, cost, and quality of LLM interactions, enabling debugging, optimization, and monitoring.
- **Features of Interest:**
  - Detailed Tracing: Langfuse captures detailed traces of LLM interactions, including inputs, outputs, latencies, token usage, and costs. This allows users to understand exactly what's happening during each interaction.
  - Performance Monitoring: Track key performance indicators (KPIs) like latency, throughput, and error rates.  Identify bottlenecks and areas for optimization.
  - Cost Tracking:** Monitor the cost of using LLMs (especially important for pay-per-use models like OpenAI).
  - Quality Evaluation: Langfuse provides tools for evaluating the quality of LLM outputs. This can involve manual evaluation (human feedback) or automated metrics (e.g., comparing outputs to known good answers).
  - Prompt Management: Experiment with different prompts and track their impact on model performance and quality.
  - User Feedback: Collect user feedback on LLM responses to improve model accuracy and relevance.
  - Alerting: Set up alerts for performance degradations, cost spikes, or quality issues.
  - SDKs: SDKs are available for Python, Javascript/Typescript.
- **Integration with Other Components:**
  - Keycloak (Authentication/Authorization):
    - Langfuse UI: Langfuse *does* support authentication (username/password, SSO). However, for consistent management, integrate it with Keycloak. Langfuse has documentation on how to integrate with various identity providers using OIDC. It may require some configuration of the Langfuse instance.
    - API Authentication (for SDKs): When your applications (FastAPI, Streamlit, etc.) use the Langfuse SDK to send traces, they will need to authenticate. This is typically done using API keys. Langfuse provides the ability to create and manage project level API keys (Secret Key and Public Key). It is best practice to store the Langfuse Secret Key securely in a secrets vault.
  - Ollama: Langfuse integrates directly with Ollama (and other LLM providers like OpenAI, Anthropic, etc.).  The Langfuse SDK provides wrappers and utilities to automatically capture traces from Ollama interactions.  This simplifies the integration process.
  - Application Layer: Applications (FastAPI, Streamlit, etc.) use the Langfuse SDK to instrument their LLM interactions.  The SDK sends traces to the Langfuse server (either self-hosted or Langfuse Cloud).
  - Monitoring (Prometheus/Grafana): Langfuse itself provides dashboards for visualizing the collected data.  It also exposes a Prometheus endpoint, allowing you to integrate Langfuse metrics into your existing Prometheus/Grafana setup. This provides a single pane of glass for monitoring the entire system.
  - Networking: If Langfuse is self hosted, the Langfuse instance should be deployed in a private subnet. If using Langfuse Cloud, applications will need to be able to connect to the Langfuse Cloud API (which can be facilitated through a NAT gateway).

### 10. The Monitoring layer
The Monitoring layer tracks the health, performance, and resource usage of all components in the system, and to provide alerts when issues arise. It is crucial for ensuring the reliability, availability, and performance of the entire data platform. It enables proactive problem detection and resolution.
#### Metrics Monitoring
##### Prometheus
- **Features of Interest:**
  - Multi-dimensional Data Model: Prometheus uses a time-series database. Data is identified by a metric name and a set of key-value pairs (labels). This allows for flexible and powerful querying.  Examples: `http_requests_total{job="api-server", method="GET", status="200"}`.
  - Pull Model: Prometheus actively *scrapes* metrics from configured targets (your applications and services).  This contrasts with "push" models where applications send metrics to the monitoring system.  The pull model simplifies configuration in many dynamic environments.
  - PromQL (Prometheus Query Language): A powerful query language specifically designed for querying time-series data.  It allows for aggregations, calculations, filtering, and more.
  - Service Discovery: Prometheus can automatically discover targets to monitor. This is particularly useful in dynamic environments (like Kubernetes) where services are constantly being created and destroyed. It supports various service discovery mechanisms (DNS, file-based, Kubernetes API, Consul, etc.).
  - Alerting: Prometheus has a built-in alerting system (Alertmanager). You define alerting rules based on PromQL expressions. When a rule's condition is met, Prometheus sends alerts to Alertmanager, which can then route them to various notification channels (email, Slack, PagerDuty, etc.).
  - Exporters: Prometheus uses *exporters* to expose metrics from various systems.  There are official exporters for many common services (databases, web servers, message queues, etc.), and a large community of third-party exporters.  You can also write your own custom exporters.
  - Storage: Prometheus stores data locally on disk in a highly optimized format. It's designed for short-term, high-resolution metrics. For very long-term storage, Prometheus can be integrated with remote storage solutions (like Thanos, Cortex, or M3DB).
  - Client Libraries: Client libraries exist for many programming languages, make it easy to instrument your applications to expose custom metrics.
- **Integration:**
  - Prometheus itself *does not* directly integrate with Keycloak for authentication or authorization. Prometheus's built-in web UI has very limited access control.
  - Best Practice: Reverse Proxy:** The standard and recommended approach is to put Prometheus behind a reverse proxy (like Nginx, Apache, or a dedicated authentication proxy like `oauth2-proxy`). This reverse proxy handles authentication and authorization against Keycloak.
    - The reverse proxy intercepts requests to the Prometheus UI.
    - If the user is not authenticated, it redirects them to Keycloak for login.
    - After successful authentication, Keycloak issues an ID token and (optionally) an access token.
    - The reverse proxy validates the token (typically by checking its signature against Keycloak's public key) and forwards the request to Prometheus *only if the token is valid and the user has the necessary roles/permissions*.
    - The reverse proxy can be configured to pass user information (e.g., username, roles) to Prometheus via HTTP headers, although Prometheus itself doesn't use this information for authorization.  This information might be useful for logging or auditing.
  - Service Accounts for Scraping:** Prometheus uses service accounts (or API tokens) to authenticate *itself* when scraping metrics from other services.  These are *not* Keycloak user accounts. For example, if Prometheus is scraping metrics from a PostgreSQL database that requires authentication, you'd configure Prometheus with a PostgreSQL username and password (or a connection string).  Similarly, if scraping from a Kubernetes cluster, Prometheus would use a Kubernetes service account with appropriate RBAC permissions.
##### Grafana
- **Features of Interest:**
  - Visualization: Grafana's core strength is creating beautiful and interactive dashboards. It supports a wide variety of visualization types (graphs, tables, heatmaps, gauges, etc.).
  - Data Sources: Grafana can connect to many different data sources, including Prometheus, Loki, databases (PostgreSQL, MySQL, etc.), cloud monitoring services (AWS CloudWatch, Azure Monitor, etc.), and more. This allows you to visualize data from multiple sources in a single dashboard.
  - Dashboards: Grafana's dashboards are highly customizable. You can create dashboards with multiple panels, each displaying data from different data sources or queries.
  - Alerting: Grafana has its own alerting system, which is separate from Prometheus's Alertmanager. Grafana alerts are defined directly within Grafana and can use data from any of its supported data sources.
  - Annotations: You can add annotations to your Grafana dashboards to mark important events (e.g., deployments, outages).
  - Templating (Variables): Grafana supports variables, which allow you to create dynamic dashboards. For example, you can create a variable for a specific instance or environment, and then use that variable in your queries to filter the data.
  - Plugins: Grafana has a rich plugin ecosystem, which allows you to extend its functionality (add new data sources, visualization types, etc.).
  - Provisioning: Grafana can be provisioned via configuration files, allowing for infrastructure-as-code approaches to managing dashboards and data sources.
- **Integration:**
  - Direct Keycloak Integration: Grafana has *built-in support* for authentication and authorization via Keycloak (and other OIDC providers). This is the *preferred* method, eliminating the need for a separate reverse proxy for Grafana itself (although you'll likely still want a reverse proxy for other reasons, like TLS termination).
  - Configuration:
    - You configure Grafana's `auth.generic_oauth` section in its configuration file (`grafana.ini`).
    - You provide Keycloak's client ID, client secret, authorization URL, token URL, and API URL (user info endpoint).
    - You can map Keycloak roles to Grafana roles (Admin, Editor, Viewer).  This is crucial for RBAC. For example, you might map the Keycloak role "grafana_admin" to Grafana's Admin role, and "grafana_viewer" to Grafana's Viewer role.
    - Grafana uses the information from the ID token (and optionally calls the user info endpoint) to determine the user's roles and map them to Grafana's internal roles.
  - RBAC within Grafana: Grafana has its own RBAC system (separate from Keycloak's roles), but the Keycloak integration allows you to *synchronize* roles.  Grafana's roles control what users can do *within Grafana* (create dashboards, edit data sources, manage users, etc.).
  - Data Source Permissions: Grafana allows you to restrict access to specific data sources based on user roles. This is important for multi-tenant environments or when you want to limit access to sensitive data.

#### Log Aggregation and Monitoring)
##### Loki
- **Features of Interest:**
  - Log Aggregation: Loki is a horizontally scalable, highly-available log aggregation system inspired by Prometheus. It's designed to be cost-effective and easy to operate.
  - Index and Labels: Loki *does not* index the full text of log messages. Instead, it indexes *metadata* (labels) associated with log streams. This makes the index much smaller and faster to query.  Log content is compressed and stored in chunks.
  - LogQL: Loki uses a query language called LogQL, which is similar to PromQL.  You query logs based on their labels.  LogQL allows for filtering, aggregations, and extracting data from log lines (using regular expressions).
  - Integration with Grafana: Loki is tightly integrated with Grafana. Grafana has native support for querying and visualizing logs from Loki.
  - Multi-tenancy: Loki supports multi-tenancy, allowing you to isolate logs from different teams or applications.
  - Storage: Loki can store log data in various object storage systems (like AWS S3, Google Cloud Storage, MinIO, or local files).
  - Retention: Loki allows you to configure retention policies to automatically delete old log data.
- **Integration:**
  - Like Prometheus, Loki *does not* have built-in Keycloak integration.
  - Reverse Proxy: The recommended approach is to use a reverse proxy (Nginx, Apache, `oauth2-proxy`) to handle authentication and authorization against Keycloak. The setup is the same as described for Prometheus.
  - Service Accounts: When Loki interacts with other services (e.g., object storage), it will typically use service accounts or API keys, not Keycloak user accounts.
##### Promtail
- **Features of Interest:**
  - Log Shipping: Promtail is an agent that ships logs to Loki. It's designed to be lightweight and efficient.
  - Discovery: Promtail can automatically discover log files on the host where it's running.  It supports various discovery mechanisms (static configuration, file globbing, systemd journal, Kubernetes API).
  - Labeling: Promtail adds labels to log streams.  These labels are crucial for querying logs in Loki.  Promtail can extract labels from the log file path, systemd journal fields, or using regular expressions to parse log lines.
  - Filtering and Transformation: Promtail can filter and transform log lines before sending them to Loki. You can drop irrelevant logs, add or modify labels, and even rewrite log messages.
  - Tail from Beginning/End: Promtail can be configured to read log files from the beginning (for initial ingestion) or from the end (for tailing new logs).
  - Multiple Output: promtail sends the logs to Loki.
- **Integration:**
  - Promtail *does not* interact directly with Keycloak. Promtail's job is to send logs to Loki.  Authentication to Loki is handled at the Loki level (via a reverse proxy, as described above).
  - Promtail *itself* might need credentials to access log files or the systemd journal, but these are local system credentials, not Keycloak user accounts.
  - Service Account (for Loki):** Promtail needs credentials to *send* logs to Loki.  This is typically configured using a basic authentication username and password (which you would store securely, e.g., in a Kubernetes secret or environment variables).  This is a *service account*, not a Keycloak user account. The reverse proxy in front of Loki will handle Keycloak authentication for *users* accessing Loki via Grafana. Promtail bypasses this user authentication.

#### Standardized Log Format
- Use a consistent log format (e.g., JSON) across all components to make it easier to analyze and correlate events.
- tba

### 11. The Authentication and Authorization layer
The Authentication and Authorization layer acts as a Centralized Identity Provider (IdP). This is *critical* for consistent access control across the *entire* system.  The other components should *not* manage users and passwords directly.
Okay, let's delve into Keycloak's features and relevant aspects for system engineers, expanding on the Authentication and Authorization layer.
#### KeyCloak
- **Core Identity and Access Management (IAM) Features**
  - Single Sign-On (SSO) and Single Sign-Out (SLO): This is *the* cornerstone feature. Users authenticate once with Keycloak and gain access to all applications integrated with it, without needing to re-login.  SLO ensures that logging out of Keycloak logs the user out of all connected applications. This improves user experience and strengthens security.
  - User Federation (LDAP/Active Directory Integration): If your system already has existing user directories (like Microsoft Active Directory or an LDAP server), Keycloak can *federate* with them.  This means users can use their existing credentials, and Keycloak synchronizes user information.  This is crucial for avoiding user duplication and simplifying migration.  Keycloak can act as a bridge between existing systems and newer OIDC/OAuth 2.0-based applications.
  - Social Login: Keycloak supports integration with social login providers (Google, Facebook, GitHub, etc.).  While less critical for a primarily internal system like OllaLab, it can be useful for external collaborators or specific use cases.
  - Identity Brokering: Keycloak can act as an intermediary between OllaLab and *other* identity providers.  This is useful if you need to integrate with external partners who have their own SSO systems.
  - User Management: Keycloak provides a web UI and a REST API for managing users: creating, deleting, updating profiles, resetting passwords, managing group memberships, and assigning roles.
  - Group Management: Organize users into groups, which simplifies role assignment and permission management.  Groups can be hierarchical.  This is more manageable than assigning roles to individual users directly, especially as the system grows.
  - Role-Based Access Control (RBAC):
    - Realm Roles: Roles that apply across the entire Keycloak realm (typically representing broad access levels, like `data_engineer`, `analyst`).
    - Client Roles: Roles that are specific to a particular client application (e.g., a role that grants permission to access a specific API endpoint in a FastAPI service). This allows for finer-grained control.
    - Composite Roles: Roles that are composed of other roles (both realm and client roles). This allows creating higher-level roles that combine multiple permissions.  For example, a `reporting_manager` role could be a composite of the `analyst` realm role and a `report_generator` client role.
  - Client Management: Keycloak uses the concept of "clients" to represent applications and services that integrate with it.  Each component in OllaLab (Streamlit, FastAPI, Grafana, etc.) that needs authentication/authorization will be registered as a client in Keycloak.
    - Client Types: Keycloak supports different client types:
      - OpenID Connect (OIDC) Clients: For web applications, single-page applications (SPAs), and mobile apps that use the OIDC protocol for authentication.
      - SAML Clients: For applications that use the SAML 2.0 protocol for SSO.  Less common in modern architectures, but still relevant for legacy systems.
      - Confidential Clients: Clients that can securely store a client secret. These are typically server-side applications (FastAPI, Node.js).
      - Public Clients: Clients that *cannot* securely store a client secret (e.g., SPAs running in a browser).
      - Bearer-Only Clients: Clients that only need to validate access tokens (e.g., backend services that are protected by an API gateway like Kong).
    - Client Scopes: Define the permissions that a client can request.  Scopes are used in OAuth 2.0 to limit the access granted to a client.  Keycloak allows defining custom scopes.  For example, a scope could be `read:data` or `write:models`.
    - Service Accounts: Clients can have associated service accounts, which are used for machine-to-machine communication (e.g., when Airflow needs to access a database).  Service accounts are granted roles, just like users.
  - Authentication Flows: Keycloak's authentication flows are highly customizable.  A flow defines the steps a user goes through during authentication (e.g., username/password, OTP, etc.).  You can create custom flows to implement specific authentication requirements.
  - Required Actions: Actions that a user *must* perform after their first login (e.g., update their profile, configure MFA).
  - Credential Management: Keycloak handles storage of user credentials (passwords, OTP secrets).  It uses secure hashing algorithms and supports various credential types.
- **Advanced Security Features**
  - Multi-Factor Authentication (MFA):
    - Time-Based One-Time Passwords (TOTP): Using authenticator apps like Google Authenticator or Authy.
    - Hardware Security Keys (FIDO2/WebAuthn): The most secure MFA method, using physical security keys.
    - OTP via SMS/Email: Less secure than TOTP or hardware keys, but still an option.
    - Conditional MFA: Enforce MFA based on certain conditions (e.g., user's IP address, device, or risk level).
  - Password Policies: Configure password complexity requirements (minimum length, required characters, etc.) and password history (preventing reuse of old passwords).
  - Brute-Force Protection: Keycloak has built-in protection against brute-force attacks.  It can temporarily lock out accounts after multiple failed login attempts.
  - Token Customization (Mappers): You can customize the contents of ID tokens and access tokens by adding custom claims.  This is done using "mappers."  For example, you could add a user's department or organization ID as a claim.  This allows propagating additional user information to applications.
  - Event Listener: Keycloak emits events for various actions (login, logout, user creation, etc.). You can configure event listeners to trigger actions based on these events (e.g., send a notification to an administrator when a new user is created, or log events to an external system).
  - Admin Console: A web-based interface for managing Keycloak (users, roles, clients, realms, etc.).  Access to the Admin Console should be *strictly* controlled.
  - REST API: Keycloak provides a comprehensive REST API for managing all aspects of the system. This is useful for automation and integration with other tools.
  - Themes: You can customize the look and feel of Keycloak's login pages, registration pages, etc., using themes.
  - Clustering and High Availability: Keycloak can be deployed in a clustered configuration for high availability and scalability.  This requires a shared database and a distributed cache.
- **Other Relevant System Engineering Aspects**
  - Realms: Realms are a core concept in Keycloak. A realm is a *security domain*.  It's like a separate Keycloak instance within a single Keycloak installation.  Each realm has its own users, roles, clients, groups, and configuration.
    - Master Realm: Keycloak has a special "master" realm that is used to manage the Keycloak installation itself.  *Do not* use the master realm for application-specific users or clients.
    - OllaLab Realm: You would create a dedicated realm for OllaLab (e.g., named "ollalab").  All OllaLab-related users, roles, clients, etc., would be defined within this realm.  This isolates OllaLab's security configuration from other potential applications.
    - Multi-tenancy: If you need to support multiple, isolated tenants within OllaLab (e.g., different departments or customers), you could consider using multiple realms (one per tenant) or a single realm with careful use of groups and client roles to achieve isolation.
  - Storage:
    - Database: Keycloak uses a relational database to store its configuration (users, roles, clients, etc.).  PostgreSQL is the recommended database.  Make sure the database is properly secured and backed up.
    - User Federation Storage: If you're using user federation (LDAP/AD), Keycloak will also interact with the external user directory.
    - Caching: Keycloak uses caching extensively to improve performance.  In a clustered environment, you'll need a distributed cache (like Infinispan, which is included with Keycloak).
  - Deployment:**
    - Docker (Recommended): The easiest way to deploy Keycloak is using Docker containers.  Official Keycloak Docker images are available.
    - Kubernetes (Recommended for Production): For production deployments, Kubernetes is highly recommended.  There are official Keycloak Helm charts available to simplify deployment and management.
    - Standalone (WildFly/JBoss EAP): Keycloak is built on top of WildFly (formerly JBoss EAP). You *can* deploy Keycloak as a standalone application server, but this is generally less convenient than using Docker or Kubernetes.
  - Networking:
    - HTTPS (TLS): Keycloak *must* be accessed over HTTPS (TLS).  This is essential for protecting user credentials and tokens.  Use a reverse proxy (Nginx, Apache) to handle TLS termination.
    - Firewall Rules: Restrict access to the Keycloak server to only authorized clients (your applications and the Admin Console).
  - Monitoring:
    - Metrics: Keycloak exposes metrics in the Prometheus format.  You can scrape these metrics with Prometheus and visualize them in Grafana.
    - Logging: Configure Keycloak's logging to capture relevant events.  Consider aggregating logs using Loki and Promtail.
    - Health Checks: Configure health checks to monitor the availability and responsiveness of Keycloak.
  - Backup and Restore:
    - Regularly back up the Keycloak database. This is essential for disaster recovery.
    - You can also export and import realm configurations using the Keycloak Admin Console or REST API.
- **Upgrading:** Keycloak releases new versions regularly. Plan for regular upgrades to benefit from bug fixes, security patches, and new features. Follow the Keycloak documentation for upgrade procedures.
- **Extending Keycloak:**
  - Service Provider Interfaces (SPIs): Keycloak is highly extensible through its SPIs. You can develop custom providers for:
    - Authentication: Implement custom authentication mechanisms.
    - User Federation: Connect to custom user stores.
    - Event Listeners: Trigger custom actions based on Keycloak events.
    - Protocol Mappers: Customize the contents of tokens.
    - And more...
  - Custom Themes: Customize the look and feel of Keycloak's UI.
  - Custom Authenticators: Create custom authentication flows.

## Networking
- **Network policies** can be used to restrict traffic flow *between* components, even within one's infrastructure. This is a "defense in depth" approach.
- **TLS/SSL:** Use TLS/SSL for *all* communication between components:  databases, APIs, web interfaces, etc.  This is non-negotiable.  Use Let's Encrypt (or a similar service) for free, valid certificates.
- **Virtual Private Cloud (VPC) / Virtual Networks (VNets):**  Isolate one's infrastructure within VPCs (AWS, GCP) or VNets (Azure).  This creates a private network for one's resources.  Use separate VPCs/VNets for development and production.
- **Subnets:** Divide one's VPC/VNet into subnets.  Common practice is to have:
  - Public Subnets:** For resources that need to be directly accessible from the internet (e.g., load balancers, NAT gateways).
  - Private Subnets:**  For most of one's resources (databases, application servers, data processing components). These should *not* be directly accessible from the internet.
  - VPN/Direct Connect:**  Establish a secure connection between one's on-premise network and one's cloud VPCs/VNets using a VPN or a dedicated connection (AWS Direct Connect, Azure ExpressRoute, Google Cloud Interconnect).
  - Network address ranges** A common practice is using a `/16` network on the VPC level and `/24` for subnets
- **Network Access Control Lists (NACLs) and Security Groups:**
  - NACLs (AWS):**  Stateless firewall rules that apply at the subnet level.
  - Security Groups (AWS/GCP/Azure):**  Stateful firewall rules that apply at the instance level.
  - Principle of Least Privilege:**  Only allow the *minimum* necessary traffic between components.  For example, one's PostgreSQL database should only be accessible from one's application servers and one's data processing components, not from the public internet.
- **Load Balancing:**
  - External Load Balancers:**  Distribute traffic from the internet to one's public-facing applications (e.g., one's Streamlit/SvelteKit apps).
  - Internal Load Balancers:** Distribute traffic between one's application servers or within one's data processing cluster.
- **NAT Gateway / Cloud NAT:**  Allow instances in one's private subnets to access the internet (e.g., for software updates or to access external APIs) without having public IP addresses.
- **DNS:** Use a consistent DNS strategy (e.g., Route 53 in AWS, Cloud DNS in GCP, Azure DNS) to manage domain names and resolve internal and external hostnames.
- **Container Networking (if applicable):**  If using containers (Docker, Kubernetes), configure networking within one's container orchestration system. This often involves overlay networks and service discovery mechanisms.
- **Kong as API Gateway:** Place Kong in a public subnet (or behind a load balancer in a public subnet).  Configure it to route traffic to one's backend services (which are in private subnets).
- **Local Development Environment:**
  - VPN Client:**  Use a VPN client on one's Macbook Pro to connect to one's development VPC/VNet. This allows access resources in one's private subnets as if on the same network.
  - SSH Tunneling:**  Use SSH tunneling to securely access services (e.g., databases) that are not directly exposed.
  - Local Development:**  For some components (e.g., Streamlit apps), one can develop and test locally without connecting to the cloud environment.

## Data Governance

### Data Ownership and Stewardship
Who is responsible for the quality and accuracy of different datasets?
Who ensures that data governance policies are followed?
### Data Quality Metrics
Define specific, measurable metrics for data quality (completeness, accuracy, consistency, timeliness, validity).  The Monitoring layer should track these.
### Data Classification
Categorize data based on sensitivity (public, internal, confidential, restricted).  This informs access control and security policies.  This is *crucial* for privacy.
### Data Retention Policy
How long should data be stored?  This is important for compliance (e.g., GDPR) and cost management.  This should be tied to Hudi's retention and cleanup capabilities.
### Data Lineage (Beyond Catalog)
While the catalog mentions lineage, you need a robust mechanism to *automatically* capture and visualize lineage.  Consider integrating with the processing layer (e.g., dbt, Spark) to capture lineage information and feed it into the metadata catalog (e.g., via Airflow operators).
### Privacy
- **At Rest (Data Lakehouse)**
  - Column-Level Encryption (with format-preserving encryption): For sensitive fields, one could encrypt them using format-preserving encryption (FPE).  This allows data encryption while maintaining its original format (e.g., encrypting a credit card number so it still *looks* like a credit card number, but is unusable without the decryption key).
- **Dynamic Data Masking (At Query Time):**
  - Database Views: Create views in PostgreSQL (or other databases) that mask sensitive columns.  Grant users access to the views, not the underlying tables.
  - Row-Level Security (RLS) in PostgreSQL: Use RLS to restrict access to rows based on user attributes.  This can be combined with masking to provide fine-grained control.
  - Proxy/Gateway: One could potentially use a proxy (like Kong) to intercept queries and apply masking rules, but this is complex and can impact performance.
- **Anonymization Techniques:**
  - Pseudonymization: Replace identifying information with pseudonyms (e.g., replace names with unique IDs).  Maintain a mapping table (securely!) to link pseudonyms back to the original identifiers if needed.
  - Generalization: Replace specific values with broader categories (e.g., replace ages with age ranges).
  - Aggregation: Report aggregate statistics instead of individual-level data.
  - Suppression: Remove sensitive fields entirely.
  - Perturbation: Add random noise to data to obscure individual values while preserving overall statistical properties (e.g., differential privacy).

## Backup and Monitoring
### Databases
#### PostgreSQL
1.  **Types of Backups:**
  - Full Backups:  A complete copy of the entire database.  These are the foundation of any backup strategy.
  - Incremental Backups: Back up only the changes since the last full or incremental backup.  These are faster and smaller than full backups.
  - WAL (Write-Ahead Log) Archiving:**  Continuously archive the WAL, which records all database changes.  This enables point-in-time recovery (PITR).

2.  **Backup Tools:**
  - pg_dump/pg_restore: PostgreSQL's built-in tools for logical backups.  `pg_dump` creates a SQL script that can be used to recreate the database. `pg_restore` restores from a `pg_dump` archive.
  - pg_basebackup: Creates a physical backup of the database files.  Faster than `pg_dump` for large databases.  Used for creating replicas and for PITR.
  - WAL-G / WAL-E: Tools specifically designed for WAL archiving and backup to cloud storage (S3, GCS, Azure Blob Storage).  Highly recommended for production deployments.
  - Barman:**  An open-source backup and recovery manager for PostgreSQL.  Provides features like remote backups, retention policies, and incremental backups.

3.  **Backup Schedule:**
  - Full Backups: At least daily.  More frequent backups (e.g., every 4-6 hours) may be needed for critical databases.
  - Incremental Backups: Hourly or more frequently, depending on the rate of data changes.
  - WAL Archiving: Continuous.

4.  **Backup Storage:**
  - Separate Location: *Never* store backups on the same server as the database.  Use a separate storage location (e.g., cloud object storage, a different server).
  - Redundancy: Store backups in multiple locations (e.g., multiple cloud regions) for disaster recovery.
  - Encryption: Encrypt backups at rest and in transit.

5.  **Recovery Procedures:**
  - Documented Procedures: Have clearly documented, step-by-step procedures for restoring from backups.
  - Regular Testing: *Test* your backup and recovery procedures regularly. This is *essential* to ensure they work correctly. Simulate failures and practice restoring the database.
  - Recovery Time Objective (RTO): How long can you afford to be down?
  - Recovery Point Objective (RPO): How much data loss is acceptable?

6.  **Monitoring Tools:**
  - Prometheus: Use the `postgres_exporter` to collect a wide range of PostgreSQL metrics (queries per second, connection counts, lock waits, replication status, etc.).
  - Grafana: Create dashboards to visualize PostgreSQL metrics from Prometheus.  Use pre-built dashboards (many are available online) or create your own.
  - pg_stat_statements: A PostgreSQL extension that tracks statistics about SQL query execution.  Essential for identifying slow queries.
  - pg_top: A command-line utility similar to `top`, but for PostgreSQL.
  - auto_explain: A PostgreSQL extension to help understand slow query execution.

7.  **Key Metrics to Monitor:**
  - Connections: Number of active connections, idle connections, and waiting connections.
  - Queries: Queries per second, query execution times (average, 95th percentile, 99th percentile).
  - Locks: Number of locks, lock wait times.
  - Replication: Replication lag (for read replicas).
  - CPU Utilization
  - Memory Usage
  - Disk I/O: Disk read/write operations per second, disk latency.
  - Disk Space: Free disk space.
  - Checkpoint Statistics: How often checkpoints occur, how long they take.
  - Error Logs: Monitor PostgreSQL's error logs for any errors or warnings.

8.  **Alerting:**
  - Prometheus Alertmanager: Configure alerting rules in Prometheus to trigger alerts based on metric thresholds.  For example:
    - High CPU utilization.
    - Low disk space.
    - High query latency.
    - Replication lag exceeding a threshold.
    - Errors in the PostgreSQL logs.
  - Grafana Alerts: You can also configure alerts directly in Grafana, based on the data from Prometheus or other data sources.

### MongoDB (Community Edition)
1.  **Backup Tools:**
  - mongodump/mongorestore: MongoDB's built-in tools for logical backups. `mongodump` creates a BSON dump of the database, and `mongorestore` restores from the dump.
  - MongoDB Cloud Manager (or Ops Manager): These are *not* available for the Community Edition.
  - Filesystem Snapshots: If you're running MongoDB on a cloud provider (e.g., AWS EBS, Azure Managed Disks, Google Persistent Disks), you can use filesystem snapshots to create backups. This is a fast and efficient way to create backups, but it requires careful coordination to ensure consistency.

2.  **Backup Strategy:**
  - Regular Full Backups: Use `mongodump` to create regular full backups of your database. The frequency depends on your RPO.
  - Oplog Backups (for Point-in-Time Recovery): The oplog (operation log) is a capped collection that records all write operations. To enable point-in-time recovery, you need to back up the oplog *in addition to* full backups.  You can use `mongodump` to back up the oplog. This is *critical* for production.
  - Sharded Clusters: In a sharded cluster, you need to back up each shard *and* the config servers.  It is strongly recommended to use filesystem snapshots of config servers and the oplog.
  - Separate Backup Storage: Store backups in a separate location (e.g., cloud storage) from the database servers.

3.  **Recovery Procedures:**
  - Documented: Have clear, step-by-step procedures for restoring from backups.
  - Testing: Regularly test your backup and recovery procedures.

4.  **Monitoring Tools:**
  - Prometheus: Use the `mongodb_exporter` to collect a wide range of MongoDB metrics.
  - Grafana: Create dashboards to visualize MongoDB metrics.
  - MongoDB's Built-in Tools:
    - `mongostat`:  A command-line tool that provides real-time statistics about a running MongoDB instance.
    - `mongotop`:  Shows the read and write activity for each collection.
    - Database Profiler:  Can be enabled to log slow queries.

5.  **Key Metrics:**
  - Connections: Number of active connections.
  - Operations: Operations per second (inserts, queries, updates, deletes).
  - Replication: Replication lag, oplog size.
  - Memory Usage: Resident memory, virtual memory, mapped memory.
  - Storage: Data size, index size, storage size.
  - Locks: Lock percentages and wait times.
  - Cursors: Number of open cursors.
  - Network: Bytes in/out.
  - Errors: Monitor MongoDB's logs for errors.

6.  **Alerting:**  Configure alerts in Prometheus or Grafana for:
  - High connection counts.
  - Slow queries.
  - Replication lag.
  - Low disk space.
  - Errors in the logs.

#### Neo4j (Community Edition)

1.  **Backup Tools:**
  - neo4j-admin backup (Community Edition): Neo4j provides the `neo4j-admin backup` command for creating consistent backups of the database.  This is an *offline* backup, meaning the database must be stopped during the backup process.
  - Incremental backups are not supported in the Community Edition
  - Online Backup (Enterprise Edition Only): Neo4j Enterprise Edition supports online backups, which can be performed while the database is running.

2.  **Backup Strategy:**
  - Regular Full Backups: Use `neo4j-admin backup` to create regular full backups.  The frequency depends on your RPO.
  - Separate Backup Storage: Store backups in a separate location.

3.  **Recovery Procedures:**
  - Documented: Have documented procedures for restoring from backups.
  - Testing: Regularly test your backup and recovery procedures.

4.  **Monitoring Tools:**
  - Prometheus: Use the `neo4j-prometheus-exporter` to collect Neo4j metrics.  (Note: You might need to find a community-maintained exporter, as official support may vary.)
  - Grafana: Create dashboards to visualize Neo4j metrics.
  - Neo4j Browser: Neo4j Browser provides some built-in monitoring capabilities (e.g., viewing query execution plans).

## Security
- **Centralized Identity Provider (IdP)**
  - Keycloak: Open-source, highly configurable, supports OIDC, OAuth 2.0, SAML, LDAP, and more.  Can be self-hosted (Docker container) in both development and production.  Excellent choice for its flexibility and features.
  - Single Sign-On (SSO): Users log in once and can access multiple services without re-authenticating.
  - Centralized User Management: Add, remove, and modify users in one place.
  - Consistent Security Policies: Enforce password policies, multi-factor authentication (MFA), etc., across your entire system.
  - Auditing: Track user logins and activity in a central location.
  - Scalability: IdPs are designed to handle large numbers of users and applications.
- **OpenID Connect (OIDC) and OAuth 2.0** These are industry-standard protocols for authentication and authorization.  OIDC builds on top of OAuth 2.0 to add identity information (who the user is). All system components will use OIDC/OAuth 2.0 to interact with the IdP.
- **Role-Based Access Control (RBAC)**
  - RBAC control *what* users can do after they've authenticated.  RBAC is a standard approach where roles (e.g., "admin," "data_engineer," "analyst," "viewer") are defined and assign permissions to those roles. Users inherit permissions based on their assigned roles.
  - Define Roles in IdP: Most IdPs (Keycloak) allow defining roles and assigning users to them.
  - Permissions Mapping: one needs to map roles to specific permissions *within* applications and services. For example:
    - `admin`: Full access to all resources.
    - `data_engineer`: Access to create/modify data pipelines, access to all databases.
    - `analyst`: Read-only access to certain databases and BI tools.
    - `viewer`: Read-only access to pre-defined dashboards.
  - Claims:  When a user authenticates, the IdP issues an access token (JWT - JSON Web Token) that contains *claims*.  These claims include the user's roles (and potentially other attributes).  The system components/applications will inspect these claims to determine the user's permissions.
- **Disk Encryption:**  Encrypt the underlying disks/volumes of one's servers (both on-premise and in the cloud).  This provides an additional layer of protection.
- **Mutual TLS (mTLS):** For highly sensitive communication (especially between services), consider using mTLS, where both the client and server present certificates to verify their identities.
- **Disaster Recovery**
tba
- **Security Audits and Penetration Testing:**
  - Schedule regular security audits and penetration tests to identify and address vulnerabilities.

Okay, let's expand the Security section with the requested information about Secrets Management (HashiCorp Vault) and Vulnerability Scanning.

### Secrets Management (HashiCorp Vault)
HashiCorp Vault is a dedicated secrets management solution designed to address the critical problem of securely storing and managing sensitive information.  It offers significant advantages over ad-hoc approaches:
  - Centralized Management:  A single source of truth for all secrets, eliminating sprawl and inconsistency.
  - Encryption at Rest and in Transit: Secrets are encrypted both when stored and when accessed.
  - Dynamic Secrets: Vault can generate short-lived, dynamically-created credentials (e.g., for databases), reducing the risk of long-term credential exposure.
  - Auditing: Vault provides detailed audit logs, tracking all access to secrets.
  - Access Control: Fine-grained access control policies (using Vault's policy language) restrict who can access which secrets.
  - Integration: Vault integrates seamlessly with many systems, including Keycloak, Kubernetes, cloud providers, and databases.
- **Keycloak Integration:** Vault can leverage Keycloak for authentication and authorization. This allows you to manage Vault access using your existing Keycloak users and groups, and ensures a single source of truth for identity.
  - Enable the OIDC Auth Method in Vault: Vault has a built-in OIDC authentication method.  You'll configure it with your Keycloak's discovery URL, client ID, and client secret.
  - Configure Vault Policies: Vault's policies control what actions users and applications can perform.  You'll map Keycloak roles (from the JWT claims) to Vault policies.
  - Role Mapping: In Vault's OIDC configuration, you'll map Keycloak roles to these Vault policies.  This ensures that when a user authenticates with Keycloak, Vault grants them the appropriate permissions based on their Keycloak roles.
- **Integration with Other Components:**
  - Databases (PostgreSQL, MongoDB, Neo4j): Vault can act as a *secrets engine* for databases.  Instead of storing database credentials directly in application configurations, applications can request dynamic credentials from Vault. Vault will generate short-lived credentials for the database, and the application will use those.  This greatly reduces the risk of credential leakage.
  - Airflow: Airflow can use Vault to store connections and variables securely. The `HashiCorp Vault Provider` is the recommended way to integrate Airflow and Vault.
  - NiFi: NiFi can use the HashiCorp Vault client APIs or custom processors to retrieve secrets from Vault.
  - Pulsar: Pulsar functions can be configured to retrieve secrets from Vault using the Vault Java client library (or similar for Python/Go).
  - Application Layer (FastAPI, Streamlit, SvelteKit):** Applications should use Vault client libraries (e.g., `hvac` for Python) to retrieve secrets.  They *never* store secrets directly in code or configuration files.
  - MLflow/Ollama: These tools would access any required secrets via Vault, likely by having the surrounding application retrieve the secrets and pass them in via environment variables or API calls.

### Vulnerability Scanning
Vulnerability scanning is the automated process of identifying security weaknesses in your infrastructure and applications.  Scanners use databases of known vulnerabilities (e.g., CVEs - Common Vulnerabilities and Exposures) to check your systems for potential problems.
1.  **Regular Network Scans:** Schedule regular network scans of your infrastructure (e.g., weekly or monthly).
2.  **Web Application Scans:**  Perform regular web application scans, especially after major code changes.
3.  **Alerting and Reporting:** Configure your scanning tools to send alerts when vulnerabilities are found.  Generate reports to track the overall security posture of your systems. Integrate with your monitoring layer (e.g., send alerts to Prometheus/Alertmanager, visualize scan results in Grafana).
4.  **Remediation:**  Establish a process for addressing vulnerabilities.  Prioritize based on severity and potential impact.
5. **Secrets Scanning:** Scan code repositories, configuration files, and environment variables for accidentally committed secrets (passwords, API keys, etc.). Tools like git-secrets, truffleHog, and Gitleaks can help. Integrate this into the CI/CD pipeline.

## Scalability
- **Complexity:**  This is a complex system with many moving parts. Consider starting with a smaller, more focused implementation and gradually adding components as needed. This will make it easier to manage and debug.
- **Scalability:**
  - Horizontal Scalability:  Most of chosen components (Pulsar, NiFi, Airflow, MinIO, databases, etc.) can be scaled horizontally by adding more instances.
  - Vertical Scalability: For some components (e.g., single-node databases), one might need to increase the resources (CPU, RAM, storage) of the instance.
- **Development/Testing/Staging/Production Environments:**
  - It's *highly recommended* to have at least one staging environment (a replica of production) for testing changes before deploying them to production.  This minimizes the risk of introducing bugs or downtime.  IAC (Terraform) is *critical* for maintaining consistency across environments.
- **High Availability (HA):**
  - Redundancy:  Deploy multiple instances of critical components (e.g., Pulsar brokers, NiFi nodes, Airflow workers, database replicas) to ensure that the system can continue operating even if one instance fails.
  - Failover: Configure automatic failover mechanisms (e.g., using ZooKeeper or a similar coordination service) to switch to a backup instance if the primary instance fails.
  - Databases: Use database replication (e.g., PostgreSQL streaming replication, MongoDB replica sets) to provide HA and read scalability.
- **Load balancing:** use load balancing for services
- **Deployment and Automation:**
  - Infrastructure as Code (IaC): Use tools like Terraform or CloudFormation to define infrastructure as code. This allows automation of the provisioning and management of resources.
  - Configuration Management: Use tools like Ansible, Chef, or Puppet to automate the configuration of servers and applications.

## Cost Optimization
While scalability is addressed, cost optimization should be a first-class consideration, especially in hybrid and multi-cloud environments.  This includes:
    - Resource Sizing:**  Right-size instances and clusters to avoid overspending.  Use monitoring to inform this.
    - Storage Tiering:**  Use different storage tiers (e.g., MinIO's warm/cold tiers, or cloud provider equivalents) based on access frequency.
    - Auto-Scaling:**  Leverage auto-scaling where possible (e.g., for NiFi, Airflow workers, databases).
    - Spot Instances (with caution):**  For non-critical, fault-tolerant workloads (e.g., some batch processing), consider using spot instances (or preemptible VMs) in the cloud to save costs.
    - Data Transfer Costs:**  Be mindful of data transfer costs, especially between regions or clouds.

- Disaster Recovery (DR):**  The document focuses on HA (within a single environment).  You *must* have a DR plan to handle a complete site outage (e.g., a data center failure).  This might involve:
    - Replication to a different region:**  Replicate data and critical infrastructure to a different geographical region.
    - Regular backups:**  Perform regular backups of all critical data (databases, Hudi data, metadata catalog) and store them in a separate location.
    - Automated failover to the DR site:**  Plan how you would switch to the DR site in case of a disaster.