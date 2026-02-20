# OCI - Helm - Chart verwenden

## Prerequisites 

  * Repo "flux" (Repo mit Deinem Namen) muss ausgecheckt sein (git clone https://gitlab.com/training.tn<nr>/<dein-repo>.git) als flux

## Schritt 1: OCIRepository einrichten 

```
cd
# Falls noch nicht existent, tut aber auch ansonsten nicht weh !
mkdir -p manifests/flux/clusters/production/apps/sources 
cd manifests/flux/clusters/production/apps/sources
nano cloudpirates.yaml 
``` 

```
# oci-cloudpirates.yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: cloudpirates-mariadb
  namespace: flux-system
spec:
  interval: 10m
  url: oci://registry-1.docker.io/cloudpirates/mariadb
  ref:
    version: "0.14.1"
```

```
# Wieder nach online pushen
git add -A
git commit -am "Added OCIRepository cloudpirates-mariadb"
git push
```

<img width="1092" height="229" alt="image" src="https://github.com/user-attachments/assets/f83848e1-6b72-4c3b-bf11-6613ef8f914a" />

## Schritt 2: Funktioniert nicht ? Warum ? 

```
# Ihr müsst einen Moment warten bis die neue "commit-id" zu sehen ist
# bzw. bis der Fehler kommt (System mach erst ein dry-run und spring noch garnicht auf die neue commit-id
```

```
# Er hat das richtige commit gezogen
flux get sources git
# aber dieses noch nicht angewendet
flux get kustomizations
```

<img width="1898" height="120" alt="image" src="https://github.com/user-attachments/assets/99830fd5-8e49-4e72-b0f1-645fb377480a" />

```
# das Feld "version" gibt es nicht
# Welches Feld gibt es denn, unter ref ?
kubectl explain OCIRepository.spec.ref
```

## Schritt 3: version ändern in tag:

```
nano cloudpirates.yaml 
``` 

```
# vi cloudpirates.yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: cloudpirates-mariadb
  namespace: flux-system
spec:
  interval: 10m
  url: oci://registry-1.docker.io/cloudpirates/mariadb
  ref:
# ---> hier: von version -> nach -> tag
    tag: "0.14.1"
```

```
# Wieder nach online pushen
git add -A
git commit -am "Added OCIRepository cloudpirates-mariadb"
git push
```

```
# Er hat das richtige commit gezogen
flux get sources git
# aber dieses noch nicht angewendet
watch flux get kustomizations
# << raus kommt ihr wieder mit CTRL+C 
```

```
# ist ociregistry da ?
kubectl -n flux-system get ocirepositories
```



## Schritt 4: HelmRelease anlegen 

   * version wird hier nicht verwendet, weil das bereits im OCIRepository ist.

```
cd
cd manifests/flux/clusters/production/apps/releases
nano mariadb.yaml
```

```
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: mariadb
  namespace: default
spec:
  interval: 10m
  chartRef:
    kind: OCIRepository
    name: cloudpirates-mariadb
    namespace: flux-system
```

```
git add -A
git commit -am "added helmrelease mariadb"
git push
```



## Schritt 6: War die installation für mariadb - erfolgreich ?

```
flux get helmreleases 
kubectl get pods
# Was ist das Problem ?
kubectl describe pods mariadb
```

## Schritt 7: csi nfs treiber installieren und einrichten

```
cat helmfile-csi-nfs.yaml
helmfile -f helmfile-csi-nfs.yaml sync
```

```
# Überprüfen
kubectl -n kubesystem get pods | grep nfs
```

```
# Was ist jetzt mit dem mariadb pod
# Creating .... Ready -> 1/1 dauert immer einen Moment 
kubectl get pods
```

## Schritt 8: flux geradeziehen 

  * Wenn es zu lange dauert, könnte es sein, dass flux immer noch einen fehler anzeigt

```
flux get helmreleases -A
```

<img width="1900" height="199" alt="image" src="https://github.com/user-attachments/assets/1e18b427-73d1-4b1e-9561-f43fad12460b" />

```
# falls dieser hängt 
flux reconcile helmrelease mariadb -n default
# machen wir das mit --force 
flux reconcile helmrelease mariadb -n default --force
helm list -A
helm -n default status mariadb
```

