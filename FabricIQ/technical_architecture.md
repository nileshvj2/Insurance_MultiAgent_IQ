# 🏗 Technical Architecture — Specialty Insurance Demo

This document covers the architecture design, data pipeline details, star schema, semantic model, ontology, setup instructions, and automation for the Specialty Insurance Demo on Microsoft Fabric.

> 📖 For a high-level overview, see [README.md](README.md).
> 📐 For complete column-level schemas, see [Insurance_Demo_Architecture.md](Insurance_Demo_Architecture.md).

---

## Table of Contents

- [Architecture Design](#architecture-design)
- [Fabric Resources](#fabric-resources)
- [Data Pipeline](#data-pipeline)
  - [Bronze Layer](#-bronze-layer--raw--cleansed)
  - [Gold Layer](#-gold-layer--star-schema)
- [Star Schema Relationships](#-star-schema-relationships)
- [Semantic Model](#-semantic-model--insurance-analytics)
- [DAX Measures](#dax-measures-33)
- [Ontology & Knowledge Graph](#-ontology--insurance_ontology)
- [Data Agent](#-data-agent--insurance_data_agent)
- [Getting Started](#-getting-started)
- [Portal Configuration](#-portal-configuration)
- [Project Structure](#-project-structure)
- [Automation & Code Generation](#-automation--code-generation)
- [Workspace Inventory](#-workspace-inventory)
- [Sample Business Questions](#-sample-business-questions)

---

## Architecture Design

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                       Microsoft Fabric Workspace                             │
│                          "InsuranceDemo"                                      │
│                                                                              │
│  ┌──────────────────────┐         ┌──────────────────────┐                   │
│  │   insurance_bronze   │         │    insurance_gold     │                   │
│  │     (Lakehouse)      │────────►│     (Lakehouse)       │                   │
│  │                      │  PySpark│                       │                   │
│  │  9 Raw/Cleansed      │  Trans- │  9 Dimension Tables   │                   │
│  │  Tables              │  form   │  6 Fact Tables        │                   │
│  └──────────────────────┘         └───────────┬───────────┘                   │
│                                               │                              │
│                                               ▼                              │
│                                   ┌───────────────────────┐                  │
│                                   │  "Insurance Analytics" │                  │
│                                   │   Semantic Model       │                  │
│                                   │  (DirectLake / DAX)    │                  │
│                                   └───────────┬───────────┘                  │
│                                               │                              │
│                              ┌────────────────┼────────────────┐             │
│                              ▼                                 ▼             │
│                  ┌───────────────────────┐         ┌───────────────────────┐  │
│                  │  "Insurance_Ontology" │         │ "Insurance_Data_Agent" │  │
│                  │   Ontology / Graph    │         │   Fabric Data Agent    │  │
│                  │   Model Layer         │         │   (Fabric IQ / NL Q&A) │  │
│                  └───────────────────────┘         └───────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

### Why Medallion Architecture?

| Layer | Purpose | Data Characteristics |
|---|---|---|
| **Bronze** | Raw ingestion / system-of-record mirror | Source-system schemas preserved, minimal transformation, append-only, full history |
| **Gold** | Business-ready analytics | Star schema, surrogate keys, derived measures, conformed dimensions, optimised for queries |

---

## Fabric Resources

| Resource | Type |
|---|---|
| InsuranceDemo | Workspace |
| insurance_bronze | Lakehouse |
| insurance_gold | Lakehouse |

---

## Data Pipeline

### 🥉 Bronze Layer — Raw / Cleansed

The Bronze layer contains **9 tables** representing core operational data from insurance source systems (policy administration, claims management, underwriting platforms, billing, reinsurance, and actuarial systems). Data is lightly cleansed but retains the source-system schema and granularity. Each table includes `source_system`, `created_date`, and `modified_date` audit columns for lineage tracking.

| Table | Description | Rows | Source Systems |
|---|---|---|---|
| `bronze_customers` | Policyholder organisations — insured entities that purchase specialty insurance | 80 | CRM, POLICY_ADMIN, LEGACY_SYSTEM |
| `bronze_brokers` | Insurance brokers and intermediaries who place business | 30 | BROKER_PORTAL, CRM |
| `bronze_policies` | Insurance contracts — coverage terms, limits, deductibles, premium, territory | 100 | POLICY_ADMIN, UNDERWRITING_PLATFORM, LEGACY_SYSTEM |
| `bronze_claims` | Loss event notifications — financial exposure, loss type, peril, severity | 75 | CLAIMS_SYSTEM, LEGACY_CLAIMS, TPA_FEED |
| `bronze_claim_transactions` | Financial transactions against claims — payments, reserves, recoveries, legal fees | 250 | CLAIMS_SYSTEM, FINANCE_SYSTEM, TPA_FEED |
| `bronze_underwriting_submissions` | Risks presented by brokers — full submission lifecycle from receipt to binding | 90 | UNDERWRITING_PLATFORM, BROKER_PORTAL, EMAIL_INGESTION |
| `bronze_premium_bordereaux` | Premium billing, collection, commission settlement detail | 500 | BILLING_SYSTEM, BORDEREAUX_FEED, PREMIUM_ADMIN |
| `bronze_reinsurance_treaties` | Reinsurance contracts — structure, retention, limits, financial terms | 40 | REINSURANCE_SYSTEM, TREATY_MGMT |
| `bronze_loss_runs` | Historical loss experience by customer, policy, accident year, valuation date | 100 | ACTUARIAL_SYSTEM, LOSS_RUN_PROVIDER, LEGACY_DATA |

**Bronze notebooks** (`nb_*.ipynb`) generate synthetic data using PySpark and load it into the `insurance_bronze` Lakehouse.

### 🥇 Gold Layer — Star Schema

The Gold layer transforms Bronze data into a **dimensional model** (star schema) with **9 dimension tables** and **6 fact tables**. All transformations are performed via PySpark notebooks reading from Bronze via cross-lakehouse `abfss://` paths. The Gold layer adds:

- **Surrogate keys** for dimensional relationships
- **Derived measures** (ratios, calculated fields, flags)
- **Conformed dimensions** shared across fact tables
- **Business classifications** (customer segments, broker tiers)

#### Dimension Tables (9)

| Table | Description | Rows | Key Derived Fields |
|---|---|---|---|
| `dim_date` | Shared calendar dimension (2018–2025) with fiscal year (April start) | 2,922 | fiscal_year, fiscal_quarter, fiscal_month |
| `dim_customer` | Policyholder dimension | 80 | **customer_segment**: Enterprise (>$1B) / Large / Mid-Market / Small |
| `dim_broker` | Broker dimension | 30 | **broker_tier**: Platinum (>$50M) / Gold / Silver / Bronze |
| `dim_underwriter` | Underwriter profiles with authority limits | ~50 | specialization, authority_limit, office_location |
| `dim_coverage_line` | Lines of business / products | 10 | line_category, coverage_type, target_market |
| `dim_geography` | Major global insurance market locations | 15 | region, country, city |
| `dim_claim_status` | Claim lifecycle stages | 8 | status_category (Active/Closed), sort_order |
| `dim_policy_status` | Policy lifecycle stages | 6 | status_category (In Force/Terminated/Pre-Inception) |
| `dim_peril` | Risk event taxonomy | 20 | peril_category (Natural/Accidental/Man-made) |

#### Fact Tables (6)

| Table | Grain | Rows | Key Measures | Key Derived Fields |
|---|---|---|---|---|
| `fact_policy` | One row per policy | 100 | gross_premium, net_premium, policy_limit, deductible | **retention_ratio**, **policy_term_days** |
| `fact_claim` | One row per claim | 75 | incurred_amount, paid_amount, reserved_amount | **reporting_lag_days**, **paid_to_incurred_ratio** |
| `fact_premium_transaction` | One row per txn | 500 | gross_premium, commission_amount, net_premium, tax | **effective_commission_rate** |
| `fact_underwriting_submission` | One row per submission | 90 | quoted_premium, technical_price, risk_score | **price_adequacy_ratio**, **is_bound**, **is_quoted**, **is_declined** |
| `fact_loss_run` | One row per loss run | 100 | incurred_total, paid_total, earned_premium, ultimate_loss | **ibnr_estimate**, **ultimate_loss_ratio**, **avg_claim_size** |
| `fact_reinsurance` | One row per treaty | 40 | cession_rate, retention, treaty_limit, premium_ceded | **layer_size**, **rate_on_line** |

---

## 🔗 Star Schema Relationships

```
                           ┌─────────────┐
                           │  dim_date   │
                           └──────┬──────┘
                                  │
            ┌─────────────────────┼─────────────────────┐
            │                     │                     │
     ┌──────┴──────┐      ┌──────┴──────┐      ┌──────┴──────┐
     │ fact_policy  │      │ fact_claim  │      │fact_premium │
     └──┬───┬───┬──┘      └──┬───┬──┬───┘      └──┬───┬──┬──┘
        │   │   │             │   │  │             │   │  │
        │   │   └─────────────┼───┼──┼─────────────┼───┘  │
        │   │                 │   │  │             │      │
        │   └─────────────────┼───┘  │             │      │
        │                     │      │             │      │
  ┌─────┴──────┐       ┌─────┴──────┐      ┌─────┴──────┐
  │dim_customer│       │ dim_broker │      │dim_coverage│
  └────────────┘       └────────────┘      │   _line    │
                                           └────────────┘

  Additional relationships:
  fact_claim         → dim_claim_status, dim_peril
  fact_policy        → dim_policy_status
  fact_loss_run      → dim_customer, dim_date, dim_peril, dim_coverage_line
  fact_uw_submission → dim_customer, dim_broker, dim_date, dim_coverage_line
  fact_reinsurance   → dim_date, dim_coverage_line
```

All relationships follow the star schema pattern: **Dimension (1) → Fact (Many)**

| From (Dimension) | To (Fact) | Join Key |
|---|---|---|
| dim_date | fact_policy | date_key (inception) |
| dim_date | fact_claim | date_key (loss date) |
| dim_date | fact_premium_transaction | date_key (transaction) |
| dim_date | fact_underwriting_submission | date_key (submission) |
| dim_date | fact_loss_run | date_key (accident) |
| dim_date | fact_reinsurance | date_key (inception) |
| dim_customer | fact_policy | customer_key |
| dim_customer | fact_claim | customer_key |
| dim_broker | fact_policy | broker_key |
| dim_broker | fact_underwriting_submission | broker_key |
| dim_underwriter | fact_policy | underwriter_key |
| dim_coverage_line | fact_policy | coverage_line_key |
| dim_geography | fact_policy | geography_key |
| dim_claim_status | fact_claim | claim_status_key |
| dim_policy_status | fact_policy | policy_status_key |

---

## 📊 Semantic Model — "Insurance Analytics"

The **Insurance Analytics** semantic model is a **DirectLake** Power BI semantic model built on top of the Gold layer lakehouse.

- **Mode:** DirectLake (queries Gold lakehouse tables directly via OneLake — no data import/duplication)
- **Compatibility Level:** 1604
- **File:** `model.bim`

```
                            ┌──────────────┐
                            │   dim_date   │
                            └──────┬───────┘
                                   │
    ┌──────────────┐    ┌──────────┴──────────┐    ┌──────────────────┐
    │ dim_customer  │────┤    fact_policy       ├────│    dim_broker     │
    └──────────────┘    │    fact_claim         │    └──────────────────┘
                        │    fact_premium_txn   │
    ┌──────────────┐    │    fact_uw_submission  │    ┌──────────────────┐
    │ dim_geography │────┤    fact_loss_run      ├────│  dim_underwriter  │
    └──────────────┘    │    fact_reinsurance    │    └──────────────────┘
                        └──────────┬──────────┘
    ┌────────────────┐             │              ┌──────────────────┐
    │dim_coverage_line├────────────┤              │  dim_claim_status │
    └────────────────┘             │              └──────────────────┘
    ┌────────────────┐             │              ┌──────────────────┐
    │   dim_peril     ├────────────┘              │ dim_policy_status │
    └────────────────┘                            └──────────────────┘
```

### Model Features

- **Hidden FK Columns:** All foreign key columns are hidden from report authors to simplify the field list
- **Column Descriptions:** Every column has a business-friendly description for Copilot and self-service discoverability
- **Currency Formatting:** All monetary measures formatted as currency ($, 2 decimal places)
- **Percentage Formatting:** Ratio measures (loss ratio, hit ratio, etc.) formatted as percentages
- **DirectLake Mode:** No data duplication — queries run directly against Delta tables in the Gold lakehouse via OneLake

### DAX Measures (33)

#### Portfolio Measures

| Measure | Formula Summary | Format |
|---|---|---|
| Total GWP | SUM(fact_policy[gross_written_premium]) | Currency |
| Total NWP | SUM(fact_policy[net_written_premium]) | Currency |
| Policy Count | COUNTROWS(fact_policy) | Whole Number |
| Average Policy Limit | AVERAGE(fact_policy[policy_limit]) | Currency |
| Average Deductible | AVERAGE(fact_policy[deductible]) | Currency |
| Active Policies | CALCULATE(Policy Count, dim_policy_status[policy_status_name]="Active") | Whole Number |

#### Claims Measures

| Measure | Formula Summary | Format |
|---|---|---|
| Total Incurred | SUM(fact_claim[incurred_amount]) | Currency |
| Total Paid | SUM(fact_claim[paid_amount]) | Currency |
| Total Reserved | SUM(fact_claim[reserved_amount]) | Currency |
| Claim Count | COUNTROWS(fact_claim) | Whole Number |
| Average Claim Severity | AVERAGE(fact_claim[incurred_amount]) | Currency |
| Loss Ratio | Total Incurred / Total GWP | Percentage |
| Open Claims | CALCULATE(Claim Count, dim_claim_status[claim_status_name]="Open") | Whole Number |

#### Premium Measures

| Measure | Formula Summary | Format |
|---|---|---|
| Total Premium Collected | SUM(fact_premium_transaction[premium_amount]) | Currency |
| Total Commission | SUM(fact_premium_transaction[commission_amount]) | Currency |
| Average Commission Rate | AVERAGE(fact_premium_transaction[commission_rate]) | Percentage |
| Net Premium After Commission | Total Premium Collected - Total Commission | Currency |
| Premium Transaction Count | COUNTROWS(fact_premium_transaction) | Whole Number |

#### Underwriting Measures

| Measure | Formula Summary | Format |
|---|---|---|
| Submission Count | COUNTROWS(fact_underwriting_submission) | Whole Number |
| Bound Submissions | CALCULATE(Submission Count, fact_underwriting_submission[status]="Bound") | Whole Number |
| Hit Ratio | Bound Submissions / Submission Count | Percentage |
| Average Quoted Premium | AVERAGE(fact_underwriting_submission[quoted_premium]) | Currency |
| Average Technical Price | AVERAGE(fact_underwriting_submission[technical_price]) | Currency |
| Pricing Adequacy | Average Quoted Premium / Average Technical Price | Percentage |
| Total Sum Insured | SUM(fact_underwriting_submission[sum_insured]) | Currency |

#### Actuarial Measures

| Measure | Formula Summary | Format |
|---|---|---|
| Ultimate Loss | SUM(fact_loss_run[ultimate_loss]) | Currency |
| IBNR Reserve | SUM(fact_loss_run[ibnr_amount]) | Currency |
| Average Development Factor | AVERAGE(fact_loss_run[development_factor]) | Decimal |
| Loss Run Count | COUNTROWS(fact_loss_run) | Whole Number |

#### Reinsurance Measures

| Measure | Formula Summary | Format |
|---|---|---|
| Total Ceded Premium | SUM(fact_reinsurance[ceded_premium]) | Currency |
| Average Cession Rate | AVERAGE(fact_reinsurance[cession_percentage]) | Percentage |
| Total Reinsurer Limit | SUM(fact_reinsurance[reinsurer_limit]) | Currency |
| Treaty Count | COUNTROWS(fact_reinsurance) | Whole Number |

---

## 🧠 Ontology — "Insurance_Ontology"

The **Insurance_Ontology** defines the domain knowledge graph for the specialty insurance data.

- **Backing GraphModel:** Auto-generated by Fabric when the ontology is created
- **Auto-created Lakehouse:** Auto-provisioned by Fabric for ontology storage

### Entity Model

```
                    ┌────────────┐
                    │  Customer  │
                    └─────┬──────┘
                          │ HAS_POLICY
                          ▼
┌────────┐  PLACED_BY  ┌────────┐  COVERS_LINE  ┌───────────────┐
│ Broker │◄────────────│ Policy │──────────────►│ CoverageLine  │
└────────┘             └───┬────┘               └───────────────┘
                           │
          ┌────────────────┼────────────────┬──────────────────┐
          │                │                │                  │
          ▼                ▼                ▼                  ▼
   ┌─────────────┐  ┌───────────┐  ┌─────────────┐   ┌──────────────┐
   │ Underwriter │  │   Claim   │  │  Geography  │   │ Reinsurance  │
   └─────────────┘  └─────┬─────┘  └─────────────┘   └──────────────┘
                          │
                          ▼
                    ┌───────────┐
                    │   Peril   │
                    └───────────┘
```

### Entities (9)

| Entity | Source Table | Description |
|---|---|---|
| Customer | dim_customer | Insured organisations and policyholders |
| Broker | dim_broker | Insurance brokers and intermediaries |
| Policy | fact_policy | Insurance policies covering specialty lines |
| Claim | fact_claim | Claims filed against specialty policies |
| CoverageLine | dim_coverage_line | Specialty insurance lines of business |
| Underwriter | dim_underwriter | Underwriters who assess and price risks |
| Geography | dim_geography | Locations for risk assessment and compliance |
| Peril | dim_peril | Perils and risk types covered |
| Reinsurance | fact_reinsurance | Treaties and facultative placements |

### Relationships (8)

| Relationship | Source → Target | Description |
|---|---|---|
| HAS_POLICY | Customer → Policy | Customer holds an insurance policy |
| PLACED_BY | Policy → Broker | Policy was placed by a broker |
| UNDERWRITTEN_BY | Policy → Underwriter | Policy was underwritten by an underwriter |
| HAS_CLAIM | Policy → Claim | Policy has an associated claim |
| COVERS_LINE | Policy → CoverageLine | Policy covers a specific line of business |
| LOCATED_IN | Policy → Geography | Policy risk is located in a geography |
| CAUSED_BY_PERIL | Claim → Peril | Claim was caused by a specific peril |
| CEDED_TO | Policy → Reinsurance | Policy risk is ceded to a reinsurance treaty |

### Configuration Files

| File | Purpose |
|---|---|
| `ontology_definition.json` | Domain ontology entity and relationship definitions |
| `graphDefinition.json` | Graph model structure for Fabric |
| `graphType.json` | Graph type configuration |
| `graph_update_body.json` | Graph update REST API payload |
| `stylingConfiguration.json` | Visual styling for graph rendering |

> ⚠️ **Note:** The Ontology item was created programmatically via the Fabric REST API. However, the GraphModel definition must be configured through the **Fabric portal UI** — the API does not yet support persisting graph definitions for ontologies in preview.

---

## 🤖 Data Agent — "Insurance_Data_Agent"

The **Insurance_Data_Agent** enables **natural language Q&A** (Fabric IQ) over the insurance portfolio data.



### Pre-loaded AI Instructions

- **Domain expertise:** Specialty insurance analytics across Marine, Aviation, Cyber, D&O, E&O, Professional Liability
- **Key terminology:** GWP, NWP, Loss Ratio, Combined Ratio, IBNR, Cession, Bordereaux, Hit Ratio, Severity, Frequency
- **Business context:** Star schema structure, metric targets (loss ratio < 65%, combined ratio < 100%), fiscal year alignment
- **Response guidance:** Line-of-business context awareness, actionable insights focus

### Demo Questions

| Category | Example |
|---|---|
| Portfolio | "What is our total gross written premium by line of business?" |
| Claims | "What is the current loss ratio by coverage line?" |
| Underwriting | "What is the quote-to-bind hit ratio by broker?" |
| Reinsurance | "What is our net retention after reinsurance cessions?" |
| Actuarial | "Show the IBNR reserves by accident year" |
| Executive | "Give me a combined ratio breakdown by line of business" |

---

## 🚀 Getting Started

### Prerequisites

- **Microsoft Fabric** workspace with Lakehouse capability
- **Two Lakehouses** created:
  - `insurance_bronze` — Raw data landing zone
  - `insurance_gold` — Transformed star schema
- PySpark notebook execution environment (Fabric Notebooks)

### Step 1: Load Bronze Data

Run the Bronze notebooks to generate synthetic data:

```powershell
# Run all Bronze notebooks via PowerShell
.\run_bronze.ps1

# Or run individually in Fabric:
# 1. nb_policies.ipynb
# 2. nb_claims.ipynb
# 3. nb_claim_txns.ipynb
# 4. nb_premium.ipynb
# 5. nb_uw_subs.ipynb
# 6. nb_loss_runs.ipynb
# 7. nb_reinsurance.ipynb
```

### Step 2: Transform to Gold

Run the Gold notebooks to build the star schema (order matters — dimensions before facts):

```
1. gold_dim_date.ipynb
2. gold_dim_cust_brk.ipynb
3. gold_dims.ipynb
4. gold_fact_policy.ipynb
5. gold_fact_claim.ipynb
6. gold_fact_prem.ipynb
7. gold_fact_uw.ipynb
8. gold_fact_loss.ipynb
9. gold_fact_reins.ipynb
```

### Step 3: Deploy Semantic Model

Import `model.bim` into Fabric as a semantic model connected to `insurance_gold` via DirectLake.

### Step 4: Configure Data Agent & Ontology

See [Portal Configuration](#-portal-configuration) below.

---

## 🖥 Portal Configuration

The following items require manual configuration through the [Microsoft Fabric Portal](https://app.fabric.microsoft.com):

### 1. Configure the Data Agent (Required for Fabric IQ Demo)

1. Navigate to the **InsuranceDemo** workspace in the Fabric portal
2. Open **Insurance_Data_Agent**
3. Click **+ Data source** in the Explorer pane
4. From the OneLake catalog, select **Insurance Analytics** (Semantic Model)
5. Click **Add** to attach it as a data source
6. In the Explorer pane, ensure all **15 tables** are checked (9 dim_* + 6 fact_*)
7. The AI instructions are already pre-loaded — no additional configuration needed
8. Test the agent by typing a question in the chat pane, e.g.: *"What is the total GWP by coverage line?"*
9. Once satisfied with responses, click **Publish** to make the agent available to colleagues

### 2. Configure the Ontology (Optional — for Graph-based Exploration)

1. Navigate to the **InsuranceDemo** workspace in the Fabric portal
2. Open **Insurance_Ontology**
3. Add the **Insurance Analytics** semantic model as the data source
4. Map the business entities (Customer, Policy, Claim, Broker, etc.) to the corresponding tables using the visual designer
5. Define relationships between entities (e.g., Customer → HAS_POLICY → Policy)
6. Save and validate the ontology

### 3. Build Power BI Reports (Optional)

- Connect to the **Insurance Analytics** semantic model from Power BI Desktop or the Fabric portal
- Use the 33 pre-built DAX measures for instant visualizations
- Build dashboards for Underwriting Performance, Claims Analytics, Portfolio Overview, and Executive KPIs

### 4. Cleanup (Optional)

Remove debug artifacts from the workspace:
- Delete `test_table` from the `insurance_bronze` lakehouse
- Delete `schema_info` and `bronze_schema_info` tables from the `insurance_gold` lakehouse
- Remove any unused notebooks from the workspace

---

## 📁 Project Structure

```
InsuranceDemo/
│
├── README.md                        # High-level project overview
├── technical_architecture.md        # This file — technical details
├── Insurance_Demo_Architecture.md   # Complete column-level specification
│
├── # ── Bronze Layer Notebooks ──
├── 01_Bronze_Layer_Load.ipynb       # Master Bronze load orchestrator
├── nb_policies.ipynb                # Generate bronze_policies
├── nb_claims.ipynb                  # Generate bronze_claims
├── nb_claim_txns.ipynb              # Generate bronze_claim_transactions
├── nb_premium.ipynb                 # Generate bronze_premium_bordereaux
├── nb_uw_subs.ipynb                 # Generate bronze_underwriting_submissions
├── nb_loss_runs.ipynb               # Generate bronze_loss_runs
├── nb_reinsurance.ipynb             # Generate bronze_reinsurance_treaties
├── run_bronze.ps1                   # PowerShell script to run all Bronze notebooks
│
├── # ── Gold Layer Notebooks ──
├── 02_Gold_Layer_Transform.ipynb    # Master Gold transform orchestrator
├── gold_dim_date.ipynb              # Transform dim_date
├── gold_dim_cust_brk.ipynb          # Transform dim_customer & dim_broker
├── gold_dims.ipynb                  # Transform remaining dimensions
├── gold_fact_policy.ipynb           # Transform fact_policy
├── gold_fact_claim.ipynb            # Transform fact_claim
├── gold_fact_prem.ipynb             # Transform fact_premium_transaction
├── gold_fact_uw.ipynb               # Transform fact_underwriting_submission
├── gold_fact_loss.ipynb             # Transform fact_loss_run
├── gold_fact_reins.ipynb            # Transform fact_reinsurance
│
├── # ── Semantic Model ──
├── model.bim                        # TMDL / BIM semantic model definition
├── dataSources.json                 # Data source connection definitions
│
├── # ── Ontology & Graph ──
├── ontology_definition.json         # Domain ontology definition
├── graphDefinition.json             # Graph model structure
├── graphType.json                   # Graph type configuration
├── graph_update_body.json           # Graph update API payload
├── stylingConfiguration.json        # Graph visual styling
│
├── # ── Code Generation Scripts ──
├── generate_notebooks.py            # Python notebook generator
├── generate_notebooks.js            # JS notebook generator
├── gen_bronze_nbs.js                # Generate Bronze notebooks
├── gen_gold_nbs.js                  # Generate Gold notebooks
├── gen_gold_dims_split.js           # Generate dimension notebooks
├── gen_bim.js                       # Generate semantic model (BIM)
├── gen_ontology.js                  # Generate ontology definition
├── gen_graph_def.js                 # Generate graph definition
├── gen_diag.js                      # Generate diagnostic scripts
│
└── # ── Debug / Test Notebooks ──
    ├── diag_schema.ipynb            # Schema diagnostics
    ├── test_small_data.ipynb        # Small data test
    └── test_3cells.ipynb            # Quick cell tests
```

---

## 🛠 Automation & Code Generation

Several scripts automate notebook and artifact generation:

| Script | Purpose |
|---|---|
| `generate_notebooks.py` | Python-based notebook generator |
| `generate_notebooks.js` | JS notebook generator |
| `gen_bronze_nbs.js` | Generate all Bronze layer notebooks |
| `gen_gold_nbs.js` | Generate all Gold layer notebooks |
| `gen_gold_dims_split.js` | Generate individual dimension notebooks |
| `gen_gold_dims_fix.js` | Fix dimension notebook generation |
| `gen_dim_fix2.js` | Additional dimension fixes |
| `gen_bim.js` | Generate the semantic model (model.bim) |
| `gen_ontology.js` | Generate the ontology definition |
| `gen_graph_def.js` | Generate the graph model definition |
| `gen_diag.js` / `gen_diag2.js` | Generate diagnostic notebooks |
| `gen_remaining_bronze.js` | Generate remaining Bronze notebooks |

These scripts use the architecture specification (`Insurance_Demo_Architecture.md`) as the source of truth and produce Fabric-compatible `.ipynb` notebooks and JSON artifacts.

---

## 📦 Workspace Inventory

| Type | Item Name |
|---|---|
| **Workspace** | InsuranceDemo |
| **Lakehouse** | insurance_bronze |
| **Lakehouse** | insurance_gold |
| **Semantic Model** | Insurance Analytics |
| **Ontology** | Insurance_Ontology |
| **Data Agent** | Insurance_Data_Agent |
| **GraphModel** | Insurance_Ontology_graph_* (auto-created) |
| **Lakehouse** | Insurance_Ontology_lh_* (auto-created) |
| **SQL Endpoints** | 3 (one per lakehouse) |
| **Notebooks** | 16 (Bronze + Gold data generation) |

---

## 📈 Sample Business Questions

| Domain | Question |
|---|---|
| **Portfolio** | What is the GWP split by line of business and territory? |
| **Portfolio** | Which customer segments generate the highest premium volume? |
| **Underwriting** | What is the quote-to-bind ratio by broker tier and coverage line? |
| **Underwriting** | How does pricing adequacy (quoted vs technical price) vary by underwriter? |
| **Claims** | What is the average reporting lag by line of business? |
| **Claims** | Which perils drive the highest catastrophic severity claims? |
| **Actuarial** | What are the loss development patterns by accident year? |
| **Actuarial** | What is the IBNR estimate by line and accident year? |
| **Premium** | What is the outstanding receivables balance by broker? |
| **Premium** | How does commission rate vary between broker tiers? |
| **Reinsurance** | What is the net retention by coverage line after treaty cessions? |
| **Reinsurance** | How does rate-on-line compare across reinsurers? |
| **Executive** | What is the combined ratio (loss + expense) by line of business? |
| **Executive** | What is the renewal retention rate by customer segment? |

---

*Specialty Insurance Demo — Microsoft Fabric Medallion Architecture*
*Bronze: 9 tables | Gold: 15 tables (9 dims + 6 facts) | Semantic Model: 33 DAX measures | Ontology: 9 entities | Data Agent: Fabric IQ ready*
