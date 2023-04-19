TODO:

1. what does this command mean:

command:
 - sh
 - -c
 - echo hello


sh -c is a way to run a shell command inside a container, and the -c option is used to pass a command string to the shell for execution.

1.1 yaml file - command: easier to put them in json format:
    command: ["/bin/sh", "-c", "echo Hello, World!"]


2. k run sega --image busybox --dry-run=client -o yaml > sega.yaml -- sleep 3600


3. force to replace:
k replace --force -f=/tmp/xxx.yaml


4. difference in pod.yaml file:
.containers[0].args:
 - sleep
 - "3600"
.containers[0].command:
 - sleep
 - "3600"


 Docker recommends to always explicily specify your port mappings as strings because of the way YAML parses numbers.

Practise these again:
1.Recap Core Concepts

2.Configuration
2.1 Commands: 10 (diff between command and args in the yaml file!)
2.4 Security Context: 2 (securityContext in .spec.securityContext vs .spec.containers[0].securityContext)
2.5 Resource Limits? 🐱继续

3.Mult-Container

4.POD Design
4.1 Multi-contianer: nr.8 (namespace)
5.Services
5.2 Network Policies
5.3 Ingress-1: last task => due to rewrite annotation
5.4 Ingress-2: last task => didnt work, 503??

6.State Persistaence
6.2 Storage Class: 4

