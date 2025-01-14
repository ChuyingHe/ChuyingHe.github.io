# 1. The Cluster Update Process
With the new distribution system **Red Hat Enterprise Linux CoreOS**, oc cluster can perform **Over-the-Air updates (OTA)**. 

OTA:

- substantially decreases downtime due to upgrades
- a cluster can use new features and new bug fixes when they are available.
- you can update faster by skipping intermediate versions. `4.14.1` -> `4.14.3` directly


!!! warning
    Starting with OpenShift 4.10, the OTA system requires a persistent connection to the internet.


UI acccess: [https://console.redhat.com/openshift](https://console.redhat.com/openshift). The console looks like this:

<img src="../imgs/updates-console.png" />

## Concepts
|Concept|Explanation|
|:-|:-|
|**upgrade path**|cluster eligibility for certain updates, belongs to update **channel**|
|**channel**|representation of the upgrade path. The **channel** controls the frequency and stability of updates. A **channel** name consists of the following parts: <br/>1. the tier (`candidate`, `fast`, `stable`, and `eus` - extended update support)<br/>2. the major version (4)<br/>3. the minor version (.12)<br/><br/>Examples: `candidate-4.14`, `fast-4.14`, `stable-4.14`, `and eus-4.14`|


# 2. Detect Deprecated Kubernetes API Usage
# 3. Update Operators with the OLM