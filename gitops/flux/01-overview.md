# Flux v2.7 – Überblick, Controller, CRDs und Ablauf (am Beispiel Helm Charts)

> Ziel dieses Dokuments: Verständlich erklären, **wie Flux arbeitet**, **welche Controller es gibt**, **welche CRDs zu Flux gehören**, **wofür sie sinnvoll sind** und **welche Aufgaben die Controller haben**.

---

## 1) Grundprinzip: GitOps + Reconciliation

Flux arbeitet „pull-based“:

1. **Quelle beobachten** (Git/OCI/Helm-Repo/S3-Bucket)
2. **Artefakt bauen** (z. B. tar.gz mit Repo-Snapshot oder Helm-Chart-Artifact)
3. **Zielzustand ableiten** (Kustomize oder Helm)
4. **Ist-Zustand im Cluster angleichen** (apply/upgrade)
5. **Wiederholen** (in Intervallen oder bei Triggern), inkl. Status/Events

Wichtig: Flux „macht nichts einmalig“, sondern **reconciled** immer wieder, bis Ist = Soll.

---

## 2) Die Flux-Controller (Komponenten) und ihre Aufgaben

Flux besteht aus mehreren Controllern (Deployments), die jeweils **bestimmte CRDs** beobachten und „reconciled“ ausführen:

### 2.1 source-controller (Quellen + Artefakte)
**Aufgabe**
- Holt Inhalte aus **Git**, **OCI**, **HelmRepositories** oder **Buckets**
- Erzeugt **versionierte Artefakte** (z. B. tar.gz) und stellt sie für andere Controller bereit
- Verifiziert optional Signaturen/Checksums (je nach Setup)

**Typische Outputs**
- „Artifact“: Ein referenzierbares Paket (URL + Digest + Revision)

**CRDs**
- `GitRepository` – Git Repo als Source (Branch/Tag/Commit, Auth, Interval)
- `OCIRepository` – OCI-Artifact als Source (z. B. “oci://…”)
- `HelmRepository` – Helm Chart Repo Index als Source
- `Bucket` – S3/GCS/etc. Bucket als Source
- `HelmChart` – (meist automatisch) „Chart-Build“ aus HelmRepository + Chart + Version/Constraints
- `HelmPullRequest` – (optional/selten) für PR-basiertes Chart Pulling (nicht in jedem Setup genutzt)

**Wofür sinnvoll?**
- Trennung „**beschaffen**“ (Source) von „**anwenden**“ (Helm/Kustomize)
- Einheitlicher Artefakt-Mechanismus für alle nachfolgenden Controller

---

### 2.2 helm-controller (Helm Releases)
**Aufgabe**
- Installiert/Upgraded/Uninstallt Helm Releases anhand von `HelmRelease`
- Nutzt als Input Helm-Chart-Artefakte (aus source-controller: `HelmChart`/Artifacts)
- Verarbeitet Values (inline, ConfigMap/Secret refs, etc.)
- Wartet optional auf Readiness, führt Rollbacks durch (je nach Policy)

**CRDs**
- `HelmRelease` – beschreibt Release (Chart-Quelle, Version, Values, Drift/Upgrade Strategy)

**Wofür sinnvoll?**
- „Helm as reconciliation“: Release ist deklarativ, Upgrades reproduzierbar, Status nachvollziehbar

---

### 2.3 kustomize-controller (YAML / Kustomize)
**Aufgabe**
- Rendert und applied Kubernetes Ressourcen aus einem Source-Artefakt
- Unterstützt Kustomize (overlays, patches, images, etc.)
- Kann (optional) Helm-Rendering über Kustomize nutzen, ist aber typischerweise „YAML/Kustomize-first“

**CRDs**
- `Kustomization` – beschreibt Pfad im Artifact, Prune, Health Checks, DependsOn, Interval etc.

**Wofür sinnvoll?**
- Deklaratives Ausrollen von „plain YAML“ oder Kustomize-Overlays
- Gute Orchestrierung über `dependsOn`, Health Checks und `prune`

---

### 2.4 notification-controller (Events/Alerts/Inbound Webhooks)
**Aufgabe**
- Sendet Benachrichtigungen über Zustandsänderungen/Events (z. B. Slack/Webhook/Teams)
- Kann Webhooks empfangen (z. B. GitHub/GitLab/Harbor) und dadurch „reconcile now“ auslösen

**CRDs**
- `Provider` – Zielsystem (Slack/Webhook/etc.) + Credentials
- `Alert` – Regeln: Welche Events von welchen Objekten (z. B. HelmRelease, Kustomization) wohin
- `Receiver` – Inbound Webhook Endpoint (z. B. GitHub) + Token/Secret

**Wofür sinnvoll?**
- Feedback-Kanal: „Was passiert gerade im Cluster?“
- Schnellere Reconciliation durch Webhook statt Polling (optional)

---

### 2.5 image-reflector-controller (Image Tags/Registry beobachten)
**Aufgabe**
- Scannt Container Registries und speichert Metadaten (Tags, Digests)
- Liefert Grundlage für „Image Update Automation“

**CRDs**
- `ImageRepository` – Welche Registry/Repo scannen? Auth? Interval?
- `ImagePolicy` – Welche Tags/Digests sind „gewünscht“ (semver, regex, latest digest, …)
- `ImageScanResult` – (intern/implizit) Status-Daten, je nach Version/Setup

**Wofür sinnvoll?**
- Automatisches „finden“ neuer Images (z. B. neuestes semver)

---

### 2.6 image-automation-controller (Git Updates/Commits)
**Aufgabe**
- Schreibt Updates (z. B. neue Image Tags) in ein Git Repo zurück (Commit/Push)
- Dadurch triggert Flux anschließend selbst wieder „normalen“ GitOps-Flow

**CRDs**
- `ImageUpdateAutomation` – welches Repo/Branch, Commit Message, Update-Strategie (YAML Setter, kustomize images, …)

**Wofür sinnvoll?**
- Git bleibt „source of truth“, aber Image Updates passieren automatisiert

---

## 3) Welche CRDs gibt es in Flux v2.7 (kompakt)

> Hinweis: Die CRDs sind in APIs gruppiert (z. B. `source.toolkit.fluxcd.io`, `helm.toolkit.fluxcd.io`, …).
> Die wichtigsten „Alltags-CRDs“ sind:

### source.toolkit.fluxcd.io
- `GitRepository`
- `OCIRepository`
- `HelmRepository`
- `Bucket`
- `HelmChart` (Chart artifact builder)

### helm.toolkit.fluxcd.io
- `HelmRelease`

### kustomize.toolkit.fluxcd.io
- `Kustomization`

### notification.toolkit.fluxcd.io
- `Provider`
- `Alert`
- `Receiver`

### image.toolkit.fluxcd.io
- `ImageRepository`
- `ImagePolicy`
- `ImageUpdateAutomation`

---

## 4) Ablauf erklärt: Helm Charts Deployment mit Flux (Schritt-für-Schritt)

### Beispielziel
Wir wollen ein Helm Chart (z. B. `ingress-nginx`) deklarativ ausrollen.

### Die beteiligten Objekte (minimal)
1. `HelmRepository` (Quelle: Chart Repo)
2. `HelmRelease` (Release Definition)

Optional/implizit:
- `HelmChart` wird häufig **automatisch** vom source-controller erzeugt (Artifact-Build)
- Notification- und Image-Komponenten sind optional

---

### 4.1 Ablauf (in Worten)

**A) source-controller: Chart-Quelle bereitstellen**
1. `HelmRepository` wird reconciled:
   - source-controller lädt `index.yaml` des Chart Repos
   - speichert Status/Revision (z. B. index digest)
