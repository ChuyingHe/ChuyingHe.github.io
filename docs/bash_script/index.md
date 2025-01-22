Several shells are installed on your PC, do this at your root:
```bash
cat /etc/shells
```
You might see this:

```txt
/bin/bash
/bin/csh
/bin/dash
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

The current shell in usage can be found by:
```bash
echo $SHELL
```

-------

# Introductions
To get the helping doc:
```bash
man [command]
```
## Grammar
```bash
# [Command] [Flags] [Arguments]
ls -lrt /home
```

- **Flags** can be written together with one `-` in front
- **Arguments** should be separated by space

## Script file (`*.sh`)
```bash
#!/bin/bash
npm -v
```
The first line starts with `#!` is called **shebang line** and it indicates what shell to use.

!!! warning 
    In a script file, the commands will be executed "line by line".

!!! note "Semicolon (`;`)"
    The semicolon (`;`) has the same funcitonality of a newline.
    ```bash
    echo "hello"; sleep 4; echo "world"
    ```
    is equal to:
    ```bash
    echo "hello"
    sleep 4
    echo "world"
    ```


## Execution
**Shell Script** is a sequence of commands pasted in a text file. The purpose is to automate repetitive tasks in our Unix/Linux environment. You can execute shell by:

1. Writing the command directly: `java -version`

2. Executing *.sh file: 
    - `bash my_cmds.sh`
    - `./my_cmds.sh`
        
        If you have permission issue, do `ls -lrt` to check the permissions, and use `chmod u+x my_cmds.sh` to add ^^execution permission^^ to the file


!!! note "`bash` VS `.`"
    `.` needs ^^execution permission^^

-------




# Variables
```bash
#!bin/bash
a=2
b=$((a+3))
c=read
d=/etc/shells
e=$(hostname)
```
!!! info "Grammar"
    - `$(())` syntax is used for arithmetic evaluation 
    - `$(command)` syntax is used to capture the output of a command


## Types
There are 3 types:

1. User defined variables
2. System defined variables such as `${USER}`
    - defined with CAPITAL LETTERS
    - a list of variables set by **bash** can be found [herer](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)
3. Special variables such as `$$`, `$?`, `$@`...

!!! info 
    to see all set variables, run `set`

## Exit Status
Many commands in bash follow this convention:

- `0`: Successful execution.
- `1`: Command executed but did not succeed in finding/matching/performing the desired task.
- `2` or higher: An error occurred during the execution.

!!! info "check Exit Status"
    To check Exit Status, you can run this direct after running a command line such as `echo "1 3 2" | grep  2`:

    ```bash
    echo $?
    ```

## Naming
- 0-9, A-Z and \_ are allowed
- should NOT start with number
- case-sensitive
- NO space while defining a variable: `x = 4` will not work
- use `""` if the values has space: `name="John Doe"`
## Actions
### > to print
```bash
# 1. `echo`
echo $a; echo $b; echo $c; echo $d; echo $e

# 2. `cat`
cat  << EOF
a:
${a}
b:
${b}
EOF
```

!!! note "Variable reference"
    Both `$a` and `${a}` work, but `${a}` is more accurate.

!!! info "EOF"
    `EOF` is commonly used to signify the end of a **Here Document** (also called **HereDoc**), which is a way to provide multiline input to a command or output in a script.

    `EOF` stands for End of File, but it’s just a convention and doesn't hold any special meaning. You can use other terms like **END** or **any other string** as the delimiter - as long as there is the same word in both start and end.

### > to delete
```bash
a=5
unset a
```
-------

# Special variables
## :
a no-op command, 等价于 `true`。它始终返回退出状态码 0（成功），但什么实际操作也不做。常见用法:

1. 作为占位符，相当于python的pass：
    ```bash
    if [ "$condition" ]; then
        : # 条件为真，但目前还没有具体操作
    fi
    ```
2. 赋予变量默认值：如果变量 `NAME` 没有被设置或为空，则将其赋值为 world
    ```bash
    : ${NAME:="world"}
    echo "Hello, $NAME"

    # 等同于：
    if [ -z "$NAME" ]; then
        NAME="world"
    fi
    ```
3. 清空文件
    ```bash
    : > file.text
    ```

