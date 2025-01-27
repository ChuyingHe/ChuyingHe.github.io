<img src="../imgs/1.OpenShift-Installation-Process-pdf-III.png" />

Step 1: The user runs the OpenShift installer. The installer asks
for the necessary cluster information and then creates the
<strong>installation configuration file</strong>
`install-config.yaml` accordingly.

<img src="../imgs/2.OpenShift-Installation-Process-pdf-III.png" />

Step 2: From the `install-config.yaml` installation configuration
file content, the OpenShift installer creates the
<strong>Kubernetes manifests</strong>. The
<strong>Kubernetes manifests</strong> contain the necessary
instructions to build the resources for the OpenShift installation.

<img src="../imgs/3.OpenShift-Installation-Process-pdf-III.png" />

Step 3: From the <strong>manifests</strong> content, the OpenShift
installation process creates the
<strong>ignition configuration files</strong> for the bootstrap node
`bootstrap.ign`, control plane nodes `master.ign`, and compute nodes
`worker.ign`.

<img src="../imgs/4.OpenShift-Installation-Process-IV.png" />

Step 4: The bootstrap node boots and fetches its remote resources
(`bootstrap.ign`) from the initial ignition data source, and then
finishes booting.<br />
➡️ At this stage, the Kubernetes API is running on the bootstrap
node. The bootstrap node hosts the remote resources required for
control plane nodes to boot (ignition configuration files) in the
<strong>Machine Configuration Server (MCS)</strong>. It also runs a
single instance of the etcd cluster.

<img src="../imgs/5.OpenShift-Installation-Process-pdf-VI.png" />

Step 5: The control plane nodes boot and fetch their remote
resources (the master.ign ignition configuration file) from the
bootstrap node, and then finish booting.

<img src="../imgs/6.OpenShift-Installation-Process-pdf-VI.png" />

Step 6: The bootstrap node starts a temporary control plane and
installs the etcd operator.

<img src="../imgs/7.OpenShift-Installation-Process-pdf-VIII.png" />

Step 7: The etcd operator running on the bootstrap node scales up
the etcd cluster to 3 instances using two control plane nodes.

<img src="../imgs/8.OpenShift-Installation-Process-pdf-VIII.png" />

Step 8: The temporary control plane running on the bootstrap node
schedules the production control plane to the control plane nodes.
The OpenShift installation process transfers the etcd cluster to the
control plane nodes.

<img src="../imgs/9.OpenShift-Installation-Process-pdf-X.png" />

Step 9: The temporary control plane shuts down, yielding to the
production control plane. At this stage, the Kubernetes API is
running on the production control plane.

<img src="../imgs/10.OpenShift-Installation-Process-pdf-X.png" />

Step 10: For full-stack automation installations, the installer
shuts down the bootstrap node. Since this stage, the bootstrap node
is no longer needed.

<img src="../imgs/11.OpenShift-Installation-Process-XI.II.png" />

Step 11: At this stage, the Production Control Plane hosts the
cluster remote resources (ignition configuration files) for Control
Plane Nodes and Compute Nodes in their MCS.

The Compute Nodes boot
and fetch their remote resources (the worker.ign ignition
configuration file) from the control plane nodes, finish booting,
and join the cluster.




!!! warning "Notice"
    - Red Hat only supports the use of 3 control plane nodes.
    - Red Hat has tested a maximum of 2000 compute nodes on a Red Hat OpenShift Container Platform 4.6 cluster.
    - Red Hat recommends the use of at least two compute nodes to ensure the high availability of the applications running on the cluster.
    - For more information, refer to the Planning your environment according to object maximums section of the Red Hat OpenShift Container Platform 4.6 documentation at https://access.redhat.com/documentation/en-us/openshift_container_platform/4.6/html-single/scalability_and_performance