# IRCTC Ticket Booking Sync to BigQuery

## Overview
This project streams IRCTC ticket booking data from Pub/Sub to BigQuery using a Dataflow pipeline with a Python User-Defined Function (UDF) for data transformation. The pipeline reads streaming data from a Pub/Sub topic, applies transformations, and loads it into a BigQuery table for analysis.

## Architecture
1. **Data Ingestion:** Mock ticket booking data is generated and published to a Pub/Sub topic.
2. **Data Transformation:** A Python UDF is used to transform data before loading it into BigQuery.
3. **Data Processing:** Google Cloud Dataflow processes the streaming data from Pub/Sub and loads it into BigQuery.
4. **Data Storage:** The transformed data is stored in a BigQuery dataset.

## Tech Stack
- **Google Cloud Platform (GCP)**
  - Pub/Sub (Message Queue)
  - Dataflow (Stream Processing)
  - BigQuery (Data Storage & Analytics)
  - Cloud Storage (Python UDF Storage)
- **Python** (Data Publishing, Transformation Logic)

## Project Setup

### 1. Create and Configure GCP Resources
- Enable necessary APIs: Pub/Sub, Dataflow, BigQuery
- Create a **Pub/Sub topic**
- Create a **BigQuery dataset and table**
- Upload the **Python UDF script** (`transform_udf.py`) to a GCS bucket

### 2. Publish Messages to Pub/Sub
#### Python Script (`publisher.py`)
```python
from google.cloud import pubsub_v1
import json
import time

def initialize_pubsub(project_id, topic_id):
    publisher = pubsub_v1.PublisherClient()
    topic_path = publisher.topic_path(project_id, topic_id)
    return publisher, topic_path

def publish_to_pubsub(publisher, topic_path, data):
    message_json = json.dumps(data)
    message_bytes = message_json.encode('utf-8')
    publisher.publish(topic_path, message_bytes)
    print(f"Published message: {data}")

project_id = "your-project-id"
topic_id = "your-topic-id"

publisher, topic_path = initialize_pubsub(project_id, topic_id)

data_samples = [
    {"ticket_id": "12345", "user": "John Doe", "train": "Rajdhani", "date": "2025-03-10"},
    {"ticket_id": "12346", "user": "Jane Smith", "train": "Shatabdi", "date": "2025-03-11"}
]

for data in data_samples:
    publish_to_pubsub(publisher, topic_path, data)
    time.sleep(1)
```

### 3. Transform Data using Python UDF (`transform_udf.py`)
```python
def transform_element(element):
    import json
    record = json.loads(element)
    record['booking_status'] = 'Confirmed' if 'ticket_id' in record else 'Pending'
    return json.dumps(record)
```

### 4. Set Up Dataflow Job
- Use the **"Pub/Sub to BigQuery using Python UDF"** template in Dataflow.
- Configure the following parameters:
  - **Input Pub/Sub Topic**: `projects/your-project-id/topics/your-topic-id`
  - **Output BigQuery Table**: `your-project-id:your_dataset.your_table`
  - **Python UDF GCS Path**: `gs://your-bucket-name/transform_udf.py`
  - **Worker Nodes**: 2 (or more, based on data volume)
  - **Disk Size**: 50GB (adjust as needed)

### 5. Verify Data in BigQuery
- Query the table in BigQuery to verify that transformed data is loaded correctly:
```sql
SELECT * FROM `your-project-id.your_dataset.your_table` LIMIT 10;
```

## Conclusion
This pipeline ensures real-time ticket booking data synchronization into BigQuery for further analytics and reporting. The combination of **Pub/Sub**, **Dataflow**, and **BigQuery** provides a scalable and efficient data processing solution.


