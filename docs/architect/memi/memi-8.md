* -ilities = Quality Attributes. Technical capabilities that should be used in order to fulfill the **non-functional
requirements**

<image src="../images/memi-8.jpg" width="600" />


|Non-Functional requirement | Quality Attribute |
| --------| -------- |
|The system must work under heavy load, but should not waste money on unused resources | Scalability |

# Scalability
Adding computing resources without any interruption.

|Non-Scalable System | Scalable System |
| --------| -------- |
| 1. Look for non-scalable code <br/> 2. Rewrite non-scalable code <br/> 3. Reinforce VM | 1. Add VM <br/> 2. Notify the Load Balancer |

## Scalability Types
1. Scale Up: make 1 server bigger
2. Scale out: take 2 or 3 server with a Load Balancer

<image src="../images/memi-8-scale.jpg" width="600" />



# Manageability
Know what’s going on and take actions accordingly. The question to ask is: Who reports the problem?

- If it's the end user, then the system is NOT manageable, and as a side effect, we have made your users a QA team, which is bad.
- If the system itself report problems, the experience is much better.

<image src="../images/memi-8-manageable.jpg" width="600" />

!!! note "Example"
    For example, a mass of request is received in a very short time and the response time degradates substantially.
    As a result, the system alerts the monitoring agent about it and the operator decides to add more VMs to support that load.

# Modularity
A system that is built from building blocks, that can be changed or replaced without affecting the whole system.

# Extensibility
A system that its functionality can be extended without modifying its existing code.

<image src="../images/memi-8-extensible.jpg" width="600" />

# Testability
How easy it to test the application with **Unit Testing** and **Integration Testing**

- **Manual**: he actual tester is sitting in front of the screen and test various functions of the system using the user interface it provides. --> has nothing to do with testability
- **Unit Testing**
- **Integration Testing**: With integration test, we don't test a specific method but a whole model or flow. We execute a method that will trigger a chain of actions that will result in a specific outcome. For example, a new record in the database should be created from the UI.

!!! tip "Testability’s Characteristics"
    - Independent modules and methods --> achieve with **Dependency Injection (DI)**
    - Single responsibility --> achieve with the single responsibility principle
    

More *-ilities: https://en.wikipedia.org/wiki/List_of_system_quality_attributes