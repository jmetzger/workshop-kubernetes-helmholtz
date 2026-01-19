# Beispiel nginx 

```
docker run --name test-nginx -d -p 8080:80 nginx

docker container ls
lsof -i
cat /etc/services | grep 8080
curl http://localhost:8080
docker container ls
# wenn der container gestoppt wird, keine ausgabe mehr, weil kein webserver
docker stop test-nginx 
curl http://localhost:8080


```

## Jeder port kann nur 1x vergeben werden

```
docker start test-nginx

# Gemecker ist gross, weil Port schon vergeben
docker run --name test-nginx2 -d -p 8080:80 nginx
```
