# Docker run 

## Beispiel (binden an ein terminal), detached

```
# optional, docker run does the same 
docker pull ubuntu:resolute # resolute ist 26.04 
docker run -t -d --name my-newubuntu ubuntu:resolute
# will wollen überprüfen, ob der container läuft
docker container ls 
# image vorhanden 
docker images

# in den Container reinwechsel 
docker exec -it my_newubuntu bash 
docker exec my_newubuntu cat /etc/os-release 
```
