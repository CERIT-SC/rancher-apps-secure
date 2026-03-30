# SeqUIaSCOPE Helm Chart

A Helm chart for deploying [SeqUIaSCOPE](https://github.com/katjur01/seqUIaSCOPE/tree/dev)- a Shiny application for genomic data visualization with IGV (Integrative Genomics Viewer) integration.

## Description

This chart deploys Sequiascope as a Kubernetes Deployment with two containers:

- **sequiascope**: The main Shiny application (port 8080)
- **sequiascope-igv**: IGV static file server (port 8081)

The application uses two separate persistent volumes:
- **Input PVC**: Read-only data volume for input files (can use existing PVC or be created)
- **Output PVC**: Writable volume for output files (created if enabled, otherwise uses emptyDir)

## Prerequisites

- Kubernetes cluster
- Helm 3.0+

## Installation

### Install from the chart directory

```bash
helm install sequiascope .
```

### Install with custom values

```bash
helm install sequiascope . -f custom-values.yaml
```

## Configuration

The chart is highly configurable. The most commonly used parameters are listed below. For a complete list of all available parameters, see [`values.yaml`](values.yaml).

### Key Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `sequiascope.image.tag` | Sequiascope container image tag | `latest` |
| `igv.image.tag` | IGV container image tag | `latest` |
| `sequiascope.resources` | CPU/memory resources for sequiascope container | See values.yaml |
| `igv.resources` | CPU/memory resources for IGV container | See values.yaml |
| `inputPersistence.enabled` | Enable input PVC | `true` |
| `inputPersistence.existingClaim` | Use existing input PVC (if set, the chart won't create a PVC) | `""` |
| `inputPersistence.size` | Input PVC size | `100Gi` |
| `outputPersistence.enabled` | Enable output PVC | `true` |
| `outputPersistence.size` | Output PVC size | `100Gi` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.hosts[0].host` | Hostname for ingress | `sequiascope.example.com` |
| `ingress.tls` | TLS configuration | `[]` |
| `ingress.auth.enabled` | Enable basic auth for ingress | `true` |
| `ingress.auth.username` | Username for basic auth | `"admin"` |
| `ingress.auth.password` | Password for basic auth | `"!cHaNgEmEpLeAsE!"` |
| `ingress.auth.existingSecret` | Use [existing secret](https://kubernetes.github.io/ingress-nginx/examples/auth/basic/) instead of creating one | `""` |

### Additional Configuration Options

- **Environment variables**: Add extra environment variables via `sequiascope.extraEnv` and `igv.extraEnv`
- **Security context**: Non-root execution with dropped capabilities (configured in `podSecurityContext` and container `securityContext`)
- **Node selection/affinity**: Use `nodeSelector`, `tolerations`, and `affinity` for scheduling
- **Service type**: Default is `ClusterIP`; change via `service.type`
- **Storage class**: Set `inputPersistence.storageClassName` and `outputPersistence.storageClassName` as needed

**Note**: If persistence is disabled (`enabled: false`), an `emptyDir` volume will be used instead. The input PVC is mounted read-only to both containers.

### Basic Authentication

The ingress supports basic authentication via the `ingress.auth` configuration:

- **Enable basic auth**: Set `ingress.auth.enabled: true` and provide `username`/`password` inline.
- **Use existing secret**: If you already have a Kubernetes secret containing `.htpasswd` data, set `ingress.auth.existingSecret` to its name (the chart will not create a new secret). When this is set, the `username` and `password` values are ignored.
- **Generated secret**: When using inline credentials, the chart creates a secret named `<releasename>-basic-auth` containing the `.htpasswd` hash.

## Volume Mounts

- **sequiascope container**:
  - `/input_files` → input-data volume (read-only)
  - `/output_files` → output-data volume
- **sequiascope-igv container**:
  - `/srv/igv-static` → input-data volume (read-only)

## Example: Custom Values

Create a `custom-values.yaml` file:

```yaml
# Override image versions and resource limits
sequiascope:
  image:
    tag: "v1.2.3"
  resources:
    requests:
      memory: 2Gi
      cpu: 500m
    limits:
      memory: 4Gi
      cpu: 1

igv:
  image:
    tag: "v1.2.3"
  resources:
    requests:
      memory: 1Gi
      cpu: 500m
    limits:
      memory: 2Gi
      cpu: 1

# Use existing input PVC
inputPersistence:
  existingClaim: my-existing-input-pvc

# Adjust output PVC size
outputPersistence:
  size: 50Gi

# Enable ingress with TLS and basic authentication
ingress:
  enabled: true
  className: nginx
  annotations:
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: sequiascope.example.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: sequiascope-tls
      hosts:
        - sequiascope.example.com
  auth:
    enabled: true
    username: "admin"
    password: "securepassword"
    # existingSecret: ""  # Uncomment to use an existing secret
```

Install with custom values:

```bash
helm install sequiascope . -f custom-values.yaml
```

## Upgrading

```bash
helm upgrade sequiascope .
```

## Uninstalling

```bash
helm uninstall sequiascope
```

## Troubleshooting

### View pod logs

```bash
kubectl logs -l app.kubernetes.io/name=sequiascope -c sequiascope
kubectl logs -l app.kubernetes.io/name=sequiascope -c sequiascope-igv
```

### Check pod status

```bash
kubectl get pods -l app.kubernetes.io/name=sequiascope
kubectl describe pod <pod-name>
```

### Verify PVCs are bound

```bash
kubectl get pvc -l app.kubernetes.io/name=sequiascope
kubectl describe pvc sequiascope-input-pvc
kubectl describe pvc sequiascope-output-pvc
```

### Test the application

```bash
# Port-forward to access the application locally
kubectl port-forward svc/sequiascope 8080:8080
```

Then open http://localhost:8080 in your browser.

## License

This chart is provided as-is for deploying SeqUIaSCOPE.
