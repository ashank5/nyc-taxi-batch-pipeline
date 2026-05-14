# NYC Taxi Data Pipeline — Azure Databricks

An end-to-end batch data pipeline built on **Databricks Community Edition** 
using **PySpark**, **Delta Lake**, and the **medallion architecture** 
(Bronze → Silver → Gold).

---

## Architecture

![Pipeline Architecture](architecture/pipeline_architecture.png)

---

## Tech Stack

| Component | Technology |
|---|---|
| Processing engine | Apache Spark (PySpark) |
| Storage format | Delta Lake |
| Platform | Databricks Community Edition |
| Dataset | NYC TLC Yellow Taxi 2023 |
| Ingestion | Manual upload to Databricks Volume |

---

## Pipeline Design

### Medallion Architecture

| Layer | Path | Description |
|---|---|---|
| Bronze | `/Volumes/.../bronze/nyc_taxi/` | Raw parquet ingested as-is with metadata columns added |
| Silver | `/Volumes/.../silver/nyc_taxi/` | Cleaned, deduplicated, enriched with derived columns |
| Gold | `/Volumes/.../gold/*/` | Five aggregated business metric tables |

### Ingestion Strategy

| Layer | Strategy | Idempotency |
|---|---|---|
| Bronze | Full load (manual) → incremental via watermark | `replaceWhere` on year/month partition |
| Silver | Incremental via Silver watermark | `replaceWhere` or MERGE |
| Gold | Triggered by RUN_DATE or Silver watermark | MERGE on business key |

---

## Gold Tables

| Table | Business Question Answered |
|---|---|
| `daily_revenue` | How does revenue trend day by day? |
| `hourly_volume` | Which hours have peak demand? |
| `zone_performance` | Which pickup zones generate most revenue? |
| `payment_breakdown` | How do customers prefer to pay? |
| `trip_category` | How do short, medium, long trips compare? |
| `monthly_cumulative` | What is the full-year monthly trend? |

---

## How to Run

### Prerequisites
- Databricks Community Edition account
- Cluster with DBR 13.x or above (includes Delta Lake)

### Step 1 — Upload Data
Download 2023 Yellow Taxi parquet files from:  
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

Upload manually to:
```
/Volumes/main/nyc_taxi_raw/landing_zone/
```

### Step 2 — Initialise Watermark
Run once before first pipeline execution:
```
00_init_watermark.py
```

### Step 3 — Run Pipeline
Run notebooks in order, setting `RUN_DATE = "2023-01"` through `"2023-12"`:

```
ingest_nyc_taxi.py       # Bronze
transform_nyc_taxi.py    # Silver
aggregate_nyc_taxi.py    # Gold
```

Set `RUN_DATE = None` to process all pending months automatically.

---

## Key Design Decisions

See [docs/design_decisions.md](docs/design_decisions.md) for full rationale.

| Decision | Choice | Reason |
|---|---|---|
| Storage format | Delta Lake | ACID transactions, time travel, schema evolution |
| Partitioning | year / month | Matches pipeline cadence, enables partition pruning |
| Idempotency | replaceWhere + MERGE | Safe reruns without duplicates |
| Gold tables | 5 separate tables | Each answers one business question cleanly |
| Watermark | Delta table | Durable, queryable, restartable |

---

## Data Quality Rules — Silver Layer

| Column | Rule | Action on failure |
|---|---|---|
| `passenger_count` | Between 1 and 6 | Row rejected |
| `fare_amount` | Greater than 0 | Row rejected |
| `trip_distance` | Greater than 0 | Row rejected |
| `dropoff > pickup` | Enforced strictly | Row rejected |
| `PULocationID` | Between 1 and 265 | Row rejected |
| Rejection rate | Must be under 5% | Pipeline fails loudly |

---

## Dataset

- **Source:** NYC Taxi & Limousine Commission (TLC)
- **Period:** January – December 2023
- **Volume:** ~37 million trips, ~500MB compressed parquet
- **License:** Public domain
