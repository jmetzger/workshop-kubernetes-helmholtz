# HelmRepository - Helm Chart Repositories mit Flux verwalten

## Hintergrund

`HelmRepository` ist eine Flux Custom Resource Definition (CRD), die ein Helm Chart Repository als Quelle definiert. Der **source-controller** ueberwacht diese Ressource und laedt periodisch den Repository-Index (`index.yaml`).

| Eigenschaft | Beschreibung |
|-------------|--------------|
| **API Group** | `source.toolkit.fluxcd.io/v1` |
| **Controller** | source-controller |
| **Funktion** | Helm Chart Repository Index bereitstellen |
| **Update** | Periodisch (interval) oder per Webhook |

## Voraussetzungen

- Flux installiert (siehe [02-installation-helm.md](02-installation-helm.md))
- `source-controller` laeuft

## Schritt 1: Vorbereitung

```
cd
mkdir -p manifests/flux
cd manifests/flux
```

## Schritt 2: HelmRepository fuer Traefik erstellen

Wir definieren das Traefik Helm Repository als Quelle:

```
# vi 01-helmrepo-traefik.yml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: traefik
  namespace: flux-system
spec:
  interval: 10m
  url: https://traefik.github.io/charts
```

**Erklaerung:**
| Feld | Wert | Bedeutung |
|------|------|-----------|
| `interval` | `10m` | Alle 10 Minuten Index neu laden |
| `url` | `https://...` | Helm Chart Repository URL |
| `namespace` | `flux-system` | Namespace wo Flux installiert ist |

## Schritt 3: HelmRepository anwenden

```
kubectl apply -f 01-helmrepo-traefik.yml
```

## Schritt 4: Status pruefen

```
kubectl get helmrepository -n flux-system
```

**Erwartete Ausgabe:**
```
NAME      URL                                  AGE   READY   STATUS
traefik   https://traefik.github.io/charts    30s   True    stored artifact for revision 'sha256:...'
```

**Status-Felder:**
- `READY: True` - Repository Index erfolgreich geladen
- `STATUS` - Zeigt Revision (SHA256 des Index)

## Schritt 5: Weitere Repository hinzufuegen (cloudpirates)

Cloudpirates ist eine Alternative zu Bitnami und bietet frei nutzbare Charts:

```
# vi 02-helmrepo-cloudpirates.yml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: cloudpirates
  namespace: flux-system
spec:
  interval: 30m
  url: https://cloudpirates-charts.storage.googleapis.com
```

```
kubectl apply -f 02-helmrepo-cloudpirates.yml
```

## Schritt 6: Alle HelmRepositories anzeigen

```
kubectl get helmrepository -n flux-system
```

**Erwartete Ausgabe:**
```
NAME            URL                                                  AGE    READY   STATUS
traefik         https://traefik.github.io/charts                    5m     True    stored artifact...
cloudpirates    https://cloudpirates-charts.storage.googleapis.com  1m     True    stored artifact...
```

## Schritt 7: HelmRepository mit Authentifizierung (optional)

Falls ein Repository Authentifizierung benoetigt:

```
# vi 03-helmrepo-private.yml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: my-private-repo
  namespace: flux-system
spec:
  interval: 10m
  url: https://charts.example.com
  secretRef:
    name: helm-repo-credentials
```

**Secret erstellen:**
```
kubectl create secret generic helm-repo-credentials \
  --from-literal=username=myuser \
  --from-literal=password=mypassword \
  -n flux-system
```

## Schritt 8: Manuelles Update triggern (optional)

Flux reconciled automatisch basierend auf `interval`. Manuelles Update:

```
flux reconcile source helm traefik
```

**Oder mit kubectl:**
```
kubectl annotate helmrepository traefik -n flux-system \
  reconcile.fluxcd.io/requestedAt="$(date +%s)"
```

## Was passiert im Hintergrund?

1. **source-controller** laedt `index.yaml` vom Repository
2. Speichert Artifact intern (mit SHA256 Digest)
3. Macht Artifact verfuegbar unter `http://source-controller.flux-system.svc.cluster.local./...`
4. Andere Controller (z.B. helm-controller) koennen darauf zugreifen

## Haeufige Szenarien

| Use Case | interval | Grund |
|----------|----------|-------|
| **Public Stable** | `30m` - `1h` | Charts aendern sich selten |
| **Public Frequent** | `5m` - `10m` | Haeufige Updates (z.B. CI/CD) |
| **Private/Intern** | `10m` | Balance zwischen Last und Aktualitaet |
| **Webhook-basiert** | `1h` + Webhook | Nur bei Push aktualisieren |

## Zusammenfassung

| Aktion | Befehl |
|--------|--------|
| HelmRepository erstellen | `kubectl apply -f 01-helmrepo-traefik.yml` |
| Status pruefen | `kubectl get helmrepository -n flux-system` |
| Manuell aktualisieren | `flux reconcile source helm traefik` |

## Naechster Schritt

Im naechsten Schritt ([04-helmrelease.md](04-helmrelease.md)) nutzen wir diese HelmRepositories, um tatsaechlich Helm Charts mit `HelmRelease` auszurollen.

## Aufraeumen

```
kubectl delete -f .
```

**Hinweis:** Loescht alle HelmRepository Objekte im aktuellen Verzeichnis.