2. Für einen `HelmRelease` wird (oft) ein `HelmChart` Artifact erzeugt:
   - source-controller lädt die Chart `.tgz` Datei (oder OCI chart)
   - baut Artifact, versieht es mit Digest, stellt es bereit

**B) helm-controller: Release ausrollen**
3. helm-controller reconciled `HelmRelease`:
   - holt Chart-Artifact Referenz (vom source-controller)
   - rendert Templates mit `values`
   - führt `helm upgrade --install`-Äquivalent aus (intern, ohne dass ihr Helm CLI nutzt)
   - optional: `wait`, `timeout`, `rollback`, `drift detection` je nach Spec
4. Status wird im `HelmRelease` aktualisiert:
   - letzte erfolgreiche Revision, ObservedGeneration, Conditions (Ready/NotReady)

**C) Wiederholung & Änderungen**
5. Neue Chart-Version im Repo oder Werte-Änderungen → nächster Reconcile
6. Flux sorgt dafür, dass der Cluster-Zustand wieder dem gewünschten Zustand entspricht

---

### 4.2 Ablauf (als Mermaid Sequenzdiagramm)

```mermaid
sequenceDiagram
  autonumber
  participant U as User/GitOps Repo
  participant SC as source-controller
  participant HC as helm-controller
  participant API as Kubernetes API

  U->>API: apply HelmRepository + HelmRelease
  API-->>SC: HelmRepository observed
  SC->>SC: fetch index.yaml, compute revision
  SC-->>API: update HelmRepository status (artifact/ref)

  API-->>SC: HelmRelease references HelmRepository
  SC->>SC: build/fetch HelmChart artifact (.tgz/oci)
  SC-->>API: publish HelmChart artifact ref (digest/url)

  API-->>HC: HelmRelease observed
  HC->>SC: request chart artifact (ref)
  HC->>HC: render templates with values
  HC->>API: apply/upgrade resources (as Helm)
  HC-->>API: update HelmRelease status (Ready/NotReady)
