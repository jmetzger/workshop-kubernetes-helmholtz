# Docker-Container analysieren

```
docker run -d --name hello-web hello-world
docker ps -a 
docker inspect hello-web # hello-web = container name 
```

```
docker run -d --name nginxly nginx
docker ps -a
# ip abfragen 
docker inspect nginxly | grep -i ipad 
``` 
