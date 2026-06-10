# Azure Cosmos DB Setup: Learning Program

## Overview
This repository contains the configuration choices, deployment evidence, and verification details for the **Azure Cosmos DB Setup Learning Program**. 

The goal of this project is to explore and implement cloud-native NoSQL databases, focusing on global distribution, horizontal partitioning (sharding), throughput management, security, and real-time monitoring.

---

## Technical Configuration Choices

| Parameter | Configuration Choice | Technical Justification |
| :--- | :--- | :--- |
| **API Type** | Azure Cosmos DB for NoSQL | Selected for native JSON document handling, flexible schema capability, and support for rich SQL-like query syntax. This allows efficient, low-latency querying on nested structures and automatic indexing of all properties. |
| **Primary Region** | `UK South` | The initial write and read region, chosen based on proximity to optimize latency for local workloads. |
| **Secondary Region** | `South Africa North` | Added to demonstrate global replication, high availability, and disaster recovery. |
| **Multi-region Writes** | Enabled | Configured to allow both regions to accept writes. This reduces write latency for globally distributed clients and ensures write availability in the event of a regional outage. |
| **Consistency Level** | Session Consistency | Selected as the optimal balance for user-centric applications (the default in Cosmos DB). It guarantees monotonic reads, monotonic writes, and read-your-writes guarantees *within* a client session, without the high latency and availability trade-offs of Strong consistency. |
| **Database Name** | `cosmosdb-lab` | Logical grouping for container workloads. |
| **Container Name** | `customer` | Used for storing customer-profile JSON documents. |
| **Partition Key** | `/customerId` | Selected due to high cardinality, uniform distribution of customer actions, and alignment with primary lookup query patterns (e.g., retrieving records by ID). This prevents "hot" physical partitions. |
| **Throughput Mode** | Autoscale (Max 10,000 RU/s) | Provisions scale-on-demand capabilities between 1,000 RU/s (10% of max) and 10,000 RU/s, protecting against traffic spikes while reducing costs during idle periods. |

---

## CAP Theorem & Consistency Trade-offs

In a distributed database system, the **CAP Theorem** dictates that a system can guarantee at most two out of three properties: **Consistency**, **Availability**, and **Partition Tolerance**. 

Azure Cosmos DB allows fine-grained consistency configuration. By using **Session Consistency** with **Multi-region Writes**, we navigate these trade-offs as follows:
* **Availability**: High. If a region fails, another region can accept write operations immediately.
* **Latency**: Low. Both read and write operations are performed locally in the closest region (`UK South` or `South Africa North`).
* **Consistency**: Guaranteed consistency within client sessions. Outside the active session, replication is asynchronous, leading to eventual consistency for other regions.

---

## Task-by-Task Implementation Walkthrough

### Task 1: Account Creation
An Azure Cosmos DB account named `cosmosdb-olamileye` was created under the resource group `cosmosdb-rg` using the **NoSQL API** and deployed to `UK South`.

* **Deployment Completion Confirmation**:
  ![Deployment Complete](screenshots/01_deployment_complete.png)

* **Account Overview and Active Status**:
  ![Cosmos DB Account Overview](screenshots/02_cosmos_db_overview.png)

---

### Task 2: Global Distribution & Replication
Replication was configured to copy data globally from `UK South` to `South Africa North`. To maximize availability and performance, **Multi-region writes** were enabled.

* **Replication Map and Failover Settings**:
  ![Global Replication](screenshots/03_global_distribution.png)

---

### Task 3: Database & Container Setup
Within the `cosmosdb-olamileye` account, a database named `cosmosdb-lab` and a container named `customer` were initialized. The partition key was set to `/customerId` to enable efficient horizontal partitioning.

* **Database and Container Details (shown in Data Explorer)**:
  ![Data Explorer View](screenshots/04_data_explorer_crud_create.png)

---

### Task 4: Data Operations (CRUD)
CRUD operations were performed directly within the Azure Portal's Data Explorer:
1. **Create/Read**: Inserted two JSON documents representing customer records for Segun Lawal (`customerId: C001`) and Elba Idris (`customerId: C002`).
2. **Delete**: Deleted the record for Elba Idris to demonstrate delete capabilities.

* **Inserted Customer Documents (Before Delete)**:
  ```json
  {
    "id": "2",
    "customerId": "C002",
    "name": "Elba Idris",
    "phone_number": "081141967419",
    "country": "Congo"
  }
  ```
  ![Inserted Documents](screenshots/04_data_explorer_crud_create.png)

* **Deletion Confirmation**:
  ![Successfully Deleted Document](screenshots/05_data_explorer_crud_delete.png)

---

### Task 5: Throughput Management
The throughput mode for the `customer` container was configured to **Autoscale** with a Maximum Request Units limit of `10,000 RU/s`. This permits Azure to dynamically scale resource allocations down to `1,000 RU/s` when traffic is light, optimizing cost-efficiency.

* **Scale & Settings Pane**:
  ![Autoscale Configuration](screenshots/06_throughput_scale_settings.png)

---

### Task 6: Security and Networking
To secure the database, access keys were inspected. Access control is enforced using Master Keys (Primary/Secondary Read-Write and Read-Only keys) to restrict unauthorized actions.

* **Access Keys Pane**:
  ![Access Keys](screenshots/07_security_keys.png)

---

### Task 7: Monitoring & Metrics
Azure Metrics were utilized to track the performance and health of the database:
1. **Request Volume**: Tracked the total number of requests (17 requests during setup operations).
2. **Latency**: Verified that the average server-side latency for Gateway requests was `6.65 ms`, ensuring low-latency operations.
3. **Custom Dashboard**: Pinned the request count and average latency metrics to a private monitoring dashboard for easy observability.

* **Total Request Count Chart**:
  ![Total Requests Chart](screenshots/08_monitoring_total_requests.png)

* **Server-Side Latency Chart**:
  ![Average Latency Chart](screenshots/09_monitoring_average_latency.png)

* **Private Monitoring Dashboard**:
  ![Private Dashboard](screenshots/10_monitoring_dashboard.png)