## &>
`&>` 是一种 重定向操作符，用于同时将 **标准输出（stdout）** 和 **标准错误（stderr）** 重定向到同一个目标。

!!! note "stdout & stderr"
    Shell 中的每个命令有两种主要输出流：
    
    - 标准输出（stdout，文件描述符为 `1`）：用于正常输出。
    - 标准错误（stderr，文件描述符为 `2`）：用于错误消息或诊断信息。

```bash
# 以下两条命令效果相同
ls &> output.txt
ls > output.txt 2>&1
```

常见用法:

1. 静默运行命令
    ```bash
    ls nonexistentfile &> /dev/null
    # 不会在屏幕上输出错误消息。
    ```
2. 同时记录标准输出和错误到文件
    ```bash
    ls existingfile nonexistentfile &> logfile.txt
    # 把成功的输出和错误消息都记录到 logfile.txt 中。
    ```
    如要分开打印：如成功，保存结果到`stdout.log`，如失败，保存结果到`stderr.log`
    ```bash
    command 1> stdout.log 2> stderr.log
    ```

-------
# Condition
```bash
case <variable> in
    <pattern1>) <commands1> ;;
    <pattern2>) <commands2> ;;
    ...
esac
```

- `<pattern1>`, `<pattern2>`: The patterns to compare with the variable.
    - Wildcards like *, ?, and [...] can be used in patterns.
    - Multiple patterns can be combined with | (logical OR).
- `)` Indicates the end of a pattern and the start of commands.
- `;;` Ends the commands for a pattern.

# Operators

## Arithmetic Operators
Addition (+): Binary operation used to add two operands.
Subtraction (-): Binary operation used to subtract two operands.
Multiplication (*): Binary operation used to multiply two operands.
Division (/): Binary operation used to divide two operands.
Modulus (%): Binary operation used to find remainder of two operands.
Increment Operator (++): Unary operator used to increase the value of operand by one.
Decrement Operator (- -): Unary operator used to decrease the value of a operand by one

## Numeric Comparison Operators

-eq: Equal to.
-ne: Not equal to.
-lt: Less than.
-le: Less than or equal to.
-gt: Greater than.
-ge: Greater than or equal to.

## Relational Operators

‘==’ Operator: Double equal to operator compares the two operands. Its returns true is they are equal otherwise returns false.
‘!=’ Operator: Not Equal to operator return true if the two operands are not equal otherwise it returns false.
‘<‘ Operator: Less than operator returns true if first operand is less than second operand otherwise returns false.
‘<=’ Operator: Less than or equal to operator returns true if first operand is less than or equal to second operand otherwise returns false
‘>’ Operator: Greater than operator return true if the first operand is greater than the second operand otherwise return false.
‘>=’ Operator: Greater than or equal to operator returns true if first operand is greater than or equal to second operand otherwise returns false

## Logical Operators

Logical AND (`&&`, `-a`): This is a binary operator, which returns true if both the operands are true otherwise returns false.
Logical OR (`||`): This is a binary operator, which returns true if either of the operands is true or if both the operands are true. It returns false only if both operands are false.
Not Equal to (`!`): This is a unary operator which returns true if the operand is false and returns false if the operand is true.

## Bitwise Operators
Bitwise And (&): Bitwise & operator performs binary AND operation bit by bit on the operands.
Bitwise OR (|): Bitwise | operator performs binary OR operation bit by bit on the operands.
Bitwise XOR (^): Bitwise ^ operator performs binary XOR operation bit by bit on the operands.
Bitwise complement (~): Bitwise ~ operator performs binary NOT operation bit by bit on the operand.
Left Shift (<<): This operator shifts the bits of the left operand to left by number of times specified by right operand.
Right Shift (>>): This operator shifts the bits of the left operand to right by number of times specified by right operand.

## File Test Operators
-b operator: returns true if the file is a block special file otherwise false.
-c operator: returns true if it is a character special file otherwise false.
-d operator: If given directory exists then operators returns true otherwise false.
-e operator: If given file exits this operator returns true otherwise false.
-r operator: If given file has read access then it returns true otherwise false.
-w operator: If given file has write then it returns true otherwise false.
-x operator: If given file has execute access then it returns true otherwise false.
-s operator: If the size of given file is greater than 0 then it returns true otherwise it is false.

