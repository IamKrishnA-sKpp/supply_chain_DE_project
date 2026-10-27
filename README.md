## supply_chain_DE_Project

📌 Extraction, Transformation & Loading Supply Chain Data.

### 🏭 Core Business Model
Company Name : 'Enigma'
1. Enigma is an Electronics Manufacturing services company which provides full system assembly, printed circuit board assembly (PCBA), final product testing.
2. It gets business from various customers globally to manufacture the required electronic component.
3. To manufacture an Electronic component based on customers requirement(OM[Order Management]), Enigma procures various parts from various vendors/suppliers.
4. To procure these parts, Enigma raises multiple Purchase Orders(PO) to vendors based on the price and required quantity.
5. Procured/Received(RCV) parts will be coming to Inventory(INV) for Quality(QA) Inspection and defected parts will be rejected.
6. After Quality Inspection, parts will be moved to Warehouse(WMS[Warehouse Management System]) for picking by shopfloor or production(WIP[Work In Progress]).
7. Business users raises BOM reports for getting exact number of parts required at various levels.
8. Business users will also run DRP/MRP(Distribution Requirement Planning / Material Requirement Planning) reports for forecasting material requirements and production schedules based on actual demand.
9. From the procurement to Production and shipping, each part movement at various BOM levels(Part Level 0 to n & Finished Good) will be tracked using MTL Tables under Inventory(INV).
10. Once the Finished Good is ready, again a quality inspection(QA) will be done and it will be ready for shipping(WSH[Warehouse Shipping]) to the customer.
11. For every finished good that is shipped to customer, an SO([Sales Order]) would be raised to the customer.
12. All the finance transactions from vendors to customers will be tracked by various Finance modules like General Ledger (GL), Accounts Payable (AP), Accounts Receivable (AR) etc..

- Note that there will be multiple tables under each schema/module, we will be extracting most of the tables for reporting purpose either CDC/Full load method.    

## 

### 🛠️ Tech Stack
- Source : Oracle ERP Database
- Cloud Platform : Microsoft Azure
- Orchestration : Azure Data Factory(ADF)
- Cloud Storage : Azure Delta Lake Storage(ADLS Gen2)
- Compute Engine : Databricks (Apache Spark)
- File Formats : csv → parquet → Delta
- Target : Snowflake DWH
- Programming Language : Python(Pyspark) & SQL
- Version Control : GitLab
- Project Tracking & Ticketing : Enigma PlanView & SR Tool

##

## 📁 Project Architecture & Flow : Bronze → Silve → Gold (Medallion Architecture)

### ✅ STEP 1 : Data Extraction from Source to Bronze Layer

#### Flow of Steps ⬇️
1. For CDC Method → Extract the Database Logs using Oracle Golden Gate as .csv files and copy them into a sftp server.
2. For Full Load/Refresh Method → In-Database PL/SQL Procedure to extract the full table into one or more .csv files and copy them into a sftp server.
3. sftp server will act as a staging between the Source Oracle Database and the Bronze Layer Storage in ADLS.
4. All the .csv files under a particular location in the sftp server will be copied to the bronze/raw_data under 'enigmastoragxx.enigma-container' storage account in ADLS.
5. File naming convention will be as '<source_system>_<schema>_<table>_<extract_timestamp>_<batch_id>.csv' which is very critical for dataloading into particular tables.
6. Once the files are loaded into the raw_data folder, we will now convert the .csv files into .parquet files and move them into the folder bronze/processed_data.
7. Oracle Database logs would be read by the Oracle Golden Gate tool and generates the .csv files directly under a Filesystem in sftp server hosted in Azure Virtual Machine.
8. The .csv files generated under the filesystem would be copied to bronze/raw_data using 'COPY ACTIVITY' under Azure Data Factory.
9. For this activity we will have two Linked Services "Source → SFTP Linked Service(Azure VM)"  <--->  "Sink  → Azure Data Lake Storage Gen2 Linked Service".
10. Once the 1st 'COPY ACTIVITY' completes, we will initiate another 'COPY ACTIVITY' to convert the .csv files under bronze/raw_data into .parquet files into bronze/processed.

<br>
  
```text
Data Extraction from Source to Bronze Layer
│
└── adf/
   ├── linked_services/
   │   ├── LS_SFTP_Source.json           # SFTP server on Azure VM
   │   └── LS_ADLS_Gen2_Bronze.json      # ADLS Gen2 Bronze storage
   │
   ├── datasets/
   │   ├── DS_SFTP_Source_CSV.json       # Source CSV dataset
   │   └── DS_ADLS_Bronze_CSV.json       # Destination CSV dataset
   │
   └── activities/
       ├── COPY ACTIVITY 1          #Source sftp to bronze raw_data
       │      ├── Source/ (etlsftp.enigma.com)                            # Source Server
       │      │         └── /etl_data/                                    # Source Filesystem
       │      │                ├── orders_20261027T153000Z_batch001.csv
       │      │                ├── products_20261027T153000Z_batch001.csv
       │      │                ├── customers_20261027T153000Z_batch001.csv
       │      │                └── inventory_20261027T153000Z_batch001.csv
       │      │
       │      └── Sink/ (ADLS : enigmastoragxx.enigma-container)          # Sink Storage
       │                 └── /bronze/raw_data/                            # Sink Filesystem
       │                        ├── orders_20261027T153000Z_batch001.csv
       │                        ├── products_20261027T153000Z_batch001.csv
       │                        ├── customers_20261027T153000Z_batch001.csv
       │                        └── inventory_20261027T153000Z_batch001.csv
       │
       └── COPY ACTIVITY 2          #csv to parquet
                ├── Source/ (ADLS : enigmastoragxx.enigma-container)               # Source Storage
                │         └── /bronze/raw_data/                                    # Source Filesystem
                │                ├── orders_20261027T153000Z_batch001.csv
                │                ├── products_20261027T153000Z_batch001.csv
                │                ├── customers_20261027T153000Z_batch001.csv
                │                └── inventory_20261027T153000Z_batch001.csv
                │
                └── Sink/ (ADLS : enigmastoragxx.enigma-container)          # Sink Storage
                        └── /bronze/processed/                              # Sink Filesystem
                              ├── orders_20261027T153000Z_batch001.parquet
                              ├── products_20261027T153000Z_batch001.parquet
                              ├── customers_20261027T153000Z_batch001.parquet
                              └── inventory_20261027T153000Z_batch001.parquet
```
##

### ✅ STEP 2 : Data Loading from Bronze to Silver with Data Quality Checks
