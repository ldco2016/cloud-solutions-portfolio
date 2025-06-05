# Data Ingestion Methods

[![AWS](https://img.shields.io/badge/AWS-100000?style=flat&logo=amazon&logoColor=FFFFFF&labelColor=5C5C5C&color=FF7300)](https://docs.aws.amazon.com/quicksight/latest/user/signing-up.html)
[![DynamoDB](https://img.shields.io/badge/DynamoDB-4053D6?style=flat&logo=amazonaws&logoColor=white&labelColor=232F3E&color=4053D6)](https://aws.amazon.com/dynamodb/)
[![Kinesis Firehose](https://img.shields.io/badge/Kinesis_Firehose-FFA500?style=flat&logo=amazonaws&logoColor=white&labelColor=232F3E&color=FFA500)](https://aws.amazon.com/kinesis/firehose/)
[![S3](https://img.shields.io/badge/S3-569A31?style=flat&logo=amazonaws&logoColor=white&labelColor=232F3E&color=569A31)](https://aws.amazon.com/s3/)
[![Lambda](https://img.shields.io/badge/Lambda-FDDB33?style=flat&logo=amazonaws&logoColor=white&labelColor=232F3E&color=FDDB33)](https://aws.amazon.com/lambda/)
## OVERVIEW

Managing data streams can get expensive and you might need to preprocess your streaming data and run analytics in real time. Amazon Managed Service for Apache Flink is a great way to transform and analyze streaming data in real time. It also takes care of everyhing required to run streaming applications, and it scales automatically to match the volume and throughput of your incoming data.
There are no servers to manage, no minimum fee or setup cost, and you pay only for the resources your streaming applications consume.

Create an Amazon Kinesis Data Firehose delivery stream.
Ingest and store clickstream data in an Amazon S3 bucket.
Use Amazon Kinesis Data Analytics to preprocess data with AWS Lambda.
Configure real-time analytics to count active page views.
Configure a Kinesis Data Analytics application to send real-time analytics results to an AWS Lambda function that populates a DynamoDB table.

<p align="center">
  <img src="./img/1.png" alt="" style="display: block; margin: auto;" />
</p>

## Table of Contents

- [Requirements](#requirements)
- [Steps](#Steps)
- [Conclusion](#conclusion)
- [Contributors](#contributors)

## Requirements

To complete this quest, you will need access to the following AWS services:

- Amazon Kinesis Data Firehose
- Amazon S3
- Amazon Kinesis Data Analytics
- AWS Lambda
- Amazon DynamoDB

## Steps

### Step 1: This solution uses Amazon Data Firehose to ingest, transform, and make real-time data available for business analysis

<p align="center">
  <img src="./img/2.png" alt="" style="display: block; margin: auto;" />
</p>

### Step 2: Data Firehose reliably loads streaming data indo data lakes, data stores, and analytics services.

### Step 3: Amazon Kinesis is a fully managed service that automatically scales to match data throughput. Kinesis requires no ongoing administration.

### Step 4: A source application generates data from activities on a webpage. The data is represented by a sequence of clicks on each website link (clickstream data)

### Step 5: The data is sent to Data Firehose, which runs a custom transformation through an AWS Lambda function. The transformed data is sent to its destination.

### Step 6: For this solution, the destination is an Amazon Simple Storage Service (Amazon S3) bucket.

### Step 7: After the data is stored in Amazon S3, AWS Glue can be used to catalog the data and its schema, and continually make updates as new data arrives.

### Step 8: An AWS Glue crawler is used to discover new data and schema changes. After the crawler runs, the new data is ready to be queried by Amazon Athena.

### Step 9: Amazon Athena is an interactive query service that can be used to analuze data directly in Amazon S3 by using standard SQL

### Step 10: Another Lambda function is used to receive and save the data to an Amazon DynamoDB table, which serves as the data source for one or more analytics dashboard applications.

Open the AWS Lambda console.

Click Create function and choose Author from scratch.

Enter a name for your function (e.g., PreprocessClickstreamData) and choose Python 3.9 as the runtime.

Copy and paste the following code into the editor:

bash

```python
# Creating a Lambda package with runtime dependencies
# https://docs.aws.amazon.com/lambda/latest/dg/python-package-create.html#python-package-create-with-dependency
from datetime import datetime
import pandas as pd
import boto3
import os
import base64


s3 = boto3.client('s3')
data_bucket = os.environ.get('DATA_BUCKET_NAME')


def handler(event, context):
    """Example delivery stream record event
    {
        "invocationId":"00540a87-5050-496a-84e4-e7d92bbaf5e2",
        "applicationArn":"arn:aws:kinesisanalytics:us-east-1:12345678911:application/lambda-test",
        "streamArn":"arn:aws:firehose:us-east-1:AAAAAAAAAAAA:deliverystream/lambda-test",
        "records":[
            {
                "recordId":"49572672223665514422805246926656954630972486059535892482",
                "data":"aGVsbG8gd29ybGQ=",
                "kinesisFirehoseRecordMetadata":{
                    "approximateArrivalTimestamp":1520280173
                }
            }
        ]
    }
    """

    """Example response format
    {
        "records": [
            {
                "recordId": "49572672223665514422805246926656954630972486059535892482",
                "result": "Ok",
                "data": "SEVMTE8gV09STEQ="
            }
        ]
    }
    """

    # Response will be a list of records.
    response = {
        "records": []
    }

    # Clickstream data labels.
    labels = ['prev', 'curr', 'type', 'n']

    # Iterate list of input records.
    for record in event.get('records'):
        records = []

        # Get data from Kinesis record.
        data = base64.b64decode(record.get('data')).decode('utf-8')

        # Split into lines.
        lines = data.split('\n')

        for line in lines:
            # Skip any empty lines.
            if line == "":
                continue

            try:
                cols = line.split('\t')
                records.append((cols[0], cols[1], cols[2], cols[3]))
            except:
                continue

        # Convert to data frame.
        df = pd.DataFrame.from_records(records, columns=labels)

        response['records'].append({
            "recordId": record['recordId'],
            "result": "Ok",
            "data": base64.b64encode(df.to_csv(header=False, index=False).encode('utf-8'))
        })

    return response

```

<p align="center">
  <img src="./img/3.png" alt="" style="display: block; margin: auto;" />
</p>

### Step 4: Define a Kinesis Data Analytics SQL

The final step is to create an alarm for memory usage. Here are the steps to follow:


```SQL
-- Approximate distinct count  - Counts the number of distinct items in a stream using HyperLogLog.
-- Returns the approximate number of distinct items in a specified column over a tumbling window.
-- Note that when there are less or equal to 10,000 items in the window, the function returns exact count.
CREATE OR REPLACE STREAM DESTINATION_SQL_STREAM (NUMBER_OF_DISTINCT_ITEMS BIGINT);
CREATE OR REPLACE PUMP "STREAM_PUMP" AS INSERT INTO "DESTINATION_SQL_STREAM"
SELECT STREAM NUMBER_OF_DISTINCT_ITEMS FROM TABLE(COUNT_DISTINCT_ITEMS_TUMBLING(
  CURSOR(SELECT STREAM * FROM "SOURCE_SQL_STREAM_001"),
  'column1', -- name of column in single quotes
  60 -- tumbling window size in seconds
  )
);
  
  ```

## Conclusion

In conclusion, backing up your data is essential to ensure its safety and continuity in the event of an unexpected event. AWS provides multiple solutions for backing up your data, including creating a custom backup vault, configuring automated backup plans, and using tags to manage resources. With the knowledge gained from this guide, you can now confidently create and manage backup solutions for your AWS resources, ensuring that your data is always protected and available. Remember to regularly review and test your backup plans to ensure their effectiveness and make any necessary adjustments.

<p align="center">
  <img src="./img/4.png" alt="" style="display: block; margin: auto;" />
</p>


