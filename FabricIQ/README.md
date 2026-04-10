# 🏢 Specialty Insurance Demo — Microsoft Fabric

A reference **Medallion Architecture** (Bronze → Gold) on **Microsoft Fabric** for the **Specialty Insurance** domain. Covers six major specialty lines with synthetic data modelling the full insurance value chain — from underwriting submission through claims settlement.


## What This Project Does

This demo builds a complete specialty insurer analytics platform on Microsoft Fabric:

```
  Bronze Lakehouse          Gold Lakehouse           Analytics Layer
 ┌──────────────┐        ┌────────────────┐       ┌─────────────────┐
 │ 9 Raw Tables │──────► │ 9 Dimensions   │──────►│ Semantic Model  │
 │ (~1,200 rows)│PySpark │ 6 Fact Tables  │Direct │ (33 DAX KPIs)   │
 └──────────────┘        └────────────────┘ Lake  ├─────────────────┤
                                                   │ Ontology/Graph  │
                                                   │ Data Agent (IQ) │
                                                   └─────────────────┘
```
This project primarily focuses on building Fabric IQ as a end product for Insurance Underwriting Agentic AI workflow. Refer other folders in this project to understand end to end multi agent workflow orchestration.

## Data at a Glance

| Data | Volume | Examples |
|---|---|---|
| Policyholders | 80 | Technology, Maritime, Healthcare companies |
| Brokers | 30 | Marsh, Aon, Willis, regional specialists |
| Policies | 100 | Across 10 specialty coverage lines |
| Claims | 75 | With full incurred / paid / reserved tracking |
| Premium Transactions | 500 | Bordereaux billing and commission detail |
| Underwriting Submissions | 90 | Full pipeline from receipt to bind/decline |
| Reinsurance Treaties | 40 | Quota Share, Excess of Loss, Cat XL |
| Loss Runs | 100 | Actuarial development triangles |

## Specialty Lines

Marine Cargo · Marine Hull · Aviation Hull · Aviation Liability · Cyber Liability · Cyber Privacy · D&O · E&O · Professional Liability · Environmental Liability

## Use Cases

- **Underwriting** — Pipeline velocity, quote-to-bind ratio, pricing adequacy
- **Claims** — Loss experience, severity, reserving, catastrophe exposure
- **Premium Accounting** — Bordereaux reconciliation, commission tracking, receivables
- **Actuarial** — Loss development triangles, IBNR, ultimate loss projections
- **Reinsurance** — Cession analysis, rate-on-line, capacity utilisation
- **Portfolio & Executive** — Combined ratio, geographic exposure, renewal retention

## Key Components

| Component | Description |
|---|---|
| **insurance_bronze** | Lakehouse with 9 raw/cleansed tables from source systems |
| **insurance_gold** | Lakehouse with star schema (9 dims + 6 facts) |
| **Insurance Analytics** | DirectLake semantic model with 33 DAX measures |
| **Insurance_Ontology** | Knowledge graph with 9 entities and 8 relationships |
| **Insurance_Data_Agent** | Fabric Data Agent for natural language Q&A |

## How to Implement

There are two ways to set up this project in your Microsoft Fabric workspace:

### Option 1: Manual Setup (Step-by-Step)

Follow the detailed instructions in [technical_architecture.md](technical_architecture.md):

1. **Load Bronze** — Run `nb_*.ipynb` notebooks (or `run_bronze.ps1`) to generate synthetic data
2. **Transform to Gold** — Run `gold_*.ipynb` notebooks to build the star schema
3. **Deploy Semantic Model** — Import `model.bim` into Fabric (DirectLake mode)
4. **Configure Data Agent** — Attach the semantic model in Fabric portal for NL Q&A

### Option 2: AI-Assisted with Fabric Skills for GitHub Copilot

Use **GitHub Copilot** with **Fabric Skills** to build, deploy, and manage this project through natural language — directly from your terminal or IDE.

**What are Fabric Skills for Copilot?**
Fabric Skills are specialized agents for GitHub Copilot that understand Microsoft Fabric workloads — Lakehouses, Spark notebooks, semantic models, pipelines, and more. They can generate code, create infrastructure, troubleshoot issues, and orchestrate data engineering workflows conversationally.

**Prerequisites:**
- [GitHub Copilot](https://github.com/features/copilot) subscription (Individual, Business, or Enterprise)
- [GitHub Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli) installed and authenticated
- Fabric Skills extension installed — see the [Fabric Skills for GitHub Copilot](https://github.com/microsoft/fabric-skills-for-github-copilot) repository for installation instructions

**Getting Started:**
```bash
# Install GitHub Copilot CLI (if not already installed)
gh extension install github/gh-copilot

# Install Fabric Skills for GitHub Copilot
# Follow instructions at: https://github.com/microsoft/fabric-skills-for-github-copilot
```

**Example prompts you can use:**
- *"Create a Lakehouse called insurance_bronze in my Fabric workspace"*
- *"Generate a PySpark notebook to load synthetic policy data into Bronze"*
- *"Transform Bronze tables into a Gold star schema with dimension and fact tables"*
- *"Create a DirectLake semantic model with DAX measures for insurance KPIs"*
- *"Help me build a data pipeline to orchestrate Bronze and Gold notebook execution"*

**References:**
- 📘 [Fabric Skills for GitHub Copilot — GitHub Repository](https://github.com/microsoft/fabric-skills-for-github-copilot)
- 📘 [GitHub Copilot CLI Documentation](https://docs.github.com/en/copilot/github-copilot-in-the-cli)
- 📘 [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)

---

## Documentation

| Document | Purpose |
|---|---|
| [README.md](README.md) | This file — project overview |
| [technical_architecture.md](technical_architecture.md) | Architecture design, data pipeline, star schema, semantic model, ontology, setup guide, and automation |
|

## Glossary

| Term | Definition |
|---|---|
| **GWP / NWP** | Gross / Net Written Premium |
| **Loss Ratio** | Incurred losses ÷ earned premium |
| **Combined Ratio** | Loss + expense ratio (< 100% = profit) |
| **IBNR** | Incurred But Not Reported reserves |
| **Hit Ratio** | Bound ÷ total submissions |
| **Bordereaux** | Transactional premium/claims report |
| **Rate-on-Line** | Reinsurance premium ÷ treaty limit |

---

*Demonstration project with synthetic data. Not intended for production use.*
