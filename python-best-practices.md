# Python Best Practices

## 1. Code Readability and Structure

### 1.1 Follow Standard Style Guides

- Adhere to established style guides for consistent and maintainable code. Use these as default references:
    - [Google Style Guide](https://google.github.io/styleguide/)
    - [PEP 8 – Python Style Guide](https://peps.python.org/pep-0008/)
- Automate enforcement using tools like **black**, **flake8**, or **pylint** for Python, and **scalafmt** for Scala.
- Use pre-commit hooks to ensure compliance before code gets pushed.

---

### 1.2 Descriptive Naming

- Use clear, descriptive names for variables, functions, and classes.
- Avoid vague terms or abbreviations. Names should reflect their intent and context.

**Best Practices**
- **Variables**: Name them based on their role or content.  
    Example:
    ```python
    rows_processed = 500   # Instead of 'rp'
    ```
- **Functions**: Use verb-noun pairs to convey action and object.  
    Example:
    ```python
    def transform_dataframe(input_df):  
        ...
    ```
- **Classes**: Use singular nouns with PascalCase.  
    Example:
    ```python
    class DataPipelineConfig:  
        ...
    ```

**Example**
```python
# Avoid this:
def calc(a, b):
    return a + b

# Use this:
def calculate_sum(first_number: int, second_number: int) -> int:
    """
    Calculate the sum of two integers.

    Args:
        first_number (int): First operand.
        second_number (int): Second operand.

    Returns:
        int: Sum of the two numbers.
    """
    return first_number + second_number
```

---

### 1.3 Consistent Code Layout

A structured layout improves code readability and maintainability. Ensure every module/script follows a predictable order:

**Module Structure**
1. **Module docstring**: Describe the purpose of the file.
    - Example:
    ```python
    """
    Module: data_loader.py
    Purpose: Contains functions and classes for reading and preparing raw data.
    """
    ```

2. **Imports**:
    - **Standard libraries** (e.g., `os`, `sys`) first.
    - **Third-party libraries** (e.g., `pandas`, `pyspark`) next.
    - **Internal project modules** last.  
        Use tools like **isort** to enforce import order.
3. **Constants and global variables**:
    - Capitalize constant names, and avoid unnecessary globals.  
        Example:
    ```python
    FILE_PATH = "/data/source/"
    ```
    
4. **Classes and functions**:
    - Order classes and functions logically, grouping related functionality together.
5. **Main logic**:
    - Keep the script entry point under `if __name__ == "__main__":`.

**Example Script**
```python
"""
Module: transform_sales_data.py
Purpose: Transforms raw sales data into cleaned, structured dataframes for reporting.
"""

import os
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

# Constants
RAW_DATA_PATH = "/data/raw/sales/"
PROCESSED_DATA_PATH = "/data/processed/sales/"

# Functions
def read_raw_data(spark: SparkSession, file_path: str):
    """
    Read raw sales data from the specified path.
    """
    return spark.read.csv(file_path, header=True, inferSchema=True)

def clean_data(df):
    """
    Clean and format raw data by removing nulls and standardizing column names.
    """
    return df.filter(col("sales_amount").isNotNull()).withColumnRenamed("sales_amount", "amount")

# Main Logic
if __name__ == "__main__":
    spark = SparkSession.builder.appName("Sales Data Transformation").getOrCreate()
    
    raw_data = read_raw_data(spark, RAW_DATA_PATH)
    clean_data_df = clean_data(raw_data)
    clean_data_df.write.parquet(PROCESSED_DATA_PATH, mode="overwrite")
```

---

### 1.4 Additional Best Practices

1. **Use Type Hints**:
    - Add type annotations for functions and variables to make code self-documenting and easier to debug.
    - Use tools like **mypy** to check type compliance.  
        Example:
    ```python
    def transform_data(input_data: list[int]) -> list[int]:
        ...
    ```

2. **Limit Function Scope**:
    - Avoid writing overly long functions. Break them into smaller, reusable functions.
3. **Leverage Comments Sparingly**:
    - Write comments only when necessary to explain _why_, not _what_. Let the code speak for itself.
4. **Avoid Hardcoding**:
    - Use configuration files (e.g., YAML, JSON) or environment variables for dynamic values.

---
## 2. Modularizing Code

Modularizing code improves reusability, scalability, and maintainability in data engineering projects. Here's how to structure and organize your code effectively:

### 2.1 Split Code into Modules

Break down large scripts into smaller, logically grouped modules. Each module should focus on a single responsibility to adhere to the **Single Responsibility Principle (SRP)**.

**Example Project Structure**:
A typical data pipeline project might have the following directory structure:
```bash
project/
├── ingestion/              # Responsible for data input and extraction
│   ├── __init__.py
│   ├── s3_loader.py        # Fetches data from S3 buckets
│   └── db_extractor.py     # Reads data from databases
├── transformation/         # Handles data processing and cleaning
│   ├── __init__.py
│   ├── cleaner.py          # Removes nulls, standardizes data
│   ├── aggregator.py       # Aggregates data for reporting
│   └── validator.py        # Validates schema and data integrity
├── orchestration/          # Coordinates and runs pipelines
│   ├── __init__.py
│   ├── pipeline_manager.py # Orchestrates the pipeline flow
├── config/                 # Centralized configuration
│   ├── __init__.py
│   ├── config.yaml         # YAML file with environment-specific settings
│   └── loader.py           # Parses and loads configuration
├── utils/                  # Utility functions and shared code
│   ├── __init__.py
│   ├── s3_utils.py         # S3 helper functions
│   ├── logging_utils.py    # Standardized logging setup
│   └── file_utils.py       # Local file handling helpers
├── tests/                  # Unit and integration tests
│   ├── test_ingestion.py
│   ├── test_transformation.py
│   └── test_utils.py
└── main.py                 # Entry point to trigger the pipeline
```

**Tips:**
1. **Use `__init__.py` Files**: Even in modern Python, include `__init__.py` in every module directory to explicitly mark it as a package and enable relative imports.
2. **Group Related Code**: Keep modules narrowly focused on their domain (e.g., ingestion, transformation).
3. **Configuration-Driven Code**: Centralize all configurable parameters in the `config/` folder to reduce hardcoding and improve portability.

---
### 2.2 Avoid Code Duplication
Duplicate code increases maintenance overhead and leads to inconsistencies. Extract reusable functionality into shared modules.

**Creating a Reusable Utility Module**
Here’s an example of extracting common S3 operations into a utility module:

**File: `utils/s3_utils.py`**
```python
import boto3

def get_s3_client(region_name: str = "us-west-2"):
    """
    Create and return a boto3 S3 client.
    
    Args:
        region_name (str): AWS region name. Default is "us-west-2".
    
    Returns:
        boto3.client: S3 client object.
    """
    return boto3.client('s3', region_name=region_name)

def download_from_s3(bucket_name: str, object_key: str, local_file_path: str):
    """
    Download a file from S3 to the local file system.

    Args:
        bucket_name (str): Name of the S3 bucket.
        object_key (str): S3 object key (file path).
        local_file_path (str): Local file path to save the file.
    """
    s3 = get_s3_client()
    s3.download_file(bucket_name, object_key, local_file_path)
    print(f"Downloaded {object_key} from {bucket_name} to {local_file_path}")
```

**Example Usage**
**File: `ingestion/s3_loader.py`**
```python
from utils.s3_utils import download_from_s3

def load_raw_data(bucket_name: str, object_key: str, local_file_path: str):
    """
    Load raw data from S3 and save it locally.
    """
    download_from_s3(bucket_name, object_key, local_file_path)
    print("Raw data loaded successfully.")
```

---
### 2.3 Best Practices for Modularization

#### 1. Keep Modules Small and Focused

- Ensure each module has a single, well-defined responsibility.  
    Example: `s3_loader.py` should handle S3-specific ingestion, not data transformation.
#### 2. Use Clear Boundaries Between Stages
- For ETL pipelines, separate each stage: ingestion, transformation, and loading into distinct directories or modules. Avoid mixing logic.
#### 3. Enforce Config-Driven Design
- Use a central configuration file (e.g., `config.yaml`) to store values like S3 bucket names, database credentials, or file paths.
- Use libraries like **PyYAML** or **dotenv** to load configurations dynamically.

**Example: `config/loader.py`**
```python
import yaml

def load_config(file_path: str = "config/config.yaml"):
    """
    Load configuration from a YAML file.

    Args:
        file_path (str): Path to the configuration file.

    Returns:
        dict: Parsed configuration dictionary.
    """
    with open(file_path, "r") as file:
        return yaml.safe_load(file)
```

---
### 2.4 Practical Tooling for Modularization
- **Automated Dependency Management**: Use tools like `poetry` or `pip-tools` to handle Python dependencies effectively.
- **Test-Driven Modularization**: Write unit tests for individual modules to ensure modularity doesn’t introduce hidden bugs. Use **pytest** for testing.
---
### 2.5 Modularizing Practices for Standalone Scripts (Glue Jobs with Utilities in S3)

When utilities and configuration files are stored in S3, you can still apply modularization principles to make your AWS Glue Jobs maintainable and reusable. Here's how to structure and utilize such setups:

**Best Practices for Glue Jobs with Utilities in S3**:

- **Store Shared Utilities in S3**:
    - Upload commonly used Python files (`.py`) or zipped modules to an S3 bucket.
    - Use AWS Glue's `--extra-py-files` option to dynamically include these utilities in the job runtime.
- **Centralized Configuration in S3**:
    - Store YAML, JSON, or `.properties` files in S3 for environment-specific configurations. Load these dynamically at runtime.
- **Dynamic Utility Loading**:
    - Download utility files from S3 during the job initialization, and add them to the Python path using `sys.path`.
- **Temporary Storage**:
    - Use Glue's `/tmp/` directory for storing downloaded utilities and configuration files during runtime.

**Example Structure in S3**:

```
s3://my-project/
├── utils/
│   ├── s3_utils.py
│   ├── logging_utils.py
│   └── db_utils.py
├── config/
│   ├── dev_config.yaml
│   ├── prod_config.yaml
├── glue_jobs/
│   ├── transformations.py
```


**Example Glue Job: Using Utilities and Configs from S3**

**Step 1: Upload Utility Files to S3**  
Store your Python utility scripts (e.g., `s3_utils.py`, `logging_utils.py`) in an S3 bucket (`s3://my-project/utils/`).

**Step 2: Glue Job Script (`main_job.py`)**

```python
import os
import sys
import boto3
import yaml

# Add downloaded utilities to the Python path
def add_utilities_to_path(s3_bucket: str, s3_prefix: str, tmp_dir: str = "/tmp/"):
    """
    Download utility scripts from S3 and add them to the Python path.

    Args:
        s3_bucket (str): Name of the S3 bucket.
        s3_prefix (str): S3 prefix for utility files.
        tmp_dir (str): Temporary directory to store utilities.
    """
    s3 = boto3.client("s3")
    utilities = s3.list_objects_v2(Bucket=s3_bucket, Prefix=s3_prefix)["Contents"]

    for utility in utilities:
        file_name = os.path.basename(utility["Key"])
        local_path = os.path.join(tmp_dir, file_name)
        s3.download_file(s3_bucket, utility["Key"], local_path)
        sys.path.insert(0, tmp_dir)

# Load configurations dynamically
def load_config_from_s3(s3_bucket: str, config_key: str):
    """
    Load configuration file from S3.

    Args:
        s3_bucket (str): S3 bucket name.
        config_key (str): S3 key for the configuration file.

    Returns:
        dict: Parsed configuration dictionary.
    """
    s3 = boto3.client("s3")
    obj = s3.get_object(Bucket=s3_bucket, Key=config_key)
    return yaml.safe_load(obj["Body"])

# Glue job main logic
def main():
    # Initialize Spark session
    from pyspark.sql import SparkSession
    spark = SparkSession.builder.appName("GlueJobWithS3Utilities").getOrCreate()

    # Add utilities to path
    add_utilities_to_path(s3_bucket="my-project", s3_prefix="utils/")

    # Import downloaded utilities
    from s3_utils import download_from_s3
    from transformations import clean_data

    # Load configurations
    config = load_config_from_s3(s3_bucket="my-project", config_key="config/dev_config.yaml")

    # Process data
    raw_data_path = config["raw_data_path"]
    processed_data_path = config["processed_data_path"]

    raw_df = spark.read.csv(raw_data_path, header=True)
    clean_df = clean_data(raw_df)
    clean_df.write.parquet(processed_data_path, mode="overwrite")

if __name__ == "__main__":
    main()
```

**Key Points**:
1. **S3 Utilities**:
    - Use the `boto3` client to download scripts from S3.
    - Add `/tmp/` to the Python path dynamically using `sys.path`.
2. **Dynamic Configuration**:
    - Centralize configs in S3, allowing seamless switching between environments (e.g., dev, prod).
3. **AWS Glue Parameters**:
    - Use Glue's `--extra-py-files` option for smaller utility `.zip` packages stored in S3.
    - Example:
```bash
aws glue start-job-run \
--job-name "MyGlueJob" \
--arguments '--extra-py-files=s3://my-project/utils.zip'`
```
4. **Temporary Downloads**:
    - Store downloaded files in Glue’s `/tmp/` directory to avoid conflicts across concurrent job runs.
---
## 3. Error Handling and Logging

To build reliable data engineering pipelines, robust error handling and logging practices are crucial. These ensure transparency, traceability, and quick debugging during failures.
### 3.1 Granular Error Handling
- Handle exceptions contextually to provide actionable insights.
- Avoid generic `except:` blocks that mask errors.
- Example for a data pipeline:
```python
    import logging
    
    logger = logging.getLogger(__name__)
    
    def process_data(file_path):
        try:
            # File loading logic
            with open(file_path, 'r') as file:
                data = file.read()
            logger.info(f"File loaded successfully: {file_path}")
            return data
    
        except FileNotFoundError:
            logger.error(f"File not found: {file_path}")
            raise
    
        except ValueError as ve:
            logger.error(f"Data processing error: {ve}")
            raise
    
        except Exception as e:
            logger.critical(f"Unexpected error: {e}", exc_info=True)
            raise
```

---
### 3.2 Log Structuring
- **Structured Logs**: Use JSON-format logs to integrate seamlessly with log monitoring tools (e.g., ELK Stack, CloudWatch, Datadog).
- **Metadata Enrichment**: Include relevant metadata like timestamps, job IDs, or environment for context.
- **Dynamic Fields**: Log details like row counts, file paths, or S3 buckets dynamically.
**Example: Structured Logging**

```python
import json
import logging
from datetime import datetime

# Setup logger
logger = logging.getLogger("structured_logger")
handler = logging.StreamHandler()
formatter = logging.Formatter('%(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
logger.setLevel(logging.INFO)

def log_as_json(level, message, **kwargs):
    log_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "level": level,
        "message": message,
        **kwargs
    }
    logger.info(json.dumps(log_entry))

# Usage example
log_as_json("INFO", "Data ingestion completed", rows=5000, source="S3", job_id="ingest_001")
```

---
### 3.3 Logging Best Practices

1. **Centralized Logging**:
    - Push logs to a central location (e.g., AWS CloudWatch, ELK Stack, or Datadog) for monitoring and alerting.
2. **Log Levels**:
    - Use appropriate log levels (`DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL`) to manage verbosity:
        - `DEBUG`: For troubleshooting.
        - `INFO`: For successful operations.
        - `WARNING`: For recoverable issues.
        - `ERROR`: For failed operations requiring intervention.
        - `CRITICAL`: For severe failures.
3. **Tracebacks for Debugging**:
    - Log detailed stack traces for unexpected errors:
        ```python
        import logging
        logger = logging.getLogger(__name__)
        
        try:
            1 / 0  # Simulated error
        except Exception as e:
            logger.error("Critical failure", exc_info=True)
        ```
4. **Environment-Specific Logging**:
    - Configure logging verbosity and destinations based on the environment (dev, staging, prod).
    - Use YAML or `.env` files for configurations.
5. **Redaction of Sensitive Data**:
    - Mask sensitive information (e.g., credentials, API keys) before logging:
        ```python
        def redact_sensitive_data(data):
            return {k: ("*****" if "key" in k.lower() else v) for k, v in data.items()}
        ```
---
### 3.4 Error Retry Mechanisms
For recoverable errors (e.g., temporary network issues, S3 timeouts), implement retry logic with exponential backoff to avoid immediate failure.
**Example: Retrying with Exponential Backoff**

```python
import time
import logging

logger = logging.getLogger(__name__)

def retry_operation(func, retries=3, backoff_factor=2, *args, **kwargs):
    for attempt in range(retries):
        try:
            return func(*args, **kwargs)
        except Exception as e:
            wait_time = backoff_factor ** attempt
            logger.warning(f"Retry {attempt + 1}/{retries} failed: {e}. Retrying in {wait_time} seconds...")
            time.sleep(wait_time)
    logger.error(f"Operation failed after {retries} retries.")
    raise

# Example usage
def simulate_error_operation():
    raise Exception("Simulated failure")

try:
    retry_operation(simulate_error_operation, retries=3)
except Exception:
    logger.critical("All retries exhausted.")
```

---
### 3.5 Logging in Distributed Systems

For distributed environments like Spark or Lambda:
- **Glue Jobs**:
    - Use `glueContext.get_logger()` for native logging in AWS Glue.
    ```python
    glue_logger = glueContext.get_logger()
    glue_logger.info("Started processing job")
    ```

- **Spark**:
    - Use Spark’s `log4j`-based logging:
    ```python
    log4j_logger = sc._jvm.org.apache.log4j
    logger = log4j_logger.LogManager.getLogger(__name__)
    logger.info("Spark job started")
    ```
    
- **Lambda**:
    - Use AWS-provided `logging` module for structured output:
    ```python
    import logging
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    logger.info("Lambda function invoked")
    ```
    
---
### 3.6 Logging Pipeline Runs to a Database

For detailed and persistent tracking of pipeline runs, you can log key details such as job start time, end time, status (success/failure), and error messages directly into a database table. This approach allows you to monitor pipeline performance and debug issues effectively over time.


**Database Table Structure:**
**Table Name:** `pipeline_run_logs`

| **Column Name**         | **Data Type** | **Description**                              |
| ----------------------- | ------------- | -------------------------------------------- |
| `run_id`                | `UUID`        | Unique identifier for the pipeline run.      |
| `job_name`              | `VARCHAR`     | Name of the job or pipeline.                 |
| `start_time`            | `TIMESTAMP`   | Start time of the pipeline.                  |
| `end_time`              | `TIMESTAMP`   | End time of the pipeline.                    |
| `status`                | `VARCHAR`     | Status of the run (`SUCCESS`, `FAILURE`).    |
| `error_message`         | `TEXT`        | Detailed error message if the run failed.    |
| `processed_records`     | `INTEGER`     | Number of records processed (optional).      |
| `execution_environment` | `VARCHAR`     | Execution environment (e.g., `dev`, `prod`). |

**Example Implementation:**
Here’s how to implement logging to a database for your pipeline runs:
1. **Database Connection Setup**  
    Use a library like `psycopg2` for PostgreSQL or `sqlalchemy` for ORM-based logging.
2. **Insert Logging Functions**  
    Define utility functions for inserting and updating log records.
3. **Track Execution Lifecycle**  
    Log the start of the job, update it on success or failure, and include error details if applicable.

**Code Example: Logging Pipeline Runs to a Database**

```python
import uuid
import psycopg2
from datetime import datetime

# Database connection details
DB_CONFIG = {
    "dbname": "pipeline_logs",
    "user": "admin",
    "password": "password",
    "host": "localhost",
    "port": 5432,
}

def log_pipeline_run(job_name, start_time, status=None, end_time=None, error_message=None, processed_records=None, environment="prod"):
    """
    Logs pipeline run details to the database.

    Args:
        job_name (str): Name of the pipeline job.
        start_time (datetime): Start time of the job.
        status (str): Current status ('SUCCESS' or 'FAILURE').
        end_time (datetime): End time of the job (if available).
        error_message (str): Error details in case of failure.
        processed_records (int): Number of records processed (optional).
        environment (str): Environment (e.g., 'dev', 'prod').
    """
    conn = None
    run_id = str(uuid.uuid4())
    try:
        conn = psycopg2.connect(**DB_CONFIG)
        cur = conn.cursor()

        # Insert or update the log record
        if status is None:
            # Insert a new log record for job start
            cur.execute(
                """
                INSERT INTO pipeline_run_logs (run_id, job_name, start_time, execution_environment)
                VALUES (%s, %s, %s, %s)
                """,
                (run_id, job_name, start_time, environment),
            )
        else:
            # Update the log record with status, end_time, and error_message
            cur.execute(
                """
                UPDATE pipeline_run_logs
                SET status = %s, end_time = %s, error_message = %s, processed_records = %s
                WHERE run_id = %s
                """,
                (status, end_time, error_message, processed_records, run_id),
            )

        # Commit transaction
        conn.commit()
        cur.close()
        return run_id
    except Exception as e:
        print(f"Failed to log pipeline run: {e}")
    finally:
        if conn:
            conn.close()

# Example: Pipeline execution lifecycle
try:
    job_name = "daily_data_ingestion"
    start_time = datetime.utcnow()
    run_id = log_pipeline_run(job_name, start_time)

    # Simulate job execution
    print("Pipeline running...")
    processed_records = 5000  # Example record count

    # Mark the job as successful
    log_pipeline_run(
        job_name,
        start_time,
        status="SUCCESS",
        end_time=datetime.utcnow(),
        processed_records=processed_records,
        environment="prod",
    )
except Exception as e:
    # Log job failure with error details
    log_pipeline_run(
        job_name,
        start_time,
        status="FAILURE",
        end_time=datetime.utcnow(),
        error_message=str(e),
        environment="prod",
    )
```

**Benefits of Logging in a Database:**
1. **Centralized Visibility**: Track all pipeline runs in one place.
2. **Historical Analysis**: Use SQL queries to analyze trends, e.g., average execution time or failure rate.
3. **Automation Triggers**: Use failure logs to trigger alerts or auto-remediation workflows.
4. **Auditing**: Maintain an audit trail for compliance or debugging purposes.

**Query Example:**
To analyze the average runtime of successful jobs in production:

```sql
SELECT job_name, AVG(EXTRACT(EPOCH FROM (end_time - start_time))) AS avg_runtime_sec
FROM pipeline_run_logs
WHERE status = 'SUCCESS' AND execution_environment = 'prod'
GROUP BY job_name;
```

---
## 4. Data Validation and Testing in Pipelines
Ensuring data quality and reliability is essential for robust pipeline development. By integrating automated data validation and testing practices, data engineers can proactively identify and resolve issues.

### 4.1 Automated Data Validation
Automate validation of incoming data and intermediate results using specialized libraries.
1. **Schema Validation**: Validate the schema of incoming data (e.g., column names, data types).  
    **Example: Using `pandera` for Schema Validation**
    
    ```python
    import pandas as pd
    import pandera as pa
    from pandera import Column, DataFrameSchema
    
    schema = DataFrameSchema({
        "id": Column(pa.Int, nullable=False),
        "name": Column(pa.String, nullable=False),
        "age": Column(pa.Int, checks=pa.Check.ge(0)),
    })
    
    df = pd.DataFrame({"id": [1, 2], "name": ["Alice", "Bob"], "age": [25, 30]})
    schema.validate(df)  # Raises an error if validation fails
    ```
    
2. **Value Validation**: Validate the values in specific columns (e.g., ranges, sets, or patterns).  
    **Example: Using `great_expectations` for Column Validation**
    
    ```python
    from great_expectations.dataset import PandasDataset
    
    data = {"age": [25, 30, 45], "gender": ["M", "F", "M"]}
    df = PandasDataset(data)
    df.expect_column_values_to_be_in_set("gender", {"M", "F"})
    df.expect_column_values_to_be_between("age", min_value=18, max_value=60)
    ```
    
---
### 4.2 Mock Data for Testing
Generate realistic test data to simulate various scenarios, including edge cases.
1. **Synthetic Data Generation**: **Example: Using `faker` to Generate Mock Records**
    ```python
    from faker import Faker
    
    fake = Faker()
    test_data = [
        {"name": fake.name(), "email": fake.email(), "phone": fake.phone_number()}
        for _ in range(5)
    ]
    print(test_data)
    ```
2. **Edge Case Simulation**:  Create datasets with missing values, invalid types, or unexpected patterns to test pipeline resilience.

---
### 4.3 Pipeline Test Automation

Test pipeline logic using automated tests that simulate real-world scenarios.

1. **Unit Testing**: Validate individual functions or transformations within the pipeline.  
    **Example: Testing a Data Transformation Function**

    ```python
    def calculate_discount(price, discount_rate):
        return price * (1 - discount_rate)
    
    def test_calculate_discount():
        assert calculate_discount(100, 0.1) == 90
        assert calculate_discount(200, 0.2) == 160
    ```
    
2. **Integration Testing**: Test interactions between pipeline components. Use mock services like `moto` for AWS testing.  
    **Example: Testing S3 Interaction with `moto`**
    
    ```python
    import boto3
    from moto import mock_s3
    
    @mock_s3
    def test_s3_upload():
        s3 = boto3.client("s3", region_name="us-east-1")
        s3.create_bucket(Bucket="test-bucket")
        s3.put_object(Bucket="test-bucket", Key="file.txt", Body="data")
    
        response = s3.list_objects(Bucket="test-bucket")
        assert response["Contents"][0]["Key"] == "file.txt"
    ```
    
3. **End-to-End Testing**: Test the entire pipeline with controlled inputs and expected outputs.
    
    **Example: Simulating an End-to-End Pipeline Execution**
    
    ```python
    def test_pipeline_execution():
        input_data = [{"id": 1, "value": 10}, {"id": 2, "value": 20}]
        expected_output = [{"id": 1, "value": 15}, {"id": 2, "value": 25}]
    
        result = pipeline_function(input_data)
        assert result == expected_output
    ```

---
## 5. Parameterization and Configuration Management

Effective parameterization and configuration management ensure flexibility, scalability, and maintainability in data engineering workflows. By abstracting configurations, you can adapt pipelines for different environments without modifying code.

### 5.1 Environment-Aware Configurations

1. **Use Environment-Specific Configuration Files**:  
    Organize configurations for different environments (development, testing, production).  
    Example Folder Structure:
    ```
    config/
    ├── dev_config.yaml
    ├── test_config.yaml
    └── prod_config.yaml
    ```
    
    **Sample Configuration File (`dev_config.yaml`)**:
    ```yaml
    database:
      host: "localhost"
      port: 5432
      user: "dev_user"
      password: "dev_password"
    s3:
      bucket_name: "dev-data-bucket"
      region: "us-west-2"
    ```
    
    **Loading Configuration Based on Environment**:
    ```python
    import os
    import yaml
    
    def load_config(env="dev"):
        config_file = f"config/{env}_config.yaml"
        with open(config_file, 'r') as file:
            return yaml.safe_load(file)
    
    current_env = os.getenv("ENV", "dev")  # Default to 'dev'
    config = load_config(current_env)
    print(config["database"]["host"])
    ```

---
### 5.2 Parameterization

1. **Pipeline Parameterization**:  
    Pass parameters dynamically to pipelines or functions for reusability.  
    **Example: Glue Job Arguments**:
    ```python
    import sys
    from awsglue.utils import getResolvedOptions
    
    args = getResolvedOptions(sys.argv, ["JOB_NAME", "S3_BUCKET", "TABLE_NAME"])
    print(f"Running job {args['JOB_NAME']} with bucket {args['S3_BUCKET']} and table {args['TABLE_NAME']}")
    ```

2. **Using Environment Variables**:  
    Access sensitive or dynamic values using environment variables.  
    **Example**:
    ```python
    import os
    
    DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:password@localhost:5432/dbname")
    print(f"Connecting to database: {DATABASE_URL}")
    ```

---
### 5.3 Best Practices

1. **Centralized Configuration Storage**:
    - Use centralized systems like AWS Systems Manager Parameter Store, AWS Secrets Manager, or HashiCorp Vault for secure and consistent configuration storage.
    - **Example: Fetching from AWS Parameter Store**:
        ```python
        import boto3
        
        ssm_client = boto3.client("ssm", region_name="us-west-2")
        
        def fetch_parameter(name):
            response = ssm_client.get_parameter(Name=name, WithDecryption=True)
            return response["Parameter"]["Value"]
        
        db_password = fetch_parameter("/myapp/database/password")
        print(f"Database password: {db_password}")
        ```

2. **Validation of Configurations**:
    - Validate configurations before using them in pipelines to catch issues early.  
        **Example**:
    ```python
    def validate_config(config):
        required_keys = ["database", "s3"]
        for key in required_keys:
            if key not in config:
                raise ValueError(f"Missing required configuration key: {key}")
    
    validate_config(config)
    ```
    
3. **Version-Controlled Configurations**:
    - Store non-sensitive configuration files in version control systems like Git for auditability and collaboration.

---

## 6. Documentation of Functions/Code

Proper documentation is essential for code maintainability, collaboration, and ensuring that other team members can quickly understand and reuse code. Adopting standardized documentation practices improves readability and reduces onboarding time for new engineers.

### 6.1 Writing Docstrings
- Use **docstrings** to document all public functions, classes, and modules.
- Follow a standard format like **Google Style** or **reStructuredText (reST)**.
- Example using **Google Style**:
    ```python
    def load_data_from_s3(bucket: str, key: str) -> str:
        """
        Load data from an S3 bucket.
    
        Args:
            bucket (str): Name of the S3 bucket.
            key (str): Key of the file in the S3 bucket.
    
        Returns:
            str: The content of the file.
    
        Raises:
            FileNotFoundError: If the file does not exist in the bucket.
            boto3.exceptions.Boto3Error: If there is an error accessing S3.
        """
        # Implementation here
    ```

### 6.2 Documenting Modules

- Add a module-level docstring at the top of each file.
- Example:
    ```python
    """
    data_pipeline.py
    
    This module contains functions and classes for orchestrating data pipelines.
    Includes functionality for data ingestion, transformation, and validation.
    
    Classes:
        DataPipelineManager: Manages the lifecycle of a data pipeline.
    
    Functions:
        run_pipeline: Executes the full pipeline based on provided configuration.
    
    Usage:
        python data_pipeline.py --config config/prod.yaml
    """
    ```

---
### 6.3 Inline Comments

- Use inline comments for complex or non-intuitive code logic.
- Avoid over-commenting; ensure the code itself is self-explanatory.
- Example:

    ```python
    # Filter rows where the "status" column is not null
    df_filtered = df.filter(df.status.isNotNull())
    ```

---
### 6.4 Best Practices
1. **Be Concise Yet Informative**:
    - Avoid redundancy in comments; focus on explaining "why" rather than "what" when the code is self-explanatory.
2. **Document Edge Cases**:
    - Clearly state assumptions, edge cases, and limitations in the docstring.
3. **Version-Controlled Documentation**:
    - Maintain version history of documentation to track updates alongside code changes.
4. **Periodic Reviews**:
    - Schedule team reviews to ensure documentation accuracy and relevance.

---