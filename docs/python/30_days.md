# String operation

|                                                             |                                                       |
| :---------------------------------------------------------- | :---------------------------------------------------- |
| `print("hello world")` <br/> `print("hello world", end="")` | `end=""` overwrites the default 'newline' after print |
| `strip()`                                                   |                                                       |
| `if... elif... else`                                        | `elif` instead of `else if`                           |
| `!=` <br/> `==`                                             |                                                       |
| `a = int(input())`                                          | Read input value from STDIN, as Integer               |

<!--
|``||
|``||
|``||
|``||
|``||
|``|| -->

## Python Arithmetic Operators

| Operator | Description    | Example                                                                                                          |
| :------- | :------------- | :--------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ---------------------------- | --------------------------------------------------------- |
| `+`      | Addition       | Adds values on either side of the operator.                                                                      | a + b = 30                                                                                                       |
| `-`      | Subtraction    | Subtracts right hand operand from left hand operand.                                                             | a – b = -10                                                                                                      |
| `*`      | Multiplication | Multiplies values on either side of the operator                                                                 | a \* b = 200                                                                                                     |
| `/`      | Division       | Divides left hand operand by right hand operand                                                                  | b / a = 2                                                                                                        |
| `%`      | Modulus        | Returns remainder                                                                                                | b % a = 0                                                                                                        |
| `**`     | Exponent       | Performs exponential (power) calculation on operators                                                            | a\*\*b =10 to the power 20                                                                                       |
| `//`     |                | Floor Division - The division of operands where the result is the quotient in which the digits after the decimal | point are removed. But if one of the operands is negative, the result is floored, i.e., rounded away from zero ( | towards negative infinity) − | 9//2 = 4 and 9.0//2.0 = 4.0, -11//3 = -4, -11.0//3 = -4.0 |

## Python Comparison Operators

| Operator | Description                                                                                                       | Example                                                  |
| :------- | :---------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------- | ----------------- |
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
$$
|                         | List                                                                                                                                                                                                                | Tuple | Set | Dictionary |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- | --- | ---------- |
| Ordered?                | Yes                                                                                                                                                                                                                 | Yes   | No  | Yes        |
| Changeable?             | Yes                                                                                                                                                                                                                 | No    | No  | Yes        |
| Allow duplicates?       | Yes                                                                                                                                                                                                                 | Yes   | No  | No         |
| Operations:             |                                                                                                                                                                                                                     |       |     |            |
| Create                  | `my_list = ["apple", "banana", "cherry"]` <br/> `my_list = list(("apple", "banana", "cherry"))`                                                                                                                     |       |     |            |
| - allows different type | `my_list = ["abc", 34, True, 40, "male"]`                                                                                                                                                                           |       |     |            |
| Length                  | `len(my_list)`                                                                                                                                                                                                      |       |     |            |
| Get Type                | `type(my_list)`                                                                                                                                                                                                     |       |     |            |
| Access items            | `my_list[1]` <br/>`my_list[-1]` Get the last one <br/> `my_list[2:5]` Get a range of elements [2,5) <br/> `my_list[:4]`                                                                                             |       |     |            |
| Checking existence      | `if "apple" in my_list:`                                                                                                                                                                                            |       |     |            |
| Change                  | `my_list[1]="kiwi"` <br/> `my_list[1:3]=["kiwi", "passionfruit"]`                                                                                                                                                   |       |     |            |
| Add                     | `my_list.insert(2, "watermelon")` <br/> `my_list.append("orange")`                                                                                                                                                  |       |     |            |
| Concatenate             | `my_list.extend(list2)` <br/> `my_list.extend(tuple)` Concatenate List and Tuple                                                                                                                                    |       |     |            |
| Remove                  | `my_list.remove("banana")` Remove the **first** occurance of "banana" <br/> `my_list.pop(1)` removes specified index and get the value for return <br/> `del my_list[1]` Just removes the one with index, no return |       |     |            |
| Clear up                | `my_list.clear()`                                                                                                                                                                                                   |       |     |            |
