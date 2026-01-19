# Container - Image - Delete - Kill

```
docker run -t -d --name ubuntu-container ubuntu:resolute
docker stop ubuntu-container 
# Kill it if it cannot be stopped -be careful
docker start ubuntu-container 
docker kill ubuntu-container

# Get nur, wenn der Container nicht mehr läuft 
docker rm ubuntu-container

# oder alternative
docker rm -f ubuntu-container 


# image löschen 
docker rmi ubuntu:resolute 

# falls Container noch vorhanden aber nicht laufend 
docker rmi -f ubuntu:xenial 

```
