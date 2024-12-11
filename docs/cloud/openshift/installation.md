

# IPI vs UPI
Reference: [https://www.redhat.com/zh/blog/red-hat-openshift-disconnected-installations](https://www.redhat.com/zh/blog/red-hat-openshift-disconnected-installations)

**IPI (Installer-Provisioned Infrastructure):**

- OpenShift installer automates the provisioning of infrastructure.
- Less manual control over node placement, networking, and resource assignment.
- Customizations, such as ensuring site affinity, require post-install configuration or creative use of installer features (e.g., node labels, taints).

**UPI (User-Provisioned Infrastructure)**:

- You are responsible for provisioning the underlying infrastructure (e.g., VMs, networking, storage) before installing OpenShift.
- Offers full control over how nodes are distributed across sites, which allows:
    - Pre-assigning nodes to specific data centers or availability zones.
    - Customizing networking, storage, and VM placement based on your requirements.