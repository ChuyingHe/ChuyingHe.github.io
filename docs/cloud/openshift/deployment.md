# 1. Namespace
Create new Namespace/Project:
```bash
oc create ns my-project
```

Use the created NS:
```bash
oc ns my-project
```
or
```bash
oc project my-project
```

# 2. Secrets
Create **Source Secret** to pull source code from a private git repo ([reference](https://www.redhat.com/en/blog/private-git-repositories-part-4-hosting-repositories-gitlab)):

- the "username" is optional
- the password is a PAT (Github > Personal access tokens (classic))

<img src="../deployment/source-secret.png" width=500 />



# 3.1 Build Image without Pipeline
## ImageStream
```bash
oc create is backend
```
## BuildConfig
# 3.1 Build Image with Pipeline
