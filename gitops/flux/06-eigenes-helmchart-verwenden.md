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
  url: https://gitlab.com/training-tn<nr>/final-helm-chart-<nr>.git
  ref:
    branch: master
```

```
git add -A
git commit -am "added gitrepository"
git push
```

## Schritt 5: Überprüfen ob es geklappt hat 

  * Das kann 1 Minute dauern bis man es sieht

```
flux get source git 
```

## Schritt 6: helmRelease einpflegen 

```
mkdir -p manifests/flux/clusters/production/apps/releases  
cd manifests/flux/clusters/production/apps/releases
```

```
nano final-chart.yaml
```

```
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: final-chart
  namespace: flux-system
spec:
  interval: 1m
  targetNamespace: ns-final-chart
  install:
    createNamespace: true
  chart:
    spec:
      chart: final-chart         # Pfad zum Chart im Repo
      version: "0.1.0"
      sourceRef:
        kind: GitRepository
        name: mein-helm-chart
        namespace: flux-system
  values:
    replicaCount: 2
    service:
      type: NodePort
```

```
git add -A
git commit -am "added final-chart"
git push
```

## Schritt 7: Überprüfen 

```
flux get source git
flux get kustomization -A
flux get helmrelease -A
helm list -A
kubectl -n ns-final-chart get pods
```
