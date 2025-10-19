- [Udemy Course](https://ibm-learning.udemy.com/course/learn-ansible/learn/lecture/16681102#overview)
- [Kodekloud](https://learn.kodekloud.com/user/courses/labs-ansible-for-the-absolute-beginners?utm_source=udemy&utm_medium=labs&utm_campaign=ansible)

# 1. Install

```bash
brew install ansible
```

!!! error
    Ansible is VERY STRICT about identation

# 2. Config
double check the version and where the default config file is:

```bash
> ansible --version
ansible [core 2.18.6]
  config file = None
  configured module search path = ['/Users/<User>/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /opt/homebrew/Cellar/ansible/11.7.0_1/libexec/lib/python3.13/site-packages/ansible
  ansible collection location = /Users/<User>/.ansible/collections:/usr/share/ansible/collections
  executable location = /opt/homebrew/bin/ansible
  python version = 3.13.7 (main, Aug 14 2025, 11:12:11) [Clang 17.0.0 (clang-1700.0.13.3)] (/opt/homebrew/Cellar/ansible/11.7.0_1/libexec/bin/python)
  jinja version = 3.1.6
  libyaml = True
```

!!! note "Config File"
    - commonly used location in iOS: `/Users/<User>/ansible.cfg`
    - the **configuration parameters** are divided into **sections**:
        - `[default]`
        - `[inventory]`
        - `[privileges_escalation]`
        - `[paramiko_connection]`
        - `[ssh_connection]`
        - `[persistent_connection]`
        - `[colors]`

    - example of `[default]` section:
        ```cfg
        [defaults]
        inventory       = /etc/ansible/hosts
        1og_path        = /var/log/ansible.log

        library         = /usr/share/my_modules/
        roles_path      = /etc/ansible/roles
        action_plugins  = /usr/share/ansible/plugins/action
        
        gathering       = implicit

        # SSH timeout
        timeout         = 10
        forks           = 5
        ```
    
!!! note "Overwriting config file"
    you can overwrite the default config in different way:

    1. define another in Playbook Directory: `/db-playbooks/ansible.cfg`
    2. use ENV `$ANSIBLE_CONFIG`:
      ```bash
      ANSIBLE_CONFIG=/opt/ansible-common.cfg ansible-playbook playbook.yml
      ```
    3. overwriting <u>single parameter</u>:
      |in *.cfg|as ENV|
      |:-|:-|
      |`gathering=implicit`|`ANSIBLE_GATHERING=explicit`|

    **Priority Chain**:
    
    1. single parameter as ENV: 
        - temporary oneliner:
          ```bash
          ANSIBLE_GATHERING=explicit ansible-playbook playbook.yml
          ```
        - shell-wide:
          ```bash
          export ANSIBLE_GATHERING=explicit 
          ansible-playbook playbook.yml
          ```
    1. `*.cfg` pass in ENV: 
      ```bash
      ANSIBLE_CONFIG=/opt/ansible-common.cfg ansible-playbook playbook.yml
      ```
    1. `*.cfg` in Playbook's directory 
    1. `~/.ansible.cfg` (HOME directory) 
    1. default `ansible.cfg` (`/Users/<User>/ansible.cfg`)

!!! info "View Configuration"
    ```bash
    # render all config
    ansible-config list

    # show current config file
    ansible-config view

    # show all config, and where got them from
    ansible-config dump
    ```


# 3. Inventory
**inventory** is a list of all machines that involved in task executions. The inventory file is in an <u>INI like format</u>. Example:

```yaml
10.24.0.100

[webservers]
web1.myserver.com
web2.myserver.com

[databases]
10.24.0.7
10.24.0.8
```

- you can use either <u>IP address</u> or <u>hostnames</u>
- the IP could be from Cloud servers, virtual or bare metal servers
- default inventory file: `/etc/ansible/hosts`


## alias
here we specifiy alias: `web`, `db`, `mail`, `web2`
```bash
web   ansible_host=server1.company.com
db    ansible_host=server2.company.com
mail  ansible_host=server3.company.com
web2  ansible_host=server4.company.com
```

### inventory parameters
common **inventory parameters**:
    
- `ansible_host`: the FQDN or IP address of a server
- `ansible_connection`: how to connect to the server
- `ansible_port`: the connection port --> by default `22`for ssh
- `ansible_user`: the user used to make remote connections --> by default is `root` for linux
- `ansible_ssh_pass` defines the SSH password for Linux.
  - NOT ideal -> PW will be plain text
    
Example:
```bash
web   ansible_host=server1.company.com  ansible_connection=ssh    ansible_user=root
db    ansible_host=server2.company.com  ansible_connection=winrm  ansible_user=admin
mail  ansible_host=server3.company.com  ansible_connection=ssh    ansible_ssh_pass=p@#
web2  ansible_host=server4.company.com  ansible_connection=winrm

# 有一个名为 "localhost" 的目标主机
# 使用 localhost 连接插件（即不在远程服务器上执行，而是在当前控制机本地执行任务）
localhost ansible_connection=localhost
```


!!! note "winrm"
    `WinRM` stands for **Windows Remote Management**. It is Microsoft's protocol for remotely managing Windows systems, similar to what SSH is for Linux/Unix systems.

    |`ansible_connection`|**inventory parameters** for password|
    |:-|:-|
    |`ssh`|`ansible_ssh_pass`|
    |`winrm`|`ansible_password`|


## groups
use `[group_name]` to group servers together, so that you can reference them in the **Play**

```bash
[webservers]
web1.myserver.com
web2.myserver.com
```

**group of groups**:
```bash
[web_servers]
web1
web2

[db_servers]
db1

[all_servers:children]    # <-- 注意这里要加 :children
web_servers
db_servers
```

!!! note "常见的应用场景"
    make groups out of different perspectives:  
    ```bash
    [db_nodes]
    sql_db1
    sql_db2

    [web_nodes]
    web_node1
    web_node2
    web_node3

    [boston_nodes]
    sql_db1
    web_node1

    [dallas_nodes]
    sql_db2
    web_node2
    web_node3
    ```


## Inventory Format
the **Inventory** can be written in INI or YAML formats, here are examples:

- INI
  ```ini
  [webservers]
  webl.example.com
  Web2.example.com
  
  [dbservers]
  db1.example.com
  db2.example.com

  [allservers:children]
  webservers
  dbservers
  ```
- YAML: suitable for more complex organisation
  ```yaml
  allservers:
    children:
      webservers: 
        hosts:
          webl.example.com: 
          web2.example.com:
      dbservers: 
        hosts:
          db1.example.com:
          db2.example.com:
  ```

# 4. Variables
**Variables** store information that varies with each host using <u>Jinja2 Tempating</u>([Mini-course](https://learn.kodekloud.com/user/courses/jinja2-basics-mini-course)). **Inventory Parameter** is also **variables**. 

!!! note "how to define"
    - in **playbook**
    - in dedicate variable file


```yaml
name: Add DNS server to resolv.conf
hosts: localhost
vars:
  dns_server: 10.1.250.10                 # variable definition
tasks:
  - lineinfile:
      path: /etc/resolv.conf 
      line: 'nameserver {{ dns_server }}' # variable usage
```

!!! note "variable usage"
    - pure variable: `source: '{{ dns_server }}'` --> WITH quotes!
    - variable concatenated in text: `source: blabla{{ dns_server }}blabla` --> WITHOUT quotes!

## Types
- **string**
- **number**: integer or float
- **boolean**: 
    - `true` = `True`, `'true'`, `'t'`, `'yes'`, `'y'`, `'on'`, `'1'`, `1`, `1.0`
    - `false` = `False`, `'false'`, `'f'`, `'no'`, `'n'`, `'off'`, `'0'`, `0`, `0.0`
- **list**: list of value(of any type):
    - example:
      ```yaml
      packages:
        - nginx
        - postgresql
        - git
      ```
    - usage:
      ```yaml
      # whole list
      loop: "{{ packages }}"

      # first value in the list
      package0: "{{ packages[0]] }}"
      ```
- **dictionary**: key and value can be any type
    - example:
      ```yaml
      user:
        name: "admin"
        password: "secret"
      ```
    - usage
      ```yaml
      # whole dict
      user: "{{ user }}"

      # first element in the dict
      name: "{{ user.name }}"
      ```

## Variable Precedence
you can define **variable** in **group**, a **host**, a **play** or as extra CLI variable:

1. in **group**
    ```yaml
    [web_servers:vars]
    dns_server=10.5.5.3
    ```
1. in **host**
    ```yaml
    web2 ansible_host=172.20.1.101  dns_server=10.5.5.4
    ```
1. in **play**
    ```yaml
    - name: Configure DNS Server
      hosts: all
      vars:
        dns_server: 10.5.5.5
      tasks:
      - nsupdate:
        server: '{{ dns_server }}'
    ```
1. in **CLI**:
    ```bash
    ansible-playbook playbook.yml --extra-vars "dns_server=10.5.5.6"
    ```

!!! info
    [variable precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)

## Registering Variables
in a **play**, you can pass result of a **task** to the upcoming **task**:
```yaml
- name: Check filesystem mount
  shell: df -h /opt/instana
  register: result

- name: Fail if mount point doesn't exist
  fail:
    msg: "/opt/instana mount point not found"
  when: result.rc != 0

- name: Validate available space
  fail:
    msg: "Insufficient disk space on /opt/instana"
  when: result.stdout | regex_search('([0-9]+)%') | int >= 90
```


!!! note "`result`"
    a example of how variable `result` could looks like

    ```json
    "result": {
        "changed": true,
        "cmd": "cat /etc/hosts",
        "delta": "0:00:00.005621",
        "end": "2024-01-15 10:30:45.123456",
        "failed": false,
        "rc": 0,
        "start": "2024-01-15 10:30:45.117835",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "127.0.0.1   localhost\n127.0.1.1   my-ubuntu-server\n192.168.1.10 webserver01\n192.168.1.20 dbserver01\n# The following lines are desirable for IPv6 capable hosts\n::1     ip6-localhost ip6-loopback\nfe00::0 ip6-localnet\nff00::0 ip6-mcastprefix\nff02::1 ip6-allnodes\nff02::2 ip6-allrouters",
        "stdout_lines": [
            "127.0.0.1   localhost",
            "127.0.1.1   my-ubuntu-server",
            "192.168.1.10 webserver01",
            "192.168.1.20 dbserver01",
            "# The following lines are desirable for IPv6 capable hosts",
            "::1     ip6-localhost ip6-loopback",
            "fe00::0 ip6-localnet",
            "ff00::0 ip6-mcastprefix",
            "ff02::1 ip6-allnodes",
            "ff02::2 ip6-allrouters"
        ]
    }
    ```

    important attributes:

    - `.changed`: Whether the task changed the system
    - `.cmd`:  The command that was executed
    - `.rc` Return code (0 = success)
    - `.stdout`:  Full command output as a string
    - `.stdout_lines`:  Output as a list (array) of lines
    - `.stderr`:  Error output (if any)
    - `.failed`:  Whether the task failed
    - `.start/.end`:  Timestamps of execution

### How to debug
2 ways to debug and see what a **registering variable** content

1. use `debug` mode
    ```yaml
    - name: Check /etc/hosts file
      hosts: all
      tasks:

      - shell: cat /etc/hosts # ---- register mode ----
        register: result      # 1. register a variable named "result", saved shell result to it

      - debug:                # ---- debug mode ----
        var: result           # 2. print "result"
    ```
2. add flag `-v` while running the playbook:
    ```bash
    ansible-playbook –i inventory playbook.yml –v
    ```


!!! warning "scope"
    **registering variable** is associated to its **host** and is available for the rest of the **playbook** execution.

## Scope
**variables** has scope depends on where they are declared:

### in inventory
```ini
# accessible in all hosts
[all:vars] 
app_list=['vim', 'sqlite', 'jq']

# accessible in current host
localhost ansible_connection=local nameserver_ip=8.8.8.8 snmp_port=160-161  # snmp_port is ONLY accessible for "localhost"
node01 ansible_host=node01 ansible_ssh_pass=caleston123
```

### in playbook
```yaml
---
- hosts: localhost
  vars:
    car_model: 'BMW M3'
    country_name: USA
    title: 'Systems Engineer'
  tasks:
    - command: 'echo "My car is {{ car_model }}"'
    - command: 'echo "I live in the {{ country_name }}"'
    - command: 'echo "I work as a {{ title }}"'
```
## Magic Variables

### `hostvars` 
#### variables defined in other **hosts**
**Question**: is it possible for `web1` and `web3` to get the `dns_server` variable defined in `web2` scope?
```yaml
web1 ansible_host=172.20.1.100
web2 ansible_host=172.20.1.101 dns_server=10.5.5.4
web3 ansible_host=172.20.1.102
```
Answer: yes if using **magic variable** `hostvars`:

```yaml
- name: Print dns server
  hosts: web3                                   # in web3
  tasks:
  - debug:
    msg: '{{ hostvars['web2'].dns_server }}'    # get the variable defined in web2
```

#### info about other **hosts**
**magic variable** can be used to get information about other **hosts** too:

```yaml
msg: '{{ hostvars['web2'].ansible_host }}'
msg: '{{ hostvars['web2'].ansible_facts.architecture }}'
msg: '{{ hostvars['web2'].ansible_facts.devices }}'
msg: '{{ hostvars['web2'].ansible_facts.mounts }}'
msg: '{{ hostvars['web2'].ansible_facts.processor }}  # 也可以这样写： msg: '{{ hostvars['web2']['ansible_facts']['processor'] }}'
```

### `groups['boston_nodes']`
it return all **hosts** under it:
```yaml
msg: '{{ groups['boston_nodes'] }}'

# result:
web1
web2
```

### `group_names`
it returns all the **groups** the current **host** belongs to:
```yaml
# web1
msg: '{{ group_names }}'

# result:
boston_nodes
webservers
```

# 5. Facts
When you run a playbook and when Ansible connects to a target machine, a `setup` module runs automatically, and collects **facts** and saves in variable `ansible_facts`:

1. basic system information: system architecture, operating system version, processor details, memory details, serial numbers etc
1. host's network connectivity: interfaces, IP addresses, FQDN, MAC address etc
1. device information: volumes, mounts, available space etc
1. date and time


!!! note "automatic fact gathering module"
    ```yaml
    - name: Print hello message
      hosts: all
      tasks:
      - debug:
        msg: Hello from Ansible!
    ```

    if you run the above playbook you will see 2 tasks:
    
    <img src="../imgs/setup_module.png" width=600 />


!!! info "disable **facts**"
    ```yaml
    - name: Print hello message
      gather_facts: no  # disable fact gathering
      hosts: all
      ...
    ```

    only one task will be executed:

    <img src="../imgs/setup_module_disabled.png" />

    in `ansible.cfg` there is also a config to enable the fact gathering
    ```cfg
    gathering = implicit  # change it to explicit if you want to disable it
    ```

    ⚠️ the `gather_facts` in **playbook** > `gathering` in ansible.cfg


# 6. Playbook
A **Playbook** is a single YAML file containing a set of **plays**. A **Play** defines set of **tasks** to be run on host. A **task** is a single action 

!!! note "Example: Playbook contains 1 Play"
  ```yaml
  - name: Play 1      # <-- name of the play
    hosts: localhost  # <-- the host you want to run is ALWAYS on "play" level
    tasks:            # <-- tasks to excute
      - name: Execute command ‘date’
        command: date
      - name: Execute script on server
        script: test_script.sh
      - name: Install httpd service
        yum:
          name: httpd
          state: present
      - name: Start web server
        service:
          name: httpd
          state: started
  ```

!!! warning
    - `dict` has no order
    - `list` has order, therefore **tasks**(**plays**) could be depending on other **tasks**(**plays**)

!!! note
    `hosts` is defined in **inventory**

## Task
is an action that accomplished by **module**. For example, the highlighted parts are all **modules**:

<img src="../imgs/module.png" width=400 />

!!! note
    all available **modules** can be found either on the online doc or using:
    ```bash
    ansible-doc -l
    ```

## to run
```bash
ansible-playbook playbook.yml

ansible-playbook --help
```

## to verify
two ways to verify:

1. **Syntax Check mode**:
    ```bash
    ansible-playbook playbook.yml --syntax-check
    ```
1. **Check mode**: dry run without actual change. --> Not all **modules** support this 
    ```bash
    ansible-playbook playbook.yml --check
    ```
1. **Diff mode**: provides before & after comparison
    ```bash
    ansible-playbook playbook.yml --diff
    ```

## to lint
```bash
ansible-lint playbook.yml
```

<!--

## remote_user
```yaml
remote_user: root
```

## vars
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
-->