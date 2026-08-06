# BigQuery External Tables: Complete Guide

## What Is an External Table?

An external table in BigQuery is a table whose **data lives outside of BigQuery's managed storage** — in Google Cloud Storage, Google Drive (Sheets), Bigtable, or a few other supported sources. BigQuery stores only the table's **metadata and schema**; when you query it, BigQuery reads the data live from the external source at query time.

This is different from a **native table**, where data is physically loaded and stored inside BigQuery's own columnar storage (Capacitor format), fully managed and optimized by BigQuery.

```
Native Table:    Data lives IN BigQuery storage → fast, managed, billed for storage
External Table:  Data lives OUTSIDE BigQuery    → queried live, no storage cost in BQ, but slower
```

---

## Supported External Data Sources

| Source | Format | Notes |
|---|---|---|
| Google Cloud Storage | CSV, JSON, Avro, Parquet, ORC | Most common use case |
| Google Sheets | Native Sheets format | Great for small, human-edited data |
| Google Drive | CSV, JSON, Avro | Similar to GCS but Drive-hosted |
| Bigtable | Bigtable format | For querying Bigtable directly |
| Amazon S3 / Azure Blob (BigQuery Omni) | CSV, JSON, Parquet, etc. | Cross-cloud querying without moving data |

---

## Why Use an External Table?

**Good use cases:**
- Querying data before deciding whether it's worth permanently loading into BigQuery
- Small reference/config/lookup data maintained by humans (e.g., a Google Sheet of category mappings)
- Ad hoc analysis on files sitting in GCS without an ETL step
- Federated queries across clouds (via BigQuery Omni) without duplicating data
- Data that changes very frequently and where "always live" matters more than query speed
- Avoiding storage duplication/costs for infrequently queried data

**Poor use cases:**
- High-frequency queries or dashboards (external tables are much slower — no caching, no BQ-native optimization)
- Large datasets (multi-GB+) queried often — performance and cost degrade
- Production pipelines needing guaranteed schema stability
- When you need partitioning/clustering optimizations (native tables only)
- When query cost predictability matters (external table scans can be less predictable)

---

## Creating an External Table

### 1. From Google Cloud Storage (CSV example)

```sql
CREATE EXTERNAL TABLE `my_project.my_dataset.gcs_external_table`
OPTIONS (
  format = 'CSV',
  uris = ['gs://my-bucket/data/*.csv'],
  skip_leading_rows = 1
);
```

- `uris` supports wildcards (`*`) to match multiple files.
- `format` can be `CSV`, `NEWLINE_DELIMITED_JSON`, `AVRO`, `PARQUET`, `ORC`.
- `skip_leading_rows` skips header rows for CSV.

### 2. From Google Sheets

```sql
CREATE EXTERNAL TABLE `my_project.my_dataset.sheet_external_table`
OPTIONS (
  format = 'GOOGLE_SHEETS',
  uris = ['https://docs.google.com/spreadsheets/d/SHEET_ID/edit#gid=0'],
  sheet_range = 'Sheet1!A1:Z1000',  -- optional
  skip_leading_rows = 1
);
```

- `sheet_range` is optional — omit to read the entire first sheet/tab.
- Multiple URIs can be listed if sheets share identical schema (BigQuery will union them under the hood).

### 3. Using the `bq` CLI

Create a table definition file (`table_def.json`):

```json
{
  "sourceFormat": "CSV",
  "sourceUris": ["gs://my-bucket/data/*.csv"],
  "csvOptions": {
    "skipLeadingRows": 1
  }
}
```

Then run:

```bash
bq mk --external_table_definition=./table_def.json my_dataset.my_table
```

### 4. Using the BigQuery Console (UI)

1. Go to your dataset → **Create Table**
2. Source: choose **Google Cloud Storage**, **Drive**, etc.
3. Select file format
4. Table type: **External table**
5. Configure schema (autodetect or manual) and options
6. Click **Create Table**

---

## Schema Handling

You have two options:

**Autodetect (default/simplest):**
```sql
OPTIONS (
  format = 'CSV',
  uris = ['gs://my-bucket/data/*.csv'],
  autodetect = true
)
```
BigQuery infers column names/types by sampling the source. Convenient, but risky — a stray text value in a numeric column can cause type mismatches or silently wrong inference.

**Explicit schema (recommended for production):**
```sql
CREATE EXTERNAL TABLE `my_project.my_dataset.my_table`
(
  id INT64,
  name STRING,
  created_date DATE
)
OPTIONS (
  format = 'CSV',
  uris = ['gs://my-bucket/data/*.csv'],
  skip_leading_rows = 1
);
```
More reliable, avoids silent schema drift.

