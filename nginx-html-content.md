# Example nginx with content

## Schritt 1: Simple Example 

```
# das gleich wie cd ~
# Heimatverzeichnis des Benutzers root 
cd
mkdir -p nginx-test/html
cd nginx-test/html
# vi index.html
nano index.html
```

```
Text, den du rein haben möchtest
``` 

```
cd ..
nano Dockerfile
``` 

```
FROM nginx:latest
COPY html /usr/share/nginx/html
```

```
# nameskürzel z.B. jm1 
docker build -t dockertrainereu/<dein-namenskuerzel>-hello-web . 
docker images
```


## Schritt 2: Push build 

```

# eventually you are not logged in 
docker login -u dockertrainereu
docker push dockertrainereu/<dein-namenskuerzel>-hello-web 
#aus spass geloescht
docker rmi dockertrainereu/<dein-namenskuerzel>-hello-web

```

## Schritt 3: docker laufen lassen

```
# sicherstellen, dass kein Alter container mit dem läuft
docker container ls -a | grep hello-web
# ansonsten
docker rm -f hello-web
```

```
# und direkt aus der Registry wieder runterladen 
docker run --name hello-web -p 8080:80 -d dockertrainereu/jm1-hello-web

# laufenden Container anzeigen lassen
docker container ls 
# oder alt: deprecated 
docker ps 

curl http://localhost:8080 


# 
docker rm -f hello-web 

```
