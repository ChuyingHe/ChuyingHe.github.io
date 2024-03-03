# 0. Understand the system's goal
We've discussed it in the last chapter, its a preparation phrase, the real architect process starts with 1.

# 1. Understand the system's requirements (with @System analyst)
Requirements = what the system should do, they usually begin with a high-level task, such as 

- allow end users to view telemetry data
- workflows
- logical services
- user interface elements

Requiremenets are usually defined by the system analyst, who works directly with the client.

# 2. Understand the non-functional requirements
Non-functional requirements: are a special kind of requirements that define some technical and service level attributes of the system. For example:

- concurrent number of users
- heavy loads
- volumes of data and performance

The client and the system analyst are usually not aware of the non-functional requirements, it's the architect's job to help them formulate those requirements. 

!!! warning 
	For the architects, the non-functional requirements are much more important than the regular requirements.

	Never begin to work on a system before knowing exactly what its non-functional requirements are

# 3. Map the components
The components = various tasks of the system, functional as well as non-functional. 

In this step, we map the various software units and define what each of them does. The component map serves two goals:

1. To help you understand the system and its various parts. 
2. To communicate to the client your understanding of the system, thus making sure you are not missing anything. 

!!! note
	The component map is completely non-technical. You are yet to decide on the platform, the development tool, the database type. This is just a map that displays the various capabilities of the system.
# 4. Select the technology stack

- one of the most important steps
- together with the development team
- usually you will have to select backend, frontend and data store

# 5. Design the architecture (with @Developers)

We glue 

- the requirements
- the non-functional requirements
- the components 
- the technology stack

together and will result in a system that is fast, secure, reliable, and easy to maintain.

!!! note
    We will learn about the qualities of a well-designed system, such as loose coupling, stateless, scaling, caching, messaging, and lots more, and see how those qualities are used as the building blocks of the architecture. When you are done, you will have a complete architecture in place, but it won't be formalized yet, which brings us to the next step.

# 6. Write the architecture document

This is a combination of all the efforts you've put into the system, and this is your greatest creation. The architecture document describes the whole process you have been through and gives the developers and management a full picture of the system that is going to be built. 

A good architecture document is relevant for all the levels in the organization:

- the CEO
- the CIO
- the project manager
- the developers

They will all find great value in it. We'll talk a lot about the structure and the content of the document in the relevant section and explain how to maximize its value.

# 7. Support the development team

Architect's job is not done when the document is done.

Subtle architecture is a living, breathing creature, and it changes all the time. You have to be there for the developers to help them to make sure they're developing according to the architecture and to be part of the dilemmas that are going to be raised. And there are going to be a lot of dilemmas, arguments, and talks, and the architecture will be changed and not only once. S

o you have to support the team if you don't want the documents to become a glorified paperweight. And remember, you are not done until the system is in production, and even then, you probably will have a lot to do.