---

## Permissions Required

| Source | Requirement |
|---|---|
| GCS | Service account/user needs `storage.objects.get` on the bucket/files |
| Google Sheets | The querying identity (user or service account) needs at least **view access** to the actual sheet in Drive — dataset permissions alone are NOT enough |
| Cross-cloud (Omni) | Requires a BigQuery Omni connection configured with appropriate cloud provider credentials |

A common failure mode: someone can query the BigQuery dataset but gets a permission error, because they don't have Drive access to the underlying sheet.

---

## Querying an External Table

Once created, you query it exactly like a native table:

```sql
SELECT *
FROM `my_project.my_dataset.sheet_external_table`
WHERE created_date > '2026-01-01'
```

You can also `JOIN` or `UNION ALL` external tables with native tables:

```sql
SELECT a.id, a.name, b.category
FROM `my_project.my_dataset.sheet_external_table` a
JOIN `my_project.my_dataset.native_lookup_table` b
  ON a.id = b.id
```

---

## Key Limitations & Gotchas

1. **Performance** — every query re-reads the source live. No BigQuery-native caching, partitioning, or clustering applies. Expect noticeably slower queries versus native tables, especially at scale.
2. **No partition pruning** — for GCS-based external tables, BigQuery must scan more data than it would with a partitioned native table (though Hive-partitioned layouts can help).
3. **Fragility** — if the underlying file/sheet is moved, renamed, deleted, or permissions change, queries fail immediately with no warning beforehand.
4. **Schema drift** — column order/type changes in the source (especially Sheets) can silently break autodetected schemas.
5. **No transactional guarantees** — data can be edited or deleted mid-query since BigQuery doesn't own the storage.
6. **Concurrency limits** — Sheets-based external tables in particular are not designed for high query volume; the Drive API imposes hidden bottlenecks.
7. **Security exposure** — anyone with BigQuery query access effectively gets read access to the live source data (sheet/file), so BQ IAM permissions alone don't fully control access.
8. **Cost model differs** — you don't pay for storage in BigQuery, but you still pay for query bytes processed depending on the source, and repeated reads from GCS/Sheets aren't cached the way native table scans can be.

---

## Common Pattern: External Table as a Staging Layer (with dbt)

A popular and pragmatic pattern is to use external tables purely as an ingestion/staging layer, then materialize a cleaned, typed, native table downstream using dbt (or scheduled queries):

```
Google Sheet / GCS file
        │
        ▼
External Table (staging, live read)
        │
        ▼
dbt model: SELECT with explicit CAST/SAFE_CAST + tests
        │
        ▼
Native materialized table (fast, validated, historized)
```

Example dbt staging model:

```sql
-- models/staging/stg_external_source.sql
SELECT
  SAFE_CAST(id AS INT64) AS id,
  TRIM(name) AS name,
  SAFE_CAST(created_date AS DATE) AS created_date,
  CURRENT_TIMESTAMP() AS _loaded_at
FROM {{ source('raw_external', 'sheet_external_table') }}
```

This gets you:
- Convenience of external tables (no custom ingestion code)
- Reliability of native tables downstream (fast queries, schema enforcement, dbt tests for data quality)
- A clear failure point (dbt test failures) instead of silent data corruption

---

## External Table vs Native Table — Quick Comparison

| | External Table | Native Table |
|---|---|---|
| Data location | Outside BigQuery (GCS, Drive, etc.) | Inside BigQuery managed storage |
| Query speed | Slower (live read each time) | Fast (optimized columnar storage) |
| Storage cost in BQ | None | Billed per GB stored |
| Setup effort | Minimal | Requires load/ingestion step |
| Data freshness | Always current (reads live) | As fresh as last load |
| Schema stability | Can drift, fragile to source changes | Enforced, stable |
| Partitioning/clustering | Not supported (GCS Hive-partitioning is a partial exception) | Fully supported |
| Best for | Prototyping, small reference data, ad hoc analysis | Production pipelines, dashboards, frequent queries |

---

## Summary

External tables are best treated as a **convenient entry point** for data BigQuery doesn't own — useful for quick access, prototyping, and human-edited reference data like Google Sheets. For anything powering production reporting, dashboards, or frequent queries, materialize the data into a native table (ideally via a transformation layer like dbt) to get better performance, reliability, and schema control.