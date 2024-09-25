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
Datapro when:
- have dependencies on specific tools/packages in the Hadoop/Spark ecosystem
- favour a DevOps/hands-on approach to operations over a serverless one

Dataflow when:
- no existing Hadoop/Spark ecosystem
- prefer a serverless/hands-off approach to ops
