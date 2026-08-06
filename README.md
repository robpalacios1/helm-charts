# Helm Charts

Learning-focused Helm chart that deploys an Nginx server to Kubernetes, with optional Ingress support and a Redis dependency (subchart). Built step by step as a hands-on project to learn Helm 3.

## Requirements

- [Helm 3](https://helm.sh/docs/intro/install/)
- A Kubernetes cluster reachable via `kubectl` (Docker Desktop with Kubernetes enabled, minikube, kind, etc.)
- [ingress-nginx](https://kubernetes.github.io/ingress-nginx/deploy/) installed on the cluster if you plan to use `ingress.enabled=true`

## Chart structure

```
.
├── Chart.yaml              # Chart metadata and dependencies (Redis)
├── values.yaml              # Default configuration
├── values-dev.yaml          # Overrides for the dev environment
├── values-prod.yaml         # Overrides for the prod environment
├── .helmignore
└── templates/
    ├── _helpers.tpl          # Reusable naming and label helper functions
    ├── deployment.yaml        # Nginx Deployment
    ├── service.yaml             # Service exposing the Deployment
    ├── configmap.yaml            # ConfigMap mounted as a file inside the container
    ├── ingress.yaml               # Conditional Ingress (only if ingress.enabled=true)
    ├── NOTES.txt                   # Message shown after install/upgrade
    └── tests/
        └── test-connection.yaml    # helm test hook that checks the Service responds
```

## Installation

```bash
git clone <this-repo-url>
cd helm-charts

# If you're using the Redis dependency, fetch it first
helm repo add bitnami https://charts.bitnami.com/bitnami
helm dependency update .

# Validate before installing
helm lint .
helm template demo .

# Install
helm install demo .
```

Access the app locally with:

```bash
kubectl port-forward svc/demo-mychart 8081:80
```

Then open `http://localhost:8081/message.txt`.

## Configuration

| Parameter | Description | Default |
|---|---|---|
| `replicaCount` | Number of Deployment replicas | `1` |
| `image.repository` | Container image | `nginx` |
| `image.tag` | Image tag | `1.25-alpine` |
| `image.pullPolicy` | Pull policy | `IfNotPresent` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Port exposed by the Service | `80` |
| `resources.requests` / `resources.limits` | Reserved and max CPU/memory | `100m` / `128Mi` |
| `config.message` | Message injected via ConfigMap, served at `/message.txt` | see `values.yaml` |
| `ingress.enabled` | Enables the Ingress resource | `false` |
| `ingress.className` | Ingress class to use | `nginx` |
| `ingress.host` | Host for the Ingress rule | `mychart.local` |
| `ingress.path` | Path for the Ingress rule | `/` |
| `redis.enabled` | Installs Redis as a subchart | `false` |

## Environments (dev / prod)

This chart uses a base `values.yaml` plus per-environment override files:

```bash
# Preview the differences without applying anything
helm template demo . -f values-dev.yaml
helm template demo . -f values-prod.yaml

# Apply an environment
helm upgrade demo . -f values-prod.yaml
```

## Ingress

1. Install an ingress controller (e.g. ingress-nginx) on your cluster.
2. Point the host to your machine in your local hosts file:
   ```
   127.0.0.1 mychart.local
   ```
3. Enable the Ingress:
   ```bash
   helm upgrade demo . --set ingress.enabled=true
   curl http://mychart.local
   ```

## Redis dependency (subchart)

Redis (Bitnami) is declared as a conditional dependency in `Chart.yaml`. To install it alongside this chart:

```bash
helm dependency update .
helm upgrade demo . --set redis.enabled=true
```

It gets installed as part of the same release; `helm uninstall demo` removes everything together.

## Testing

The chart includes a `helm test` hook that verifies the Service responds:

```bash
helm test demo --logs
```

## Release lifecycle

```bash
helm upgrade demo . --set replicaCount=3   # update
helm history demo                            # view revisions
helm rollback demo <revision>                  # revert
helm uninstall demo                             # uninstall
```

## Packaging

```bash
helm package .
```

## Author
**Roberto Palacios** 
- [LinkedIn](https://www.linkedin.com/in/robpalacios1)
- [Portfolio](https://www.robpalacios1.com/)
