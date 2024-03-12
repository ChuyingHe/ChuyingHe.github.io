# 2 types of Requirements
## Functional requirements
Functional requirements = What the system should do

- business flows: example, login, storing photos, receiving and crunching telemetry data
- business services: login service, data access service, telemetry receiver, telemetry credential.
- user interface: the look and feel, general guidance, responsiveness

!!! note
    For Architect: 

    - Functional requirements should not be ignored
    - but Non-functional requirements has even a higher priority

## Non-functional requirements
Non-functional requirements = What should the system deal with? 

Non-functional requirements describe the various aspects of the system's operation, and are not tied to a specific behavior or logic. The most common 5 non-functional requirements are:

1. performance
2. load
3. data volume
4. concurrent users
5. SLA 

Non-functional requirements basically describe what is expected environment for the system, with emphasis on edge cases, e.g.:
- Large number of concurrent users
- Server crash
- Extremely high load of requests

### 1. Performance: Latency & Throughput
!!! tip "Tipps"
    Its important to always talk in numbers: E.g.: How fast is fast? less than 1 sec or 1 ms

1. Latency: How much time does it take to perform a single task in the application?
    - E.g.: How much time does it need for the API to save the user data in the database?
2. Throughput: How many tasks can be performed in a given time unit?
    - E.g.: How many users can be saved in the database in a minute?
    - E.g.: How many files can be read in a second?


!!! note
    For human, one second or 700 milliseconds looks almost the same.

!!! tip "Latency VS Throughput"
    Let's say the **latency** of saving user data is one second. (This is quite slow.)
    
    Now, what would be the **throughput**? Can we know how many users can be saved in one minute? The answer is **NO**. 
    
    - If the application is well-designed, deployed on a strong hardware and knows its way around threads, it might have a throughput of 1000 users saved in one minute. 
    - If the code is buggy and there are a lot of memory leaks and no concurrency at all, we want to be able to reach a throughput of 60, which is a latency multiplied by 60, the number of seconds in a minute. 
   
### 2. Load
Quantity of work without crashing, it can be understood "approximately" as maximal Thoughput the system can handle. 

E.g.: for a web API-based application, the load will usually be defined as how many concurrent requests are going to be received by the system without crashing. 

!!! tip "Load VS Throughput"
    For example, the performance requirement can dictate **throughput** of 100requests/s, but the system should be able to handle 500 concurrent requests without crashing _--> even if those requests will take more than a second to complete._

This definition is important, because the worst thing that can happen to a system is to crash under heavy load.
.
!!! note "Example"
    for an e-commerce website, the regular load might be up to 200 concurrent requests, but on Black Friday, we are looking at more than 2000 concurrent requests. In that case, Load = 2000

### 3. Data volume
This requirement defines how much data in gigabytes or terabytes our system will _accumulate over tim_e. It's important for helping us to decide:

- what kind of database? since not all databases can handle large quantities of data equally. 
- what type of queries? because a query in a table of 100,000 rows will be completely different from a query in a table of 100 million rows.
- its helpful to plan ahead the storage we need to allocate.

The data volume usually has two aspects:

1. how much data is required on day one 
2. what is a forecasted data growth? For example, a system might need 500MB on its first day and is expected to grow by 2TB annually. 

### 4. Concurrent users
This requirement defines how many users will be using the system simultaneously. There is a big difference between **Concurrent users** and **load**. 

> The concurrent users requirement describes how many users will be using the system, not how many users will be performing requests. 

This distinction is important. When a user is using a system, there are a lot of dead times when no action is actually taken.

For example, a user is asking the system to display all the data. The system executes an API that goes to the database and retrieves the data. This is an actual action. Now, the user is looking at the data. During this time, the system is doing nothing. The API is not working. The database just sits there and the network is silent.

!!! tip "The rule of thumb"
    Concurrent users = 10 * Concurrent request

### 5. SLA (Service Level Agreement)
describes what is the required uptime for this system in percentage.

For example, Azure Cosmos DB takes pride with its 99.99% SLA. This is translated to less than an hour of downtime in a year.

!!! tip 
    One important thing to note about SLA is client expectations. 
    
    If you'll ask the client what is a required SLA for the system, he will usually give you an answer along the lines of 100% or the famous five nines, which is 99.999%. 
    
    When this happens, I usually tell him, no problem. For this, we'll need to build at least **3 data centers** in different continents with **independent and dual power stations** and **automatic failover** between them. What do you say? This generally brings him down to earth and we discuss more realistic SLA goals. 


## Who defines the requirements?
Usually, we expect the client, together with the system analyst, to define the **functional requirements**. But the client has no idea about **non-functional requirements**, during the discussion, you as architect should:

- Framining the realistic requirements' boundaries: no 100% uptime, its not realistic and probably also not necessary
- always using numbers