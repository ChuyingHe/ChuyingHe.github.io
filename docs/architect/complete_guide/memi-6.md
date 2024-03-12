After defining functional requirements and non-functional requirements, you need to determine the application type. Be aware, once this decision is made, its not easy to switch to other types.

# 1. Web App
It consists of:

- web server (ususally more than 1)
- web browser
  
\* Those two communicate with each other via HTTP protocol

Web App are best for systems that require:

- **User interface**
- **User-initiated actions**: meaning the user is the one making the request to do something, such as viewing all their data, saving username and so on. 
- **Large scale**: with a large number of users and a lot of data. 
- **Short, focused action**: as opposed to long-running processes. For example, web apps are not a good fit for a process that should crunch billions of numbers and produce business intelligence.
  
!!! tip "Rule of thumb"
    if the system you are working on can be described as a request-response application, then you are probably looking at a web app.

# 2. WebAPI
Web API is quite similar to web app with two important differences:

1. it does not serve HTML pages but JSON data 
2. instead of web browsers, other applications are calling it

Web API exposes an API, application programming interface, which allows other programs to access it and execute various actions. The most popular API is definitely the REST API. 

WebAPI is a combination of:

- URL (`https://api.thecatapi.com/v1/images/search`)
- Parameter (`has_breeds=true&order=RANDOM`)
- HTTP Verb (`GET`)

> I believe that almost every language can access web API and use it!

WebAPI Apps are best for systems that require:
- Data retrieval and store - though not huge amount of data in each action,
- Client initiated actions: meaning the caller is the one making the request to do something, such as getting all the data, saving username and so on. 
- Large scale: with a large number of users and a lot of data
- short, focused action: as opposed to long running processes. 

**Web API** and **web apps** are both based on the request-response model and should be avoid for Long running processes.

# 3. Mobile
Mobile Apps on Android or Apple, they always uses **WebAPI**

Mobile Apps are best for systems that require:
- User interaction (games, social apps)
- Frontend for WebAPI (news)
- Location sensitive - will benefit from phone's GPS

# 4. Console
Console App, pr command line applications, often called CLI, are applications that run inside the command line of the operating system. 


- No fancy UI: the only thing the end user sees are the lines of text displayed in the command line window.
- Requires some level of technical knowledge: normal end users who are accustomed to point-and-click applications will have a hard time working with these applications. 
- Limited interaction
- Long or short running Processes

Console App are best for systems that require:
- Long or short running Processes
- short, focused actions such as exporting data

# 5. Service
Service is like Console, but:

- no UI at all
- manages by Service Manager in the OS

Services App are best for systems that require:

- long running processes (monitoring folder on the disc) - when no user intervention is required.


# 6.Desktop Application
a poor kid in the application world, usually you dont encounter it:

- has all resources on the PC <--> not the Rich Client App that access WebAPI
- Might connect to the web (for some arbitrary task such as validating license or sending activity logs), but it's still be fully-functional when its offline

!!! note "Example"
    Microsoft word

are best for:

- user-centric actions (word processing or calculations, and heavy gaming)

# Other types
AWS Lambda or Azure Functions, which allows the developer to write a short focused code segments and not worry about servers, scale, or SLA.

# Summary
- App type should be set early
- App types can be more than one

!!! note "WebApp + Service"
    In addition to the main application, many **web apps** have also a **service** that run continuously and performs maintenance tasks such as removing all data records, monitoring various processes, and so on.