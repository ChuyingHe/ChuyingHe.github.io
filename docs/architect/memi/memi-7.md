Why is it important?

- Irreversible
- Emotional

Decision must be:

-  made with clear mind (very rational)
-  heavily documented
-  group effort (discuss with your developers)


Technology Stack Considerations

1. Can perform the required tasks?
2. Community ([stackoverflow tags](https://stackoverflow.com/tags))
3. Popularity ([google trend](https://trends.google.com/trends/explore?date=today%205-y&geo=DE&q=%2Fm%2F012l1vxv,%2Fg%2F11c6w0ddw9&hl=en-GB))

# Frontend

## Web 
Frontend app consists of HTML, CSS and JS. There are some popular JS Frameworks, including ReactJS and AngularJS
<image src="../images/memi-7-frontend-techs.jpg" width="600" />

## Mobile 
There are 3 types of mobile app:

1. **Native**: Native apps are developed for a specific mobile operating system (OS) using the
platform's native programming languages and tools. *--> Such as Swift for iOS, Kotlin for Android*
2. **Hybrid**: Hybrid apps are built using _web technologies (HTML, CSS, JavaScript)_ and are
wrapped in a native container for deployment. *--> Such as Ionic*
3. **Cross-Platform**: Cross-platform development uses frameworks or technologies that allow
developers to write code once and deploy it on multiple platforms. *--> Such as Flutter, React Native*

<image src="../images/memi-7-frontend-mobile.jpg" width="600" />

## Desktop
<image src="../images/memi-7-frontend-desktop.jpg" width="600" />


# Backend
WebApps, WebAPI, Console and Service could all be counted as **Backend**. Here are some popular backend applications:
## 1. .NET Classic
- founded by Microsoft in 2001 as a response for Java
- General purpose
- Object-oriented
- Statically-typed 
- IDE: Visual Studio
- Windows only
- Performance okish
- Very mature
- Blurred roadmap

It is default choice for Windows-based application.

!!! note "Statically-typed vs Dynamically-typed"
    In **statically-typed** languages, the data types of variables are known and checked at compile-time, before the program is run. Suitable for large-scale projects where errors can have severe consequences, because it enables error detection and ensures type safety. E.g.: C, C++, Java, C#, Swift, Go

    In contrast, **dynamically-typed** languages where type checking is done at runtime. Suitable for prototyping and development, or when data types may change during runtime. E.g.: Python, JS, TS, PHP, R

## 2. .NET Core
- Next generation of .NET,
- Cross-platform support and performance.
- a flexible and fast platform
- but not fully baked yet
- fast growing community

## 3. Java
- founded in 1995 by Sun Microsystems
- very popular
- General purpose
- Object-oriented
- Statically-typed 
- Huge community


## 4. node.js
- founded on 2009 by Ryan Dahl
- optimized for highly concurrent Web Apps.
- JavaScript-based
- dynamically typed
- large community
- great performance


As mentioned, Node.js is NOT targeted for long running processes so don't try to build services with it. But for Web Apps that require a lot of short, concurrent I/O operations, its the best choice.

## 5. PHP
- founded in 1994 by Rasmus Lerdof
- Messy
- easy to learn
- not polished enough.
- large community.

PHP is focused on Web Apps and Web API, not suitable for long running component

## 6. Python
- Founded on 1989 by Guido van Rossum
- scripting language
- the easiest language to learn
- large community
- can perform almost any task

You can consider Python for almost any type of application including Web App, Web API, console, or service.

<image src="../images/memi-7-backend-techs.jpg" width="600" />

# Data Store



## SQL Database
SQL database or relational database. The most popular relation databases are Microsoft SQL Server, Oracle and MySQL, postgresql.

- Stores data in tables
- Tables have concrete set of columns
- Tables can have relationships with each other --> hence the "relational" in the name.
- Transaction(TA): use SQL, ACID Structured Query Language

!!! note "ACID!"
    - Atomicity: ALL or NONE changes of the Transaction is made
    - Consistency: follows all pre-defined rules, such as primary key not empty etc
    - Isolation: 2 parallel Transaction should not effect each other
    - Durability: once changes have been made in the DB, it should always be there

## NoSQL Database
not SQL or not only SQL. MongoDB, CouchDB

- Emphasis on scale and performance
- Schema-less: **SQL** databases store the data in a well-defined table with well-defined columns, which defines an actual schema for the entities, **NoSQL** database do not force any schema. They can store completely different entities with completely different fields in the same table.
- Data usually stored in JSON format
- Transaction(TA): no standard, can be frustrating, every DB has its own language.

Data in NoSQL can be temporarily inconsistent, but "Eventual Consistency", which means that the database guarantees that the action will be performed, but it will not guarantee when exactly it will be performed. (partially ACID) --> usually less than 1 second

!!! tip "Usage"
    This is great if your application is going to store semi-structured or unstructured data, which does not have a concrete schema.

!!! note "ACID?"
    Each NoSQL database fulfills partially ACID, but differently. It's important to look closely at the transaction support of the NoSQL database you are going to work with.

    BUT, MongoDB is fully ACID-compliant with ACID transactions!


|| SQL     | NoSQL |
| --------| -------- | ------- |
|Storage| Not huge, < 10 Terabytes  | Huge    |
| Type of Data|Structured | unstructured     |


!!! note "Data size"
    |Name | Equal To |
    | --------| -------- |
    |Bit | 1 Bit |
    |Byte | 8 Bits |
    |Kilobyte | 1024 Bytes | 1024 |
    |Megabyte | 1024 Kilobytes | 
    |Gigabyte | 1024 Megabytes | 
    |Terabyte | 1024 Gigabytes |
    |Petabyte | 1024 Terabytes | 
    |Exabyte | 1024 Petabytes |
    |Zettabyte | 1024 Exabytes | 
    |Yottabyte | 1024 Zettabytes|

## New technology
**PostgreSQL** is an advanced, enterprise-class open-source **relational (SQL) database** that supports both SQL (relational) and JSON (non-relational) querying.