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

### Project Architecture

#### Extraction
1. For CDC Method → Extract the Database Logs using Oracle Golden Gate as .csv files and copy them into a sftp server.
2. For Full Load/Refresh Method → In-Database PL/SQL Procedure to extract the full table into one or more .csv files and copy them into a sftp server.
3. 
