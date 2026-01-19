# Dockerfile design for small images 

## Übung - Schritt 1

```
cd
mkdir buildtest
cd buildtest
```

```
nano Dockerfile
```

```
FROM ubuntu:24.04
RUN apt-get update && \
    apt-get install -y inetutils-ping
```

```
docker build -t ubuntu-ping .
```


## Übung: Schritt 2:

```
# Dockerfile anpassen
FROM ubuntu:24.04
RUN apt-get update && \
    apt-get install -y inetutils-ping && \
    rm -rf /var/lib/apt/lists/*
```

```
docker build -t ubuntu-ping-small .
docker images
```



