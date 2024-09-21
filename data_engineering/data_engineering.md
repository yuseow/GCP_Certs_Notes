# [WIP] GCP Data Engineering Notes

## Data Lifecycle and corresponding GCP services
**Ingest** = Pub/sub, Dataflow, Composer, Storage Transfer Service, Application logs from App Engine/Compute Engine/ GKE



## Types of windows
Windows are good to group streaming data for processing. Group elements by timestamp. 
When your data is generated, it will have a timestamp. But the time when the data is generated may not be the time that your pipeline receives that data. Hence there is a need to separate "event time" from "processing time".

- Fixed Windows
- Sliding Windows
- Session Windows
