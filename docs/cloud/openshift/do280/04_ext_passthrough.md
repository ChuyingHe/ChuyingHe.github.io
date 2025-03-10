
<table>
    <tr>
        <th>初始状态的文件</th>
        <th>本步骤结束之后的文件</th>
    </tr>
    <tr>
        <td>
<pre><code>
openssl-commands.txt
passphrase.txt
training.ext
training-CA.key
training-CA.pem
</code></pre>
        </td>
        <td>
<pre><code>
openssl-commands.txt
passphrase.txt
training.ext
training-CA.key
training-CA.pem
<span style="background-color: #ccd1f0">
training.key
training.csr
training-CA.srl
training.crt</span>
</code></pre>
        </td>
    </tr>
</table>

## original files
### `training-CA.key`
**private key** for the Certificate Authority (CA). Use it to sign other certificates

```txt
-----BEGIN ENCRYPTED PRIVATE KEY-----
MIIEqzCCAx0gAwIBAgIUTcgzLN5PmsMqvkifCXuaSxzLJlcwDQYJKoZIhvcNAQEL
VQQHDAdSYWxlaWdoMRAwDgYDVQQKDAdSZWQgSGF0MRkwFwYDVQQDDBBvYЗA0LmV4
...
-----END ENCRYPTED PRIVATE KEY-----
```

### `training-CA.pem`
**certificate** that includes **public key** and some **metadata**. Use it to distribute to clients or servers, to establish a trust chain for certificates signed by this CA.

```txt
-----BEGIN CERTIFICATE-----
MIIEqzCCAx0gAwIBAgIUTcgzLN5PmsMqvkifCXuaSxzLJlcwDQYJKoZIhvcNAQEL
VQQHDAdSYWxlaWdoMRAwDgYDVQQKDAdSZWQgSGF0MRkwFwYDVQQDDBBvYЗA0LmV4
...
-----END CERTIFICATE-----
```

### `training.ext`
ext=extended file system, created for Linux kernel. It contains additional configuration for the certificate:

```txt
subjectAltName = @alt_names

[alt_names]
DNS. 1 = *.ocp4.example.com
DNS. 2 = *.apps.ocp4.example.com
```


## created files

### `training.key`
Private Key for the Application that has the host `todo-https.apps.ocp4.example.com`

### `training.crt`
Signed public Certificate for the Application

### `training.csr`
is a Certificate Signing Request (CSR), an essential part of the process for obtaining a signed certificate. It contains:

1. Information about the **entity** that is requesting the certificate, such as:

    - Common Name (CN): The fully qualified domain name (e.g., todo-https.apps.ocp4.example.com).
    - Organization (O): The name of the organization (e.g., Red Hat).
    - Location details: Country (C), State (ST), and City (L).
    - Other optional metadata, such as email addresses.

2. **Public Key**: the public key that corresponds to the private key (`training.key`) you generated.

### `training-CA.srl`
The `training-CA.srl` file is used by **OpenSSL** (or other certificate tools) to keep track of the serial numbers of certificates issued by a **Certificate Authority (CA)**. It ensures that every certificate signed by the CA has a unique serial number, which is required by the X.509 standard for certificates.


!!! warning "*.pem VS *.crt"
    `*.pem` could contain more info types, while `*.crt` only contains certificate.
    
    ||PEM (.pem)|CRT (.crt)|
    |:-|:-|:-|
    |**Stands for**|Privacy-Enhanced Mail|Certificate|
    |**Format**|Base64-encoded text with header/footer|Can be in PEM (Base64) or DER (Binary)|
    |**Content**|Typically contains only a public certificate.|Can contain certificates, private keys, certificate chains, or other cryptographic data.|
    |**Common Headers**|`-----BEGIN CERTIFICATE-----` (for a public certificate)<br/>`-----BEGIN PRIVATE KEY-----` (for a private key)<br/>`-----BEGIN CERTIFICATE REQUEST-----` (for a CSR)|`-----BEGIN CERTIFICATE-----`|
    |**Usage**|Used in OpenSSL, Apache, and many other applications.|Used by web servers, applications, and certificate authorities.|

# 1. Generate TLS certificates for the application

<img src="../imgs/generate_certifcate_passthrough.png" />

```bash
# Generate a private key `training.key`
openssl genrsa -out training.key 4096

# Generate the certificate signing request (CSR) `training.csr`:
# - use the generated training.key`
# - specify the hostname: todo-https.apps.ocp4.example.com
openssl req -new \
    -key training.key -out training.csr \
    -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/\
    CN=todo-https.apps.ocp4.example.com"

# Generate the signed certificate `training.crt`
# - use the request: `training.csr`
# - use the defined password: `passphrase.txt `
# - use the CA info:
#       -  public key of CA: `training-CA.pem`
#       -  private key of CA: `training-CA.key`
# Output:
#   - a signed Certificate: `training.crt`
#   - a list for documenting purpose: `training.ext`
openssl x509 -req -in training.csr \
    -passin file:passphrase.txt \
    -CA training-CA.pem -CAkey training-CA.key -CAcreateserial \
    -out training.crt -days 1825 -sha256 -extfile training.ext
```

# 2. Generate TLS secret with Certificate and Key
- Certificate: `training.crt`
- Key: `training.key`

```bash
oc create secret tls todo-certs \
    --cert certs/training.crt --key certs/training.key
```
# 3. Create App and Service with Secret
Use the `secret` in the Deployment file:

```yaml
apiVersion: apps/v1
kind: Deployment
...output omitted...
        volumeMounts:
        - name: tls-certs
          readOnly: true
          mountPath: /usr/local/etc/ssl/certs
...output omitted...
      volumes:
      - name: tls-certs
        secret:
          secretName: todo-certs
---
apiVersion: v1
kind: Service
...output omitted...
  ports:
  - name: https
    port: 8443
    protocol: TCP
    targetPort: 8443
...output omitted..
```

# 4. Create the encrypted route with Service
```bash
oc create route passthrough todo-https \
    --service todo-https --port 8443 \
    --hostname todo-https.apps.ocp4.example.com
```