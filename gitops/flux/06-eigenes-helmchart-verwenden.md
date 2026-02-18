# Eigenes Helmchart verwenden 

  * helmchart liegt im git

## Schritt 1: Chart erstellen 

```
cd
mkdir helm-chart-test/
cd helm-chart-test
helm create final-chart
```

## Schritt 2: Repo einrichten 

  * Achtung: Repo muss leer sein
  * URL rauskopieren, z.B. https://gitlab.com/training-tn<nr>/final-chart-<dein-kuerzel>.git


## Schritt 3: lokal git initialisieren und pushen

```
# in git einchecken
git init
git remote add origin https://gitlab.com/training-tn<nr>/final-chart-<dein-kuerzel>.git
git add -A
git commit -am "Chart hochschicken"
git push -u origin master
```

## Schritt 4: In Flux einpflegen (GIT) 

```
cd
mkdir -p manifests/flux/clusters/production/apps/sources 
cd manifests/flux/clusters/production/apps/sources
```

```
nano mein-chart.yaml
```

```
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: mein-helm-chart
  namespace: flux-system
spec:
  interval: 1m
  url: https://gitlab.com/training-tn<nr>/final-helm-chart-<nr>
  ref:
    branch: master
```

```
git add -A
git commit -am "added gitrepository"
git push
```
