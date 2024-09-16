docker compose vs kubernetes/Openshift
All of them are frameworks for container orchestration.

# Frequently used CLIs
```sh
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi $(docker images)
```