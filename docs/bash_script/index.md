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
## to print
```bash
# 1. `echo`
echo $a; echo $b; echo $c; echo $d; echo $e

# 2. `cat`
cat << EOF
e:
${e}
a:
${a}
EOF
```

!!! note "refer to a variable"
    Both `$a` and `${a}` work, but `${a}` is more accurate.


## to delete
```bash
a=5
unset a
```

# Input 
1. `read [-rp] Variable(s)`
Example:
```bash
read -p "Enter your shell type: " myShell myName
echo "Your shell type is ${myShell}, Your name is ${myName}"
```

Add **field separater**:
```bash
IFS=':' read -p "Enter your shell type and name, separate them by ':': " myShell myName
echo "Your shell type is ${myShell}, Your name is ${myName}"

```

2. `source` concept
3. `export`

```bash

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


```bash
```

```bash
```