<img src="../imgs/ocp-edge.png" width="600" />

Edge certificates encrypt the traffic between <ins>the client and the router</ins>, but leave the traffic between <ins>the router and the service</ins> unencrypted. 

👉 OpenShift generates its own certificate that it signs with its CA.

```bash
oc create route edge todo-https \
    --service todo-http \
    --hostname todo-https.apps.ocp4.example.com
```

!!! warning "NO `http://` or `https://` in --hostname"
    When creating a route in OpenShift, the `--hostname` flag specifies only the host portion of the URL (e.g., `todo-https.apps.ocp4.example.com`) without including `http://` or `https://`. This is because:

    - Hostname is separate from protocol – --hostname only specifies the domain name, not http:// or https://.
    - Protocol is determined by route type – edge termination enables HTTPS at the OpenShift router level.
    - Flexibility – Different TLS termination options control how HTTPS is handled, independent of the hostname.

- `edge` indicates its encrypted
- If the --key and --cert options are omitted, then the **RHOCP ingress operator** provides a certificate from the **internal Certificate Authority (CA)**.



!!! info "Certificates provided from internal 'Certificate Authority'"
    to view the **certificates** that the internal **CA** provides:
    ```bash
    oc get secrets/router-ca -n openshift-ingress-operator -o yaml
    ```


<img src="../imgs/network-ingress-edge-padlock.png" width=700 />
<img src="../imgs/network-ingress-edge-more.png" width=700 />
<img src="../imgs/network-ingress-edge-page-info.png" width=400 />