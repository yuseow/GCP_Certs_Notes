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

## Parallet Data Processing - Cloud Dataflow
- Dataflow is the GCP managed version of Apache BEAM (aka **B**atch/str**eam**). It is auto-scaling, serveless/no-ops.
- Natively integrates with Pub/Sub, BigQuery
- Connectors available for Bigtable, Apache Kafka
- Pipelines must be located with a single region

**Key Terms**
Element = single entry of data (i.e. a row of data)
PCollection = Distributed data set, data input and output
ParDo = a type of transform applied to individual elements. ParDo is good for filtering/extracting elements from a large group of data
Transform = processing operation applied to data


types of windows to handle streaming data:
1. Tumbling windows (aka fixed windows)
  - fixed duration, non-overlap, sequential
2. Hopping windows (aka sliding windows)
  - overlap, has a frequency (called a hop interval), fixed duration (e.g. each window is 30s, but refresh/hop every 10s)
  - for fast based, time sensitive data to ensure there is the most up-to-date data (aka stock analysis)
3. session-based windows
  - irregular/dynamic duration, based on how long the session is
  - as long as the event happened within a designated gap threshold (e.g. 5 min threshold) as long as the interval between each event is lesser than the threshold, it will be classified as the same session
  - e.g. clicks in a websession


**Other Dataflow concepts you should know**
- Watermarks = timestamps that keep track of progress in your pipeline
  - If a step fails or stalls, the watermark does not advance
  - Watermarks are timestamps that help track the progress of data processing in a pipeline. They essentially tell the system how far along the stream of data it has processed. If a step in your data pipeline fails or stalls, the watermark does not move forward, meaning that the system recognizes that certain data hasn't been processed yet. This helps ensure that no data is left behind or processed out of order.
  - Example: If your data is streaming from multiple sources with different latencies, the watermark ensures that you don’t process data from one source too early before data from slower sources has arrived.
- Triggers = conditions that determine when the aggregated results of data, such as those in a window, should be emitted/outputted, esp in systems with unbounded data
  - Especially important in unbounded/streaming data pipelines
  - Triggers are the conditions that determine when data (usually aggregated or transformed data) is outputted from the system, especially in streaming systems. Triggers decide at what point the aggregated data (like data grouped by time windows) should be emitted.In unbounded or streaming pipelines, you can't wait forever to process data, so you need to decide how often or under what conditions to output the results (e.g., after a certain time window, after receiving a certain number of elements, etc.).
  - Example: Imagine you're counting the number of events per minute (time window). A trigger might decide to emit the count every minute, or every time a batch of 100 events arrives, or some other custom condition. Triggers can be tuned to fit the needs of the pipeline (early results, late results, etc.).
  - Triggers are especially important in unbounded or streaming pipelines because these pipelines handle continuous data streams that don’t have a natural endpoint. Therefore, you need a way to decide when to finalize or partially emit results without waiting for all data to arrive (which might never happen).

## Scaling 
Horizontal scaling: increasing the number of nodes/workers (vs Vertical scaling which is to add more compute to existing nodes)

## Preventing Fusion
Fusion: Dataflow may combine several steps of the pipeline into one (do all the steps together)

However it might not always be efficient for fusion to happen cause it might think a task is smaller than it really is, and affect how it distributes tasks to the workers.

Techniques to overcome fusion (to optimise worker usage, hence speeding the ops up):
- GroupByKey + Ungroup: group the data and immediately separate it again. forces Dataflow to treat the data as more significant and not combine the steps
- SideInput: take the data an expand into a sideinput for another operations. forces Dataflow to take a detour rather than doing and combining the next steps. Refers to a secondary data source that is used alongside the main input in a transform. Allows additional data to be accessed and used within the transformation process
- Reshuffle: rearranges data, hence acts a break in the pipeline that stops Dataflow from combining steps. will help to ensure that each datapoint is unique?

