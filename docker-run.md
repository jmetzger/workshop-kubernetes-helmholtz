# Docker run 

## Beispiel (binden an ein terminal), detached

```
# optional, docker run does the same 
docker pull ubuntu
docker run -t -d --name my-newubuntu ubuntu:xenial
# will wollen überprüfen, ob der container läuft
docker container ls 
# image vorhanden 
docker images

# in den Container reinwechsel 
docker exec -it my_newubuntu bash 
docker exec my_newubuntu cat /etc/os-release 
```
