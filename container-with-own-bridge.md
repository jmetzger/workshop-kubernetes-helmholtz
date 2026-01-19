# Container with own bridge 

## Schritt 1: Netzwerk anlegen und 2. Netzwerk hinzufügen 

```
# use bridge as type
# docker network create -d bridge test_net
# but bridge is default 
docker network create test_net
docker network ls
docker network inspect test_net

# Container mit netzwerk starten 
docker container run -d --name nginx1 --network test_net nginx
docker network inspect test_net

# Weiteres Netzwerk (bridged) erstellen
docker network create demo_net
docker network connect demo_net nginx1

# Analyse 
docker network inspect demo_net
docker inspect nginx1

# Verbindung lösen 
docker network disconnect demo_net nginx1

# Schauen, wir das Netz jetzt aussieht 
docker network inspect demo_net

```

## Schritt 2: anpingen von neuem Container 

```
docker run -it --rm busybox
```

```
# anpingen des pods aus schritt 2
ping -c4 172.18.0.2
exit 
``` 

## Schritt 3: anpingen von container im gleichen netzwerk 

```
docker run -it --rm --network test_net busybox 
```

```
# anpingen des pods aus schritt 2
ping -c4 172.18.0.2
exit
```
