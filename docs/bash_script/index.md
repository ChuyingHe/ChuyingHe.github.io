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


!!! note "bash VS ."
    `./script.sh` needs ^^execution permission^^


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
    - to see all, run `set`
    - are defined with CAPITAL LETTERS
3. Special variables such as `$$`, `$?`, `$@`...

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
heredoc> a:
heredoc> ${a}
heredoc> b:
heredoc> ${b}
heredoc> EOF
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

# Input 
1. `read [-rp] Variable(s)`
    Example:
    ```bash
    read -p "Enter shell type and your name: " myShell myName
    echo "Your shell type is ${myShell}, Your name is ${myName}"
    ```

    Another example: add **field separater**:
    ```bash
    IFS=':' read -p "Enter shell type and your name, separate them by ':': " myShell myName
    echo "Your shell type is ${myShell}, Your name is ${myName}"
    ```
    !!! info "IFS"
        IFS = internal field separator

2. `source` concept
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


# String operations
We assign a variable first: 
```bash
myStr=Iamacatperson
yourStr=Iamadogperson
```

|Script|Function|Description|
|:--|:--|:--|
|`${myStr}`|refer to the string||
|`${#myStr}`|length of the string||
|`"${myStr} ${yourStr}"`|Concatenation|Needs to add quotes|
|``|Lowercase||
|``|||
|``|||











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