!!! note "Goal"
The goal of this page is to give you a quick recap of important FastAPI concepts after you detached from it for a while

# Concurrency vs Parallelism

| Concurrent                                       | Parallel               |
| :----------------------------------------------- | :--------------------- |
| Ordering burger in Burger King                   | Odering Burrito        |
| + good for the task that takes long waiting time | + good for quick tasks |

!!! tip "Coroutine"
**co.routine** has 2 possible meanings: 1. Result of a async function 2. Technique that uses `async` and `await` to write code

**_Once async, always async_** - all the function that uses async function have to be async too!

# Path

Endpoint, route or path are all the same thing, they are mean `@app.get("/users/{user_id}")`

# Parameters

There are 4 types of Parameters. They all inherit from the same common `Param` class. Though `Header` has some extra functionality.

## 1. Path parameter

Path parameter needs to be defined "twice":

- in path
- in the function head in parentheses ()
  ```python
  @app.get("/users/{user_id}")
  async def read_user(user_id: str):
  ```

Besides `str` there are also other **type annotation**, for example `Enum` will lead to a dropdown UI in swagger doc. You need to define the enum:

```python
class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"
```

!!! note ""
optional:
`python
    q: str | None = None
    `

    default:
    ```python
    q: str = "hi"
    ```

## 2. Query parameter

When declare other parameters that are NOT a part of the path.

- key-value pairs
- use it after `?`, separated by `&`. E.g. `http://127.0.0.1:8000/items/?skip=0&limit=10`
- it can be mixed with path parameters: order doesnt matter

!!! note ""
required: just give no default values

## 3. Cookie parameter

To declare cookies, you need to use Cookie, because otherwise the parameters would be interpreted as query parameters. E.g.

```python
from typing import Annotated
from fastapi import Cookie

@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

!!! note "Annotated"
`Annotated` is used to specify that the `ads_id` parameter is a cookie parameter and can be either a `string` or `None`.

## 4. Header parameter

To declare cookies, you need to use Header, e.g.:

```python
from typing import Annotated
from fastapi import  Header

@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

Header parameter has extra functionality comparison to other 3 parameters:

- Header will convert the parameter names from hyphen (-) to underscore (\_) (because name user-agent is invalid)
- its case-sensitve
- Duplicate headers:
  ```python
  async def read_items(x_token: Annotated[list[str] | None, Header()] = None):
  ```

# Request Body

operations that have Request Body:

- POST
- PUT
- DELETE
- PATCH

Data type: use `Pydantic`! It also adds support in your IDE

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
async def create_item(item: Item):
    return item.name
```

!!! note "How does it work in background?" 1. FastAPI reads the Request Body as JSON 2. convert the corresponding types 3. validate

!!! tip "Differentiation: Parameter vs Body" - If the parameter is also declared in the path, it will be used as a **path parameter**. - If the parameter is of a singular type (like int, float, str, bool, etc) it will be interpreted as a **query parameter**. - If the parameter is declared to be of the type of a Pydantic model, it will be interpreted as a **request body**.

    💡 **query parameter** and **request body** ONLY differ in data-type!

# Typing

## Annotated

`Annotated` is a Python type hint that allows you to attach additional metadata or annotations to a type. This metadata can provide extra context or instructions for how the type should be interpreted or used. Here's an explanation of the code:

# Onion Architecture

<image src="./images/onion-architecture.png" width=500 />

1. **Domain**: This is the innermost layer and represents the core domain logic of the application. It contains domain models, business rules, and interfaces defining the contracts for interacting with domain objects. It can includes `models/` for domain model definitions, and `repositories/` for interface definitions for interacting with data.
2. **Service**: This layer sits around the domain layer and contains the application-specific business logic. It orchestrates interactions between different domain objects, applies business rules, and handles use cases.
3. **Infrastructure**: This layer wraps around both the domain and application layers and deals with external concerns such as databases, file systems, external services, and configuration settings. It provides implementations for interacting with external resources while keeping the core domain and application layers decoupled from such details. It can also be further refined into subfolder such as `database/`, `external_service/`
4. **API**: This is the outermost layer and represents the user interface or the entry point of the application. It handles HTTP requests, translates them into application-specific commands, and sends them to the application layer for processing. This layer is responsible for presenting information to the user and accepting user inputs.
