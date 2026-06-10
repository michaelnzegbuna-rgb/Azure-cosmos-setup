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




# Azure Cosmos DB Setup — Step-by-Step Guide

This document walks through the complete setup and configuration of an Azure Cosmos DB account as demonstrated in the screenshots, from initial deployment to monitoring.

---

## Screenshot 1 — Deployment Complete

**File:** `01_deployment_complete.jpg`

The Azure portal confirms a successful Cosmos DB deployment. Key details visible:

- **Deployment name:** `Microsoft.Azure.CosmosDB-20260606213014`
- **Start time:** June 6, 2026 at 21:30:16
- **Subscription:** Azure subscription 1
- **Resource group:** `cosmosdb-rg`
- **Correlation ID:** `f29bbe41-9e84-48b6-a5d3-76a3c...`

The green checkmark and "Your deployment is complete" message confirm the account was provisioned successfully. From here, the **"Go to resource"** button navigates directly into the new Cosmos DB account. The right panel also highlights optional next steps: setting up Cost Management alerts and enabling Microsoft Defender for Cloud.

---

## Screenshot 2 — Cosmos DB Account Overview

**File:** `02_cosmos_db_overview.jpg`

After navigating to the resource, the Cosmos DB account overview page (`cosmosdb-nzemikez`) is displayed. Important properties shown:

| Property | Value |
|---|---|
| Status | Online |
| Resource group | cosmosdb-rg |
| Subscription | Azure subscription 1 |
| Read Locations | UK South |
| Write Locations | UK South |
| Total throughput limit | 1000 RU/s |
| Free Tier Discount | Opted In |
| Capacity mode | Provisioned throughput |

The left sidebar reveals the full set of management options available: **Data Explorer**, **Mirroring in Fabric**, **Container Copy**, **Resource Visualizer**, **Settings**, **Integrations**, **AI Capabilities (Preview)**, **Containers**, **Monitoring**, and more. The **Setup & Optimization** tab in the main pane provides guided links for getting started, loading data, and going production-ready.

> **Note:** A banner recommends upgrading the Azure Cosmos DB Java SDK to version 4.48.2 or later for optimal performance and stability.

---

## Screenshot 3 — Global Distribution / Replicate Data Globally

**File:** `03_global_distribution.jpg`

This screen shows the **"Replicate data globally"** configuration under Settings. The account is configured with **multi-region writes enabled**, and two replica regions are active:

| Region | Write Region | Availability Zone | Status |
|---|---|---|---|
| UK South | ✓ | — | Online |
| South Africa North | ✓ | — | Online |

Both regions are set as write regions, meaning the database supports active-active multi-region writes. This setup ensures low-latency access for users in both the UK and Africa, and provides resilience against regional outages.

From this screen you can also **Manage regions**, **Configure failover policy**, **Change write region**, or take a region **Offline** using the top action bar.

---

## Screenshot 4 — Monitor Metrics: Total Requests

**File:** `8_monitoring_total_request.png`

The **Azure Monitor → Metrics** blade shows a line chart for **Total Requests (Count)** scoped to the `cosmosdb-nzemikez` account over the last 24 hours (auto-refreshing every 5 minutes).

- The chart is mostly flat (near zero activity) for most of the day.
- A sharp spike appears toward the end of the 24-hour window, reaching a peak of approximately **7–8 requests**.
- The legend at the bottom confirms a total count of **17 requests** for the period.

This view is useful for understanding traffic patterns, detecting unexpected spikes, or verifying that your application is successfully connecting and sending requests to the database.

---

## Screenshot 5 — Monitoring Dashboard (My Dashboard)

**File:** `10_monitoring_dashboard.png`

This is a custom **Azure Dashboard** ("My Dashboard") that has been configured to display multiple Cosmos DB metrics in one place. Two pinned charts are visible:

1. **Server Side Latency Gateway (Avg)** — Shows average gateway latency; the legend value reads **6.65ms**, indicating consistently low latency for the account.
2. **Count Total Requests for cosmosdb-nzemikez** — Mirrors the Metrics chart from Screenshot 4, showing the same spike pattern with a total of **17 requests**.

Dashboards like this are ideal for ongoing operational visibility. Charts can be added from any Metrics view using the **"Save to dashboard"** option.

---

## Screenshot 6 — Data Explorer: CRUD — Create / Read

**File:** `04_data_explorer_crud_create.jpg`

The **Data Explorer** shows the `cosmosdb-lab` database with a `customer` container. Two documents have been created:

