
# Enable and Use Microsoft Fabric

---

## 1. Enabling Microsoft Fabric

Before accessing Microsoft Fabric's end-to-end capabilities, it must be enabled at the organizational level[cite: 1].

### Admin Roles Required to Enable Fabric
Fabric activation requires one of the following administrator roles[cite: 1]:
* **Fabric Administrator:** Manages overall Fabric platform settings and configurations[cite: 1].
* **Power Platform Administrator:** Oversees Power Platform services, including Fabric integration[cite: 1].
* **Global Administrator:** Possesses implicit organization-wide administrative permissions[cite: 1].

### Activation Process
Administrators enable Fabric via **Admin portal > Tenant settings** in the Power BI service[cite: 1]. It can be enabled for:
* The entire organization[cite: 1].
* Specific Microsoft 365 or Microsoft Entra security groups[cite: 1].
* Delegated capacity levels[cite: 1].

> **Note:** Organizations not currently using Fabric or Power BI can sign up for a free Fabric trial[cite: 1].

---

## 2. Workspace Management & Governance

Workspaces serve as collaborative environments where users create and manage assets like lakehouses, warehouses, and reports[cite: 1]. All workspace assets store data in OneLake and support native data lineage views[cite: 1].

### Workspace Settings Configuration
* License type assignment[cite: 1]
* OneDrive access settings[cite: 1]
* Azure Data Lake Storage Gen2 (ADLS Gen2) connection setups[cite: 1]
* Git integration for continuous integration/version control[cite: 1]
* Spark workload settings for performance optimization[cite: 1]

### Workspace Roles & Item Permissions

| Workspace Role | Typical Scope & Purpose |
| :--- | :--- |
| **Admin** | Full administrative rights over workspace settings, permissions, and assets[cite: 1]. |
| **Member** | Collaboration rights to manage workspace items and add users[cite: 1]. |
| **Contributor** | Can create, edit, and delete workspace content/items[cite: 1]. |
| **Viewer** | Read-only access to view and interact with workspace content[cite: 1]. |

> **Best Practice:** Use workspace roles for general collaboration[cite: 1]. For granular access control, apply item-level permissions on specific lakehouses or reports[cite: 1].

---

## 3. Data Discovery via OneLake Catalog

The **OneLake Catalog** allows users to discover and connect to governed data assets shared across the organization[cite: 1]. 

* **Scoping:** Results can be narrowed by specific workspaces or organizational domains[cite: 1].
* **Filtering:** Search datasets by keywords, default categories, or specific item types[cite: 1].
* **Security:** Users only see items explicitly shared with them[cite: 1].

---

## 4. Microsoft Fabric Workloads & Items

Fabric integrates capabilities from Azure Data Factory, Azure Synapse Analytics, and Power BI into a unified data mesh architecture with decentralized data ownership[cite: 1]:

* **Data Engineering:** Lakehouses, Spark jobs, and pipelines to curate data estates[cite: 1].
* **Data Factory:** Data ingestion, dataflows, and multi-system orchestration[cite: 1].
* **Data Warehouse:** Enterprise SQL analytical warehousing[cite: 1].
* **Real-Time Intelligence:** Streaming data processing, monitoring, and analytics[cite: 1].
* **Data Science:** ML model training, trend detection, and predictive analytics[cite: 1].
* **Databases:** Management and querying of relational databases[cite: 1].
* **Fabric IQ (preview):** Semantic modeling and ontology creation to unify OneLake data[cite: 1].
* **Power BI:** Report generation, dashboards, and self-service BI[cite: 1].
* **Industry Solutions:** Out-of-the-box domain-specific data solutions[cite: 1].

---

## 5. Enterprise AI & Copilot Integration

Fabric provides extensive AI infrastructure for built-in productivity and custom AI development[cite: 1].

### Fabric IQ & Microsoft IQ Suite (Preview)
**Fabric IQ** organizes OneLake data using business ontologies—defining concepts, relationships, and rules so AI agents can reason over domain context rather than raw schema tables[cite: 1].

| IQ Workload | Scope & Function |
| :--- | :--- |
| **Fabric IQ** | Models business data (ontologies, graphs, semantic models) across OneLake & Power BI[cite: 1]. |
| **Foundry IQ** | Connects structured/unstructured data across Azure, SharePoint, OneLake, and web sources[cite: 1]. |
| **Work IQ** | Captures collaboration signals (documents, chats, meetings, workflows)[cite: 1]. |

### Fabric Data Agents
Fabric data agents provide conversational interfaces allowing users to query lakehouses, warehouses, and semantic models using natural language[cite: 1]. They connect directly to Fabric IQ ontologies to ground responses in business context[cite: 1].

### Copilot Capabilities
Copilot is built into Fabric workloads by default (configurable in Tenant settings by admins)[cite: 1]:
* **Code Completion & Generation:** Intelligent inline code suggestions in notebooks, SQL generation from natural language, and Kusto Query Language (KQL) translation[cite: 1].
* **Data Transformation Guidance:** Code generation and plain-language logic explanations in Data Factory[cite: 1].
* **Insight & Report Generation:** Automated Power BI report creation, page summaries, and Q&A analytics[cite: 1].

```