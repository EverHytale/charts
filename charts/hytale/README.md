# Hytale Server Helm Chart

A Helm chart for deploying Hytale dedicated game servers on Kubernetes.

> 🎮 **Docker Image:** `everhytale/hytale-server`
> 📡 **Protocol:** QUIC/UDP on port 5520

## ⚠️ Disclaimer

This is an unofficial community project. Hytale is a trademark of Hypixel Studios.

## Prerequisites

- Kubernetes 1.33+
- Helm 3.x
- PersistentVolume provisioner (for persistence)
- Docker image `everhytale/hytale-server` available

## Installation

### Simple Installation

```bash
helm install my-hytale ./charts/hytale
```

### Installation with Custom Values

```bash
helm install my-hytale ./charts/hytale \
  --set jvm.maxMemory=16G \
  --set service.type=LoadBalancer
```

### Installation with Values File

```bash
helm install my-hytale ./charts/hytale -f my-values.yaml
```

## Configuration

### Image Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Image repository | `everhytale/hytale-server` |
| `image.tag` | Image tag | `""` (uses Chart.AppVersion) |
| `image.pullPolicy` | Pull policy | `IfNotPresent` |
| `imagePullSecrets` | Secrets for private registries | `[]` |

### Server Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.port` | Server port (QUIC/UDP) | `5520` |
| `server.bind` | Bind address | `0.0.0.0` |
| `server.authMode` | Authentication mode | `authenticated` |
| `server.disableSentry` | Disable Sentry | `false` |
| `server.useAotCache` | Use AOT cache | `true` |
| `server.extraArgs` | Extra arguments | `""` |

### JVM Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `jvm.minMemory` | Minimum memory | `4G` |
| `jvm.maxMemory` | Maximum memory | `8G` |
| `jvm.javaOpts` | Custom JVM options | `""` |

### Backup Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `backup.enabled` | Enable automatic backups (`--backup`) | `false` |
| `backup.dir` | Backup directory path (`--backup-dir`) | `/server/backups` |
| `backup.frequency` | Frequency in minutes (`--backup-frequency`) | `30` |
| `backup.maxCount` | Maximum backups to keep (`--backup-max-count`) | `5` |

### Console Access

| Parameter | Description | Default |
|-----------|-------------|---------|
| `console.tty` | Enable TTY for container (required for `kubectl attach -it`) | `true` |
| `console.stdin` | Enable stdin for container (required for `kubectl attach -it`) | `true` |

### Authentication

Authentication can be done in two ways:

#### Option 1: Tokens via Values

```yaml
auth:
  ownerName: "MyUsername"
  ownerUuid: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  sessionToken: "session-token"
  identityToken: "identity-token"
```

#### Option 2: Existing Secret

```yaml
auth:
  existingSecret: "my-hytale-auth-secret"
```

The secret must contain keys: `SESSION_TOKEN`, `IDENTITY_TOKEN`, `OWNER_NAME`, `OWNER_UUID`

#### Option 3: Interactive Authentication

Without configured tokens, authenticate after startup:

```bash
kubectl attach -it deployment/my-hytale
# In the server console:
/auth login device
/auth persistence Encrypted
```

| Parameter | Description | Default |
|-----------|-------------|---------|
| `auth.ownerName` | Owner name | `""` |
| `auth.ownerUuid` | Owner UUID | `""` |
| `auth.sessionToken` | Session token | `""` |
| `auth.identityToken` | Identity token | `""` |
| `auth.existingSecret` | Existing secret | `""` |

### Machine-ID (for Encrypted Persistence)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `machineId.mountFromHost` | Mount /etc/machine-id | `false` |
| `machineId.hostPath` | Host path | `/etc/machine-id` |

### Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `5520` |
| `service.nodePort` | NodePort (if applicable) | `null` |
| `service.externalTrafficPolicy` | Traffic policy | `""` |
| `service.annotations` | Annotations | `{}` |

### Persistence

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistence | `true` |
| `persistence.existingClaim` | Existing PVC | `""` |
| `persistence.storageClass` | Storage class | `""` |
| `persistence.accessMode` | Access mode | `ReadWriteOnce` |
| `persistence.size` | Volume size | `10Gi` |

### Gateway API (UDPRoute)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `udpRoute.enabled` | Enable UDPRoute | `false` |
| `udpRoute.annotations` | Annotations | `{}` |
| `udpRoute.parentRefs` | Gateway references | see values.yaml |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.requests.cpu` | Requested CPU | `1000m` |
| `resources.requests.memory` | Requested memory | `4Gi` |
| `resources.limits.cpu` | CPU limit | `4000m` |
| `resources.limits.memory` | Memory limit | `8Gi` |

### Probes

Probes use `pgrep -f HytaleServer.jar` to verify the process is running.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `livenessProbe.initialDelaySeconds` | Initial delay | `120` |
| `readinessProbe.initialDelaySeconds` | Initial delay | `60` |
| `startupProbe.failureThreshold` | Failure threshold | `30` |

## Deployment Examples

### Development Server

```yaml
# dev-values.yaml
jvm:
  minMemory: "2G"
  maxMemory: "4G"

server:
  disableSentry: true
  authMode: "unauthenticated"

persistence:
  enabled: false

resources:
  requests:
    cpu: 500m
    memory: 2Gi
  limits:
    cpu: 2000m
    memory: 4Gi
```

### Production Server

```yaml
# prod-values.yaml
jvm:
  minMemory: "8G"
  maxMemory: "16G"

backup:
  enabled: true
  frequency: 15

auth:
  existingSecret: "hytale-auth-prod"

machineId:
  mountFromHost: true

service:
  type: LoadBalancer
  externalTrafficPolicy: Local

persistence:
  enabled: true
  size: 50Gi
  storageClass: "fast-ssd"

resources:
  requests:
    cpu: 2000m
    memory: 8Gi
  limits:
    cpu: 8000m
    memory: 16Gi

nodeSelector:
  node-type: gaming
```

### With Gateway API

```yaml
# gateway-values.yaml
service:
  type: ClusterIP

udpRoute:
  enabled: true
  parentRefs:
    - name: main-gateway
      namespace: gateway-system
      sectionName: hytale-udp
```

## Data Structure

The `/server` volume contains:

```
/server/
├── universe/          # World data
├── logs/              # Server logs
├── config/            # Configuration
├── auth.enc           # Encrypted authentication
└── backups/           # Backups (if enabled)
```

## Troubleshooting

### Pod Fails to Start

```bash
# Check events
kubectl describe pod -l app.kubernetes.io/name=hytale

# Check logs
kubectl logs -l app.kubernetes.io/name=hytale --previous
```

### Memory Issues

Ensure `jvm.maxMemory` matches `resources.limits.memory`:

```yaml
jvm:
  maxMemory: "8G"
resources:
  limits:
    memory: 10Gi  # Slightly more than maxMemory
```

### Authentication Fails

If encrypted authentication doesn't work:

1. Mount the machine-id: `machineId.mountFromHost: true`
2. Or use in-memory persistence:
   ```
   /auth persistence Memory
   ```

### Cannot Attach to Container Console

If `kubectl attach -it` fails with "Unable to use a TTY", ensure console access is enabled:

```yaml
console:
  tty: true
  stdin: true
```

## Uninstallation

```bash
helm uninstall my-hytale
```

⚠️ The PVC is not automatically deleted. To delete data:

```bash
kubectl delete pvc my-hytale
```

## License

MIT License - see [LICENSE](../../LICENSE)
