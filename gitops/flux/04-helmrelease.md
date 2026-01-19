# HelmRelease - Helm Charts deklarativ mit Flux ausrollen

## Hintergrund

`HelmRelease` ist die zentrale Flux CRD zum Ausrollen von Helm Charts. Der **helm-controller** reconciled diese Ressource und fuehrt `helm upgrade --install` automatisch aus.

| Eigenschaft | Beschreibung |
|-------------|--------------|
| **API Group** | `helm.toolkit.fluxcd.io/v2` |
| **Controller** | helm-controller |
| **Funktion** | Helm Release deklarativ verwalten |
| **Reconciliation** | Automatische Upgrades bei Aenderungen |

## Voraussetzungen

- Flux installiert (siehe [02-installation-helm.md](02-installation-helm.md))
- HelmRepository erstellt (siehe [03-helmrepository.md](03-helmrepository.md))

## Schritt 1: Vorbereitung

```
cd
mkdir -p manifests/flux-releases
cd manifests/flux-releases
```

## Schritt 2: HelmRelease fuer nginx erstellen

Wir rollen nginx aus dem cloudpirates Repository aus:

```
# vi 01-helmrelease-nginx.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: nginx
  namespace: default
spec:
  interval: 5m
  chart:
    spec:
      chart: nginx
      version: "1.3.3"
      sourceRef:
        kind: HelmRepository
        name: cloudpirates
        namespace: flux-system
      interval: 10m
  values:
    replicaCount: 2
    service:
      type: ClusterIP
      port: 80
```

**Erklaerung:**
| Feld | Wert | Bedeutung |
|------|------|-----------|
| `interval` | `5m` | Pruefe alle 5 Minuten auf Drift/Updates |
| `chart` | `nginx` | Chart Name aus dem Repository |
| `version` | `1.3.3` | Spezifische Chart Version |
| `sourceRef` | `cloudpirates` | Referenz auf HelmRepository |
| `values` | ... | Ueberschreibt Chart Default-Values |

## Schritt 3: HelmRelease anwenden

```
kubectl apply -f 01-helmrelease-nginx.yml
```

## Schritt 4: Status pruefen

```
kubectl get helmrelease -n default
```

**Erwartete Ausgabe:**
```
NAME    AGE   READY   STATUS
nginx   1m    True    Release reconciliation succeeded
```

**Status-Felder:**
- `READY: True` - Helm Release erfolgreich installiert
- `STATUS` - Details zur Reconciliation

## Schritt 5: Helm Release verifizieren

Flux fuehrt intern Helm-Befehle aus. Verifizierung:

```
kubectl get pods -n default -l app.kubernetes.io/name=nginx
```

**Erwartete Ausgabe:**
```
NAME                     READY   STATUS    RESTARTS   AGE
nginx-123abc-xxxxx       1/1     Running   0          2m
nginx-123abc-yyyyy       1/1     Running   0          2m
```

## Schritt 6: Service pruefen

```
kubectl get svc -n default -l app.kubernetes.io/name=nginx
```

**Erwartete Ausgabe:**
```
NAME    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   10.245.xxx.xxx  <none>        80/TCP    2m
```

## Schritt 7: Details des HelmRelease anzeigen

```
kubectl get helmrelease nginx -n default -o yaml | grep -A 10 status
```

**Wichtige Informationen:**
- `lastAppliedRevision` - Chart Version die installiert wurde
- `lastAttemptedRevision` - Letzte versuchte Version
- `conditions` - Status der Reconciliation

## Schritt 8: Helm Release mit helm CLI pruefen

Flux nutzt Helm intern. Verifizierung:

```
helm list -n default
```

**Erwartete Ausgabe:**
```
NAME    NAMESPACE  REVISION  UPDATED                   STATUS    CHART          APP VERSION
nginx   default    1         2026-01-18 21:00:00 UTC   deployed  nginx-1.3.3    1.27.2
```

## Schritt 9: Values anpassen (Declarative Update)

Wir aendern die Replica-Anzahl:

```
# vi 01-helmrelease-nginx.yml
# Aendere replicaCount: 2 -> replicaCount: 3
```

```
kubectl apply -f 01-helmrelease-nginx.yml
```

**Flux reconciled automatisch:**
```
kubectl get pods -n default -l app.kubernetes.io/name=nginx
```

Nach wenigen Sekunden sollten 3 Pods laufen.

## Schritt 10: Nginx testen

Port-Forward zum Service:

```
kubectl port-forward -n default svc/nginx 8080:80
```

In einem anderen Terminal:
```
curl http://localhost:8080
```

Erwartete Ausgabe: nginx Welcome-Seite

## Schritt 11: Automatische Upgrades mit Semver (Vorsicht!)

