### Data Pipeline Overview

This project implements a full data pipeline that ingests datasets from a GitHub repository and processes them through multiple layers in Azure until they are ready for analytics in Power BI. The pipeline is designed to be dynamic, so adding or removing datasets only requires updating a JSON configuration file.

Ingestion

The process starts in Azure Data Factory, where datasets are pulled from GitHub using HTTP. Each file is stored in the Landing container in ADLS Gen2. The folder structure is based on the file name, making it easy to track each dataset as it arrives.
If a file can't be ingested or has issues, it is automatically sent to a separate Failed container.
Monitoring is built in using Azure Monitor and Logic Apps, which send notifications for every pipeline run, whether successful or not.

Bronze Layer

ADF Data Flows read the raw Landing data and load it into the Bronze layer. Data is stored under a folder named after the dataset, followed by the date of ingestion (e.g., dataset_name/2025-01-01/).
Before writing to Bronze, the flow adds three metadata fields so I can track when and how each file was ingested.

Silver Layer

From there, Databricks takes over. A Service Principal is mounted to give Databricks access to ADLS.
The Bronze data is loaded, and each dataset goes through a thorough cleaning process, handling issues column by column. Once cleaned, the data is written to the Silver container in Delta format, which makes it more efficient for downstream processing.

Gold Layer

For the Gold layer, another Databricks notebook reads the Silver tables, removes the metadata added earlier, and partitions the data by geography and processed time.
The partitioned data is then exposed to Azure Synapse, where I create views that will be used for reporting.

Reporting

Synapse connects directly to Power BI. From there, I build the visualizations using the cleaned and partitioned Gold layer data.