| # | ID | Customer ID | Name | Phone | Country |
|---|---|---|---|---|---|
| 1 | — | C001 | Segun Lawal | 0801419674... | Nigeria |
| 2 | — | C002 | Elba Idris | 0811419674... | Congo |

The right pane displays the raw JSON for the selected document (C002 — Elba Idris), including system-generated fields such as `_rid`, `_self`, `_etag`, `_attachments`, and `_ts` (timestamp).

This demonstrates successful **Create** and **Read** operations via the Data Explorer. The SQL-like query bar at the top (`SELECT * FROM c`) allows filtering documents by any field.

---

## Screenshot 7 — Monitor Metrics: Average Latency

**File:** `09_monitoring_average_latency.jpg`

Back in **Azure Monitor → Metrics**, this chart tracks the **Server Side Latency Gateway (Avg)** for `cosmosdb-nzemikez` over the last 24 hours. The metric configuration panel shows:

- **Scope:** cosmosdb-nzemikez
- **Metric Namespace:** Cosmos DB standard metrics
- **Metric:** Server Side Latency Gateway
- **Aggregation:** Avg

The chart shows a brief spike peaking at just over **15ms**, returning quickly to the baseline of approximately **5ms** (shown as a dotted reference line). The legend confirms an average of **6.65ms** — well within acceptable bounds for a globally distributed NoSQL database.

---

## Screenshot 8 — Keys and Connection Strings

**File:** `7_Security_Key.png`

The **Keys** blade under Settings exposes the credentials needed to connect applications to the Cosmos DB account. The following are provided:

- **URI:** `https://cosmosdb-olamileye.documents.azure.com:443/`
- **PRIMARY KEY** *(masked)*
- **SECONDARY KEY** *(masked)*
- **PRIMARY CONNECTION STRING** *(masked)*
- **SECONDARY CONNECTION STRING** *(masked)*

Both keys were last regenerated on **06/06/2026**. The eye icon next to each field reveals the value, and the copy icon allows direct copying.

> ⚠️ **Security Note:** Never commit these keys to source control. Store them in Azure Key Vault or use environment variables / managed identity for application access.

---

## Screenshot 9 — Data Explorer: CRUD — Delete

**File:** `5_data_explorer_crud_delete.png`

This screenshot captures the result of a **Delete** operation in the Data Explorer. The status bar at the bottom of the screen confirms:

> ✅ **Successfully deleted 1 document(s)**

After the deletion, only one document remains in the `customer` container:

| # | ID | Customer ID | Name | Phone | Country |
|---|---|---|---|---|---|
| 1 | — | C001 | Segun Lawal | 0801419674... | Nigeria |

The document for C002 (Elba Idris) has been removed. This demonstrates a successful **Delete** operation, completing the full CRUD cycle in the Data Explorer.

---

## Screenshot 10 — Throughput Scale Settings (Autoscale)

**File:** `06_throughput_scale_settings.png`

The **Scale & Settings** tab inside the `customer` container shows the throughput configuration. The account is being configured for **Autoscale** mode (as opposed to Manual):

- **Minimum RU/s:** Determined automatically (shown as NaN pending input)
- **Maximum RU/s:** Configurable; the slider ranges from **400** to **1,000,000 RU/s**
- **Scale range:** Autoscale adjusts between 10% of max and the configured maximum — scaling is **Instant** at the low end and **4–6 hours** at the high end
- **Storage capacity:** Unlimited
- **Regions:** 2 (matching the global replication setup)
- **Price per RU:** $0.00016/RU

> Autoscale is recommended for workloads with unpredictable or variable traffic — the system scales down automatically during idle periods, reducing cost, and scales up on demand to handle traffic spikes without manual intervention.

---

## Summary

The screenshots document a complete Azure Cosmos DB setup workflow:

1. **Deploy** the Cosmos DB account via the Azure Portal
2. **Review** the account overview (region, throughput, capacity mode)
3. **Configure global distribution** with multi-region writes (UK South + South Africa North)
4. **Retrieve connection keys** for application integration
5. **Perform CRUD operations** (Create, Read, Delete) via Data Explorer
6. **Configure autoscale throughput** for cost-efficient scaling
7. **Monitor** the account using Azure Monitor Metrics and custom Dashboards

This setup reflects a production-ready configuration with geo-redundancy, autoscaling, and observability in place.


* **Server-Side Latency Chart**:
  ![Average Latency Chart](screenshots/09_monitoring_average_latency.png)

* **Private Monitoring Dashboard**:
  ![Private Dashboard](screenshots/10_monitoring_dashboard.png)
