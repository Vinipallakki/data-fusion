# **GCP Pipeline: Cloud SQL → Data Fusion → BigQuery**

## **Overview**
This project demonstrates a simple ETL pipeline using Google Cloud Platform (GCP) services. Data is extracted from a Cloud SQL database, transformed using Cloud Data Fusion, and loaded into BigQuery for analysis.

---

## **Architecture**
1. **Source**: Cloud SQL (MySQL/PostgreSQL)
2. **Transform**: Cloud Data Fusion
   - Transformation logic applied to the data.
3. **Sink**: BigQuery

---

## **Project Structure**
The pipeline consists of the following stages:
1. **Batch Source**:
   - Extracts data from a Cloud SQL database.
2. **Transformation**:
   - Uses Data Fusion Wrangler for data cleaning and transformations.
3. **Sink**:
   - Loads the transformed data into a BigQuery table.

---

## **Prerequisites**
Before running this pipeline, ensure the following:
1. A GCP project is set up.
2. Cloud SQL instance is configured with a sample dataset.
3. BigQuery dataset and table are created for storing transformed data.
4. Cloud Data Fusion is enabled and a workspace is ready.

---

## **Steps to Set Up the Pipeline**

### 1. **Cloud SQL Configuration**
- Create a Cloud SQL instance (MySQL/PostgreSQL).
- Import a sample dataset into your Cloud SQL instance. Example SQL to populate the `employees` table:
  ```sql
  CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    department VARCHAR(100),
    salary DECIMAL(10, 2)
  );

  INSERT INTO employees (employee_id, first_name, last_name, department, salary)
  VALUES (1, 'John', 'Doe', 'Engineering', 75000.00),
         (2, 'Jane', 'Smith', 'HR', 65000.00);
  ```
- Note the instance connection name (e.g., `project-id:region:instance-id`).

### 2. **Cloud Data Fusion Pipeline**
- Open the Cloud Data Fusion UI.
- Create a new pipeline.
- Add stages:
  - **Source**: CloudSQL Batch Source plugin.
    - Configure the plugin with the Cloud SQL instance connection name, database name, and query:
      ```sql
      SELECT * FROM employees WHERE $CONDITIONS;
      ```
    - Set `numSplits` for parallelism.
  - **Transformation**: Wrangler Transform plugin.
    - Apply data cleansing or transformations (e.g., add computed columns or filter rows).
  - **Sink**: BigQuery Sink plugin.
    - Configure with the BigQuery dataset and table name.

- Validate and deploy the pipeline.

### 3. **BigQuery Configuration**
- Create a dataset (e.g., `employee_data`).
- Create a table (e.g., `transformed_employees`) with a schema matching the transformed data.

---

## **Pipeline Configuration**

### Data Fusion Pipeline JSON
Example pipeline configuration (as JSON):
```json
{
  "stages": [
    {
      "id": "CloudSQL-MySQL-Source",
      "name": "CloudSQL MySQL Source",
      "type": "batchsource",
      "properties": {
        "connectionName": "project-id:region:instance-id",
        "database": "employees_db",
        "importQuery": "SELECT * FROM employees WHERE $CONDITIONS;",
        "numSplits": "4",
        "user": "username",
        "password": "password"
      }
    },
    {
      "id": "Wrangler",
      "name": "Wrangler Transform",
      "type": "transform",
      "properties": {
        "field": "*",
        "schema": "{\"type\":\"record\",\"name\":\"outputSchema\",\"fields\":[{\"name\":\"employee_id\",\"type\":\"int\"},{\"name\":\"first_name\",\"type\":[\"string\",\"null\"]},{\"name\":\"last_name\",\"type\":[\"string\",\"null\"]},{\"name\":\"department\",\"type\":[\"string\",\"null\"]},{\"name\":\"salary\",\"type\":[{\"type\":\"bytes\",\"logicalType\":\"decimal\",\"precision\":10,\"scale\":2},\"null\"]}]}"
      }
    },
    {
      "id": "BigQuery-Sink",
      "name": "BigQuery Sink",
      "type": "batchsink",
      "properties": {
        "referenceName": "bigquery_sink",
        "dataset": "employee_data",
        "table": "transformed_employees"
      }
    }
  ],
  "connections": []
}
```

---

## **Running the Pipeline**
1. Deploy the pipeline in Cloud Data Fusion.
2. Start the pipeline and monitor the progress in the Data Fusion UI.
3. Verify the results in BigQuery.

---

## **Result**
- **Input (Cloud SQL)**: The raw data from the `employees` table.
- **Output (BigQuery)**: The transformed data in the `transformed_employees` table.

Example transformed output:
| employee_id | first_name | last_name | department    | salary   |
|-------------|------------|-----------|---------------|----------|
| 1           | John       | Doe       | Engineering   | 75000.00 |
| 2           | Jane       | Smith     | HR            | 65000.00 |

---

## **Future Enhancements**
- Automate pipeline execution using Cloud Scheduler.
- Add error handling and logging in Data Fusion.
- Enable streaming data ingestion for real-time updates.

---

## **Contributing**
Contributions are welcome! Feel free to submit issues or pull requests.
