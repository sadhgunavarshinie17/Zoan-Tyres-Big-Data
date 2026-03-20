# ZoanTyresPipeline

> **Cloud-Based Big Data Pipeline for Enterprise Tyre Distribution Analytics**

ZoanTyresPipeline is an end-to-end Azure data engineering solution built for **Zoan Core Tyre Trading LLC** — a UAE-based importer and distributor of Wanli and Joyroad tyre brands. The project modernizes the company's fragmented, manual data workflows into a scalable, cloud-native pipeline with automated batch processing, a star-schema data warehouse, and interactive Power BI dashboards.

> 📄 **Full implementation details, code, and screenshots are documented in [`ISIT312_ProjectReport.pdf`](./ISIT312_ProjectReport.pdf)**

Built for **ISIT312 — Big Data Management** at the University of Wollongong in Dubai.

---

## Problem Statement

Zoan Tyres relied on manual Excel entry, disconnected cloud storage (Google Cloud + iCloud), and a legacy ERP (Tally → ERPNext) with no analytics layer. Key pain points:

- Data fragmented across ERPNext, Google Cloud, Apple iCloud, and local Excel files
- No automated pipeline — all updates and backups performed manually
- Zero analytics or reporting on sales, inventory, or customer credit risk
- 365 PDF documents (invoices, bank statements, trade licenses) with no metadata indexing

---

## Architecture Overview

```
[Source Data]
CSV / JSON / PDF (ERPNext exports, credit applications, invoices)
         │
         ▼
[Storage Layer]
Azure Blob Storage / ADLS Gen2
  └── raw/csv/    → dim_customers, dim_items, dim_suppliers, dim_warehouses
  └── raw/json/   → erp_exports.json
  └── raw/pdfs/   → invoices, bank_statements, delivery_notes, etc.
         │
         ▼
[Processing & Transformation Layer]
Azure Databricks + PySpark (batch jobs)
  - Deduplication, null handling, primary key validation
  - Feature engineering (credit tiers, profit margins, supplier scores)
  - PDF metadata extraction via regex
  - Output: Parquet files → processed/
         │
         ▼
[Data Warehouse Layer]
Azure Synapse Analytics (Serverless SQL Pool)
  - External tables over Parquet: dim_*, fact_*, analytics_*
  - Star schema: fact_sales_invoices + fact_invoice_line_items
         │
         ▼
[Visualization Layer]
Power BI Dashboards
  - Sales Overview | Customer Analysis | Product Analysis
  - Warehouse & Supplier | Territory & Credit | Document Analysis
         │
         ▼
[Pipeline Automation]
Azure Data Factory
  - Event trigger on blob upload → runs Databricks notebook automatically
```

---

## Tech Stack

| Layer | Tool |
|---|---|
| Cloud Platform | Microsoft Azure |
| Storage | Azure Blob Storage / ADLS Gen2 |
| Processing | Azure Databricks, Apache Spark (PySpark) |
| Data Warehouse | Azure Synapse Analytics (Serverless SQL Pool) |
| Visualization | Microsoft Power BI |
| Pipeline Orchestration | Azure Data Factory |
| Data Formats | CSV, JSON, PDF → Parquet |
| Schema Pattern | Star Schema (ELT + Schema-on-Write) |

---

## Key Transformations

### Customer Enrichment
| Feature | Logic |
|---|---|
| `credit_tier` | Bronze / Silver / Gold based on credit limit thresholds |
| `customer_value_score` | `credit_limit / 10000` |
| `payment_risk_score` | Net 30 → 1, Net 60 → 2, Net 90 → 3 |
| `estimated_clv` | `credit_limit × 12 × 0.15` |

### Item Enrichment
| Feature | Logic |
|---|---|
| `profit_margin` | `(price - cost) / price × 100` |
| `profit_category` | Low / Medium / High |
| `reorder_urgency` | High / Medium / Low based on reorder level |

### Supplier Enrichment
| Feature | Logic |
|---|---|
| `payment_flexibility_score` | Net 30 → 1, Net 60 → 2 |
| `regional_preference_score` | China → 3, Thailand/Malaysia → 2, other → 1 |
| `supplier_score` | Composite of both scores |

---

## Data Warehouse Schema

**Fact Tables:**
- `fact_sales_invoices` — invoice headers with totals, status, days outstanding
- `fact_invoice_line_items` — line-level quantity, unit price, VAT, subtotals

**Dimension Tables:**
- `dim_customers` — enriched customer master (200 records)
- `dim_items` — enriched tire product catalog (100 SKUs)
- `dim_suppliers` — enriched supplier info (25 records)
- `dim_warehouses` — warehouse locations (4 records)

**Analytics Tables:**
- `analytics_territory` — pre-aggregated credit exposure by UAE territory
- `document_metadata` — PDF index with file metadata and customer linkage (365 documents)

---

## Power BI Dashboards

| Dashboard | OLAP Operations Used |
|---|---|
| Sales Overview | Drill-Down (year → month → quarter) |
| Customer Analysis | Slice & Dice, Pivot, Drill-Down, Roll-Up |
| Product Analysis | Slice & Dice, Drill-Down, Roll-Up |
| Warehouse & Supplier | Slice & Dice, Pivot, Roll-Up |
| Territory & Credit | Drill-Down (territory → customer group) |
| Document Analysis | Slice by document type |

---

## Repository Contents

```
ZoanTyresPipeline/
├── README.md
├── ISIT312_ProjectReport.pdf    # Full implementation: code, SQL, screenshots & analysis
└── raw/
    └── the CSV, JSON and PDF files
```

The project was built and deployed entirely on Microsoft Azure. The complete PySpark notebook, T-SQL scripts, pipeline configurations, and Power BI dashboard screenshots are all documented in the project report above.

---

## ⚠️ Limitations

- Dataset was small-scale and partially simulated due to limited enterprise data availability
- Azure Student subscription restricted certain regions and compute tiers
- Batch processing only — streaming (Kafka) not required at Zoan's current data velocity
- No ML models in this phase

---

## Future Improvements

- Real-time streaming with Apache Kafka as data volume grows
- OCR integration for automated PDF data extraction (Azure Form Recognizer)
- ML models for credit scoring and demand forecasting
- Pipeline monitoring and alerting via Azure Monitor

---

## Team

| Name |
|---|
| Saadiyah Asif |
| Roshni Warrier |
| Sadhguna Kumar |
| Zainab Gawai |
| Tanisha Ghansham Das |
| Nahan Salam |
| Fathima Noorain |
| Simran Khan |
| Danish Tanwar |
| Mohammed Amer |

Supervised by **Dr. Patrick Mukala**

---

## 📄 License

Developed for academic purposes as part of ISIT312 — Big Data Management at the University of Wollongong in Dubai. All rights reserved by the authors.
