# Aufräumen 

## Alle nicht verwendeten container und images löschen 

```
# Alle container, die nicht laufen löschen 
docker container prune 

# Alle images, die nicht an eine container gebunden sind, löschen 
docker image prune 

# container händisch, gleich viel am Stück
docker rm -f 9c 58 b0 45 d8 4b

```
