# Images

```bash
# to find available images from remote or local registries
podman search [ImageName]
podman search ubi           # default: in Docker
podman search quay.io/ubi   # specified in quay registry

# to fetch the image and save it locally
podman pull [ImageName]

# to list local imgs
podman images
```

# Container
```bash
podman run [ImageName] --name [ContainerName] echo 'Hello world!'
podman run -it [ImageName] --name [ContainerName] /bin/bash

podman run --name [ContainerName] -d -p 8080 [ImageName]
```