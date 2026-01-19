# Docker run 

## Beispiel (binden an ein terminal), detached

```
# optional, docker run does the same 
docker pull ubuntu:resolute # resolute ist 26.04
docker images 
docker run -t -d --name my-newubuntu ubuntu:resolute
# will wollen überprüfen, ob der container läuft
docker container ls 
# image vorhanden 
docker images

# in den Container reinwechsel 
docker exec -it my-newubuntu bash
```

```
# in der bash
cat /etc/os-release
exit
```


```
docker exec my-newubuntu cat /etc/os-release 
```

## umschauen logs 

```
docker run --name log-nginx -d nginx
docker exec -it log-nginx bash
```

```
cd /var/log/nginx
# alles auf /dev/stdout und /dev/stderr 
ls -la 
```

## Beispiele direkte Kommandos 

```
# kommando was nur in der bash existiert cd
docker run --name log-nginx2 -d nginx
docker exec log-nginx2 bash -c "cd /etc; ls -la"

docker exec log-nginx ls -la /var/log/nginx
```
