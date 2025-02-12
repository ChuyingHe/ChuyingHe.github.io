[Source](https://ibm-learning.udemy.com/course/the-complete-python-course/learn/lecture/9445238#overview)

## Basics

### Lamda function

double = [x*2 for x in Sequence]
double = map(func, Sequence) # apply each item in Squence with func()
double = map(lambda x: x\*2, Sequence)

### Postion args

func(\*args)

### Keyword args

func(\*\*kwargs)

### Magic methods

Magic methods are the auto-called methods in CLASS:

- `__init__()`: when CLASS object is created
- `__str__()`: when you want to print object
- `__repr__()`: to print xxx in order to reproduce the CLASS object

### Methods types

1. instance methods
   need the object -> in order to call them

```python
@classmethod
def instance_method(self):
```

2. class methods
   need the CLASS -> in order to call them
   --> can be used to create Object

```python
@classmethod
def class_method(cls):
```

3. static methods
   a function instead of method. It has no info about the CLASS or OBJECT

```python
@staticmethod
def static_method():
```

### Inheritance

```python
class Child(Parent):
def **init**(self):
super().**init**()
```

### Type hinting

```python
def func(sequence: list) -> float
```

- input(sequence): has LIST type
- return a FLOAT type
- return the own class: "Book"
- return another class: Bookshelf

### Useful env variables

```python
import sys
sys.path # to see where to look for the python packages
sys.modules #to see all imported modules
__name__ # current file
```

### Errors

```python
def divide():
  if ... raise ZeroDivisionError("msg...")
```

Example:

```python
try: ...
except ZeroDivisionError as e :
else: # when no error
finally: # always output
```

### Custom Error Class

```python
class SportInconsistency(ValueError):
    pass
```

### First-class function

to pass Function as Parameter

```python
func # pass a function
func() # call a function
```

### Decorators

make functions secure
Decorators: @make_secure <-- the secure function
import functools
@functools.wrap(func)
Decorators: pass with variable

### Multability

Do not use default-parameter-values that are mutable

### Comment

This is the official document that shows when hover on the class/method

```python
"""
Document
"""
```

## Error Handlung

When you deliver your program to your client. You should always try to "catch the errors" so that your program doesn't crash!

| ERROR TYPE          | Reason                                     |
| :------------------ | :----------------------------------------- |
| IndexError          | out of range                               |
| KeyError            | used wrong key                             |
| NameError           | var is not defined                         |
| AttributeError      | attr of a list not exit                    |
| NotImplementedError | the same as `pass`                         |
| RuntimeError        | basically could be anything                |
| SyntaxError         | wrong python syntax                        |
| IndentationError    | indentation missing                        |
| TabError            | Should have use "spaces" instead of "tabs" |
| TypeError           | wrong type                                 |
| ValueError          | correct type, but invalid value            |
| ImportError         |                                            |
| DeprecationWarning  |                                            |

### Raise Error

```python
raise NotImplementedError('This function is not implemented yet')

if not isintance(car, Car):
    raise TypeError(f'Tried to add a `{car.__class__.__name__}` to the garage')

# in the new specification, this Error can be prevent by using "type specification":
def myfunc(self, car: Car):
```

### Create Own Error

```python
class RuntimeErrorWithCode(TypeError):
    def __init__(self, message, code):
        super().__init__(f'Error code {code}: {message}')
        self.code = code
```

### Dealing with Error

1. Silence the ERROR
   > > with finally you silenced the ERROR -> no error info will be printed.
   > > for PROD

```python
try:
    # do sth
except TypeError:
   print("TypeError: your car is not a Car")
except ValueError:
   print("ValueError...")
finally:
    #code that always run
   print(f'The garage now has {}')

```

2. Make the ERROR laut
   > > for DEV

```python
try:
    user.score = perform_calculation(user.engagement_metrics)
except KeyError:
    print('Incorrect values provided to our calculation function.')
    raise
else:
    if user.score > 500:
        send_engagement_notification(user)
```

!!! `raise`` has 2 use cases: - raise NotImplementedError() --> outside, to call an Error - raise --> inside a except Error, to unsilence the Error

# String operation

| Operation                                                   | Explaination                                          |
| ----------------------------------------------------------- | ----------------------------------------------------- |
| `print("hello world")` <br/> `print("hello world", end="")` | `end=""` overwrites the default 'newline' after print |
| `strip()`                                                   |                                                       |
| `if... elif... else`                                        | `elif` instead of `else if`                           |
| `!=` <br/> `==`                                             |                                                       |
| `a = int(input())`                                          | Read input value from STDIN, as Integer               |
| `.strip()`,`.lstrip()`,`.rstrip()`                          | 删掉多余空格                                          |
| `.title()`，`.upper()`，`.lower()`                          | 改罗马字母大小写                                      |

<!--
|``||
|``||
|``||
|``||
|``||
|``|| -->

## Python Arithmetic Operators

| Operator | Description    | Example|
| -------- | -------------- | --- | 
| `+`      | Addition       | Adds values on either side of the operator.                                                                      | a + b = 30                                                                                                       |
| `-`      | Subtraction    | Subtracts right hand operand from left hand operand.                                                             | a – b = -10                                                                                                      |
| `*`      | Multiplication | Multiplies values on either side of the operator                                                                 | a \* b = 200                                                                                                     |
| `/`      | Division       | Divides left hand operand by right hand operand                                                                  | b / a = 2                                                                                                        |
| `%`      | Modulus        | Returns remainder                                                                                                | b % a = 0                                                                                                        |
| `**`     | Exponent       | Performs exponential (power) calculation on operators                                                            | a\*\*b =10 to the power 20                                                                                       |
| `//`     |                | Floor Division - The division of operands where the result is the quotient in which the digits after the decimal | point are removed. But if one of the operands is negative, the result is floored, i.e., rounded away from zero ( | towards negative infinity) − | 9//2 = 4 and 9.0//2.0 = 4.0, -11//3 = -4, -11.0//3 = -4.0 |

## Python Comparison Operators

| Operator | Description    | Example|
| -------- | -------------- | --- | 
| `==`     | If the values of two operands are equal, then the condition becomes true.                                         | (a == b) is not true.                                    |
| `!=`     | If values of two operands are not equal, then condition becomes true.<br/> :warning: it is NOT `!==`              | (a != b) is true.                                        |
| `<>`     | If values of two operands are not equal, then condition becomes true.                                             | (a <> b) is true. This is similar to != operator.        |
| `>`      | If the value of left operand is greater than the value of right operand, then condition becomes true.             | (a > b) is not true.                                     |
| `<`      | If the value of left operand is less than the value of right operand, then condition becomes true.                | (a < b) is true.                                         |
| `>=`     | If the value of left operand is greater than or equal to the value of right operand, then condition becomes true. | (a >= b) is not true.                                    |
| `<=`     | If the value of left operand is less than or equal to                                                             | the value of right operand, then condition becomes true. | (a <= b) is true. |

## Loop

### `for` loop

Using an array:

```py
fruits = ["apple", "banana", "cherry"]
for x in fruits:
  print(x)
```

Using a string:

```py
for x in "banana":
  print(x)
```

Using a number as range:

```py
for x in range(2, 30, 3):
  print(x)
```

### `while` loop

TODO

### Python Collections


|| List| Tuple | Set | Dictionary |
| ---| --- | --- |  --- |  --- | 
| Ordered? | Yes| Yes   | No  | Yes        |
| Changeable?             | Yes| No    | No  | Yes        |
| Allow duplicates?       | Yes| Yes   | No  | No         |
| Operations:             |    |       |     |            |
| Create                  | `my_list = ["apple", "banana", "cherry"]` <br/> `my_list = list(("apple", "banana", "cherry"))`|       |     |            |
| - allows different type | `my_list = ["abc", 34, True, 40, "male"]`|       |     |            |
| Length                  | `len(my_list)`|       |     |            |
| Get Type                | `type(my_list)`|       |     |            |
| Access items            | `my_list[1]` <br/>`my_list[-1]` Get the last one <br/> `my_list[2:5]` Get a range of elements [2,5) <br/> `my_list[:4]`|       |     |            |
| Checking existence      | `if "apple" in my_list:`|       |     |            |
| Change                  | `my_list[1]="kiwi"` <br/> `my_list[1:3]=["kiwi", "passionfruit"]`|       |     |            |
| Add                     | `my_list.insert(2, "watermelon")` <br/> `my_list.append("orange")`|       |     |            |
| Concatenate| `my_list.extend(list2)` <br/> `my_list.extend(tuple)` Concatenate List and Tuple|       |     |            |
| Remove| `my_list.remove("banana")` Remove the **first** occurance of "banana" <br/> `my_list.pop(1)` removes specified index and get the value for return <br/> `del my_list[1]` Just removes the one with index, no return |       |     ||
| Clear up| `my_list.clear()`       |       |     |            |
