<img src="../imgs/ocp-passthrough.png" width="600" />

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
is a **Certificate Signing Request (CSR)**, an essential part of the process for obtaining a signed certificate. It contains:

1. Information about the **entity** that is requesting the certificate, such as:

    - Common Name (CN): The fully qualified domain name (e.g., todo-https.apps.ocp4.example.com).
    - Organization (O): The name of the organization (e.g., Red Hat).
    - Location details: Country (C), State (ST), and City (L).
    - Other optional metadata, such as email addresses.

2. **Public Key**: the public key that corresponds to the private key (`training.key`) you generated.

### `training-CA.srl`
The `training-CA.srl` file is used by **OpenSSL** (or other certificate tools) to keep track of the serial numbers of certificates issued by a **Certificate Authority (CA)**. It ensures that every certificate signed by the CA has a unique serial number, which is required by the X.509 standard for certificates.


!!! warning "`*.pem` VS `*.crt`"
    `*.pem` > `*.crt`
    
    `*.pem` could contain more info types, while `*.crt` only contains certificate.
    
    ||PEM (.pem)|CRT (.crt)|
    |:-|:-|:-|
    |**Stands for**|Privacy-Enhanced Mail|Certificate|
    |**Format**|Base64-encoded text with header/footer|Can be in PEM (Base64) or DER (Binary)|
    |**Content**|Can contain certificates, private keys, certificate chains, or other cryptographic data.|Typically contains only a single certificate.|
    |**Common Headers**|`-----BEGIN CERTIFICATE-----` (for a public certificate)<br/>`-----BEGIN PRIVATE KEY-----` (for a private key)<br/>`-----BEGIN CERTIFICATE REQUEST-----` (for a CSR)|`-----BEGIN CERTIFICATE-----`|
    |**Usage**|Used in OpenSSL, Apache, and many other applications.|Used by web servers, applications, and certificate authorities.|

# 1. Generate TLS certificates for the application

```bash
# (1) 
# Generate a Private Key `training.key`
openssl genrsa -out training.key 4096

# (2)
# Generate the certificate signing request (CSR) `training.csr`:
# - use the generated training.key`
# - specify the hostname: todo-https.apps.ocp4.example.com
openssl req -new \
    -key training.key -out training.csr \
    -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/\
    CN=todo-https.apps.ocp4.example.com"

# (3)
# Generate the signed certificate `training.crt`
# 
# INPUT:
# - `training.csr`: the request
# - `passphrase.txt`: the defined password
# -  `training.ext`: configuration file
# - CA info:
#       -  `training-CA.pem`: public key
#       -  `training-CA.key`: private key
#       
# OUTPUT:
#   - `training.crt`: a signed Certificate
#   - `training-CA.srl`: (If NOT exist) a list for documenting purpose
openssl x509 -req -in training.csr \
    -passin file:passphrase.txt \
    -extfile training.ext \
    -CA training-CA.pem \
    -CAkey training-CA.key \
    -CAcreateserial \       # Creates training-CA.srl if NOT exist, to track certificate Serial numbers
    -out training.crt \
    -days 1825 \            # Certificate’s validity period = 5 years
    -sha256                 # Certificate signature hashing = SHA-256

```

<img src="../imgs/generate_tsl_certifcate.png" width="600" />

!!! note "`-subj`"
    The `-subj` flag in the `openssl req -new `command specifies the **distinguished name (DN)** directly in the command line, instead of interactively entering it.

!!! note "Distinguished Name(DN)"
    The **Distinguished Name (DN)** is a unique identifier for an entity (such as a `server`, `organization`, or `individual`) in a **digital certificate**. It is used in:

    - X.509 certificates
    - Certificate Signing Requests (CSR)
    - LDAP (Lightweight Directory Access Protocol)

    ⛰️ **In this case, DN is the "ID Card" for the Certificate**


    A DN consists of multiple **Attributes**:
    
    |Attribute|Meaning|Example|
    |:-|:-|:-|
    |`C`|Country|`C=US` (United States)|
    |`ST`|State or Province|`ST=North Carolina`|
    |`L`|Locality (City)|`L=Raleigh`|
    |`O`|Organization|`O=Red Hat`|
    |`OU`|Organizational Unit|`OU=Engineering`|
    |`CN`|Common Name (usually a domain or hostname)|`CN=todo-https.apps.ocp4.example.com`|
    |`emailAddress`|Email Address (optional)|`emailAddress=admin@example.com`|


# 2. Generate TLS secret with Certificate and Key
- Certificate: `training.crt`
- Private Key: `training.key`

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