Design Patterns is a collection of general, reusable solutions to **Common Problems** in software design. We use Patterns to design consistent, flexible, readable and easy to maintain software.

!!! note "Common Problems"
    Examples:

    - How to communicate between classes
    - How to initialize interface implementations
    - How to access data stores
    - And more…

Benefits:

- Popular: tested and used by other developers
- Make your code more readable and easy to modify
- Avoid strong coupling

# Factory Pattern
Creating objects without specifying the exact class of the object. Goal is to avoid strong coupling between classes.

<image src="../images/memi-10-factory.jpg" width="600" />

# Repository Pattern
Modules not handling the actual work with the datastore should be oblivious to the datastore type.

More advanced implementations:
- Generic classes
- Inheritance
- Extension Frameworks

# Facade Pattern
Creating a layer of abstraction to mask multiple complex actions.
<image src="../images/memi-10-facade.jpg" width="600" />


# Command Pattern
All the action’s information is encapsulated within an object