## Dataflow Best Practices 
- **Catching errors**: Catch errors within the pipeline by using a try-catch, and then output the errors to a new PCollection. Send the PCollection somewhere, such as Pub/Sub, for later analysis.
- **Missing messages**: If you notice some messages are missing from your pipeline, you can run a batch of the streaming data and check the output.
- **Out-of-order data**: Use a combination of windows, watermarks, and triggers to logically separate your time windows, determine when the results were generated/submitted to Dataflow, allow stragglers to catch up, and decide when you should emit them.
- **Updating a pipeline**: When you need to update Dataflow pipelines with new code, do "update job." This technically creates a new job with the same name but a new jobID.
  - GCP will perform a compatibility check, so you have to make them compatible with a "transform mapping" JSON file that you provide. It maps old job transforms to new job transforms.
 
## When to use Dataflow vs Dateproc
Dataproc when:
- have dependencies on specific tools/packages in the Hadoop/Spark ecosystem
- favour a DevOps/hands-on approach to operations over a serverless one

Dataflow when:
- no existing Hadoop/Spark ecosystem
- prefer a serverless/hands-off approach to ops
- generally better option


# Dataproc: ondemand, managed version of Apache Hadoop and Spark
MapReduce: Hadoop function to distributed/parallel compute to process large data in parallel. 
- Map phase: input data divided into smaller chunks, each chunk processed by a map function that transform the data to a key, value pair
- Shuffle phase: key-value pairs are grouped by keys and sent to different reduce nodes
- Reduce phase: reduce function applied key-values pairs in the reduce nodes to aggregate the values together under the same key to produce a final results (combine all the values of same keys together)

**Storing data for Dataproc jobs**
- when moving from Hadoop or Spark to Dataproc, usually good to store data in Google Cloud storage and jobs on the Dataproc cluser
- Possible exceptions depending on the type of data you work with:
  - Apache HBase data to Bigtable (fully managed no-sql database service designed for large analytical and operational workload)
  - Apache Impala data to BigQuery (serverless multi-cloud data warehouse, sql-based)
 
