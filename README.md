# End-to-End Data Pipeline Using Azure Data Factory, ADLS Gen2, Databricks, Synapse, and Power BI
<img width="1806" height="774" alt="Intech Azure Data Engineering Architecture" src="https://github.com/user-attachments/assets/72caf73a-145f-4a5c-b0d9-1434f2a9ed3b" />

## Overview

This project demonstrates a fully dynamic, scalable data ingestion and transformation pipeline using Azure Data Factory (ADF), Azure Data Lake Storage Gen2 (ADLS Gen2), Azure Databricks, Azure Synapse Analytics, and Power BI. The solution automates ingestion from a GitHub repository, applies quality checks, organizes data into bronze/silver/gold layers, and serves curated data to Synapse and Power BI for analytics.

---

## Architecture Summary

1. **ADF Ingestion (HTTP Connector → ADLS Gen2)**

   * ADF ingests datasets directly from a GitHub repository using an **HTTP linked service**.
   * Data is dynamically ingested based on configurations stored in a **JSON file**, enabling easy addition or removal of datasets.
   * Ingested files are stored in a **landing container** in ADLS Gen2.

     * Files are stored in folders named after the filename.
   * Any ingestion failures are automatically redirected to a **failed container**.
   * ADF **monitoring** and a **Logic App** are used to send notifications for each pipeline run.

---

## Bronze Layer (ADF Data Flow)

After landing, ADF executes a **data flow** to process data into the Bronze layer:

* Reads data from the **landing** container.
* Creates subfolders named after each dataset and execution date.
* Adds **three metadata columns** before writing to Bronze for tracking:

  * Ingestion timestamp
  * Source filename
  * Pipeline run ID
* Writes processed raw-format files into the **bronze container**.

---

## Silver Layer (Databricks)

Using Databricks, the pipeline prepares clean, structured, analytics-ready data:

1. Mounts ADLS Gen2 using a **service principal**.
2. Loads raw data from the Bronze layer.
3. Applies thorough **column-by-column data cleaning**, including:

   * Type casting
   * Null handling
   * Standardization
   * Formatting corrections
4. Writes cleaned datasets into the **silver container** in **Delta format**.

---

## Gold Layer (Databricks)

Gold-level data is optimized for analytics and reporting:

* Loads Silver data into a Gold notebook.
* Removes metadata columns used during ingestion.
* Applies **partitioning by geography**, then by **processed time**.
* Writes partitioned Delta files to the **gold container**.

---

## Synapse Integration

* The Gold Delta tables are exposed to **Azure Synapse Analytics**.
* Synapse **views** are created on top of the Gold tables.
* These views serve as the semantic layer for BI tools.

---

## Power BI Reporting

* Power BI connects directly to Synapse views.
* Basic visualizations are built on top of cleaned, curated, and well-modeled Gold data.

---

## Dynamic Dataset Ingestion (JSON Configuration)

The ingestion framework is fully dynamic:

* A JSON configuration file controls which datasets are ingested.
* Adding or removing a dataset is as simple as modifying the JSON.
* ADF pipelines read this configuration at runtime to determine the ingestion list.

Example structure:

```json
{
  "datasets": [
    {
      "name": "dataset1",
      "url": "https://raw.githubusercontent.com/..."
    },
    {
      "name": "dataset2",
      "url": "https://raw.githubusercontent.com/..."
    }
  ]
}
```

---

## Key Features

* Fully dynamic ingestion driven by JSON
* Raw → Bronze → Silver → Gold medallion architecture
* Automated failure handling and notifications
* Metadata tracking from ingestion onward
* Delta Lake storage for versioning, ACID transactions, and performance
* End-to-end integration with Synapse and Power BI

---

## Technologies Used

* **Azure Data Factory** (Pipelines, Data Flows)
* **Azure Data Lake Storage Gen2**
* **Azure Logic Apps**
* **Azure Databricks** (Delta Lake, notebooks)
* **Azure Synapse Analytics** (SQL serverless views)
* **Power BI**
* **GitHub (source data)**

If you want, I can also generate a diagram, folder structure, or add step-by-step instructions.
