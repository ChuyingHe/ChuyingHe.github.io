# on vSphere
- [Amazing walk-through video](https://www.youtube.com/watch?v=9__hpUWK5vw&ab_channel=OCPdude)
- [OC official doc](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html-single/installing_on_vsphere/index#upi-vsphere-installation-reqs)


## Bastion Host
1. download `openshift-install-linux.tar` and `openshift-client-linux.tar` 
2. configure DNS/Ingress?, so that:
    - `api.openshift.mycluster`: IP1 pointing to **Master Node** and **Bootstrap Node**
    - `apps.openshift.mycluster`: IP2 pointing to **Worker Node**
    - note: you can also use the same IP and do port-forwarding
3. generate/write & modify `install-config.yaml`
4. generate **manifests**
5. generate **ignition files** for the VMs
6. encode **ignition files** with base64 - results: `master.64`, `worker.64`
7. run a HTTP server
8. copy the **ignition files** to the HTTP server location (`var/www/html`)
9. create an **append file** that points to the base64-encoded **ignition file** in the HTTP server
    - the reason behind this: while creating a VM machine, you can configure it with ignition file
    - however the **ignition file** that generated in step 5 is too big 
    - the maximal **ignition file** you can directly upload while creating is 300kb
    - so we need the **the base64 encoded append file**(`append-bootstrap.64`) - which is in fact a small **ignition file** - to provide the source of a REAL ignition file

-- Now we take a break from the Bastion Host and go to create the Boostrap, Master and Worker Node --

10. export kubeconfig `expoert KUBECONFIG=~/openshift/mycluster/auth/kubeconfig`
11. now we can see the nodes: `oc get nodes`
12. list all pending Certificate Signing Requests (CSRs) for both Master and Worker Nodes: `oc get csr | grep Pending`
13. approve pending CSRs: `oc adm certificate approve <CSR-Name-1> <CSR-Name-2> ...`
14. check status of the Cluster Operator: `oc get co` --> we need the `console` CO to be able to access the UI
15. login to UI with credential:
    - username: `kubeadmin
    - you get the password from `cat mycluster/auth/kubeadmin-password`


!!! note "when do I need to approve a CSR?"

    1. **NEW (Master/Workder) Node joins OpenShift cluster**<br/>When a new worker node starts, it sends a CSR to request a client certificate.
    2. **Cluster certificate rotation** <br/>OpenShift automatically rotates node certificates when they expire, generating new CSRs.


## Boostrap Node
create VM machine for the Boostrap Node with the following configuration parameters:

    - `disk_EnableUUID=TRUE`
    - `guestinfo.ignition.config.data.encoding=base64`
    - `guestinfo.ignition.config.data=append-bootstrap.64` <- **the base64 encoded append file**
    - `guestinfo.afterburn.initrd.network-kargs=<Network Parameters>`, example:
        - `ip=172.16.1.13::172.16.1.1:255.255.255.192:bootstrap.mycluster.land:ens192:none nameserver=172.16.1.1`
        - the example has the format of `ip=<Boostrap Node's IP>::<Default Gateway>:<Subnet>...`

## Master Nodes 0-2
create VM machine for the Master Nodes 0-2 with the following configuration parameters:

    - `disk_EnableUUID=TRUE`
    - `guestinfo.ignition.config.data.encoding=base64`
    - `guestinfo.ignition.config.data=master.64` <- **the base64 ignition file for master node**
    - `guestinfo.afterburn.initrd.network-kargs=<Network Parameters>`, example:
        - `ip=172.16.1.10::172.16.1.1:255.255.255.192:bootstrap.mycluster.land:ens192:none nameserver=172.16.1.1`
        - the example has the format of `ip=<Master Node's IP>::<Default Gateway>:<Subnet>...`

## Worker Nodes 0-2
create VM machine for the Master Nodes 0-2 with the following configuration parameters:

    - `disk_EnableUUID=TRUE`
    - `guestinfo.ignition.config.data.encoding=base64`
    - `guestinfo.ignition.config.data=workder.64` <- **the base64 ignition file for worker node**
    - `guestinfo.afterburn.initrd.network-kargs=<Network Parameters>`, example:
        - `ip=172.16.1.20::172.16.1.1:255.255.255.192:bootstrap.mycluster.land:ens192:none nameserver=172.16.1.1`
        - the example has the format of `ip=<Worker Node's IP>::<Default Gateway>:<Subnet>...`

Worker Node's IP depends on the current machine. Our 3 nodes have:
- `172.16.1.20`
- `172.16.1.21`
- `172.16.1.22`