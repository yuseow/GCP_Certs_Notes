# [WIP] GCP Data Engineering Notes

## Data Lifecycle and corresponding GCP services
**Ingest** = Pub/sub, Dataflow (Google's Apache Beam), Composer, Storage Transfer Service, Application logs from App Engine/Compute Engine/ GKE

**Store** = Cloud storage, BigQuery, Bigtable, Cloud SQL, Firestore, Memorystore, Spanner

**Process/Analyse** = Dataflow (Google's Apache Beam), Dataproc, BigQuery, Data Fusion, Dataprep, Data Loss Prevention API (DLP)

**Explore/Visualise**= Datalab, Looker Studio, custom dashboards, etc

## Types of datas
1. Structured
- Highly organised, easily accessed, processed, analysed
- Typically stored in relational databases
- Adheres to pre-defined format or schema
- e.g. transaction records, CRM system
  - GCP product for it: BigQuery, Cloud SQL, Spanner
2. Unstructured
- "free-form", lacks a predefined structure or schema
- wide range of formats and types
- e.g. text data, image, video data
  - GCP product for it: Cloud Storage
3. Semi-structured
- No fixed schema or structure, but maybe some metadata that describes the data and makes it easier to process
- e.g. JSON, XML, YAML, sometimes data from NoSQL databases
  - GCP product for it: Bigtable, Firestore, Memorystore


ACID Compliance in a database?:
The presence of four properties — 
1. atomicity: broken down into smaller parts
2. consistency: only data which follows the appropriate data validation rules is permitted to be written to the database
3. isolation: ability to concurrently process multiple transactions in a way that one does not affect another
4. durability: make failures invisible to the end-user, e.g. in databases that possess durability, data is saved once a transaction is completed, even if a power outage or system failure occurs.

[Link to explanation](https://mariadb.com/resources/blog/acid-compliance-what-it-means-and-why-you-should-care/)

## Apache vs GCP tools
| Open Source                        | GCP Tool             |
|-------------------------------------|----------------------|
| Apache Kafka                        | Cloud Pub/Sub        |
| Apache Beam                         | Cloud Dataflow       |
| Apache Hadoop and Apache Spark      | Cloud Dataproc       |
| Apache HBase                        | Cloud Bigtable       |
| Apache Hive                         | BigQuery             |
| Apache Airflow                      | Cloud Composer       |


## Types of windows
Windows are good to group streaming data for processing. Group elements by timestamp. 
When your data is generated, it will have a timestamp. But the time when the data is generated may not be the time that your pipeline receives that data. Hence there is a need to separate "event time" from "processing time".

- Fixed Windows
- Sliding Windows
- Session Windows


## IAM - Identity Access Management
Principal = entity that can be granted access to GCP resources

Typers of Principals:
1. User accounts (associated with a human, used for accessing cloud services, authentication done via username/pwd)
2. Service accounts (used by applications to enable automated/scheduled tasks, authentication done via keys/tokens)
   - 2 types of service accounts: google managed (automatically created by Google) or user-managed (created by user, usually has the domain @[project-id].iam
  
There's "roles" (collection of permissions), "permissions" (granular action that can be taken).
3 types of roles:
- basic roles: broad permissions at project level
  - owner
  - editor
  - viewer
- predefined: more specific and tailored for common use cases and job functions --> set and suggested by Google
  - Specific to a given GCP service or API, correlates to common roles/use cases, intended to minimise overhead while reducing security risk
- custom:
