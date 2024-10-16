Its about "The Big Picture". To answers the questions:

- How will the system work under heavy load?
- What will happen if the system will crash at this exact moment in the
business flow?
- How complicated can be the update process?
- And more…

System Architecture should define:

- Software Components (Services)
- Communication between Components
- System capabilities: scalability, redundancy, performance etc.

# Loose Coupling
Prevents:
- platform coupling
- URL coupling: e.g. an exposed Java Remote Method Invocation (RMI) can only be called by another Java App


## The Spiderweb
If your services map looks like a spider web, be ready for a nasty surprise when even one of the URL is changed. The spider web is a sure sign of a strong coupling system.

<image src="../images/memi-11-coupling.png" width="600" />

**Solution 1 - Directory**

Services only need to know the Directory’s URL
<image src="../images/memi-11-coupling-directory.jpg" width="600" />

**Solution 2 - Gateway**

Services only need to know the Gateway’s URL
<image src="../images/memi-11-coupling-gateway.jpg" width="600" />

# Stateless
The application’ s state is stored in only two places – the data store and the user interface. No state should be stored in the application code.

> State = Application's data

This is a "stateFUL" example: the User will not be identified by the system if the next request is sent to the Service Nr.2 by the Load Balancer!

<image src="../images/memi-11-stateless.jpg" width="600" />

- Always use stateless architecture
- Supports Scalability and Redundancy

## Scalability
- Grow and shrink as needed
- Scale Out > Scale Up

## Redundancy
Allows the system to function properly when resource is not working. E.g.: A system with more than one server, when a server goes down, the other continue working.

<image src="../images/memi-11-scale-redundant.jpg" width="600" />

# Caching
Bring data closer to its consumer so that its retrieval will be faster. Data that saved in Cache should be:

- frequently accessed: fast, optimal UX
- rarely modified: syncing cache and DB is a challenge

<image src="../images/memi-11-cache.jpg" width="500" />

## Types

<image src="../images/memi-11-cache-types-compare.jpg" width="600" />

<image src="../images/memi-11-cache-types.jpg" width="600" />

# Messaging
Communication between the various services. Messaging Criteria:

- Performance
- Message Size
- Execution Model
- Feedback & Reliability
- Complexity

<image src="../images/memi-11-msg.jpg" width="600" />


## 1. REST API

## 2. HTTP Push (WebSocket)
- Uses advanced web techniques (ie. Web Sockets)
- Very popular in chats
  

## 3. Queue
- Messages will be handled once and only once
- Messages will be handled in order


## 4. File-based & Database-based


# Logging & Monitoring
Central Logging Service: The logs of the entire system must be accessible from a central point and must be present as a single format.


**Correlation ID**

Correlation ID is an identifier that is assigned to a flow at the beginning of it and is passed from service to service. 
It is included in every log record and enable us to track the flow from start to end. 
The correlation ID itself can be whatever identifier you will want to use. 