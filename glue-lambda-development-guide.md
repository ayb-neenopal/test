# Go-To Guide for Developing Glue Jobs and Lambda Functions

This guide consolidates best practices for developing AWS Glue jobs and Lambda functions, covering everything from the development process to configuration and error handling. 

Also refer to [python-best-practices](python-best-practices.md) for more tips and guidelines.

---
## 0. Best Coding Practices for Glue Jobs and Lambda Functions

- **Code Readability**:  
    - Write clean, readable code by following industry-standard Python style guides, such as [PEP 8](https://peps.python.org/pep-0008/) or [Google Style Guide](https://google.github.io/styleguide/). 
    - This ensures consistency across codebases, making it easier for team members to understand and maintain code, especially in collaborative environments.
    - **Use meaningful variable names**.
    - **Follow consistent indentation** and spacing conventions.
    - **Document functions and classes** with docstrings to explain their purpose, inputs, and outputs.
    - **Limit line length** to 79 characters to enhance readability.

- **Error Handling**:  
    - Proper error handling ensures your code is robust and can gracefully handle unexpected issues. 
    - Catch exceptions and log useful error messages to **CloudWatch Logs** (for Glue and Lambda) to ensure visibility into job failures.
    - Wrap significant operations (e.g., data loading, transformation, API calls) in try-except blocks.
    - Use detailed error messages with context (e.g., job name, step, or dataset). 
    - Log errors in **CloudWatch Logs** for Glue and Lambda to facilitate debugging and monitoring.

- **Logging and Monitoring**:  
	- CloudWatch Logs** should be integrated for all Glue and Lambda jobs to track execution and monitor for failures.
    - Ensure that both **Glue** and **Lambda** jobs output logs to CloudWatch. This helps you monitor job status, track execution time, and troubleshoot errors.
    - Use structured logging (JSON format) for easy parsing and monitoring.
    - Set up **CloudWatch Alarms** to receive notifications for failures, retries, or specific thresholds (e.g., job duration, error rates).

- **Modularization**:  
    - Split your code into smaller, logical modules to enhance **reusability** and **maintainability**. 
    - For example:
	    - Separate the **ETL pipeline** into distinct modules for **extraction**, **transformation**, and **loading**. This makes it easier to modify or extend any part of the pipeline without affecting the entire process.
	    - Use Python functions or classes to organize reusable code.
	    - Example: `etl.py`, `extract.py`, `transform.py`, `load.py`

- **Parameterization and Configuration**:  
    - Manage environment-specific configurations and sensitive data effectively.
    - Store **sensitive data** (e.g., API keys, database credentials) securely in **AWS Secrets Manager** to avoid hardcoding them in the code. Fetch the secrets programmatically when required.
    - Manage **environment-specific configurations** (e.g., S3 bucket names, database connections) using external configuration files like **YAML** or **JSON**. 

- **Performance Considerations**:  
    - Optimize the performance of your jobs by using efficient algorithms, minimizing data shuffling, and applying parallel processing where possible.
    - For **Glue**, adjust the **DPUs** to balance job resource allocation and execution time. Monitor the **DPU usage** and adjust accordingly.
    - In **Lambda**, monitor the **memory allocation** and **timeout settings** to ensure jobs complete within the required time limits.

---
## 1. Code Readability, Structure, and Best Practices

- **Follow Standard Style Guides**: Adhere to Google Style or PEP 8 guidelines. For example, maintain clear, descriptive naming conventions for functions, variables, and classes. Always use docstrings for modules, classes, and functions.
    - Example: For a Glue job that loads data from S3, provide clear documentation of inputs, outputs, and dependencies.
- **Descriptive Naming and Modularization**:
    - Keep your code modular, breaking down large scripts into smaller files or modules. For instance, separate the data extraction, transformation, and loading (ETL) steps into their respective scripts (e.g., `extractor.py`, `transformer.py`, `loader.py`).
    - For Lambda functions, use helper functions and make them reusable by separating logic into distinct modules.
- **Consistent Code Layout**:
    - Ensure consistent structure in all Glue and Lambda jobs. A typical Python-based Glue job should follow:
        1. **Module-level docstring** describing the Glue job’s purpose.
        2. **Imports** (standard libraries, AWS SDKs, third-party libraries).
        3. **Constants and configurations** (e.g., S3 bucket names, Glue connection strings).
        4. **Core functions/classes** implementing the main logic.
        5. **Main function** or **entry point** under `if __name__ == "__main__":`
---
## 2. Error Handling, Logging, and Monitorin

### 2.1 Structured Error Handling
- Use try-except blocks to handle errors gracefully and log them using structured formats. For Glue and Lambda jobs, use **CloudWatch Logs** for logging and monitoring errors.
    - Example in Glue job:
        ```python
        import logging
        logger = logging.getLogger(__name__)
        logger.setLevel(logging.INFO)
        
        try:
            # Glue Job Logic
            logger.info("Data loaded successfully")
        except Exception as e:
            logger.error(f"Glue job failed with error: {str(e)}")
            raise
        ```

---
### 2.2 CloudWatch Integration

To effectively monitor and respond to errors and performance issues in AWS Glue jobs and Lambda functions, use **CloudWatch Logs** and **CloudWatch Alarms**. Follow these steps to set up both standard error logging and custom error metrics.

#### 2.2.1 Set Up CloudWatch Logs for Glue and Lambda
- **AWS Glue**:
	- In the Glue Console, under your job’s **Monitoring options**, ensure that **CloudWatch Logs** are enabled.
	- When setting up the Glue job, you can specify the **Log Group** and **Log Stream** for the logs to be saved.
	- Glue will automatically push logs to CloudWatch, and you can monitor these logs using CloudWatch Logs insights.
- **AWS Lambda**:
	- In the Lambda Console, under **Configuration**, go to the **Monitoring and Operations** section.
	- Enable **CloudWatch Logs** for the Lambda function.
	- AWS Lambda will automatically send logs to CloudWatch Logs for each invocation of the Lambda function. You can configure the **Log Group** and **Log Stream** names.

---
#### 2.2.2 Set Up CloudWatch Alarms for Critical Error Thresholds
CloudWatch Alarms can be configured to notify you of any failures, retries, or performance issues based on the metrics logged by Glue or Lambda. Below are the steps to set up alarms for critical thresholds.

- **Glue Job Alarm Setup:**
	- Go to CloudWatch > Alarms > Create Alarm.
	- Select the Glue Job Metrics from the Glue namespace.
		- Failed Jobs: This metric counts the number of failed job runs. You can set an alarm for the threshold of failed job runs.
		- Job Runs: This metric counts the total number of job runs. Set a threshold for a sudden increase in job runs or errors.
	- Choose the metric you want to monitor (e.g., Failed Jobs) and select the statistic you wish to use (e.g., Sum, Average).
	- Set the condition (e.g., if the failed jobs count is greater than 0 for the past 5 minutes).
	- Configure the alarm action (e.g., send a notification to SNS or email).
	- Provide a name for the alarm and review.

- **Lambda Alarm Setup**:
	- Go to **CloudWatch** > **Alarms** > **Create Alarm**.
	- Select **Lambda Metrics** from the **AWS/Lambda** namespace.
	    - **Errors**: This metric tracks the number of errors (e.g., invocation errors) for Lambda functions. Set an alarm to notify you when errors exceed a threshold.
	    - **Duration**: Monitor the function execution time to identify long-running Lambda invocations.
	    - **Throttles**: Set up alarms for throttling (when the Lambda function is rate-limited).
	- Choose the appropriate metric (e.g., **Errors**) and define the statistic (e.g., **Sum**).
	- Set the condition (e.g., if the errors exceed 3 in the last 5 minutes).
	- Configure the alarm action (e.g., send an SNS notification or email).
	- Name the alarm and review.

---
#### 2.2.3. Setting Up Custom Error Metrics for Lambda and Glue

- **Custom Error Metrics for Lambda**
	- Use the `PutMetricData` API to emit custom metrics to **CloudWatch** for tracking specific errors or retries.
	- Example: Emit a metric `LambdaErrorCount` whenever an error occurs in a Lambda function.
	- Steps:
	    1. Enable CloudWatch logging for the Lambda function.
	    2. Add a call to `PutMetricData` in the error-handling logic to emit a custom metric.
	    3. Use CloudWatch Alarms to monitor the emitted metric and trigger notifications if thresholds are crossed.

- **Custom Error Metrics for Glue**
	- AWS Glue lacks native support for custom metrics; use **CloudWatch Logs** to analyze job execution data and emit custom metrics via AWS Lambda.
	- Steps:
	    1. Enable CloudWatch logging for the Glue job.
	    2. Use a Lambda function to process Glue logs and identify error patterns (e.g., "Exception", "Failed").
	    3. Emit custom metrics (e.g., `GlueJobErrorCount`) to **CloudWatch** using the `PutMetricData` API.
	    4. Configure CloudWatch Alarms on these metrics for proactive monitoring.
---

### 2.3 Glue Job Monitoring and Failure notifications

Why?
- **Custom Notifications**: Glue job state changes are sent to your email.
- **Proactive Monitoring**: Failures and successes are logged and notified.
- **Event-driven Alerts**: Automated workflows handle Glue job monitoring.

To monitor **AWS Glue jobs** and receive notifications for job state changes (e.g., success or failure), follow these summarized steps.
#### 1. Create an SNS Topic
1. Go to the **SNS Console**.
2. Create an SNS topic and note the ARN.
3. Add an **email subscription** to receive notifications:
    - Protocol: **Email**
    - Endpoint: Your desired email address.
#### 2. Create a Lambda Function for Notifications
1. Navigate to the **Lambda Console** and create a new function:
    - Runtime: **Python 3.x**.
    - Attach an execution role with `SNS` and `CloudWatch` permissions.
2. Paste the following example code to notify based on Glue job state:
    
    ```python
    import json
    import boto3
    import datetime
    
    sns_arn = "arn:aws:sns:your-region:your-account-id:your-topic-name"
    
    def lambda_handler(event, context):
        client = boto3.client("sns")
        state = event['detail']['state']
        timestamp = datetime.datetime.strptime(event['time'], '%Y-%m-%dT%H:%M:%SZ') + datetime.timedelta(hours=5, minutes=30)
        
        if state == 'FAILED':
            message = f"""
            Hi Team,
            Glue Job Failed:
            - Job Name: {event['detail']['jobName']}
            - Severity: {event['detail']['severity']}
            - Job State: {state}
            - Job Run ID: {event['detail']['jobRunId']}
            - Message: {event['detail']['message']}
            - Time of Failure: {timestamp}
            """
            client.publish(TargetArn=sns_arn, Message=message, Subject="Glue Job Failure Alert")
        
        elif state == 'SUCCEEDED':
            message = f"""
            Hi Team,
            Glue Job Succeeded:
            - Job Name: {event['detail']['jobName']}
            - Job State: {state}
            - Job Run ID: {event['detail']['jobRunId']}
            - Time of Completion: {timestamp}
            """
            client.publish(TargetArn=sns_arn, Message=message, Subject="Glue Job Success Notification")
        return {"statusCode": 200, "body": json.dumps("Notification Sent")}
    ```
    
3. Update the SNS ARN and add custom layers (if required).
#### 3. Create an EventBridge Rule
1. Go to the **EventBridge Console** and create a rule:
    - Event Source: **Glue Job State Change**.
    - Event Pattern:
        ```json
        {
          "detail-type": ["Glue Job State Change"],
          "source": ["aws.glue"],
          "detail": {
            "jobName": ["your-glue-job-name-1", "your-glue-job-name-2"],
            "state": ["FAILED", "SUCCEEDED"]
          }
        }
        ```
    - Target: Select the **Lambda function** created earlier.

#### 4. Add Trigger in Lambda
1. Go to the Lambda function created earlier.
2. Under **Triggers**, add the **EventBridge rule** created in the previous step.

---
## 3. Parameterization and Configuration Management

- **Use Secrets Manager**: Store sensitive information such as database credentials, API keys, and access tokens in AWS Secrets Manager. This ensures that secrets are encrypted and managed securely, preventing accidental exposure in code. Fetch secrets dynamically in your Glue job or Lambda function, as shown below:
    ```python
    import boto3

	# Create a Secrets Manager client
	secrets_client = boto3.client('secretsmanager')
	
	# Fetch secret value
	secret = secrets_client.get_secret_value(SecretId="my_secret")
	secret_value = secret['SecretString']
    ```

- **Environment-Aware Configurations**: Store environment-specific configurations (like S3 bucket names, database connections) in separate config files (`dev_config.yaml`, `prod_config.yaml`) or environment variables.

---
## 4. Glue Job and Lambda Function Configuration

### 4.1 Basic Config Settings for Glue Jobs
When setting up AWS Glue jobs, consider the following configuration options to ensure efficient execution and resource management:
- **IAM Role**:  
    - The Glue job needs an **IAM role** with appropriate permissions to access AWS services such as S3, RDS, and any other services your job interacts with. Ensure the role has policies that allow actions like `s3:GetObject`, `s3:PutObject`, and `rds:DescribeDBInstances`. You can create a dedicated IAM role for Glue jobs or reuse existing roles depending on your security policy.

- **Job Type**:  
    Choose the correct job type based on the processing required:
    - **Spark**: Ideal for large-scale distributed processing with Apache Spark. This is suited for batch processing or data transformation tasks where you need to scale across multiple workers.
    - **Python Shell**: For simpler Python-based ETL jobs, data extraction, or transformation that don’t require distributed processing.
    - **Scala**: Similar to Spark jobs but using Scala as the programming language, often chosen for high-performance ETL jobs.

- **Timeouts**:
    - The default timeout for a Glue job is **48 hours**. This is suitable for long-running jobs but may need adjustment based on your specific use case. You can increase or decrease this timeout depending on job complexity and expected run time.
    - You can modify the timeout via the Glue console or using the AWS CLI or SDKs

- **Max Concurrent Runs**:  
    - Control the maximum number of concurrent job runs to manage resource usage and avoid overloading the system. You can specify the maximum number of concurrent runs in the job configuration settings. If multiple jobs can run in parallel without impacting performance, increase this value. Otherwise, set it to 1 or a low number to prevent resource contention.

- **DPU (Data Processing Units) Configuration for Glue Jobs**
    - A **DPU** is a unit of computational power allocated to Glue jobs, which is a combination of CPU, memory, and storage. DPUs are critical for determining the performance and cost of Glue jobs. A DPU is a relative measure of processing power that consists of 4 vCPUs of compute capacity and 16 GB of memory. 
    - The allocation of DPUs depends on the type of job (Spark or Python shell) and the scale of data you are processing:
	    - **Python Shell Jobs**: For simpler ETL tasks, you may not need many DPUs. The minimum is 0.1 DPUs, but you can increase this up to 1 DPU for better performance or handling larger datasets.
	    - **Spark Jobs**: For Spark-based transformations, you can choose from **2-100 DPUs** based on the amount of data processed. A higher number of DPUs allows for parallel processing and can significantly reduce the time required to process large datasets.

- **Python Script vs Spark Script for Glue Jobs**
	- **Python Script**:  Python scripts in Glue are typically used for jobs that require simpler transformations and data extraction processes. These scripts are executed in a single-threaded environment with access to Glue's built-in libraries (like `boto3` for S3 interaction). Python shell jobs are generally used for small to medium-sized datasets
	- **Spark Script**:  Spark scripts in Glue are used for distributed processing across multiple nodes. These are ideal for large-scale transformations where parallelization can significantly reduce processing time. Spark jobs can run on multiple DPUs, making them more suitable for handling large datasets.

---
### 4.2 Basic Config Settings for Lambda Functions
When configuring Lambda functions, several settings affect the function's performance and cost efficiency. Below are key configuration options you should consider:
- **IAM Role**:  
	- Ensure that the Lambda function has an **IAM role** with the necessary permissions to access AWS resources like S3, DynamoDB, or RDS. 
	- This role should be attached to the Lambda function for it to authenticate and authorize requests to other services.

- **Memory**:  
    - Lambda allows you to allocate memory between **128 MB and 10,240 MB** (10 GB) in increments of 1 MB. 
    - The amount of memory you assign impacts both the CPU power and the cost of running the function. 
    - For CPU-intensive tasks, allocate more memory to speed up the execution, as Lambda allocates CPU resources based on the amount of memory selected. 
    - For I/O-bound tasks, such as interacting with databases, a smaller memory allocation may suffice.
    - To modify memory - Go to your Lambda function, click on **Configuration**, and modify the **Memory** setting.

- **Timeout**:  
    - The default timeout for Lambda functions is **3 seconds**, which might be insufficient for more complex tasks. 
    - The maximum timeout is **15 minutes (900 seconds)**. 
    - Ensure that you set the timeout according to the nature of your Lambda function’s workload.
    - To modify - Go to the Configuration tab, set the Timeout value based on your expected execution time.

- **Concurrency**:
	- Lambda automatically scales based on the number of incoming requests, but you can set a **reserved concurrency** to ensure a minimum number of concurrent executions for your Lambda function. 
	- Additionally, setting **provisioned concurrency** ensures that a certain number of Lambda instances are pre-warmed and ready to handle requests, reducing latency.

- **Environment Variables**:  
	- Use **environment variables** to manage configuration values, such as database connection strings or API keys, that might change between development, staging, and production environments. 
	- These values can be securely retrieved inside the Lambda function, and you can also use services like **AWS Secrets Manager** for securely storing sensitive information.

---
## 5. Networking and VPC Configuration

### 5.1 VPC Configurations for Glue

When configuring AWS Glue to access private resources, like data sources within a VPC (e.g., RDS, Aurora), there are several key considerations:

- **Glue in a VPC**:  AWS Glue jobs can run within a VPC, allowing them to access resources that are not publicly accessible. To configure Glue jobs to run in a VPC:
    - Specify the **VPC**, **subnets**, and **security groups** for the Glue job.
    - Ensure that Glue is placed in subnets with the necessary **route tables** and access to other resources like RDS/Aurora.

- **VPC Endpoints**:
    - **S3 VPC Endpoint**: If your Glue job needs to read or write to S3 while running within a VPC, use a **S3 VPC Endpoint**. This keeps the traffic between Glue and S3 within the AWS network, avoiding the public internet. For S3, **Gateway VPC Endpoint** should be used.
    - **Glue VPC Endpoint**: To ensure secure communication between your Glue job and the Glue service, create a **Glue VPC Endpoint**. This ensures that the Glue job communicates privately without routing traffic over the public internet. For Glue, **Interface VPC Endpoint** should be used.

- **Security Groups and Subnets**:
    - Ensure that the **security group** associated with the Glue job allows inbound and outbound traffic to the private data sources (e.g., RDS, Aurora).
    - The **subnet** in which Glue runs should have access to the private network where your resources reside, including proper **route tables** and **network ACLs**.

---
### 5.2 VPC Configurations for Lambda

Lambda functions can also run within a VPC to access private resources. The configuration steps are similar to those for Glue but tailored to Lambda’s specific needs.

- **Lambda in a VPC**:  When Lambda functions need access to private resources (e.g., RDS, Aurora), configure the function to run within the same VPC:
    - Specify the **VPC**, **subnets**, and **security groups** that will allow Lambda to securely access the required resources.
    - Lambda functions that run in a VPC can access the VPC's resources (e.g., RDS, EC2), but you must ensure that the function is granted the appropriate access permissions and network configurations.

- **Security Groups**:
    - Assign a **security group** to the Lambda function that permits traffic to the required resources in your VPC.
    - Ensure the **security group** associated with the Lambda function allows outbound traffic to the necessary resources (e.g., RDS, Aurora) within the VPC.

---
### 5.3 VPC Endpoints

For both Glue and Lambda, setting up VPC endpoints ensures secure, private communication between services within your VPC without routing traffic over the public internet. Here are the key VPC endpoints:

- **S3 VPC Endpoint**:  
	- S3 VPC endpoints allow Glue and Lambda to interact with S3 without going through the public internet. 
	- This improves security and performance by keeping traffic within the AWS network.
	- **Endpoint Type**: **Gateway VPC Endpoint** for S3.
    - **How to Create an S3 VPC Endpoint**:
        - Go to the **VPC console** > **Endpoints** > **Create Endpoint**.
        - Select **S3** as the service and choose your VPC, the appropriate subnets, and security groups.

- **Glue VPC Endpoint**:  
	- AWS Glue can communicate securely within a VPC through a Glue VPC endpoint. 
	- This ensures that the Glue job can access data sources, such as RDS, or perform transformations privately.
	- **Endpoint Type**: **Interface VPC Endpoint** for Glue.
    - **How to Create a Glue VPC Endpoint**:
        - Navigate to the **VPC console** > **Endpoints** > **Create Endpoint**.
        - Select **Glue** as the service and configure it with your VPC, subnets, and security groups.

- **Secrets Manager VPC Endpoint**:  
	- When using AWS **Secrets Manager** to store sensitive information (e.g., database credentials), it is advisable to set up a **Secrets Manager VPC endpoint** to avoid exposing sensitive traffic to the public internet.
	- **Endpoint Type**: **Interface VPC Endpoint** for Glue.
    - **How to Create a Secrets Manager VPC Endpoint**:
        - Go to the **VPC console** > **Endpoints** > **Create Endpoint**.
        - Choose **Secrets Manager** as the service, and configure it with your VPC, subnets, and security groups.

---

## 6. Retry Mechanisms

### 6.1 Retry Mechanisms in Glue
- **Automatic Retries**:  
    AWS Glue jobs support automatic retries for transient failures (e.g., network issues, temporary service unavailability). By default, Glue will retry a failed job **2 times**.
    - **To configure retries**: In the **Glue job settings**, under **Job Details**, adjust the **Retry policy** and set the number of retries. If your job is expected to fail intermittently, you can increase the number of retries to ensure the job completes successfully after multiple attempts.
    - **Types of Failures**: Glue retries are useful for errors that are temporary and can be resolved with a retry, such as network failures or minor issues with downstream systems.
- **Custom Retry Logic**:  
    For more granular control, you can implement custom retry logic using **AWS Step Functions** to orchestrate Glue jobs. Step Functions provide more flexibility in controlling retries, including exponential backoff strategies and handling different error types differently.
---
### 6.2 Lambda Retry Mechanisms
- **Asynchronous Invocations**:  
    - For Lambda functions invoked asynchronously (e.g., via S3 events, SNS notifications), AWS automatically retries the invocation **2 times** within a 7-day period if the function fails. 
    - The retry is done with an **exponential backoff** approach.
    - **To configure retry settings** for asynchronous invocations, use the **EventBridge** or **SNS** to configure retry policies at the source of the event trigger.
- **Synchronous Invocations**:  
    - For synchronous invocations (e.g., via API Gateway or direct calls), Lambda does not automatically retry failed executions. 
    - To handle retries in these cases, implement retry logic manually within your Lambda function. 
    - This could involve catching specific exceptions and calling the function again after a delay or using a service like **Step Functions** to manage retries based on business logic.

---

## 7. Managing Python Packages

For more reference: [Using Python libraries with AWS Glue - AWS Glue](https://docs.aws.amazon.com/glue/latest/dg/aws-glue-programming-python-libraries.html)

### 7.1 Installing Supported Python Packages
- **AWS Glue's Default Library Set**:  
	- AWS Glue comes pre-installed with several common Python libraries, such as **pandas**, **numpy**, and **boto3**. 
	- These libraries are available for use without any additional configuration.
- **Adding Additional Supported Packages**:  
    - If you need to use additional libraries that are supported by Glue but not included by default, you can specify their path in the **Python library path** during job configuration.
    - For example, if your package is available in **PyPI**, you can specify the package name and it will be installed when the job runs.
    - To include these packages, either:
        - Use the **Glue job script editor** to provide a list of Python dependencies in the job settings (via `--additional-python-modules`).
        - Or, if you're using a script file, simply add the necessary package via `pip` in your script setup.

---
### 7.2 Installing Non-Supported Python Packages

- If the Python package you need is not natively supported by AWS Glue, you can still use it by uploading the package to **S3** as a `.whl` (Wheel) file and referencing it in your Glue job. 
- This process requires internet access to download the package and its dependencies. However, if the Glue job is running inside a **private VPC**, there could be connectivity issues, as it might not have direct access to the public internet. 
- Since the package gets installed every time the job runs, the installation step might add unnecessary overhead to the job's runtime, especially for large packages or if the installation process is slow.

**Steps:**
- **Create a Wheel File**:
    - First, package the library you want to use into a `.whl` file. This can be done using the following steps:
        ```bash
        pip install <package_name> --target /path/to/directory
        cd /path/to/directory
        zip -r package.zip *
        ```
        - Then convert the package into a `.whl` file using tools like `python setup.py bdist_wheel`.
- **Upload to S3**:
    - Upload the `.whl` file to an S3 bucket. For example:
- **Configure Glue Job to Use the Wheel File**:
    - In the Glue job configuration, specify the **S3 path** to the wheel file under the **Python library path**:
        - In the AWS Glue console, under **Job Details**, find the **Python library path** and add the S3 path of the `.whl` file.
        - Alternatively, you can pass the path to the `.whl` file through the **--extra-py-files** argument if you are running the Glue job through the CLI.

---