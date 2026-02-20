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

- Flux installiert (siehe [02-installation-flux-cli.md](02-installation-flux-cli.md))
- HelmRepository erstellt (siehe [03-helmrepository.md](03-helmrepository.md))

## Schritt 1: Vorbereitung

  * **Achtung: unser Flux - Replo muss geklont worden sein**

```
cd
mkdir -p manifests/flux/clusters/production/infrastructure/releases
cd manifests/flux/clusters/production/infrastructure/releases 
```

## Schritt 2: HelmRelease fuer traefik erstellen

Wir rollen traefik aus dem traefik Repository aus:

```
nano traefik.yml
```

```
# vi traefik.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: traefik
  namespace: ingress
spec:
  interval: 5m
  chart:
    spec:
      chart: traefik
      version: 39.0.1
      sourceRef:
        kind: HelmRepository
        name: traefik
        namespace: flux-system
  install:
    createNamespace: true
  values:
    replicas: 2
```

```
git add -A
git commit -am "Added HelmRelease"
git push
```

**Erklaerung:**
| Feld | Wert | Bedeutung |
|------|------|-----------|
| `interval` | `5m` | Pruefe alle 5 Minuten auf Drift/Updates |
| `chart` | `traefik` | Chart Name aus dem Repository |
| `version` | `1.3.3` | Spezifische Chart Version |
| `sourceRef` | `cloudpirates` | Referenz auf HelmRepository |
| `values` | ... | Ueberschreibt Chart Default-Values |

## Schritt 4: Status pruefen

```
kubectl get helmrelease -A
# Fehler
```

<img width="1902" height="112" alt="image" src="https://github.com/user-attachments/assets/569129b6-03c7-474e-ada4-9ff6c91c8d69" />

```
kubectl -n ingress describe helmrelease traefik 
# Fehler
```

```
 2026-02-17T17:45:18.943742569Z: preparing upgrade for traefik
2026-02-17T17:45:19.054850747Z: resetting values to the chart's original version
  Warning  UpgradeFailed  55s  helm-controller  (combined from similar events): Helm upgrade failed for release ingress/traefik with chart traefik@39.0.1: values don't meet the specifications of the schema(s) in the following chart(s):
traefik:
- at '': additional properties 'replicas' not allowed
```


## Schritt 5: Richtige values für deployments recherchieren 

```
in artifacthub.io
```

## Schritt 6: values korrigieren 

```
nano traefik.yml
```

```
# vi infrastructure/releases/traefik.yml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: traefik
  namespace: ingress
spec:
  interval: 5m
  chart:
    spec:
      chart: traefik
      version: 39.0.1
      sourceRef:
        kind: HelmRepository
        name: traefik
        namespace: flux-system
  install:
    createNamespace: true
  values:
    deployment:
      replicas: 2
```

```
git add -A
git commit -am "adjusted replicas"
git push
```

```
flux get source git
flux get helmreleases -A
kubectl -n ingress get helmrelease traefik
helm -n ingress list 
helm -n ingress history traefik
helm -n ingress get values traefik
```

<img width="357" height="90" alt="image" src="https://github.com/user-attachments/assets/5ca35cde-0395-4e6c-b177-ba25bdc9e9ae" />



## Schritt 7: Details des HelmRelease anzeigen

```
kubectl get helmrelease traefik -n ingress -o yaml | grep -A 10 status
```

**Wichtige Informationen:**
- `lastAppliedRevision` - Chart Version die installiert wurde
- `lastAttemptedRevision` - Letzte versuchte Version
- `conditions` - Status der Reconciliation



