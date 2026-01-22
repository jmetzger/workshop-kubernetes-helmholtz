# Helm - Grundlagen

## Wo kann ich Helm-Charts suchen ? 

 * Im Telefonbuch von helm [https://artifacthub.io/](https://artifacthub.io)

## Komponenten 

### Chart

  * beeinhaltet Beschreibung und Komponenten 

### Chart - Bereitstellungsformen 

  * url
  * .tgz (abkürzung tar.gz) - Format 
  * oder Verzeichnis 

```
Wenn wir ein Chart installieren, wird eine Release erstellen 
(parallel: image -> container, analog: chart -> release)
```

## Installation 

### Was brauchen wir ? 

  * helm  client muss installiert sein

### Und sonst so ? 

```
# Beispiel ubuntu 
# snap install --classic helm

# Cluster auf das ich zugreifen kann und im client -> helm (und eine entsprechende .kube/config) 
# (helm ist nichts als anderes als ein Client-Programm)  

# Test
kubectl cluster-info 
```

