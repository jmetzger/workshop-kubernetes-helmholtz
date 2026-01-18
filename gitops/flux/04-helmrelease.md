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

Wir rollen nginx aus dem Bitnami Repository aus:

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
      version: ">=18.0.0 <19.0.0"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
      interval: 10m
  values:
    replicaCount: 2
    service:
      type: ClusterIP
```

**Erklaerung:**
| Feld | Wert | Bedeutung |
|------|------|-----------|
| `interval` | `5m` | Pruefe alle 5 Minuten auf Drift/Updates |
| `chart` | `nginx` | Chart Name aus dem Repository |
| `version` | `>=18.0.0 <19.0.0` | Semver Constraint (automatische Minor Updates) |
| `sourceRef` | `bitnami` | Referenz auf HelmRepository |
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

## Schritt 6: Details des HelmRelease anzeigen

```
kubectl describe helmrelease nginx -n default
```

**Wichtige Informationen:**
```
Status:
  Conditions:
    Last Transition Time:  2026-01-18T20:30:00Z
    Message:               Release reconciliation succeeded
    Reason:                ReconciliationSucceeded
    Status:                True
    Type:                  Ready
  Last Applied Revision:   18.2.4
  Last Attempted Revision: 18.2.4
  Observed Generation:     1
```

## Schritt 7: Helm Release mit helm CLI pruefen

Flux nutzt Helm intern. Verifizierung:

```
helm list -n default
```

**Erwartete Ausgabe:**
```
NAME    NAMESPACE  REVISION  UPDATED                   STATUS    CHART          APP VERSION
nginx   default    1         2026-01-18 20:30:00 UTC   deployed  nginx-18.2.4   1.27.3
```

## Schritt 8: Values anpassen (Declarative Update)

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

## Schritt 9: Erweitertes Beispiel - MariaDB mit ConfigMap Values

```
# vi 02-helmrelease-mariadb.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: mariadb
  namespace: databases
spec:
  interval: 5m
  chart:
    spec:
      chart: mariadb
      version: "19.x"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  values:
    auth:
      rootPassword: changeme123
    primary:
      persistence:
        enabled: false
```

**Namespace erstellen:**
```
kubectl create namespace databases
```

**HelmRelease anwenden:**
```
kubectl apply -f 02-helmrelease-mariadb.yml
```

## Schritt 10: Automatische Upgrades (Semver)

Flux unterstuetzt Semver-Constraints fuer automatische Updates:

| Constraint | Bedeutung | Beispiel |
|------------|-----------|----------|
| `18.2.4` | Exakte Version | Keine Updates |
| `>=18.0.0 <19.0.0` | Minor + Patch Updates | 18.2.4 → 18.3.0 |
| `18.x` | Alle 18.x Versions | 18.2.4 → 18.9.9 |
| `*` | Neueste Version | Alle Updates |

**Vorsicht:** `*` kann Breaking Changes einfuehren!

## Schritt 11: Rollback bei Fehler (automatisch)

Flux kann automatisch Rollback durchfuehren:

```
# vi 03-helmrelease-with-rollback.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: app-with-rollback
  namespace: default
spec:
  interval: 5m
  chart:
    spec:
      chart: nginx
      version: "18.x"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
  upgrade:
    remediation:
      remediateLastFailure: true
      retries: 3
  rollback:
    cleanupOnFail: true
    recreate: true
```

**Erklaerung:**
- `remediateLastFailure: true` - Rollback bei Fehler
- `retries: 3` - 3 Versuche vor Rollback
- `cleanupOnFail: true` - Resources bei Fehler aufraeumen

## Schritt 12: HelmRelease Suspend (Pausieren)

```
flux suspend helmrelease nginx -n default
```

**Oder mit kubectl:**
```
kubectl patch helmrelease nginx -n default \
  --type merge \
  -p '{"spec":{"suspend":true}}'
```

**Resume:**
```
flux resume helmrelease nginx -n default
```

## Haeufige Szenarien

### Drift Detection

Flux erkennt manuelle Aenderungen:

```
# Manuell Helm Release aendern (nicht empfohlen!)
helm upgrade nginx bitnami/nginx --set replicaCount=5 -n default
```

**Flux reconciled automatisch** und setzt `replicaCount` wieder auf den Wert in `values` (z.B. 3).

### Dependencies zwischen HelmReleases

```
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: wordpress
  namespace: default
spec:
  interval: 5m
  dependsOn:
    - name: mariadb
      namespace: databases
  chart:
    spec:
      chart: wordpress
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
```

**Wordpress wartet** bis `mariadb` HelmRelease erfolgreich ist.

## Troubleshooting

### HelmRelease haengt in "Not Ready"

```
kubectl describe helmrelease nginx -n default
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

## Zusammenfassung

| Aktion | Befehl |
|--------|--------|
| HelmRelease erstellen | `kubectl apply -f 01-helmrelease-nginx.yml` |
| Status pruefen | `kubectl get helmrelease -n default` |
| Details anzeigen | `kubectl describe helmrelease nginx -n default` |
| Suspend | `flux suspend helmrelease nginx -n default` |
| Resume | `flux resume helmrelease nginx -n default` |
| Helm Releases anzeigen | `helm list -n default` |

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

## Aufraeumen

```
kubectl delete -f .
```

```
kubectl delete namespace databases
```
