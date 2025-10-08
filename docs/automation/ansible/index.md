- [Reference](https://www.youtube.com/watch?v=1id6ERvfozo)

# Intro
a tool to automate IT tasks, such as:

- upgrade of a distributed app on 10 remote servers
- repetitive tasks: create backup, system reboot, create user etc

!!! note "Ansible brings conveniences"
    1. Execute tasks from my machine
    1. Config steps in one single YAML file
    1. Re-use same file for different environments
    1. More reliable with less human error

!!! info
    Ansible supports ALL infrastructures: from <u>local operating systems</u> to <u>Cloud Providers</u>

## Agentless

agentless: in the example above, there is no need to install Ansible on the 10 remote servers

## How it works
### Modules
**Modules** are <u>granular, modular and specific</u> programs that do the actual work. They:

1. get pushed to target servers
1. get the work done
1. get removed

\* [a list of **Modules**](https://docs.ansible.com/ansible/latest/collections/index_module.html)

### Grammar: YAML
taking usage of Docker **Module** for example:
```yaml
# Create container
- name: Create a data container 
  docker_container:
    name: mydata 
    image: busybox 
    volumes:
        - /data

# Start container
- name: Start a container with a comman 
  docker_container:
    name: sleepy
    image: buntu: 14.04
    command: ["sleep", "infinity"]

# Apply configurations
- name: Add container to networks 
  docker_container:
    name: sleepy 
    networks:
        - name: TestingNet
          ipv4_address: 172.1.1.18
          links:
            - sleeper
        - name: TestingNet2
          ipv4_address: 172.1.10.20
```

### Playbook
**Playbook** orchestrates the **module** execution.

#### task
sequential **Modules** are grouped in **Tasks**. For example:
```yaml
tasks:
- name: Create a data container 
...

- name: Start a container with a comman 
...

- name: Add container to networks 
...
```

#### hosts
where should these **tasks** execute:
```yaml
hosts: databases
```

!!! note
    the value `databases` is defined in **Ansible Inventory List**

#### remote_user
```yaml
remote_user: root
```

!!! error
    Ansible is VERY STRICT about identation

#### vars
```yaml
vars:
    tablename: foo 
    tableowner: someuser

tasks:
    - name: Rename table {{ tablename }} to bar 
      postgresql_table:
        table: {{ tablename }} 
        rename: bar
```

### Play

a combination of <u>host</u>, <u>task</u>, <u>user</u> and <u>name(optional)</u> is called a **Play**, a **Playbook** contains a sequence of **Play**

```yaml
- name: this Play aims to create a table, set an owner etc
  hosts: xxx
  remote_user: xxx
  tasks:
    - ...
    - ...
```

!!! note
    **Plays** could be depending on each other


### Inventory
a list of all machines that involved in task executions. Example:
```yaml
10.24.0.100

[webservers]
web1.myserver.com
web2.myserver.com

[databases]
10.24.0.7
10.24.0.8
```

- the syntax `[databases]` groups multiple <u>IP address</u> together, so that you can reference them in the **Play**
- you can use either <u>IP address</u> or <u>hostnames</u>
- the IP could be from Cloud servers, virtual or bare metal servers

## Interesting Facts
### Ansible vs Docker
1. Ansible allow you to create the same App across many envs, not only as a Container.
    <img src="./imgs/ansible_vs_docker.png" />
1. with Ansible, you can also control other things besides Container:
    - the Host the container is running on
    - the Host Storage
    - the Network

### Ansible Tower
an UI dashboard from Redhat

- centrally store automation tasks
- across teams
- configure permissions
- manage inventory

### Alternatives
||Ansible|Puppet and Chef|
|:-|:-|:-|
|Language|YAML|Ruby: more difficult to learn|
|Agent|agentless|Installation needed|
