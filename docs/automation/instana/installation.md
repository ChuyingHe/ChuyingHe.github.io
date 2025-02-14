# Instana Standard Installation


# 1. Create a VM on fyre
Create a machine on fyre with at least 16Core, 64 Memory CoreOS. 

Using the <username>:<api_key> and the [fyre API](https://fyre.ibm.com/help#fyre-api) request to create the machine. Use `ssh` to access the VM's console

# 2. Preparing for a single-node deployment
> Doc: https://www.ibm.com/docs/en/instana-observability/current?topic=cluster-preparing

- Follow the instructions until th "**Create the directories**"
- Step **Add mount paths** ensures the device names remain the same after reboot. Skipping it for now
- `<base_domain>` is set to the fyre VM's public URL --> this can be changed in the CRD also

TO CHECK:
- Firewall rules
- Configuring an HTTP proxy

# 3. Installing
> Doc: https://www.ibm.com/docs/en/instana-observability/current?topic=edition-installing

In this step, we basically use the command `stanctl up` to automatically set up everything. Required parameters:
- download key or an official agent key: from Instana
- sales key: from instana
- domain: fyre VM's public URL
- tenant: tenant0
- unit: unit0
- admin password: setup yourself
- TLS: auto generate

> Tipp: 
> - use `stanctl --help`
> - all configs are in the local directory `.stanctl/`


## Namespaces
### instana-core
all backend components, get the components' logs one by one to check potential
### instana-unit
web & tenant. The `username` & `pw` to login to the Instana UI are stored in a secret `instana-unit` in this NS. Decode the secret to get the credential:

```bash
echo xxx | base64 -d
```

# Install Instana Agent 
One Agent for one Node!

<img src="../imgs/agent" />