Flux unterstuetzt Semver-Constraints fuer automatische Updates:

| Constraint | Bedeutung | Beispiel |
|------------|-----------|----------|
| `1.3.3` | Exakte Version | Keine Updates |
| `>=1.3.0 <1.4.0` | Patch Updates | 1.3.3 → 1.3.9 |
| `1.3.x` | Alle 1.3.x Versionen | 1.3.3 → 1.3.9 |
| `*` | Neueste Version | Alle Updates |

**Vorsicht:** Automatische Updates koennen Breaking Changes einfuehren!

**Best Practice:** Spezifische Versionen verwenden und manuell upgraden.

## Schritt 12: Rollback bei Fehler (automatisch)

Flux kann automatisch Rollback durchfuehren:

```
# vi 02-helmrelease-with-rollback.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: nginx-safe
  namespace: default
spec:
  interval: 5m
  chart:
    spec:
      chart: nginx
      version: "1.3.3"
      sourceRef:
        kind: HelmRepository
        name: cloudpirates
        namespace: flux-system
  upgrade:
    remediation:
      remediateLastFailure: true
      retries: 3
  rollback:
    cleanupOnFail: true
    recreate: true
  values:
    replicaCount: 2
    service:
      type: ClusterIP
```

**Erklaerung:**
- `remediateLastFailure: true` - Rollback bei Fehler
- `retries: 3` - 3 Versuche vor Rollback
- `cleanupOnFail: true` - Resources bei Fehler aufraeumen

## Schritt 13: HelmRelease Suspend (Pausieren)

```
kubectl patch helmrelease nginx -n default \
  --type merge \
  -p '{"spec":{"suspend":true}}'
```

**Status pruefen:**
```
kubectl get helmrelease nginx -n default
```

**Resume:**
```
kubectl patch helmrelease nginx -n default \
  --type merge \
  -p '{"spec":{"suspend":false}}'
```

## Haeufige Szenarien

### Drift Detection

Flux erkennt manuelle Aenderungen:

```
# Manuell Scale aendern (nicht empfohlen!)
kubectl scale deployment nginx -n default --replicas=5
```

**Flux reconciled automatisch** und setzt `replicas` wieder auf den Wert in HelmRelease (z.B. 3).

### Dependencies zwischen HelmReleases

```
# vi 03-helmrelease-with-dependency.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: app-backend
  namespace: default
spec:
  interval: 5m
  dependsOn:
    - name: nginx
      namespace: default
  chart:
    spec:
      chart: nginx
      version: "1.3.3"
      sourceRef:
        kind: HelmRepository
        name: cloudpirates
        namespace: flux-system
```

**app-backend wartet** bis `nginx` HelmRelease erfolgreich ist.

## Troubleshooting

### HelmRelease haengt in "Not Ready"

```
kubectl get helmrelease nginx -n default -o yaml
```

**Haeufige Gruende:**
- Chart Version nicht gefunden
- HelmRepository nicht bereit
- Values Schema Fehler
- Timeout bei Installation

### Logs des helm-controller

```
kubectl logs -n flux-system deployment/helm-controller -f
```

### HelmChart Status pruefen

```
kubectl get helmchart -n flux-system
```

Flux erstellt automatisch ein `HelmChart` Objekt fuer jedes `HelmRelease`.

## Zusammenfassung

| Aktion | Befehl |
|--------|--------|
| HelmRelease erstellen | `kubectl apply -f 01-helmrelease-nginx.yml` |
| Status pruefen | `kubectl get helmrelease -n default` |
| Suspend | `kubectl patch helmrelease nginx ... suspend:true` |
| Resume | `kubectl patch helmrelease nginx ... suspend:false` |
| Helm Releases anzeigen | `helm list -n default` |
| Pods pruefen | `kubectl get pods -n default -l app.kubernetes.io/name=nginx` |

## Was haben wir erreicht?

- **Deklarativ:** Helm Charts als YAML Manifests
- **GitOps-ready:** Manifests im Git = Cluster State
- **Automatisch:** Updates bei Chart-Aenderungen oder Values-Aenderungen
- **Rollback:** Automatische Fehlerbehandlung
- **Dependencies:** Orchestrierung mehrerer Releases

## Naechster Schritt

Im naechsten Schritt koennt ihr:
- Git als Source nutzen (GitRepository) statt HelmRepository
- Kustomize mit Flux kombinieren
- Image Update Automation einrichten
- Alle HelmReleases in Git commiten (GitOps!)

## Aufraeumen

```
kubectl delete -f .
```

**Hinweis:** Dies loescht alle HelmRelease Objekte und die von ihnen erstellten Resources (Deployments, Services, etc.).
