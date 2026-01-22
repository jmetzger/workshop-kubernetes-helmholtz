# ResourceQuota - Ressourcen pro Namespace begrenzen

## Hintergrund: Arten von ResourceQuotas

| Kategorie | Beispiele |
|-----------|-----------|
| **Compute** | `requests.cpu`, `requests.memory`, `limits.cpu`, `limits.memory` |
| **Object Count** | `pods`, `services`, `configmaps`, `secrets`, `persistentvolumeclaims` |
| **Storage** | `requests.storage`, `<storageclass>.storageclass.storage.k8s.io/requests.storage` |

**Wichtig:** Wenn eine ResourceQuota fuer CPU/Memory existiert, MUESSEN alle Pods diese Werte angeben!

## Schritt 1: Namespace erstellen

```
# Ersetze <dein-name> mit deinem Namen (z.B. hans, petra, etc.)
kubectl create namespace resource-<dein-name>
```

## Schritt 2: ResourceQuota anlegen

```
cd
mkdir -p manifests
cd manifests
mkdir 20-resourcequota
cd 20-resourcequota
```

```
nano 01-resourcequota.yml
``` 

```
# vi 01-resourcequota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
spec:
  hard:
    requests.cpu: "500m"
    requests.memory: "256Mi"
    limits.cpu: "1"
    limits.memory: "512Mi"
    pods: "3"
    services: "2"
    configmaps: "3"
```

```
kubectl apply -f . -n resource-<dein-name>
```

## Schritt 3: Quota pruefen

```
kubectl describe resourcequota my-quota -n resource-<dein-name>
```

Ausgabe zeigt Used vs Hard:
```
Resource         Used  Hard
--------         ----  ----
configmaps       1     3
limits.cpu       0     1
limits.memory    0     512Mi
pods             0     3
...
```

## Schritt 4: Pod mit Ressourcen erstellen

```
nano 02-pod1.yml
```

```
# vi 02-pod1.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

```
kubectl apply -f . -n resource-<dein-name>
```

## Schritt 5: Quota-Verbrauch pruefen

```
kubectl describe resourcequota my-quota -n resource-<dein-name>
```

Jetzt sollte Used aktualisiert sein (z.B. pods: 1, requests.cpu: 100m, etc.)

## Schritt 6: Test - Zu viele Pods

Erstelle pod2 und pod3 mit gleichem Format (anderer name: pod2, pod3):

```
nano 03-pod2.yml
```

```
# vi 03-pod2.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

```
nano 04-pod3.yml
```

```
# vi 04-pod3.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod3
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

```
kubectl apply -f . -n resource-<dein-name>
```

  * Versuche nun einen 4. Pod:

```
nano 05-pod4.yml
```

```
# vi 05-pod4.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod4
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100m"
        memory: "64Mi"
      limits:
        cpu: "200m"
        memory: "128Mi"
```

```
kubectl apply -f . -n resource-<dein-name>
```

**Erwarteter Fehler:**
```
Error from server (Forbidden): pods "pod4" is forbidden: exceeded quota: my-quota, requested: pods=1, used: pods=3, limited: pods=3
```

## Schritt 7: Test - Zu viel Memory

Loesche zuerst die Pods:

```
kubectl delete pod pod1 pod2 pod3 -n resource-<dein-name>
```

Versuche einen Pod mit zu viel Memory:

```
# vi 06-pod-big.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-big
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    resources:
      requests:
        cpu: "100m"
        memory: "500Mi"
      limits:
        cpu: "200m"
        memory: "600Mi"
```

```
kubectl apply -f 06-pod-big.yml -n resource-<dein-name>
```

**Erwarteter Fehler:**
```
Error from server (Forbidden): exceeded quota: my-quota, requested: requests.memory=500Mi, limited: requests.memory=256Mi
```

## Schritt 8: Test - Pod ohne Resources

```
# vi 07-pod-no-resources.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-no-resources
spec:
  containers:
  - name: nginx
    image: nginx:alpine
```

```
kubectl apply -f 07-pod-no-resources.yml -n resource-<dein-name>
```

**Erwarteter Fehler:**
```
Error from server (Forbidden): must specify limits.cpu, limits.memory, requests.cpu, requests.memory
```

## Aufraeumen

```
kubectl delete namespace resource-<dein-name>
```

## Zusammenfassung

| Szenario | Ergebnis |
|----------|----------|
| Pod mit korrekten Resources | Akzeptiert |
| 4. Pod (Limit: 3) | Abgelehnt |
| Memory > Quota | Abgelehnt |
| Pod ohne Resources | Abgelehnt |
