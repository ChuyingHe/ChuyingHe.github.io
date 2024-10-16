# 0. Understand the system's goal
We've discussed it in the last chapter, its a preparation phrase, the real architect process starts with 1.

# 1. Understand the functional requirements (with @System analyst)
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
	The component map is completely non-technical. You are yet to decide on the platform, the development tool, the database type. This is just a map that displays the various capabilities of the system. Here is an example:
		<image src="../images/memi-4-components-mapping.png" width=500 />

# 4. Select the technology stack

- one of the most important steps
- together with the development team
- usually you will have to select backend, frontend and data store

# 5. Design the architecture (with @Developers)

We glue 

- the functional requirements
- the non-functional requirements
- the components mapping
- the technology stack

together and will result in a system that is fast, secure, reliable, and easy to maintain.


# 6. Write the architecture document
We formalize the architecture design by writing the document. The architecture document describes the whole process you have been through and gives a full picture of the system. 

A good architecture document is relevant for **all the levels** in the organization:

- the CEO
- the CIO
- the project manager
- the developers

They will all find great value in it. We'll talk a lot about the structure and the content of the document in the relevant section and explain how to maximize its value.

# 7. Support the development team

Architect's job is not done when the document is done.

Subtle architecture is a living, breathing creature, and it changes all the time. You have to be there for the developers to help them to make sure they're developing according to the architecture and to be part of the dilemmas that are going to be raised. And there are going to be a lot of dilemmas, arguments, and talks, and the architecture will be changed and not only once. 

So you have to support the team if you don't want the documents to become a glorified paperweight. And remember, you are not done until the system is in production, and even then, you probably will have a lot to do.