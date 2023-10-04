# String operation
|||
|:--|:--|
|`print("hello world")`|print|
|`strip()`||
|`if... elif... else`|`elif` instead of `else if`|
|`!=` <br/> `==`|it is NOT `!==`|
|`%`|Modulus: returns remainder|
|`a = int(input())`|Read input value from STDIN, as Integer|
|``||
|``||
|``||
|``||
|``||
|``||

## Python Arithmetic Operators

|Operator|Description|Example|
|:--|:--|:--|
|`+`| Addition|Adds values on either side of the operator.|a + b = 30|
|`-`| Subtraction|Subtracts right hand operand from left hand operand.|a – b = -10|
|`*`| Multiplication|Multiplies values on either side of the operator|a * b = 200|
|`/`| Division|Divides left hand operand by right hand operand|b / a = 2|
|`%`| Modulus|Divides left hand operand by right hand operand and returns remainder|b % a = 0|
|`**`| Exponent|Performs exponential (power) calculation on operators|a**b =10 to the power 20|
|`//`||Floor Division - The division of operands where the result is the quotient in which the digits after the decimal |point are removed. But if one of the operands is negative, the result is floored, i.e., rounded away from zero (|towards negative infinity) −|9//2 = 4 and 9.0//2.0 = 4.0, -11//3 = -4, -11.0//3 = -4.0|

## Python Comparison Operators

|Operator| Description| Example |
|:--|:--|:--|
|`==`| If the values of two operands are equal, then the |condition becomes true.| (a == b) is not true. |
|`!=`| If values of two operands are not equal, then |condition becomes true.| (a != b) is true. |
|`<>`| If values of two operands are not equal, then |condition becomes true.| (a <> b) is true. This is similar to != operator.
|`>`| If the value of left operand is greater than the value of right operand, then condition becomes true.| (a > b) is not true.
|`<`| If the value of left operand is less than the value of right operand, then condition becomes true.| (a < b) is true.
|`>=`| If the value of left operand is greater than or equal to the value of right operand, then condition becomes true.| (a >= b) is not true. |
|`<=`| If the value of left operand is less than or equal to |the value of right operand, then condition becomes true.| (a <= b) is true.

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