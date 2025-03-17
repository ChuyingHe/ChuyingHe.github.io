
Compatible versions: 4.10, 4.12, 4.14

# Tipps
- learn how to use the documentation for "big picture" stuff. 
- For the smaller scale (writing yaml) i extensively used `oc explain`. 
- `oc set volume --help`

# Resources
- [EX280 Description](https://www.redhat.com/en/services/training/red-hat-certified-openshift-administrator-exam)
- [EX280 Exam Dumps](https://github.com/bugbiteme/EX280-study/tree/main)
- [Studying & Walk through](https://www.youtube.com/watch?v=nLhBrjhQRe8&ab_channel=Cloudlearning)
- DO180：
    - Chapter 5: Manage Storage for Application Config & Data: Limit etc
    - Chapter 6: Reliability: Health Probe etc
- [OC CLI Reference 4.14](https://docs.redhat.com/en/documentation/red_hat_build_of_microshift/4.14/html/cli_tools/microshift-oc-cli-commands)

# available resource in exam???
- a Document pdf
- ?[docs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.14)
https://docs.openshift.com/container-platform/4.14

# Browser
`Ctrl+F` search on browser


```bash
oc whoami —show-console
```

# Chapters to review
## DO280

### 2. Helm Chart
- use helm chart

### 3. Authentication and Authorization
- htpasswd
- CURD
- default roles(cluster-admin, self-provisioner)
- clusterrolebinding
- remove self-provisioner role from all users
- Remove user `kubeadmin`(also in doc)


### 4. Network Security
- create cert with the given tool
- apply networkPolicy

### 6. Enable Developer Self-Service
- cluster quotas
- self-provisioner role
- create-bootstrap-project-template
- limitrage 
- template.yaml
- oc edit projects.config.openshift.io cluster

### 7. Manage k8s Operator
install operator with cli


### 8. Security Context Constraints
- create scc
- use new sa: `oc set sa`
- create cronjob, update the account

### 9. Update


## DO180
### 5. Manage Storage for Application Configuration and Data
- storageclass
- provisioning PV
- PVC

- create secret
- create configmap
- mount secret/cm: `oc set volume`

### 6. Configure Applications for Reliability
- health probe
- resource limit: trouble shooting question `.containers[].resources.limits.memory` -> need more
- auto scaling