# **Azure Data Pipeline Project**

## **Overview**
This project demonstrates how to build a **data pipeline using Azure services**, focusing on **seamless integration between various Azure components**. The pipeline follows these steps:

1. **Data Ingestion**: CSV files are stored in GitHu under `/data/` folder.
2. **Data Storage**: Files are copied from GitHub to **Azure Data Lake Storage (ADLS)**.
3. **Data Processing**: The data is processed using **Databricks**.
4. **Data Warehousing**: Processed data is stored in **Azure Synapse Analytics**.
5. **Data Visualization**: The transformed data is analyzed using **Power BI**.

## **1. Data Ingestion - Storing Data in GitHub**
All the **CSV files** are stored in a **GitHub repository** as the data source. These files will be transferred to Azure for further processing.

### **Reading Files from GitHub in Azure Data Factory**
To copy files from GitHub to ADLS, use **Azure Data Factory (ADF)**.

`using COPY Function`

#### **Source Configuration**
- **Dataset Type**: HTTP
- **Linked Service**: Create a new HTTP linked service pointing to GitHub.
- **Base URL**: `https://raw.githubusercontent.com/<GitHubUsername>/<RepoName>/main/data/`
- **File Path**: `Athletes.csv` (or any other file you want to copy)
- **Request Method**: GET
- **Authentication**: Anonymous (if public repo) or personal access token (if private repo)

#### **Sync (Sink) Configuration - ADLS**
- **Dataset Type**: Azure Data Lake Storage Gen2
- **Linked Service**: Create a new ADLS Gen2 linked service.
- **File Path**: `raw/Athletes.csv`
- **Copy Method**: Binary or Delimited Text (depending on format)

## **2. Data Storage - Azure Data Lake Storage (ADLS)**
The CSV files are stored in **Azure Data Lake Storage (ADLS)**, which is designed for hierarchical storage.

### **Blob Storage vs. ADLS**
- **Azure Blob Storage** is a general-purpose storage for unstructured data.
- **Azure Data Lake Storage (ADLS)** is an advanced version of Blob Storage with **hierarchical namespace**, which allows **folder-based** organization, enhancing performance and security.

### **Storage Tiers in Azure**
- **Hot Storage**: Frequently accessed data, high cost.
- **Cool Storage**: Infrequently accessed data, lower cost.
- **Cold/Archive Storage**: Rarely accessed data, lowest cost.

### **Replication Strategies**
- **LRS (Locally Redundant Storage)**: Data is replicated within a single region.
- **GRS (Geo-Redundant Storage)**: Data is replicated across two geographically distant regions.
- **ZRS (Zone-Redundant Storage)**: Data is replicated across multiple availability zones in a single region.
- **GZRS (Geo-Zone-Redundant Storage)**: Combines ZRS and GRS for enhanced resilience.

### **ADLS Folder Structure**
- `raw/` - Stores unprocessed CSV files.
- `transformed/` - Stores processed and transformed data.

## **3. Data Transfer - Azure Data Factory (ADF)**
We use **Azure Data Factory (ADF)** to copy data from GitHub to **ADLS (raw folder)**.

### **Steps in ADF**
1. **Create a Data Factory instance** in Azure.
2. **Create a Linked Service** to GitHub (using HTTPS).
3. **Create a Linked Service** to ADLS.
4. **Use a Copy Activity** to transfer data from GitHub to ADLS.

## **4. Data Processing - Azure Databricks**
Azure Databricks is used to process the data from ADLS.

### **Connecting Databricks to ADLS**
To authenticate Databricks with ADLS, we use **Azure App Registration**.

#### **Steps for App Registration**
1. **Register an application** in **Azure Active Directory**.
2. **Copy** the `Application (Client) ID` and `Directory (Tenant) ID`.
3. **Generate a Client Secret** and copy the value.
4. **Assign the Storage Blob Data Contributor role** to the app in ADLS.

#### **Databricks Configuration**
In Databricks, we set up configurations to access ADLS:

```python
# Set up the Azure Key Vault-backed secrets
configs = {
    "spark.hadoop.fs.azure.account.auth.type.<ADLS_ACCOUNT_NAME>.dfs.core.windows.net": "OAuth",
    "spark.hadoop.fs.azure.account.oauth.provider.type.<ADLS_ACCOUNT_NAME>.dfs.core.windows.net": "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider",
    "spark.hadoop.fs.azure.account.oauth2.client.id.<ADLS_ACCOUNT_NAME>.dfs.core.windows.net": dbutils.secrets.get(scope="<scope_name>", key="client_id"),
    "spark.hadoop.fs.azure.account.oauth2.client.secret.<ADLS_ACCOUNT_NAME>.dfs.core.windows.net": dbutils.secrets.get(scope="<scope_name>", key="client_secret"),
    "spark.hadoop.fs.azure.account.oauth2.client.endpoint.<ADLS_ACCOUNT_NAME>.dfs.core.windows.net": "https://login.microsoftonline.com/<tenant_id>/oauth2/v2.0/token"
}

# Mount ADLS Gen2 storage to Databricks
dbutils.fs.mount(
  source = "abfss://<container_name>@<adls_account_name>.dfs.core.windows.net/",
  mount_point = "/mnt/adls_mount",
  extra_configs = configs)
```

**Note**: It is recommended to use **Azure Key Vault** for storing secrets securely.

### **Reading Data from ADLS in Databricks**
```python
# Read CSV files from ADLS
raw_df = spark.read.csv("/mnt/adls_mount/raw", header=True, inferSchema=True)
raw_df.show()
```

## **5. Data Transformation**
Using **Apache Spark** in Databricks, we clean and transform the data.

### **Example Transformation**
```python
from pyspark.sql.functions import col

transformed_df = raw_df.filter(col("Age") > 18)

# Writing transformed data to ADLS
transformed_df.write.mode("overwrite").csv("/mnt/adls_mount/transformed")
```

## **6. Data Warehousing - Azure Synapse Analytics**
To analyze the processed data, we store it in **Azure Synapse Analytics**.

### **Steps to Set Up Synapse Analytics**
1. **Create an Azure Synapse Analytics workspace**.
2. **Create a dedicated SQL pool** for high-performance querying.
3. **Use Data Factory to copy transformed data** from ADLS to Synapse.

### **Synapse Pools**
- **SQL Pools**: Used for structured data querying.
- **Spark Pools**: Used for big data processing and machine learning.

## **7. Data Visualization - Power BI**
To analyze the transformed data, we connect **Power BI** to Azure Synapse Analytics.

### **Steps to Connect Power BI**
1. **Open Power BI Desktop**.
2. **Click on Get Data → Azure Synapse Analytics**.
3. **Enter the server details** (found in Azure Synapse SQL pool).
4. **Load the transformed data** and create visualizations.

## **Conclusion**
This project demonstrates how to build a **fully functional data pipeline in Azure**, integrating **Azure Data Lake Storage, Azure Data Factory, Databricks, Synapse Analytics, and Power BI** for end-to-end data processing and analytics.

