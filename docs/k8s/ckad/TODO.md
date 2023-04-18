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
https://kodekloud.com/topic/network-policies-4/
Network Policies
Ingress-1: last task => due to rewrite annotation
Ingress-2: last task => didnt work, 503??

https://kodekloud.com/topic/persistent-volumes-5/
Storage Class: 4