**Configuring a Dataproc cluster**
Choose: 
- region & zone
- cluster mode (# of masters and # of workers)
  - Single node: 1 master, 0 workers
  - Standard node: 1 master, custom no. of workers
  - High availability: 3 masters, custom no. of workers
- Disk size for master and worker nodes
- Local SSD size (cannot be changed)
- no. of preemptible nodes/VMs
  - (aka on-demand VMs that are lower in cost than standard VMs but are short-lived and can be claimed by GCP whenever there is a demand from a more higher paying customer. need min 2 worker nodes. best used for compute-intensive tasks that are resilient to interruptions and do not require persistent storage. Dataproc manages the entire leave/join process of preemptible VMs without configuration required. will help jobs to reduce time
- cloud storage bucket for staging

After a Dataproc cluster is created, can change:
- no. of workers and preemptible VMs
- labels of cluster
- toggle graceful decommissioning
resharding of data is handled automatically when you update a dataproc cluster

**Cluster migration best practices**
- move the data first (usually GCS)
- perform small-scale testing on a sub-set of data first
- think in terms of ephermeral clusters (delete cluster when done)
- Use GCP tools to optimise and save costs
- eventual goal should be to move towards a cloud-native and serverless architecture

Type of Apache data to Dataproc:
HDFS => GCS
Hive => BigQuery
Impala => BigQuery
HBase => Bigtable

**Ways to optimisze Dataproc cluster performance**
- Place Dataproc cluster in same region as storage bucket that has your data (less egress/ingress)
- Increase size of persistent disk for better performance
- Consider SSD over HDD (more expensive though)
- Allocat more VMs (choose preemptible VMs to save on costs, but this option will cost more than increasing disk size though)

**Dataproc IAM roles**
- Editor (full access to create/delete/edit clusters, jobs, workflows)
- Viewer (view access only)
- Workers (for service accoounts to read/write cloud storage and write to cloud logging)
^ can only be defined on project level

## Google Cloud Storage (GCS)
Stores in buckets, all kinds of files are called "objects". Similar to blob storage/S3

**Storage class vc Costs:**
Consider storage duration and access frequency. Lower frequency access options, lower costs, but higher retrieval costs

- Standard: Access whenever, $0.02/GB --> frequency access
- Nearline: 30 days min storage (aka recommended to store at least 30 days. if you take your data out before that, you are still charged the full price), $0.01/GB
- Coldline: 90 days min storage, $0.004/GB
- Archive: 365 days min storage, $0.0012/GB --> least accessible, lowest cost but higher retrieval fee

**Bucket management and data locality**
- Buckets act as primary storage containers
- location options when setting up a bucket:
  - regional (will be in at least 2 separate zones in the same region)
  - dual-regional 
  - multi-regional (can be across diff countries, good for global companies)
- Optimised for latency and durability

**Accessing and using GCS**
- can use to access via: UI, API, SDKs, gsutil
- Features:
  - parallel uploads for large amounts of data --> optimise upload process and data transfer
  - requester-pays model good for sharing public datasets
  - integrity checks --> ensure data integrity with built in validation mechanisms. md5 crc32c hash can be pre-calculated and uploaded together for post upload verification (to ensure no data loss)
  - transcoding --> gzip transcoding to enable storage in compress format and automatically decompress on retrieval providing the users with the original file format
 
**Lifecycle Management**
- Transition rules based on conditions from 1 storage class to another (e.g. auto-archive older files)
- Cost reduction through automated management
- Deleting outdated or unnecessary data

**Access control**
- can have signed url access (time-based, got expiration, good for temporary access)
- IAM on project level
- ACL on specific buckets
- complete control over data access permissions

**Roles in data pipelines**
- can also automatically trigger processes in other GCP services (e.g. cloud functions, pub/sub)
- GCS as a dynamic component of data pipelines
- workflow initiation upon data arrival
  
# Bigquery (BQ)
- fully managed and serveless relational database
- BQ is a database optimised for extremely fast read operations and analytical queries.
- BigQuery is a fully-managed, serverless data warehouse that is designed to be extremely fast at read operations, particularly large-scale analytical queries. This is achieved through its columnar storage format and massively parallel processing.
- Updates, or writes, are slower in comparison to reads because BigQuery is optimized for append-mostly scenarios and is not intended for transactional workloads that require rapid updates. For use cases involving frequent writes or updates in a relational database, Google Cloud recommends using Cloud SQL or Spanner, which are optimized for transactional workloads and support high-throughput writes and updates.
- accepts both batch and streaming loads
- uses standard SQL (or legacy SQL, but not recommended for new projects. mainly there for backwards compatibility)

- The BigQuery Transfer Service is designed to automate data import from Software as a Service (SaaS) applications, specifically Google advertising platforms such as Google AdWords, DoubleClick, and YouTube, into BigQuery. This service simplifies the process of integrating advertising data into BigQuery, enabling seamless and recurrent data transfers. The service is particularly tailored to work with Google's advertising services, thereby making it easy to analyze advertising data alongside other data sources within BigQuery. This integration is not about exporting data from BigQuery, migrating on-premises databases, or backing up datasets, but rather about importing and synchronizing data from specific Google services.

- Federated table: query directly from external sources like google drive, cloud or BigTable. Hence able to have real time analysis without storage costs. need to set up a schema to map to the external data structure so BQ knows how to map the data. governed by the BQ access permissions
- Looker studio is GCP's free BI and data visualisation tool.
  - results generated in BQ can be immediately explored with charts
  - alternatively, set up BQ tables as data sources in Looker Studio and write queries/views in Looker Studio UI

**Ways to Access BQ**
- cloud console
- BQ command line tool
- client libraries (GO, python, Java, Node.js, PHP, Ruby, C#)

**Ways to Load data into BQ**
- **Web UI**: Upload small datasets directly.
- **BQ Command-Line**: Automate uploads from local or cloud storage.
- **Cloud Console**: Import via browser from local or Google Cloud Storage.
- **BigQuery API**: Stream in real-time or batch load programmatically.
- **Storage Transfer**: Move <ins>large volumes from online sources</ins> like Amazon S3.
- **Dataflow**: ETL operations and streaming data processing.
- **Data Transfer Service**: Schedule data imports from SaaS applications.
- **Cloud Dataprep**: Clean and prepare data for analysis.
- **Streaming Inserts**: Real-time data capture and analysis.
- **Database Transfer Service**: Migrate data <ins>from on-premises or Cloud SQL</ins>.
- **Federated Queries**: Query external sources without loading.

When considering which way to load data, consider the size of the data, the need for real time updates, and complexity of the transformations

**Nested tables**
- Nested tables is a structure that a single table can create sub-tables, allowing the storage of hierachical data in a single column. similar to having a table within a table.
- Hierarchical Data: Store complex data hierarchies within single rows.
- implemented using RECORD/STRUCT Data Types: Define sub-tables within a column. Define a schema within a schema where 1 field can contain a repeated set of fields. useful for 1:many relationships.
- Efficient Queries: Use **UNNEST** to flatten nested data for analysis.
- Performance Advantage: Columnar storage reduces data scan costs.
- Simplified Management: Avoids multiple tables and complex joins.
- JSON Integration: Seamlessly maps JSON structures to nested fields.

**Types of Backups**
1. Automatic Replication: Data is automatically replicated across multiple locations, ensures high availability and durability.
2. Disaster Recovery: Managed by Google for large-scale system failures, not intended for individual user errors oraccidental deletions.
3. Time Travel: Query snapshots of data from the past 7 days, restore tables to previous states within this period.
4. Snapshot Tables: Create manual snapshots for long-term backups, useful for archives beyond the 7-day time travel window.
5. Export Data for Backup: Export tables to Google Cloud Storage, Formats include CSV,JSON, and Avro.

Note: No traditional backup-and-restore functionality.

**BigQuery best practices**
- Instead of using SELECT*, specify the columns you need
- Pay attention to the size of the query before running it. BQ charges based on the size of the DATA PROCESSED
- Remember: LIMIT and HAVING do not reduce costs, but WHERE does (limit and having dont return all values but you are still charged for it)
- Establish the logic of youir query on small subsets of your data first
- Save intermediate tables and then query those instead
- Streaming ingest is expensive --> if real time analytics is not important, consider batch loading instead
- Better to have more, smaller tables than fewer, larger tables
- When joining tables, use INNER JOIN instead of WHERE. Using WHERE creates more variable combinations,more computation needed. Inner join can be more efficient in processing only the mapped rows
- For table design, better to have smaller more numerous tables than larger and fewer ones. smaller tables has better cost savings
- Use expiration settings for tables to delete old data that u don't need
- Take advantage of long-term storage data pricing for data that are not accessed regularly to reduce storage costs

**BigQuery admin console**
- slot capacity management: to monitor the capacity usage and the allocation of the capacity to ensure that all the applications have the resources they need. Important for flat rate pricing
- policy tag taxonomies: classify and manage access to data by defining and applying policy tags, control who has access to sensitive data
- Query INFORMATION_SCHEMA (its a metadatabase containing information about all other databases and tables) to get a sense of the performance of the jobs and how our queries are running over time.

**BQ Views**
In BigQuery, a view is essentially a virtual table created by a query. Unlike physical tables that store data, views do not store data themselves. Instead, they represent the result of a query and can be queried against as if they were tables. This "query within a query" functionality makes views particularly useful for presenting a specific view of the data without altering the underlying tables. They provide a level of abstraction and can be used to restrict access to certain data, thus enhancing data security and management.

1. Standard views: standard see the tables views, executes the underlying query-on-demand and show a virtual table representing stored SQL queries
2. Materialised views: cached query results for faster performance, good for queries that are often used. automatically updated when base tables changes
3. Authorised views: grant a specific view of the tables, control data access by excluding sensitive details. shares query results, not the raw data with the specified users
4. Logical views (BI engine): virtual schema within BI engine for optimised performance, ideal for BI tools and connected sheets

**BQ Roles**
PROJECT LEVEL:
- BQ Admin: full control over BQ resources
- BQ User: create datasets, manage jobs
- BQ Job User: run jobs & queries

DATASET LEVEL:
- BQ Data Owner: manage and share datasets/views

DATASET / TABLE LEVEL:
- BQ Data Editor: modify and delete table data
- BQ Data Viewer: read-only access to the tables/views

PROJECT/DATASET LEVEL:
- BQ Metadata Viewer: access dataset/table metadata

**BQ Slots**
- unit of compute used by BQ to execute queries
- key to determining performance and cost
- slots are automatically assigned to queries
- more slots = faster query execution but at higher costs

<ins>Managing costs</ins>: 
- can monitor the slots usage and adjust the queries or dataset partitions to ensure the slot utilisation is efficient. need to maintain cost and performance
- knowing how many slots ur queries need, can optimise the data processing and manage the budget accordingly


**Types of Slot Pricing Models**
1. **on-demand pricing (default)**: per per query based on <ins>amount of data processed</ins>
  - good for ad-hoc analytics, variable workloads
  - pay for what you use, good for sporadic queries, when no service level agreement for production workload
2. **flat-rate pricing**: purchase dedicated slot capacity
  - Good for high and predictable workloads, some production workloads

**FLAT-RATE CONFIGURATIONS**
- Enterprise edition reservations: reserving a fixed number of slots with autoscaling options, e.g. baseline reservation with the ability to autoscale for peak demands
- Autoscaling benefits:
  - flexibility: scale up during peak ties and scale down during low usage
  - cost efficiency: pay for extra slots only when needed
 
**Cost Optimization Strategies**
1. combining pricing models
  - mixed approach: flat-rate fee for predictable workloads and on-demand for ad-hoc queries. (e.g. reserve slots for predictable production jobs and use on-demand pricing for analytics queries)
2. Monitoring and adjustments
  - regularly review/monitor usage and adjust reservations and strategies accordingly using BQ's built-in tools to track and manage slot utilisation

<ins>**Exam tips here:**</ins>
- **node outage will not affect BQ data cause storage and compute are separate in BQ.** Explanation: BigQuery is designed with a separation of storage and compute resources, which means that the data stored is independent of the compute resources used to process queries. In the event of node outages, the data within BigQuery would remain unaffected because it is not stored on the compute nodes themselves. This architecture enhances the reliability of data storage and ensures that data is not at risk during compute resource issues. Other options, such as complete data loss or temporary data unavailability, would be consequences of a system where storage and compute are not decoupled. Slowing down query performance is more related to the availability of compute resources rather than the integrity of the data itself.

## Bigtable
- High performance, massively scalable NoSQL database
- Other examples of NoSQL DBs: Redis, MongoDB, Cassandra, HBase, Firestore
- Good for flexible data models, varied data types (aka the data doesn't always have the same keys/columns, unlike structured data)
- Good for rapid read and write operations, hence good for high throughput analytics like stock data, time series data. Tables that follow this pattern tend to be tall and narrow
- IT IS NOT NO OPS, NEED TO CONFIGURE THE INSTANCES etc
- Need to establish row key carefully. Good row keys distribute the load across Bigtable's nodes, bad row keys lead to hotspotting (disproportionate load is handled by a small number of nodes, potentially impacting performance)
- tables are sparse, empty cells table up no storage space. results in significant storage savings esp for datasets that have lots of optional fields
- stores in key-value pair (think dictionary)
- Developed internally for Google, then HBase (Hadoop, open source) was developed based on it. Now Bigtable works well with HBase, so devs can now migrate HBase workload to GCP managed Bigtable services without having to re-write their applications

**Bigtable instance configuration**
3 things to config when setting up:
1. Instance type, either:
 - Dev env suggestion: 1 node, low cost, no replication
 - Prod env suggestion: 1+ clusters, 3+ nodes per cluster (for high availability and redundancy), replication available, throughput guarantee
2. Storage type (HDD or SSD): SSD is amost always the right choice (quick retrieval and low latency operations), unless storing > 10TB of infrequently-access data and latency is not a concern
3. Zone for cluster to be in (affects latency for users and also data residency requirements)

Can interact wih Bigtable through cbt (commandline tool) or HBase shell. 
configuring it correctly is important for performance, scalability and cost.
