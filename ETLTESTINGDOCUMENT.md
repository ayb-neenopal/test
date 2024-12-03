# ETL Testing Guidelines  

Testing ETL (Extract, Transform, Load) pipelines ensures the accuracy, reliability, and quality of data as it moves from source systems to the target data store. ETL pipelines are critical for data-driven decision-making, as they process, clean, and transform raw data into meaningful formats.  
![image](https://github.com/user-attachments/assets/8eeee961-201c-42c4-ba3c-b6d248cdf133)
![image](https://github.com/user-attachments/assets/00a640d2-0735-436d-836b-a886425614e9)


ETL testing verifies that:  
- Data is correctly extracted from source systems.  
- Transformations align with business requirements.  
- Data loaded into the target system is accurate and complete.  

---

## Before Making a Data Pipeline Testing Strategy  

The following things should be known and documented well:  

### 1. Identify Key Components of Your Data Pipeline  
Write down each stage of your data pipeline and functions involved.  

**For example:**  

- **Data Ingestion**  
  - Functions: `fetch_from_db()`, `fetch_from_api`  
  - Sources: Databases, APIs, flat files (e.g., CSV)  

- **Data Transformation**  
  - Functions: `clean_data()`, `aggregate_data()`, `apply_business_rules()`  
  - Transformations: Cleaning, filtering, aggregating, applying business logic  

- **Data Loading**  
  - Functions: `load_to_db()`, `write_to_csv()`  
  - Destinations: Database tables, data warehouse, or file storage  

After noting down these components, you'll be able to identify the scope of each testing type you need to apply.  

---

### 2. Define Objectives for Each Testing Stage  

For each component you’ve identified, define what a test should achieve.  

#### **Data Ingestion Objectives**  
- Verify data is loaded correctly from each source.  
- Ensure data is in the expected format (e.g., CSVs have correct columns).  

#### **Data Transformation Objectives**  
- Confirm that transformations yield expected outputs for known inputs.  
- Check that business rules are applied correctly (e.g., specific calculations or filters).  

#### **Data Loading Objectives**  
- Validate that data is correctly written to the target storage without data loss.  
- Ensure data in the target matches the schema and expected format.  

---

### Data Quality Matrix  

- **Functional Test**  
- **Source Test**  
- **Flow Test**  
- **Contract Test**  
- **Component Test**  
![image](https://github.com/user-attachments/assets/09d360a2-b55f-45e2-be67-4e967b52f767)

---

## Types of Tests  

### **Unit Tests**  
Validate each logical unit or function that is part of the ETL process. If the pipeline consists of a group of transformations, those can be tested separately using a set of input values and expected output.  

- **Purpose**: Tests individual units of software.  
- **Trigger**: Manual and Pull request (automated).  
- **Tools**: `unittest`, `pytest`.  

#### How to Set Up a Unit Test  
1. **Arrange**: Prepare dummy data into a DataFrame.  
2. **Act**: Pass that data into the transformation function.  
3. **Assert**: Match the output with the expected data.  

#### Automating the Test  
1. With a CI provider, set up an agent that can automatically execute tests.  
2. Host it on AWS/Azure, etc.  
3. Manage dependencies using Docker.  
4. Write a CI pipeline using `.yml` and set a trigger on Pull Requests to test all tests before committing.  

---
# Writing Unit Test in PySpark

## Testing a DataFrame Transformation

### Step 1: Sample CSV Data

Consider the following CSV file `animal_info.csv`:


### Step 2: Expected DataFrame Output

After applying the transformation, the resulting DataFrame should look like this:

| Species  | Family     |
|----------|------------|
| Bear     | Carnivore  |
| Monkey   | Primate    |
| Fish     | null       |

### Step 3: Create a Data Transformation Function

Write a transformation function `clean_animals` to split the `animal_info` column into `species` and `family`:

```python
from pyspark.sql.functions import split, col

def clean_animals(df):
    parts = split(col("animal_info"), "\&")
    return df.select(
        parts[0].alias("species"),
        parts[1].alias("family"),
    )
### Step 4: Write the Unit Test

Now, let's write the unit test for the `clean_animals` function. The test will check whether the transformation produces the expected output.

```python
def test_clean_animals(spark):
    # Step 1: Read the input CSV file into a DataFrame
    df = spark.read.option("header", True).csv("/tests/resources/animals.csv")
    
    # Step 2: Apply the transformation function to the DataFrame
    actual_df = df.transform(clean_animals)
    
    # Step 3: Define the expected output DataFrame
    expected_data = [("bear", "carnivore"), ("monkey", "primate"), ("fish", None)]
    expected_df = spark.createDataFrame(expected_data, ["Species", "family"])

    # Step 4: Assert that the actual DataFrame equals the expected DataFrame
    assert_df_equality(actual_df, expected_df)
```
### Explanation of the Unit Test

1. **Read the Input CSV**: 
   The test reads a CSV file (`animals.csv`) containing animal information into a DataFrame.

2. **Apply the Transformation**: 
   The test applies the `clean_animals` function to split the `animal_info` column into two columns: `species` and `family`.

3. **Define Expected Data**: 
   The expected output is defined as a list of tuples, where each tuple represents a row of the expected DataFrame. For example, it expects `("bear", "carnivore")`, `("monkey", "primate")`, and `("fish", None)` as the output.

4. **Compare DataFrames**: 
   The test uses the `assert_df_equality` function to compare the transformed DataFrame (`actual_df`) with the expected DataFrame (`expected_df`). If both DataFrames are identical, the test passes. Otherwise, it fails.


## Testing a Column Function

### Step 1: Start with the column function

```python
def life_stage(col):
    return (
      F.when(col < 13, "child")
      .when(col.between(13, 19), "teenager")
      .when(col > 19, "adult")
    )
```

### Step 2: Create a DataFrame with the expected return value
```python
df = spark.createDataFrame(
  [
    ("karen", 56, "adult"),
    ("jodie", 16, "teenager"),
    ("jason", 3, "child"),
    (None, None, None),
  ]
).toDF("first_name", "age", "expected")
```

### Step 3: Run the life_stage function to get the actual value
```python
res = df.withColumn("actual", life_stage(F.col("age")))
```

### Step 4: Use Chispa to confirm actual and expected match
(Chispa is a testing library for PySpark)

```python
import chispa
chispa.assert_column_equality(res, "expected", "actual")
```

**We can also test the queries by using pyspark and no other library by assert function in the same way **

# Unit Testing a Query with PySpark

**step 1. Query to Test**

      SELECT * FROM my_table WHERE amount > 30.0

**Step 2: parameterize query**

      query = "SELECT * FROM (df) WHERE amount > {amount}"

**Step 3: create a sample dataset**
```python
      from pyspark.sql import SparkSession
      import datetime

       # Initialize Spark session
       spark = SparkSession.builder.appName("UnitTestQuery").getOrCreate()

       # Sample dataset
       df = spark.createDataFrame(
      [
        ("socks", 7.55, datetime.date(2022, 5, 15)),
        ("handbag", 49.99, datetime.date(2022, 5, 16)),
        ("shorts", 35.00, datetime.date(2023, 1, 5)),
        ("socks", 25.08, datetime.date(2023, 12, 23)),
      ],
      ["item", "amount", "purchase_date"],
      )
```
**Step 4: Create an expected result**
```python
     expected_df = spark.createDataFrame(
       [
         ("handbag", 49.99, datetime.date(2022, 5, 16)),
         ("shorts", 35.00, datetime.date(2023, 1, 5)),
       ],
       ["item", "amount", "purchase_date"],
     )
```
**Step 5: make an assertion**
```python
       from pyspark.testing.sqlutils import **assertDataFrameEqual**

       # Execute the query with parameterized value
       actual_df = spark.sql(query.format(amount=30.0), df)

       # Compare actual and expected DataFrames
       assertDataFrameEqual(actual_df, expected_df)
```

---

### **Data Quality Tests**  

- **Check the Data Model**  
  - Are all key-references valid?  

- **Check Invariants**  
  - Is the input and output as expected?  
  - Was the input cleaned and sanitized as expected?  

- **Run Statistical Checks**  
  - Did the distribution of the values change?  
  - Did the number of orphaned records change?  

#### How to Perform Data Quality Tests  

**Testing in Pipeline**  
- At the end of each pipeline, add a validation task:  
  - Validate your data models.  
  - Assert invariants like non-null, duplicates, counts.  

**Testing Outside the Pipeline**  
- Run a dedicated job checking for distributions, input/output counts, and report deviations.  

**Tools**  
- Soda Core  
- Great Expectations  
- DBT  

---
### *Example of data quality test**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

# Initialize Spark session
spark = SparkSession.builder.appName("ETL Data Quality Test").getOrCreate()

# Sample data with missing values
data = [(1, "John", "2023-01-01"), (2, "Jane", None), (3, "Doe", "2023-02-01"), (4, None, "2023-03-01")]
columns = ["ID", "Name", "Created_At"]
df = spark.createDataFrame(data, columns)

# Data Quality Test: Check for missing values
def data_quality_check(df):
    missing_data = df.filter(col("Name").isNull() | col("Created_At").isNull())
    if missing_data.count() > 0:
        print(f"Test Failed: Found {missing_data.count()} rows with missing values.")
        missing_data.show()
    else:
        print("Test Passed: No missing values.")

# Run test
data_quality_check(df)
```



### **Integration Tests**  
Integrating tests in data pipelines ensures that the entire data workflow—from ingestion, through transformation, to final reporting—works as expected and meets business requirements.  

- **In-Pipeline Tests**: Immediately after data transformations.  
- **Outside-Pipeline Tests**: Periodic validation checks on overall data consistency.  

**Tools**  
- Soda Core  
- Great Expectations  
- DBT  
- Airflow  

---

### Example of integration test
```python
# Importing necessary PySpark modules
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

# Step 1: Initialize Spark session
# This is where we initialize a Spark session which will allow us to interact with Spark and run operations on data.
spark = SparkSession.builder.appName("ETL Integration Test").getOrCreate()

# Step 2: Sample data for extraction (Source)
# Here, we create a sample source data which simulates the data we would extract from a source (e.g., database, API, file).
# The source data consists of two rows with columns: ID, Name, and Created_At.
source_data = [(1, "John", "2023-01-01"), (2, "Jane", "2023-01-02")]
source_columns = ["ID", "Name", "Created_At"]
source_df = spark.createDataFrame(source_data, source_columns)

# Step 3: Transformation
# In this step, we simulate a transformation on the extracted data. We are adding a new column "Status" based on a condition.
# The transformation checks if the "Name" column contains "John" and assigns the string "1" to the "Status" column if true.
transformed_df = source_df.withColumn("Status", col("Name").rlike("John").cast("string"))

# Step 4: Load data (In-memory simulation of target)
# This is the simulated target of the ETL pipeline. Here, we simply use the transformed dataframe as the target.
# In a real scenario, this would be the step where data is loaded into the target system (e.g., database, data warehouse).
target_df = transformed_df

# Step 5: Integration Test - Check if transformation is correctly applied
# Now, we perform the integration test. We want to verify that the transformation works correctly.
# Specifically, the "Status" column should be "1" for the rows where the "Name" is "John".
def integration_test(source_df, target_df):
    # The filter condition checks if the "Status" column is "1", indicating that the transformation was applied correctly.
    transformed_rows = target_df.filter(col("Status") == "1")
    
    # If there are rows with the correct transformation, the test passes; otherwise, it fails.
    if transformed_rows.count() > 0:
        print(f"Test Passed: {transformed_rows.count()} row(s) correctly transformed.")
    else:
        print("Test Failed: Transformation not applied correctly.")

# Step 6: Run the test
# We invoke the integration_test function, passing both the source and transformed (target) dataframes.
integration_test(source_df, target_df)
```

### **Performance Tests**  
Assesses the resource utilization and scalability of the pipeline. This is crucial for high-volume data pipelines to meet the required SLAs.  


### Example of Performance test
```python
# Importing necessary PySpark modules
from pyspark.sql import SparkSession
from pyspark.sql.functions import col
import time

# Step 1: Initialize Spark session
# The Spark session allows us to work with data in a distributed manner, which is essential for performance testing of large datasets.
spark = SparkSession.builder.appName("ETL Performance Test").getOrCreate()

# Step 2: Sample large data for extraction (Source)
# We simulate a large dataset (for testing performance) that consists of multiple rows.
# Here, we create 100,000 rows as a sample large dataset.
source_data = [(i, f"Name_{i}", f"2023-01-{i%31 + 1:02d}") for i in range(1, 100001)]
source_columns = ["ID", "Name", "Created_At"]
source_df = spark.createDataFrame(source_data, source_columns)

# Step 3: Transformation (Add a new column for testing)
# We simulate a transformation where we add a new column "Status" based on a condition.
# The transformation checks if the "Name" column contains "Name_1" and assigns "Active" to the "Status" column if true.
transformed_df = source_df.withColumn("Status", col("Name").rlike("Name_1").cast("string"))

# Step 4: Performance Test - Measure Transformation Time
# We perform a performance test by measuring the time it takes for the transformation step to execute.
def performance_test(df):
    start_time = time.time()  # Start measuring time
    transformed_df = df.withColumn("Status", col("Name").rlike("Name_1").cast("string"))
    end_time = time.time()  # End measuring time
    
    # Calculate the time taken for the transformation
    execution_time = end_time - start_time
    print(f"Performance Test: Transformation took {execution_time:.4f} seconds.")
    
    # Define a performance threshold (e.g., transformation should complete in less than 5 seconds)
    if execution_time < 5:
        print("Test Passed: Transformation completed within acceptable time limit.")
    else:
        print("Test Failed: Transformation took too long.")

# Step 5: Run the Performance Test
# We invoke the performance_test function, passing the source DataFrame to check the time taken for transformation.
performance_test(source_df)
```
---

### **End-to-End Tests**  
Tests the data pipeline as a whole, from the source to the target or output. This could be categorized as “black box” testing because there is no need to know the internal structure of the pipeline, only the expected output given the input.  
### Example end-to-end test
```python
# Importing necessary modules
from pyspark.sql import SparkSession
from pyspark.sql.functions import col
import pandas as pd

# Step 1: Initialize Spark session
spark = SparkSession.builder.appName("ETL End-to-End Test").getOrCreate()

# Step 2: Simulate data extraction (Source)
# Simulating a small dataset as source data, typically this data would come from a database or a file.
source_data = [(1, "John", "2023-01-01"), (2, "Jane", "2023-01-02")]
source_columns = ["ID", "Name", "Created_At"]
source_df = spark.createDataFrame(source_data, source_columns)

# Step 3: Data Transformation
# Applying transformation logic (For example: converting "Name" to uppercase)
transformed_df = source_df.withColumn("Name", col("Name").upper())

# Step 4: Data Loading (Simulated as in-memory target for testing)
# Loading data to a simulated target system (can be a database, file, etc.)
target_df = transformed_df  # In practice, this would be a database write operation

# Step 5: End-to-End Test - Validate the entire pipeline
def end_to_end_test(source_df, target_df):
    # Validate Extraction: Check if source data is correct
    assert source_df.count() == 2, "Source data extraction failed."

    # Validate Transformation: Check if "Name" column is properly transformed
    transformed_rows = target_df.filter(col("Name") == "JOHN")
    assert transformed_rows.count() == 1, "Transformation failed. 'Name' was not transformed to uppercase."

    # Validate Load: Check if the data exists in the target
    target_count = target_df.count()
    assert target_count == 2, f"Data load failed. Expected 2 rows, found {target_count} rows in target."

    print("End-to-End Test Passed: Data pipeline works correctly from extraction to loading.")

# Step 6: Run the End-to-End Test
end_to_end_test(source_df, target_df)
```

---

### **Functional Tests**  
Tests individual components like task groups in isolation.  

- **No mocks**.  
- **Catches more errors**.  
- **Faster** as it does not include any orchestration layer.  
- **Tools**: `pytest`, `coverage`.  

---

## Functional Test Example Code  

```python
def test_crm_users_entity(self, snowflake_executor: SnowflakeExecutor, task_factory: TaskFactory, hubspot: HubspotSchemaTransformTaskFactory) -> None:
    # Setup
    with snowflake_executor.connection() as c:
        c.execute_sql(
            """
            CREATE OR REPLACE TRANSIENT TABLE HUBSPOT.CONTACT AS SELECT 1 AS ID, '2022-01-01' AS CREATED_AT
            """
        )

    task = task_factory.query_task("hubspot_create_crm_entity", hubspot.create_crm_entity(entity=Entity.USERS))
    with snowflake_executor.connection() as c:
        c.execute(task)

    # Validate
    with snowflake_executor.connection() as c:
        actual = c.fetch_pandas_df("SELECT * FROM HUBSPOT.CONTACT")
        expected = pd.DataFrame({"ID": [1], "CREATED_AT": [date_parse("2022-01-01")]})
        pd.testing.assert_frame_equal(actual, expected)

```
## Key Details of What It Is Testing

### The test is checking the entire flow from creating a table in Snowflake (CREATE OR REPLACE TRANSIENT TABLE HUBSPOT.CONTACT), executing a task (hubspot_create_crm_entity), and validating the final data (SELECT * FROM HUBSPOT.CONTACT).

### Task Logic
- Tests the behavior of the task `hubspot_create_crm_entity`, ensuring it processes and transforms the data correctly for `Entity.USERS`.

### Database Interaction
- Verifies whether the task interacts properly with the Snowflake database, including table creation, updates, or inserts.

### Data Consistency
- Ensures the data in the `HUBSPOT.CONTACT` table after task execution matches the expected values.

### Isolated Test Data
- The SQL query creates a transient table with mock data (`ID=1`, `CREATED_AT='2022-01-01'`). This is not production data; it is a sample dataset crafted specifically for the test.

### No Impact on Production
- The test does not involve or affect production data. The transient table exists only during the test and is automatically removed afterward.

---
# Explanation

### Setup
- Creates a transient table (`HUBSPOT.CONTACT`) in the Snowflake database with test data (`ID=1` and `CREATED_AT='2022-01-01'`).

### Execution
- Uses the `task_factory` to create a task (`hubspot_create_crm_entity`) for processing a CRM user entity and executes this task in Snowflake.

### Validation
- Fetches the data from the `HUBSPOT.CONTACT` table after execution.
- Compares it with the expected data using Pandas' `assert_frame_equal`, ensuring that the output matches the expected result.

---

## How Testing is Being Done

### Setup the Test Environment
- A transient table is created in Snowflake with predefined test data. This ensures a controlled and isolated environment for testing.

### Execute the Task
- The task (`hubspot_create_crm_entity`) is dynamically created and executed in Snowflake, simulating a real-world scenario.

### Validate Results
- Data from the table after task execution is fetched into a Pandas DataFrame (`actual`).
- An expected DataFrame is defined with the anticipated result.
- `pd.testing.assert_frame_equal` ensures:
  - Data values and structure (columns, types) are identical.
  - Any discrepancies cause the test to fail, highlighting issues.

---



# Tools and Libraries for Testing

## **Pytest**
Pytest is a powerful testing framework for Python, widely used for testing data pipelines. It helps automate the validation of data processing, transformation, and flow through the pipeline, ensuring correctness and reliability.

### Key Features of Pytest
1. **Easy Test Discovery**
   - Pytest automatically finds and runs tests in files that match the `test_*.py` or `*_test.py` naming convention.

2. **Fixtures**
   - Reusable components (like mock data, database connections, etc.) that help set up and tear down the test environment.

3. **Assertions**
   - Pytest supports native Python assertions and provides better error messages when tests fail.

4. **Parameterization**
   - Allows running a test with multiple sets of data, making it easier to test the same logic under different scenarios.

5. **Plugins**
   - Pytest has a wide range of plugins that extend its capabilities, including plugins for database testing, mock data generation, and performance testing.

---

## Integrating Pytest with ETL

### **The File Structure**
There are two main files:
- One contains the ETL functions that extract tables from a SQL Server database and load them into PostgreSQL.
- The second contains the test functions that Pytest will find and execute.
- ****![image](https://github.com/user-attachments/assets/5eb7dc85-3458-4f35-aaf0-4db0884d429f)


#### **ETL Code**
```python
# ETL Code
#import needed libraries
from sqlalchemy import create_engine
import pandas as pd
import os
from sqlalchemy.engine import URL

# Get password and username from environment variables
pwd = os.environ['PGPASS']
uid = os.environ['PGUID']
uid1 = 'postgres'
pass1 = '####'
# SQL Server DB connection details
driv = "{ODBC Driver 11 for SQL Server}"  # Updated ODBC Driver
serv = "localhost\SQLEXPRESS"  # Use localhost or "localhost\SQLEXPRESS" for named instance
datab = "AdventureWorksDW2019"
serv2 = "localhost"

# Extract data from SQL Server
def extract():
    try:
        connection_string = 'DRIVER=' + driv + ';SERVER=' + serv + ';DATABASE=' + datab + ';UID=' + uid + ';PWD=' + pwd
        connection_url = URL.create("mssql+pyodbc", query={"odbc_connect": connection_string})
        src_conn = create_engine(connection_url)
        tbl_name = "DimProduct"
        # Query and load/save data to dataframe
        df = pd.read_sql_query(f'select * FROM {tbl_name}', src_conn)
        return df, tbl_name
    except Exception as e:
        print("Data extract error: " + str(e))

# Load data to PostgreSQL
def load(df, tbl):
    try:
        rows_imported = 0
        engine = create_engine(f'postgresql://{uid1}:{pass1}@{serv2}:5432/AdventureWorks')
        print(f'importing rows {rows_imported} to {rows_imported + len(df)}... for table {tbl}')
        # Save df to PostgreSQL
        df.to_sql(f'stg_{tbl}', engine, if_exists='replace', index=False)
        rows_imported += len(df)
        # Add elapsed time to final print out
        print("Data imported successful")
    except Exception as e:
        print("Data load error: " + str(e))
```
#### **ETL Code**

```python
# Testing code
import pandas as pd
import numpy as np
import pytest
from numpy import nan
from product_pipeline import extract, load

# Get data
@pytest.fixture(scope='session', autouse=True)
def df():
    # Will be executed before the first test
    df, tbl = extract()
    yield df  # It will push the dataframe to test.
    # Yield statement gives one value, saves the local state, and resumes again.
    # Will be executed after the last test
    load(df, tbl)

# Check if column exists
def test_col_exists(df):
    name = "ProductKey"
    assert name in df.columns

# Check for nulls
def test_null_check(df):
    assert df['ProductKey'].notnull().all()

# Check values are unique
def test_unique_check(df):
    assert pd.Series(df['ProductKey']).is_unique

# Check data type
def test_productkey_dtype_int(df):
    assert (df['ProductKey'].dtype == int or df['ProductKey'].dtype == np.int64)

# Check data type
def test_productname_dtype_str(df):
    assert (df['EnglishProductName'].dtype == str or df['EnglishProductName'].dtype == 'O')

# Check values in range
def test_range_val(df):
    assert df['SafetyStockLevel'].between(0, 1000).any()

# Check values in a list
def test_range_val_str(df):
    assert set(df.Color.unique()) == {'NA', 'Black', 'Silver', 'Red', 'White', 'Blue', 'Multi', 'Yellow', 'Grey', 'Silver/Black'}
```

## Testing on ETL Pipeline
![image](https://github.com/user-attachments/assets/ba72443a-d057-4105-b9aa-9c87543523a1)

### Test Report as HTML
The test report can also be published as HTML directly using the "pytest-html" extension. It looks like this!
![image](https://github.com/user-attachments/assets/17e159a5-7a0a-46db-b994-4a1eac180e13)

---

## Automating the Testing in Production

In a production environment, testing should be automated while scheduling ETL jobs using **Airflow** or similar tools.
![image](https://github.com/user-attachments/assets/6eeedd5d-8050-4dca-aa9d-c060911730e5)
Here, we have simply used a Python function to extract the data and run automated tests. If the test fails, it will not proceed to the loading stage and will raise an error.
![image](https://github.com/user-attachments/assets/20927f29-ebeb-47c6-a82b-cc6214420efd)

---
## Testing Glue Job with the help of Pytest 
- **Dataset**:
- ![image](https://github.com/user-attachments/assets/423c267c-2be1-4714-ba12-fcbf08e4c3dc)
- **Transformed Dataset**:
- made two transformations in data : 1 - Filtering the male gender, 2 - changing name to lower case
- ![image](https://github.com/user-attachments/assets/57bc07d3-3468-4e14-8880-fbe5fb3790e5)

### **Glue Job Code** 
-This script implements an ETL pipeline using AWS Glue and PySpark to process customer data stored in S3. It reads the data into a DynamicFrame, filters it by gender, converts it to a DataFrame, transforms specific column values to lowercase, and writes the processed data back to S3 in Parquet format. The pipeline utilizes GlueContext and PySpark functions for efficient data processing and storage.

```python
import logging
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.dynamicframe import DynamicFrame
from pyspark.sql import DataFrame

# Function to create Spark context and Glue context
def create_spark_context():
    sc = SparkContext()
    glueContext = GlueContext(sc)
    logger = glueContext.get_logger()
    spark = glueContext.spark_session
    job= Job(glueContext)
    return glueContext, glueContext.spark_session

# Function to read customer dataset from AWS S3
def read_customer_dataset(glueContext):
    logging.info("Reading customer data from AWS S3")
    dyf = glueContext.create_dynamic_frame_from_options(
        connection_type="s3",
        connection_options={"paths": ["s3://data-uploads-name-useast1/customer/customer_info/"]},
        format="parquet"
    )
    return dyf

# Function to filter a column in the DynamicFrame
def filter_column_from_dynamic_frame(dyf, filter_value, column_name):
    logging.info("Filtering data based on a value in column")
    filtered_dyf = dyf.filter(lambda x: x[column_name] == filter_value)
    return filtered_dyf

# Function to convert DynamicFrame to PySpark DataFrame
def dynamic_frame_to_pyspark_dataframe(dyf):
    logging.info("converting dynamic frame to pyspark dataframe")
    df = dyf.toDF()
    return df

# Function to lowercase the values in a specific column of a PySpark DataFrame
def lowercase_column_values(df: DataFrame, column_name: str) -> DataFrame:
    logging.info(f"Lowercasing values in column: {column_name}")
    df = df.withColumn(column_name, lower(col(column_name)))
    return df

# Function to write DataFrame to S3 as Parquet format
def write_dataframe_to_s3(df, glueContext, path: str):
    logging.info(f"Writing DataFrame to S3 bucket at {path}")
    dyf = DynamicFrame.fromDF(df, glueContext, "dyf")
    glueContext = GlueContext(SparkContext.getOrCreate())
    glueContext.write_dynamic_frame.from_options(
        frame=dyf,
        connection_type="s3",
        connection_options={"path": path, "partitionKeys": []},
        format="parquet"
    )

# Main function to execute the ETL process
def main():
    # Step 1: Create Spark context and Glue context
    glueContext, spark = create_spark_context()

    # Step 2: Read customer dataset from S3
    dyf = read_customer_dataset(glueContext)

    # Step 3: Filter data based on gender column (e.g., 'male')
    dyf_filtered = filter_column_from_dynamic_frame(dyf, 'male', 'gender')
    dyf_filtered.show()

    # Step 4: Convert filtered DynamicFrame to PySpark DataFrame
    df = dynamic_frame_to_pyspark_dataframe(dyf_filtered)

    # Step 5: Apply transformation (Lowercase the 'first_name' column)
    df_transformed = lowercase_column_values(df, 'first_name')


    # Step 7: Write transformed data to S3
    output_path = "s3://data-uploads-adriano-useast1/customer/customer_info_filtered/"
    # Step 6: Display the transformed DataFrame

    df_transformed.show()

    write_dataframe_to_s3(df_transformed, glueContext, output_path)

# Execute the main function
if __name__ == "__main__":
    main()

```

### **Testing code(Pytest)**
-This code provides Pytest-based unit tests for AWS Glue PySpark transformations. It includes fixtures to initialize a Spark session and GlueContext, and to create sample PySpark DataFrames and DynamicFrames for testing. The tests verify the correctness of the lowercase_column_values and filter_column_from_dynamic_frame functions by asserting transformations and filtering logic.

```python
import pytest
import logging
from pyspark.sql.types import StructType, StructField, StringType
from pyspark.context import SparkContext
from awsglue.context import GlueContext
from awsglue.dynamicframe import DynamicFrame
from transform_customer_table import lowercase_column_values, filter_column_from_dynamic_frame


# Fixture to initialize Spark session and Glue context
@pytest.fixture(scope="session")
def spark_session():
    """
    Initializes a Spark session and Glue context for testing.
    """
    # Create a SparkContext instance
    sc = SparkContext()
    sc.setLogLevel("ERROR")
    
    # Initialize GlueContext
    glueContext = GlueContext(sc)
    
    # Return the Spark session and GlueContext
    spark = glueContext.spark_session
    return spark, glueContext


# Fixture to create a sample PySpark DataFrame
@pytest.fixture
def sample_df_dataset(spark_session):
    """
    Creates a sample PySpark DataFrame for testing.
    """
    # Define the schema for the DataFrame
    schema = StructType([
        StructField("first_name", StringType(), True),
        StructField("last_name", StringType(), True),
        StructField("email", StringType(), True)
    ])

    # Sample data for the DataFrame
    data = [
        ("John", "Doe", "john.doe@example.com"),
        ("Jane", "Smith", "jane.smith@example.com"),
        ("Alice", "Brown", "alice.brown@example.com")
    ]

    # Access the Spark session and GlueContext
    spark_session, GlueContext  = spark_session

    # Create the DataFrame
    df = spark_session.createDataFrame(data, schema=schema)
    
    # Return the created DataFrame
    return df


# Fixture to create a sample DynamicFrame
@pytest.fixture
def sample_dyf_dataset(spark_session):
    """
    Creates a sample DynamicFrame for testing.
    """
    # Define the schema for the DataFrame
    schema = StructType([
        StructField("first_name", StringType(), True),
        StructField("last_name", StringType(), True),
        StructField("gender", StringType(), True),
        StructField("email", StringType(), True)
    ])

    # Sample data
    data = [
        ("John", "Doe", "male", "john.doe@example.com"),
        ("Jane", "Smith", "female", "jane.smith@example.com"),
        ("Alice", "Brown", "female", "alice.brown@example.com")
    ]

    # Access the Spark session and GlueContext
    spark_session, glueContext = spark_session

    # Create the DataFrame
    df = spark_session.createDataFrame(data, schema=schema)

    # Convert the DataFrame to a DynamicFrame
    dyf = DynamicFrame.fromDF(df, glueContext, "dyf_dataset")

    # Return the DynamicFrame
    return dyf


# Test for lowercase transformation
def test_lower_case(sample_df_dataset):
    """
    Tests the lowercase transformation on a specific column.
    """
    # Apply lowercase transformation on the "first_name" column
    df = lowercase_column_values(sample_df_dataset, "first_name")

    # Assert the number of rows remains the same
    assert df.count() == 3

    # Collect rows for assertion
    rows = df.collect()
    # Assert that the values in the "first_name" column are lowercase
    assert rows[0].first_name == 'john'
    assert rows[1].first_name == 'jane'
    assert rows[2].first_name == 'alice'


# Test for filtering DynamicFrame by gender
def test_filter_gender(sample_dyf_dataset):
    """
    Tests the filtering of a DynamicFrame by gender column.
    """
    # Apply filter transformation on the "gender" column with value 'male'
    dyf = filter_column_from_dynamic_frame(sample_dyf_dataset, 'male', 'gender')

    # Assert that the count of rows is correct after filtering
    assert dyf.count() == 1
```
## DBT (Data Build Tool)

DBT (Data Build Tool) is a popular framework for transforming and testing data in the data warehouse. It allows analysts and data engineers to write modular SQL transformations, schedule them, and perform tests on data quality within the pipeline.

### Key Benefits
- **Data Testing**: Built-in testing for data quality and integrity.
- **Version Control**: Integrates with Git for versioned data models.
- **Collaboration**: Enables collaboration between data teams through a shared repository of models and tests.

### Use Cases
- **Transformations**: Write SQL-based transformations that process raw data into analytics-ready datasets.
- **Data Testing**: DBT supports tests like uniqueness, not null, and referential integrity to ensure data quality.
- **Data Quality Assurance**: Run tests on the transformed data to ensure it adheres to defined expectations.
- **Documentation**: Automatically generate documentation for models, tests, and dependencies within the pipeline.

### Key Features
- **Built-in Testing**: Run tests to check data quality (e.g., uniqueness, null checks).
- **Version Control**: Integrates with Git to track model and test changes.
- **Scheduling**: Automate runs via cron jobs or DBT Cloud.
- **Documentation**: Auto-generate model and test documentation for easier collaboration.
- **Data Lineage**: Visualize dependencies between models for better understanding.

## **Testing using dbt**

file structure 

![image](https://github.com/user-attachments/assets/69f519e2-8523-49fd-8223-5d99581b9c85)

## **runs tests usind dbt test command**
![image](https://github.com/user-attachments/assets/63abebad-42cf-47c4-ae9b-c8fc0f6a3af5)


---
# **GREAT_EXPECTATIONS**

Great Expectations (GE) allows you to write declarative data tests (e.g., "I expect this table to have x and y number of rows"), get validation results from those tests, and output a report that documents the current state of the data. 

An expectation can be made of two or more test cases.

Great Expectations can be used with multiple popular databases such as MySQL, PostgreSQL, AWS S3, AWS Redshift, BigQuery, Snowflake, etc., and raw files.

It can also be scheduled using tools like **Apache Airflow**.

---

## Key Features

### **Data Validation**
- Define custom expectations for dataset integrity, null values, column types, and more.

### **Automated Data Documentation**
- GE creates data documentation as tests are defined, enabling traceability and transparency.

### **Integration with Data Pipelines**
- GE integrates with common pipeline orchestrators like **Apache Airflow**, **dbt**, and **Prefect**.

### **Batch Processing**
- Tests data in "batches," allowing for fine-grained control over testing specific data subsets.

---

## File Structure

Great Expectations creates a file structure in the following format:

- **great_expectations/**
  - **expectations/**: Stores expectation suites (validation rules)
  - **checkpoints/**: Defines when and where to run validations
  - **great_expectations.yml**: Main configuration file
  - **validations/**: Stores validation results
  - **uncommitted/**: Contains sensitive files (e.g., database credentials)

### **Testing through great_expectations**
![image](https://github.com/user-attachments/assets/a3d889ba-7e4d-44ad-b223-15c38f55ccf6)

---
# Modelling Data For Testing:
1. **Mock Data**
   
   *Pros*: Easy to create with synthetic data tools (e.g., Faker).

   *Cons*: Lacks production-scale volume, variety, and velocity.


2. **Sample Production Data to Test/Dev**

   *Pros*: Easier to copy small samples.
   
   *Cons*: Sampling must reflect production reality; tests may fail on actual prod data due to insufficient volume/variety.


3. **Full Production Data Copy**
  
   *Pros*: Real-world data available.
  
   *Cons*: Risks data privacy; data may become stale and lacks real-time velocity.


4. **Anonymized Production Data Copy**
    
  *Pros*: Real-world data with privacy compliance.
  
  *Cons*: Needs frequent refreshing; anonymization is resource-intensive and error-prone.


5. **Data Versioning Tool**
  
  *Pros*: Real-world data with automated, short-lived environments.In LakeFS Branching feature lets to create a test branch from a production branch.
  Only the pointers to the underlying data is copied. We get access to all production data without actually copying it, risking the production data.
  Branch creation is a one line command
  
  *Cons*: Requires additional tool (e.g., lakeFS).


## What If a Test Fails and the Reason Is Not Known?

Here's a technique you can use to find the bug.

### The Saff Squeeze

Start with a failing test—any failing test.

1. **Run the test and watch it fail. Commit.**  
   You’ll almost certainly squash later.

2. **Inline the action of the test.**  
   This often requires raising the visibility of internal parts of a module, so do it. Don’t worry about weakening the design for now.

3. **Look for intermediate assertions to add.**  
   Such as checking the condition of an if statement. Add assertions as “high” in the test as possible. We are looking for a new assertion to fail earlier in the test.

4. **Once the failure moves earlier in the test, commit.**  
   You’ll squash later.

5. **Prune away unnecessary parts of the test.**  
   Such as paths of if statements that don’t execute and all the code after the first failing assertion. Commit. You’ll squash later.

6. **Do you now understand the cause of the defect?**  
   - If yes, document it and decide how to fix it.
   - If no, try another iteration of the steps.

At some point, you’ll find **“the missing test”**. This is a test that you probably wish that someone had written at some point in the past. We don’t need to blame anyone, even though we might want to. We have found a missing test. We can interpret that as good news about today, not bad news about yesterday.

Don’t rush. Take a moment. Write things down. Breathe. Now decide how to proceed:

- **Maybe squash all these commits,** because they add up to “here’s a failing test we should have written before; we’ll make it pass now”. The project history makes it look the same as if we’d always planned to test-drive this behavior today.
- **Throw away all those commits,** then test-drive the missing behavior using the notes you wrote when you documented the cause of the defect and how to fix it.

I imagine there are other options, but I’m not sure what they are right now. Maybe there aren’t.

---

Source: [The Saff Squeeze - The Code Whisperer](https://blog.thecodewhisperer.com/permalink/the-saff-squeeze)
****
