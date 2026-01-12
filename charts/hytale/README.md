# Hytale Server Helm Chart

A Helm chart for deploying Hytale game servers on Kubernetes.

## Prerequisites

- Kubernetes 1.33+
- Helm 3.14+
- PV provisioner support in the underlying infrastructure (for persistence)
- Gateway API CRDs installed (if using TCPRoute)

## Installation

### From OCI Registry (Recommended)

```bash
helm install my-hytale oci://ghcr.io/everhytale/hytale
```

### From Helm Repository

```bash
# Add the repository
helm repo add everhytale https://everhytale.github.io/charts

# Update your repositories
helm repo update

# Install the chart
helm install my-hytale everhytale/hytale
```

### With Custom Values

```bash
helm install my-hytale everhytale/hytale -f values.yaml
```

## Uninstallation

```bash
helm uninstall my-hytale
```

> **Note:** The PersistentVolumeClaim is not deleted by default to prevent data loss. Delete it manually if needed.

## Configuration

### Server Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.name` | Server name displayed in server list | `"Hytale Server"` |
| `server.motd` | Message of the day | `"Welcome to Hytale!"` |
| `server.maxPlayers` | Maximum number of players | `20` |
| `server.port` | Game server port | `25565` |
| `server.gameMode` | Game mode (survival, creative, adventure) | `"survival"` |
| `server.difficulty` | Difficulty (peaceful, easy, normal, hard) | `"normal"` |
| `server.pvp` | Enable PvP | `true` |
| `server.viewDistance` | View distance in chunks | `10` |
| `server.seed` | World seed (empty for random) | `""` |
| `server.whitelist` | Enable whitelist | `false` |

### RCON Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.rcon.enabled` | Enable RCON | `false` |
| `server.rcon.port` | RCON port | `25575` |
| `server.rcon.password` | RCON password | `"changeme"` |
| `server.rcon.existingSecret` | Use existing secret for password | `""` |
| `server.rcon.existingSecretKey` | Key in existing secret | `"password"` |

### Image Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Image repository | `everhytale/hytale-server` |
| `image.tag` | Image tag | `""` (uses appVersion) |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |

### Persistence

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistence | `true` |
| `persistence.existingClaim` | Use existing PVC | `""` |
| `persistence.storageClass` | Storage class | `""` |
| `persistence.accessMode` | Access mode | `ReadWriteOnce` |
| `persistence.size` | Storage size | `10Gi` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Game port | `25565` |
| `service.rconPort` | RCON port | `25575` |
| `service.nodePort` | NodePort for game (if type=NodePort) | `null` |
| `service.externalTrafficPolicy` | External traffic policy | `""` |

### Gateway API (TCPRoute)

| Parameter | Description | Default |
|-----------|-------------|---------|
| `tcpRoute.enabled` | Enable Gateway API TCPRoute | `false` |
| `tcpRoute.annotations` | TCPRoute annotations | `{}` |
| `tcpRoute.parentRefs` | Gateway references | See values.yaml |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.requests.cpu` | CPU request | not set |
| `resources.requests.memory` | Memory request | not set |
| `resources.limits.cpu` | CPU limit | not set |
| `resources.limits.memory` | Memory limit | not set |

## Examples

### Basic Installation

```yaml
# values.yaml
server:
  name: "My Hytale Server"
  maxPlayers: 50
  motd: "Join the adventure!"

persistence:
  size: 20Gi
```

### With LoadBalancer

```yaml
# values.yaml
service:
  type: LoadBalancer
  externalTrafficPolicy: Local

server:
  name: "Public Hytale Server"
```

### With Gateway API

```yaml
# values.yaml
tcpRoute:
  enabled: true
  parentRefs:
    - name: my-gateway
      sectionName: hytale
      namespace: gateway-system
```

### With RCON Enabled

```yaml
# values.yaml
server:
  rcon:
    enabled: true
    password: "my-secure-password"

# Or use an existing secret
server:
  rcon:
    enabled: true
    existingSecret: "my-rcon-secret"
    existingSecretKey: "password"
```

### Production Setup

```yaml
# values.yaml
server:
  name: "Production Server"
  maxPlayers: 100
  viewDistance: 12

persistence:
  enabled: true
  storageClass: "fast-ssd"
  size: 50Gi

resources:
  requests:
    cpu: "1000m"
    memory: "2Gi"
  limits:
    cpu: "4000m"
    memory: "8Gi"

affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-type
              operator: In
              values:
                - gaming
```

## Architecture

The chart deploys the following resources:

- **Deployment**: Single replica with Recreate strategy (to avoid concurrent volume access)
- **Service**: TCP/UDP service for game traffic + optional RCON
- **PersistentVolumeClaim**: For world data persistence
- **ConfigMap**: Server configuration (server.properties)
- **Secret**: RCON password (if enabled)
- **TCPRoute** (optional): Gateway API for TCP traffic

## Upgrading

### From 0.x to 1.x

No breaking changes expected.

## Troubleshooting

### Server not starting

1. Check pod logs: `kubectl logs -l app.kubernetes.io/name=hytale`
2. Verify PVC is bound: `kubectl get pvc`
3. Check events: `kubectl describe pod -l app.kubernetes.io/name=hytale`

### Can't connect to server

1. Verify service is running: `kubectl get svc`
2. Check if port is accessible: `kubectl port-forward svc/my-hytale 25565:25565`
3. Verify firewall rules if using NodePort/LoadBalancer

## Contributing

Please submit issues and pull requests to the [GitHub repository](https://github.com/EverHytale/charts).

## License

MIT License - see [LICENSE](../../LICENSE) for details.
