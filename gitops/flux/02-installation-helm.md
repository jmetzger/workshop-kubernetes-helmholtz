# Flux Installation mit Helm

## Hintergrund

Flux ist ein GitOps-Tool, das im Cluster als mehrere Controller laeuft. Die Installation erfolgt am einfachsten mit dem offiziellen Helm Chart der Flux Community.

| Komponente | Version |
|------------|---------|
| **Helm Chart** | 2.17.2 (neueste Version, Stand Januar 2026) |
| **Flux App** | 2.7.5 |
| **Repository** | fluxcd-community |

## Voraussetzungen

- Kubernetes Cluster (1.28+) - **jeder Teilnehmer hat sein eigenes Cluster**
- kubectl konfiguriert
- helm installiert (3.x)

## Schritt 1: Namespace vorbereiten

```
kubectl create namespace flux-system
```

```
kubectl get namespace flux-system
```

## Schritt 2: Helm Repository hinzufuegen

```
helm repo add fluxcd-community https://fluxcd-community.github.io/helm-charts
```

```
helm repo update
```

## Schritt 3: Verfuegbare Chart-Version pruefen

```
helm search repo fluxcd-community/flux2 --versions | head -5
```

**Erwartete Ausgabe:**
```
NAME                       CHART VERSION  APP VERSION  DESCRIPTION
fluxcd-community/flux2     2.17.2         2.7.5        A Helm chart for flux2
fluxcd-community/flux2     2.17.1         2.7.4        A Helm chart for flux2
...
```

## Schritt 4: Flux installieren

```
helm install flux fluxcd-community/flux2 \
  --namespace flux-system \
  --version 2.17.2 \
  --create-namespace
```

**Hinweis:** Die Installation dauert ca. 30-60 Sekunden, da mehrere Controller-Pods gestartet werden.

## Schritt 5: Installation verifizieren

```
kubectl get pods -n flux-system
```

**Erwartete Ausgabe:**
```
NAME                                       READY   STATUS    RESTARTS   AGE
helm-controller-...                        1/1     Running   0          45s
kustomize-controller-...                   1/1     Running   0          45s
notification-controller-...                1/1     Running   0          45s
source-controller-...                      1/1     Running   0          45s
```

## Schritt 6: Flux Controller ueberpruefen

```
kubectl get deployments -n flux-system
```

**Erwartete Controller:**
| Controller | Funktion |
|------------|----------|
| `source-controller` | Holt Git/Helm/OCI Sources |
| `kustomize-controller` | Applied Kustomize/YAML |
| `helm-controller` | Verwaltet Helm Releases |
| `notification-controller` | Alerts und Webhooks |

## Schritt 7: Custom Resource Definitions (CRDs) pruefen

Flux installiert mehrere CRDs:

```
kubectl get crds | grep toolkit.fluxcd.io
```

**Erwartete CRDs:**
```
alerts.notification.toolkit.fluxcd.io
buckets.source.toolkit.fluxcd.io
gitrepositories.source.toolkit.fluxcd.io
helmcharts.source.toolkit.fluxcd.io
helmreleases.helm.toolkit.fluxcd.io
helmrepositories.source.toolkit.fluxcd.io
kustomizations.kustomize.toolkit.fluxcd.io
...
```

## Schritt 8: Flux Version anzeigen

```
kubectl get deployment source-controller -n flux-system -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Das zeigt das verwendete Image und die Version (z.B. `ghcr.io/fluxcd/source-controller:v1.x.x`).

## Optionale Schritte

### Flux CLI installieren (lokal auf eurem Rechner)

Flux bietet auch eine CLI fuer einfachere Verwaltung:

```
curl -s https://fluxcd.io/install.sh | sudo bash
```

Oder mit brew (macOS):
```
brew install fluxcd/tap/flux
```

**Flux Version pruefen:**
```
flux version --client
```

### Flux Status im Cluster pruefen (mit CLI)

```
flux check
```

**Erwartete Ausgabe:**
```
✔ Kubernetes 1.28.x >=1.28.0
✔ source-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ helm-controller: deployment ready
✔ notification-controller: deployment ready
✔ all checks passed
```

## Was wurde installiert?

Nach der Installation habt ihr:

1. **4 Controller-Pods** im Namespace `flux-system`
2. **Mehrere CRDs** fuer GitOps-Workflows
3. **Keine Helm Releases/Git Repos** - Flux ist bereit, aber noch nicht konfiguriert

Im naechsten Schritt werdet ihr Flux nutzen, um Helm Charts mit `HelmRepository` und `HelmRelease` auszurollen.

## Zusammenfassung

| Aktion | Befehl |
|--------|--------|
| Repo hinzufuegen | `helm repo add fluxcd-community https://...` |
| Installieren | `helm install flux fluxcd-community/flux2 --version 2.17.2` |
| Pods pruefen | `kubectl get pods -n flux-system` |
| CRDs pruefen | `kubectl get crds \| grep toolkit.fluxcd.io` |
| CLI installieren | `curl -s https://fluxcd.io/install.sh \| sudo bash` |

## Aufraeumen

```
helm uninstall flux -n flux-system
```

```
kubectl delete namespace flux-system
```

**Hinweis:** CRDs muessen manuell geloescht werden (falls gewuenscht):
```
kubectl delete crds -l app.kubernetes.io/part-of=flux
```
