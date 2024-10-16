# Important Concepts
**Software Component**: A piece of code that runs in a single process
**Distributed Systems**:

- Composed of independent Software Components
- Deployed on
    - separate processes
    - or containers
    - or servers

!!! note "Distributed Systems"
    You have probably heard about microservices application, server application, and more. 
    
    All these systems are distributed systems that have components or services deployed independently and which communicate via some kind of network protocol, usually HTTP.

    <image src="../images/memi-9-ds.jpg" width="600" />

    
# Two Levels of Software Architecture
## (LOW) Component’s Architecture
- Inner Components
- Interaction between them
- Make the code fast and easy to maintain

!!! note "Design Pattern - Layers"
    - Forces well formed and focused code
    - Modular
    - Rules:
        - Direction: From top to bottom
        - NO backwards calling!
        - NO skipping layers
        - Loose Coupling, a.k.a. use DI
        - Proper Exception handlung: give needed but limited information
  
    <image src="../images/memi-9-layers.jpg" width="600" />

### Interface

A contract that declares the signatures of an implementation. Interface in an OO language = Abstract classes in python

```java
interface ICalculator 
{
    double Add(double numi, double num2);

    double Subtract(double numi, double num2);

    double Multiply(double numi, double num2);

    double Divide(double numi, double num2);
}
```

Using interface allows us to make our code loosely coupled, meaning we do not tie one class to other. 

For example, we want to replace a Calculator with an advanced Calculator. We'll have to change the code in the Main class and recompile it, and that means that the classes are tied and strongly coupled.

Without Interface:
```java
// Before:
public Main()
{
    Calculator calc = new Calculator();
    double result = calc.Add(5,2)
}

// After:
public Main()
{
    AdvancedCalculator calc = new AdvancedCalculator();
    double result = calc.Add(5,2)
}
```

!!! warning "Warning"
    "new" is glue!


With Interface:
```java
public Main()
{   
    // calc: the Dependency
    // GetCalculatorInstance(): the Middleman
    ICalculator calc = GetCalculatorInstance();
    double result = calc.Add(5,2)
}
```

### Dependency Injection
A technique whereby one object … supplies the dependencies of another object

The "middleman", or to say the **factory method**, simply executes some logic and decides which class should be injected.
```java
private ICalculator GetCalculatorInstance()
{  
    // Its common to define the parameter in a configuration file, instead of receiving as input parameter
    var type = ConfigurationManager.AppSettings["calcType"];

    switch (type)
    {  
        case "regular":
            return new Calculator();
        case "advanced":
            return new AdvancedCalculator();
        default:
            return new Calculator();
    }
}
```

The **factory method** is one way to implement DI, there is also another way called **Constructor Injection**. The main advantage of the constructive injection pattern over the traditional pattern is testability:
```java
public class HomeController : Controller
{  
    ILogger logging;
    // instantiate with a real class through Interface ILogger
    public HomeController (ILogger logging)
    {  
        this. logging = logging;
    }
}
```

## (HIGH) System Architecture
- Bigger Picture
- Scalable, Reliable, Fast, Easy to maintain


# SOLID
Coined by Bob Martin in 2000

Stands for:
- Single Responsibility Principle: Each class, module or method should have one, and only one, responsibility
- Open/Closed Principle
- Liskov Substitution Principle
- Interface Segregation Principle
- Dependency Inversion Principle