|Operator| Description|
|:-|:-|
|`-a`|AND|
|``||
|``||
|``||
|``||
|`-ne`| not equal|
-------

# Input 
1. `read [-rp] Variable(s)`

    Example:
    ```bash
    read -p "Enter shell type and your name: " myShell myName
    # in ZSH, it should be: 
    #       read "?Enter shell type and your name: " myShell myName
    # The ? in zsh serves the same purpose as -p in bash
    echo "Your shell type is ${myShell}, Your name is ${myName}"
    ```

    Another example: add **field separater**:
    ```bash
    IFS=':' read -p "Enter shell type and your name, separate them by ':': " myShell myName
    echo "Your shell type is ${myShell}, Your name is ${myName}"
    ```
    !!! info "IFS"
        IFS = Internal Field Separator

2. `source`

    Commonly used when the variables are shared between different environments
    ```bash
    vi [myfile]
    source [myfile]
    ```
    Example:
    ```
    # myfile: this file has no prefix
    name=JohnDoe
    course=Bash2Automation
    ```

3. `export`

    ```bash
    export name=JohnDoe
    ```

    !!! warning "Temporary variable"
        The variable is temporary and is vanish once you close your Shell

    To see all the available variables, run:
    ```bash
    export -p
    ```

4. Input as command arguments
    ```bash
    # mybash.sh
    echo "${1}"
    echo "${2}"
    echo "${5}"

    echo "Total amount: ${#}"
    echo "The whole thing: ${*}"
    echo "The whole thing: ${@}"
    
    echo "your script was: ${0}"
    ```

    By execusing `./mybash.sh hello world how are you` you will get the following result:
    ```bash
    hello
    world
    you
    Total amount: 5    
    The whole thing: hello world how are you
    The whole thing: hello world how are you

    mybash.sh
    ```

    !!! info "Special variables"
        These are all **special variables**. `${1}` is **positional special variables**
    
    You can use `shift 1` to move the arguments `1` position to the left:
    ```bash
    # mybash.sh
    shift 1
    echo "${1}"
    echo "${3}"
    ```
    By execusing `./mybash.sh hello world how are you` you will get the following result:
    ```bash
    world
    are
    ```

`$#`  number of positional parameters


-------

# Output
1. `echo [-neE] [argument]`
    ```bash
    echo hello world how are you
    echo yourName
    echo "${yourName}"
    ```

    Colorful output: `-e "[COLOR_CODE1]Hello [COLOR_CODE1] [COLOR_CODE2]World[COLOR_CODE2]"`. Example:
    ```bash
    echo -e "\033[0;32mHello\033[0;32m\033[1;33m World\033[1;33m"
    ```

2. `cat`
3. `printf`

-------

# String operations
We assign a variable first: 
```bash
myStr=Iamacatperson
yourStr=Iamadogperson
```

|Script|Description|
|:--|:--|
|`${myStr}`|refer to the string|
|`${#myStr}`|length of the string|
|`"${myStr} ${yourStr}"`|Concatenation <br/>⚠️ Needs to add quotes|
|`${myStr,,}`|Lowercase|
|`${myStr^^}`|Uppercase|
|`echo ${myStr} | tr '[a-z]' '[A-Z]'`|use Transform function to switch between lower and upper cases|
|`${myStr:3:2}`|Slicing with index: from index 3, show 2 characters|
|`echo ${myStr} | cut -c 3-5`|Slicing with index, from 3 to 5|
|`echo ${myStr/One/Two}`|replace `One` with `Two`|
|`echo ${myStr}` | sed 's/One/Two/g'`|replace `One` with `Two`|

-------

# String operations on Paths

|Script|Function|Description|
|:--|:--|:--|
|`dirname /documents/code/example.txt`|print the path of the file|result: `/documents/code`|
|`basename /documents/code/example.txt`|print the file name|result: `example.txt`|

-------

# Arrays
Bash array = indexed array = list


-------

# Conditions
## if...elif...else
```bash
read x 
read y
if [ $x -gt $y ] 
then 
echo "X is greater than Y"

elif [ $x -lt $y ]
then 
echo "X is less than Y"

elif [ $x -eq $y ]
then
echo "X is equal to Y"
fi
```

!!! error 
    Make sure there is space after `[` and before `]`






```bash
```

```bash
```