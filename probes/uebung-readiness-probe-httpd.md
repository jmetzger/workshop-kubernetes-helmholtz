# Übung: HTTP Readiness Probe in Kubernetes

## Ziel
Du lernst, wie eine HTTP Readiness Probe konfiguriert wird und wie Kubernetes damit entscheidet, ob ein Pod Traffic empfangen darf.

---

## Hintergrund

Eine **Readiness Probe** prüft, ob ein Container bereit ist, Anfragen zu empfangen.  
Schlägt sie fehl → Pod wird aus dem Service-Endpunkt entfernt (kein Traffic).  
Liveness Probe hingegen: bei Fehler → Container-Neustart.

---

## Vorbereitung: Arbeitsverzeichnis anlegen

```bash
cd
mkdir -p manifests/readinessprobe
cd manifests/readinessprobe
```

---

## Schritt 1: Namespace anlegen

```bash
kubectl create namespace probe-demo
```

---

## Schritt 2: Deployment mit HTTP Readiness Probe erstellen

Öffne die Datei zum Bearbeiten:

```bash
nano readiness-deployment.yaml
```

Inhalt einfügen:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
  namespace: probe-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: nginx:alpine
          ports:
            - containerPort: 80
          readinessProbe:
            httpGet:
              path: /                 # nginx liefert index.html mit HTTP 200
              port: 80
            initialDelaySeconds: 5    # Warte 5s nach Container-Start
            periodSeconds: 10         # Prüfe alle 10s
            failureThreshold: 3       # 3 Fehler → Pod wird als "not ready" markiert
            successThreshold: 1       # 1 Erfolg → Pod wieder "ready"
            timeoutSeconds: 2         # Timeout pro Request
```

Anwenden:

```bash
kubectl apply -f readiness-deployment.yaml
```

---

## Schritt 3: Service erstellen

Öffne die Datei zum Bearbeiten:

```bash
nano webapp-svc.yaml
```

Inhalt einfügen:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: webapp-svc
  namespace: probe-demo
spec:
  selector:
    app: webapp
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl apply -f webapp-svc.yaml
```

---

## Schritt 4: Status beobachten

```bash
# Pod-Status anzeigen (READY-Spalte beachten)
kubectl get pods -n probe-demo -w

# Endpunkte des Services prüfen
kubectl get endpoints webapp-svc -n probe-demo
```

Solange die Probe erfolgreich ist, zeigt `READY 1/1` und der Pod ist im Endpunkt gelistet.

---

## Schritt 5: Fehlerfall simulieren

Wir löschen die `index.html` im laufenden Container, sodass nginx mit 404 antwortet und die Probe fehlschlägt.

```bash
# Pod-Name ermitteln
POD=$(kubectl get pod -n probe-demo -l app=webapp -o jsonpath='{.items[0].metadata.name}')

# Datei löschen → nginx antwortet mit 404
kubectl exec -n probe-demo $POD -- rm /usr/share/nginx/html/index.html
```

Jetzt beobachten:

```bash
# Status verfolgen
kubectl get pods -n probe-demo -w

# Endpunkte prüfen → Pod verschwindet aus der Liste!
kubectl get endpoints webapp-svc -n probe-demo

# Details der Probe-Fehler
kubectl describe pod -n probe-demo $POD | grep -A 10 "Conditions\|Readiness"
```

**Erwartetes Ergebnis:** `READY 0/1` — Pod empfängt keinen Traffic mehr.

---

## Schritt 6: Fehler beheben

```bash
# index.html wiederherstellen
kubectl exec -n probe-demo $POD -- sh -c 'echo "OK" > /usr/share/nginx/html/index.html'

# Status verfolgen
kubectl get pods -n probe-demo -w
```

Nach einem erfolgreichen Probe-Check (`successThreshold: 1`) ist der Pod wieder `READY 1/1`.

---

## Schritt 7: Probe-Events im Log prüfen

```bash
kubectl describe pod -n probe-demo $POD
```

Im Abschnitt **Events** siehst du:
```
Warning  Unhealthy  Readiness probe failed: HTTP probe failed with statuscode: 404
Normal   Started    Container webapp started
```

---

## Aufräumen

```bash
kubectl delete namespace probe-demo
```

---

## Zusammenfassung

| Parameter             | Bedeutung                                  |
|-----------------------|--------------------------------------------|
| `initialDelaySeconds` | Wartezeit nach Container-Start             |
| `periodSeconds`       | Intervall zwischen den Prüfungen           |
| `failureThreshold`    | Anzahl Fehler bis Pod als "not ready" gilt |
| `successThreshold`    | Anzahl Erfolge bis Pod wieder "ready" ist  |
| `timeoutSeconds`      | Max. Wartezeit pro HTTP-Request            |

> **Merke:** Readiness Probe ≠ Liveness Probe  
> Readiness → kein Traffic | Liveness → Neustart des Containers
