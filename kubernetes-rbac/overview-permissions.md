# Kubernetes RBAC - Permissions 

## Kubernetes RBAC – Verbs (Berechtigungen)

**Quelle:** [kubernetes.io – RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

### Standard-Verbs (HTTP-äquivalent)

| Verb | Beschreibung |
|------|-------------|
| `get` | Einzelne Ressource lesen |
| `list` | Alle Ressourcen eines Typs auflisten |
| `watch` | Ressourcen auf Änderungen beobachten |
| `create` | Neue Ressource erstellen |
| `update` | Vorhandene Ressource vollständig ersetzen |
| `patch` | Ressource partiell ändern |
| `delete` | Einzelne Ressource löschen |
| `deletecollection` | Mehrere Ressourcen auf einmal löschen |

### Spezielle Verbs

| Verb | Beschreibung |
|------|-------------|
| `impersonate` | Als anderer User/Gruppe/ServiceAccount agieren |
| `bind` | RoleBindings/ClusterRoleBindings erstellen |
| `escalate` | Roles mit höheren Rechten bearbeiten als man selbst hat |
| `use` | PodSecurityPolicies verwenden (deprecated ab k8s 1.25) |
| `approve` | CertificateSigningRequests genehmigen |
| `sign` | Zertifikate signieren |

### Wildcard

```yaml
verbs: ["*"]  # alle Verbs erlaubt
```

### Alle verfügbaren Verbs per Resource anzeigen:

```bash
kubectl api-resources --sort-by name -o wide
```


## nonResourceURLs in Kubernetes RBAC

**Quelle:** [kubernetes.io – RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

Das sind API-Endpunkte, die **keine Kubernetes-Ressourcen** darstellen – also kein `get pods` etc., sondern direkte HTTP-Pfade auf dem API-Server.

### Häufig verwendete nonResourceURLs

| URL | Beschreibung |
|-----|-------------|
| `/healthz` | Health-Check des API-Servers |
| `/livez` | Liveness-Probe |
| `/readyz` | Readiness-Probe |
| `/metrics` | Prometheus-Metriken (z.B. für Prometheus selbst) |
| `/api` | API-Discovery |
| `/api/*` | Alle Core-API-Pfade |
| `/apis` | API-Groups Discovery |
| `/apis/*` | Alle erweiterten API-Pfade |
| `/version` | Kubernetes-Version abfragen |
| `/openapi/v2` | OpenAPI-Schema |
| `/logs` | Logs des API-Servers |
| `/swagger-ui/*` | Swagger UI |

### Erlaubte Verbs für nonResourceURLs

Nur **`get`** – mehr ist nicht möglich.

### Beispiel

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: metrics-reader
rules:
- nonResourceURLs: ["/metrics", "/healthz"]
  verbs: ["get"]
```

> **Wichtig:** `nonResourceURLs` sind immer **cluster-weit** → nur in `ClusterRole` sinnvoll, nicht in `Role`.
