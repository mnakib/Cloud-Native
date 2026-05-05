## What is Amazon Kinesis

Amazon Kinesis is a managed AWS service designed to ingest, process, and analyze real-time, streaming data at scale. It enables businesses to process high-volume, live data from sources like IoT devices, application logs, and website clickstreams, allowing for immediate insights rather than waiting for batch processing.


## 1. Create the Stream

Create a Kinesis Data Stream instance named __temperature-stream__ to host your streams.

Amazon Kinesis > Kinesis Data Streams > Create data stream


## 2. Create the Producer

To create a producure, you may generate random temperature data and send it to Amazon Kinesis, where you can use the `boto3` library, which is the official AWS SDK for Python.

Run below Python script on the AWS Cloud Shell.

```bash
cat > producer.py
```

```python
import boto3
import json
import random
import time

# Initialize Kinesis client and define stream name
kinesis = boto3.client('kinesis', region_name='us-east-1')
# Put your kinesis stream name here. For instance; 'temperature-stream'
STREAM_NAME = 'your-kinesis-stream-name'

def send_data():
    while True:
        data = {
            'sensor_id': f"sensor_{random.randint(1, 5)}",
            'temperature': round(random.uniform(20.0, 40.0), 2),
            'event_timestamp': time.strftime('%Y-%m-%dT%H:%M:%SZ', time.gmtime())
        }
        # Serialize and send to Kinesis
        kinesis.put_record(
            StreamName=STREAM_NAME,
            Data=json.dumps(data),
            PartitionKey=data['sensor_id'] # Ensures ordered data for same sensor
        )
        print(f"Sent: {data}")
        time.sleep(1)

if __name__ == "__main__":
    send_data()
````

Run the `producer.py` Python script to start injecting stream data into your Kinesis stream

```bash
python producer.py
```


## 3. Create a Consumer

### 3.1 Create a Managed Zeppelin Notebook 

   1. Navigate to the __Amazon Kinesis Console__.
   2. Select your stream; __temperature-stream__, and go to the __Data analytics__ tab.
   3. Choose Create notebook to launch a __Kinesis Data Analytics Studio__ environment.
   4. Once the status is Running, click __Open in Apache Zeppelin__. 

### 3.2 Create a Flink SQL Table Consumer

In a new Zeppelin note, use the `%flink.ssql` interpreter to define a table that maps to your Kinesis stream. This acts as the "consumer" that understands your data schema. 

```sql
%flink.ssql(type=update)

```sql
SHOW CATALOGS;
USE CATALOG default_catalog;
```
```sql
SHOW DATABASES;
USE default_database;
```
```sql
SHOW TABLES;
```
```sql
DROP TABLE IF EXISTS temperature_data_table;
```
```sql
CREATE TABLE temperature_data_table (
  sensor_id STRING,
  temperature DOUBLE,
  event_timestamp STRING
) WITH (
  'connector' = 'kinesis',
  'stream' = 'temperature-stream',
  'aws.region' = 'us-east-1',
  'format' = 'json',
  'scan.stream.initpos' = 'LATEST'
);
```
```sql
SHOW TABLES;
```
```

### 3.3 Extract and View Data

Once the table is created, you can run standard SQL queries to view the incoming records in real-time. Zeppelin will automatically generate interactive charts.

Once this is done, you may run different queries. 

* View raw records:

```sql
%flink.ssql(type=update)
SELECT * FROM temperature_data_table;
````

* Calculate average temperature per sensor:

```sql
%flink.ssql(type=update)
SELECT sensor_id, AVG(temperature) as avg_temp
FROM temperature_dataGROUP BY sensor_id;
````

If you encounter any issue reading the stream data, you might need to add this JSON policy to the role to grant the necessary access for kinesis Data Analysis Studio to your database and tables:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "KinesisConsumerAccess",
            "Effect": "Allow",
            "Action": [
                "kinesis:DescribeStream",
                "kinesis:DescribeStreamSummary",
                "kinesis:GetShardIterator",
                "kinesis:GetRecords",
                "kinesis:ListShards",
                "kinesis:RegisterStreamConsumer",
                "kinesis:SubscribeToShard"
            ],
            "Resource": [
                "arn:aws:kinesis:us-east-1:481207241221:stream/temperature-stream",
                "arn:aws:kinesis:us-east-1:481207241221:stream/temperature-stream/consumer/*"
            ]
        }
    ]
}
````




