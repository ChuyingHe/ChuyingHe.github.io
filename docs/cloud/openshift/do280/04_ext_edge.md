<img src="../imgs/ocp-edge.png" width="600" />

Edge certificates encrypt the traffic between <ins>the client and the router</ins>, but leave the traffic between <ins>the router and the service</ins> unencrypted. 

👉 OpenShift generates its own certificate that it signs with its CA.

```bash
oc create route edge todo-https \
    --service todo-http \
    --hostname todo-https.apps.ocp4.example.com
```

- `edge` indicates its encrypted
- When the `--key api.key` and `--cert api.crt` options are omitted, the RHOCP ingress operator creates the required certificate with its own **Certificate Authority (CA)**.



!!! info "Certificates provided from internal 'Certificate Authority'"
    to view the **certificates** that the internal **CA** provides:
    ```bash
    oc get secrets/router-ca -n openshift-ingress-operator -o yaml
    ```


<img src="../imgs/network-ingress-edge-padlock.png" width=700 />
<img src="../imgs/network-ingress-edge-more.png" width=700 />
<img src="../imgs/network-ingress-edge-page-info.png" width=400 />