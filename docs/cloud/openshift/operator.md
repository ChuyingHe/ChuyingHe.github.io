## Operator
capability levels

<img src="./assets/operator-capability-level.png" width=600 />

|Capability Level|Explaination|Example(Database Operator)|
|:--|:--|:--|
|Level 1 - Basic Install ||an Operator deploys a database by creating `Deployment`, `ServiceAccount`, `RoleBinding`, `ConfigMap`, `PersistentVolumeClaim` and `Secret` object, initializes an empty database schema and signals readiness of the database to accept queries.|
|Level 2 - Seamless Upgrades ||An Operator managing a database can **update** an existing database from a previous to a newer version without data loss. The Operator might do so as part of a configuration change or as part of an update of the Operator itself.|
|Level 3 - Full Lifecycle ||an Operator managing a database provides the ability to create an application consistent **backup** of the data by flushing the database log and quiescing the write activity to the database files.|
|Level 4 - Deep Insights ||A database Operator continues to parse the logging output of the database software and **understands noteworthy log events**, e.g. running out of space for database files and produces alerts. The operator also instruments the database and exposes application level, e.g. database queries per second|
|Level 5 - Auto Pilot ||A database operator monitors the query load of the database and **automatically scales** additional read-only slave replicas up and down. The operator also detects subpar index performance and automatically rebuilds the index in times of reduced load. Further, the operator understands the normal performance profile of the database and creates **alerts on excessive amount of slow queries**. In the event of slow queries and high disk latency the Operator automatically transitions the database files to another PersistentVolume of a higher performance class.|


## S2I
### Method 1: OpenShift BuildConfig
### Method 2