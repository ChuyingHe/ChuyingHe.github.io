# Internal Registry
References:
- [Source video](https://www.youtube.com/watch?v=4Y4dyhQT8so&ab_channel=OCPdude)
- [OC Duc](https://docs.openshift.com/container-platform/3.11/install_config/registry/index.html)

Internal Registry is usually initialized during the OpenShift Cluster Installation, but if not, you can create one manually.


Get routes of image registry:
```bash
oc get svc -n openshift-image-registry

# output:
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
image-registry            ClusterIP   172.21.203.6   <none>        5000/TCP    6d18h
image-registry-operator   ClusterIP   None           <none>        60000/TCP   6d19h
```

Login to the container image registry:
```bash
docker login -u openshift -p $(oc whoami -t) <registry_ip>:<port>
```


docker login -u openshift -p $(oc whoami -t) 172.21.203.6:5000
