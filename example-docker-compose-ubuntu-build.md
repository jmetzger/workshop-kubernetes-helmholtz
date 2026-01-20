# Example Docker Compose (Ubuntu with Dockerfile) 

## Schritt 1: Erstellen 

```
cd
mkdir composetest
cd composetest
nano docker-compose.yml
```

```
services:
  myubuntu:
    build: ./myubuntu
    restart: always
```

```
mkdir myubuntu 
cd myubuntu
nano Dockerfile
```

```
FROM ubuntu:latest
RUN apt-get update && apt-get install -y inetutils-ping
CMD ["/bin/bash"]
```

```
cd ../
# wichtig, im mit der docker-compose.yml - seiend 
#pwd 
#~/bautest
docker compose up -d 
# wird image gebaut und container gestartet 
```

## Schritt 2 - Testen 

```
docker compose exec -it myubuntu 
```

```
# in der bash 
ping -c4 www.google.de
exit
```

## Hinweis:

```
Bei Veränderung vom Dockerfile, muss man den Parameter --build mitangeben 
docker compose up -d --build 
```
