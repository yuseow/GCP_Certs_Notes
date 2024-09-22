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
- custom: user defined (good for specified use cases e.g. for external auditors)

Groups:
- manage multiple members with similar or identical access needs, best practice is often to create a Google group, add each member to that group, and add the group to the IAM with the appropriate roles
- typically created with Google workspace domain or combination of environment + project/application + purpose/role. *<environment>-<application>-<role>@domain-name.com* e.g. dev-database-readers@finsecure.com

**Principle of least privilege**

Always allocate the minimum necessary permissions for someone or a service account to do what it needs. Depending on your tolerance for role maintenance, usually this involves using either custom roles or pre-defined roles.

Benefits:
- reduces the blast radius of compromised accounts
- mitigates risk of insider threats
- lowers chances of accidents
- simplified compliance and auditing
  
(if too much permissions is given, devs might accidentally push to wrong production env without testing or hackers can gain info and damage surface is higher)

**Gcloud commands**
- `Gcloud iam roles copy`: can be used to replicate IAM roles to another project, useful for custom roles that you want to keep consistent
- can use `gcloud ____ get-iam-policy` for various resources:
  - `gcloud projects get-iam-policy`: retrieves roles and members that have access to the project
  - `gcloud storage buckets get-iam-policy gs://my-bucket`: retrieves principals with access to the bucket
  - `gcloud compute instances get-iam-policy <instance-name> --zone=<zone>`: retrieves principals with access to the instance

## Pub/sub: publishers / subscribers
2 kinds of messaging systems setup:
1. **Tightly coupled systems**: sender directly tied to the receiver. This directness makes the system more likely to fail
2. **Loosely coupled systems**: have a buffer (or a message bus) between the sender and receiver. Loosely coupling the components makes the system more fault-tolerant and scalable. Allows for message queuing. --> Pub/sub is the buffer. it's the global scale messaging buffer/coupler

***Cloud pub/sub is Google's version of open source Apache Kafka***
- No-ops, serverless (aka we do not need to configure or manage this at all and it scales automaticly)
- often used for data ingestion, can connect to other data pipeline services such as Dataflow
- Delivery of each message is guaranteed at least once
- publishers does not need to wait until the subscribers are available, hence allow scalable and flexible communication between different parts of the application. especially useful for distributed systems or microservices where each component need to deliver information effectively without being directly connected

Terms:
- Publisher: an entity (e.g. microservice or application) that creates and sends messages to cloud pub/sub. doesnt need to know who will receive or process the messages
- Subscriber: an entity that receives and processes the messages. listens for new messages being receivd by pub/sub and tells pub/sub that it received the message
- Topic: a named resource which <ins>publishers</ins> sends messages. like a channel or category under which the messages are sent
- Subscription: a named resource representing the stream of messages from a specific topic to a <ins>subscriber</ins> like signing up to receive all messages posted to a particular topic

**Steps in pub/sub**
1. Topic created within Pub/Sub
2. Publisher sends a messenge to that topic, which Pub/Sub holds in queue to be served through a subscription
3. Subscriber either pulls the messages from the Pub/Sub topic or has the message pushed to them by a Pub/Sub
4. Subscriber acknowledges to Pub/Sub that it has received the message
5. Pub/Sub deletes the message after the subscriber has acknowledged it

**Push vs pull Subscriptions**
- Pull: good for batch delivery, large volume of messages
- Push: good for real-time streaming, low latency use cases
  - to do push sub, the subscriber must be a webhook endpoint that accepts POST over HTTPS

**Metrics for to monitor subscriber health**
1. Total number of messages in the Pub/Sub queue
- indicative of the overall load and throughput of the system. consistently high number can point to insufficient processing power on the subscriber's end or the publisher sending messages at a rate that the subscribers can't handle.
2. Oldest unacknowledged message

**Using Kafka with Pub/Sub**
- May need to use when there are hybrid workloads (connecting on-prem database to GCP), collect logs from on-prem kafka and send to GCP for analysis or have an app in GCP but alsp store data on-prem using Kafka
- GCP has a Pub/Sub Group Kafka Connector to connect Kafka to GCP. 2 versions of the connector, from the perspective of the role Pub/Sub plays in the setup
  - Source connector: reads messages from a Pub/Sub topic and publishes them to Kafka
  - Sink connector: reads messages from 1 or more Kafka topics and publishes them to Pub/Sub

**Additional Pub/Sub Feature**

**Seek** feature: Allows you to change the acknowledgement state of messages, including already-acknowledged messages in bulk. can replay previously acknowledged messages and snapshot to specific time
- Seeking to a <ins>snapshot</ins> allows you to return to the message acknowledgement state of a subscription
- Seeking to a <ins>time</ins> marks every message received before the time as acknowledged and every message after the time as unacknowledged
- can set a topic message retention policy. max and default is 7 days
