# Hytale Server Helm Chart

A Helm chart for deploying Hytale dedicated game servers on Kubernetes.

> 🎮 **Image Docker:** `everhytale/hytale-server`
> 📡 **Protocol:** QUIC/UDP sur port 5520

## Prérequis

- Kubernetes 1.33+
- Helm 3.x
- PersistentVolume provisioner (pour la persistence)
- L'image Docker `everhytale/hytale-server` disponible

## Installation

### Installation simple

```bash
helm install my-hytale ./charts/hytale
```

### Installation avec valeurs personnalisées

```bash
helm install my-hytale ./charts/hytale \
  --set jvm.maxMemory=16G \
  --set service.type=LoadBalancer
```

### Installation avec fichier de valeurs

```bash
helm install my-hytale ./charts/hytale -f my-values.yaml
```

## Configuration

### Paramètres de l'image

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `image.repository` | Repository de l'image | `everhytale/hytale-server` |
| `image.tag` | Tag de l'image | `""` (utilise Chart.AppVersion) |
| `image.pullPolicy` | Pull policy | `IfNotPresent` |
| `imagePullSecrets` | Secrets pour registries privés | `[]` |

### Configuration du serveur

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `server.port` | Port du serveur (QUIC/UDP) | `5520` |
| `server.bind` | Adresse de bind | `0.0.0.0` |
| `server.authMode` | Mode d'authentification | `authenticated` |
| `server.disableSentry` | Désactiver Sentry | `false` |
| `server.useAotCache` | Utiliser le cache AOT | `true` |
| `server.extraArgs` | Arguments supplémentaires | `""` |

### Configuration JVM

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `jvm.minMemory` | Mémoire minimum | `4G` |
| `jvm.maxMemory` | Mémoire maximum | `8G` |
| `jvm.javaOpts` | Options JVM personnalisées | `""` |

### Configuration des sauvegardes

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `backup.enabled` | Activer les sauvegardes auto | `false` |
| `backup.frequency` | Fréquence en minutes | `30` |

### Authentification

L'authentification peut se faire de deux manières :

#### Option 1 : Tokens via values

```yaml
auth:
  ownerName: "MonPseudo"
  ownerUuid: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  sessionToken: "token-de-session"
  identityToken: "token-identite"
```

#### Option 2 : Secret existant

```yaml
auth:
  existingSecret: "my-hytale-auth-secret"
```

Le secret doit contenir les clés : `SESSION_TOKEN`, `IDENTITY_TOKEN`, `OWNER_NAME`, `OWNER_UUID`

#### Option 3 : Authentification interactive

Sans tokens configurés, authentifiez-vous après le démarrage :

```bash
kubectl exec -it deployment/my-hytale -- /bin/bash
# Dans la console du serveur :
/auth login device
/auth persistence Encrypted
```

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `auth.ownerName` | Nom du propriétaire | `""` |
| `auth.ownerUuid` | UUID du propriétaire | `""` |
| `auth.sessionToken` | Token de session | `""` |
| `auth.identityToken` | Token d'identité | `""` |
| `auth.existingSecret` | Secret existant | `""` |

### Machine-ID (pour persistence chiffrée)

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `machineId.mountFromHost` | Monter /etc/machine-id | `false` |
| `machineId.hostPath` | Chemin sur l'hôte | `/etc/machine-id` |

### Service

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `service.type` | Type de service | `ClusterIP` |
| `service.port` | Port du service | `5520` |
| `service.nodePort` | NodePort (si applicable) | `null` |
| `service.externalTrafficPolicy` | Traffic policy | `""` |
| `service.annotations` | Annotations | `{}` |

### Persistence

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `persistence.enabled` | Activer la persistence | `true` |
| `persistence.existingClaim` | PVC existant | `""` |
| `persistence.storageClass` | Storage class | `""` |
| `persistence.accessMode` | Mode d'accès | `ReadWriteOnce` |
| `persistence.size` | Taille du volume | `10Gi` |

### Gateway API (UDPRoute)

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `udpRoute.enabled` | Activer UDPRoute | `false` |
| `udpRoute.annotations` | Annotations | `{}` |
| `udpRoute.parentRefs` | Références Gateway | voir values.yaml |

### Ressources

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `resources.requests.cpu` | CPU demandé | `1000m` |
| `resources.requests.memory` | Mémoire demandée | `4Gi` |
| `resources.limits.cpu` | CPU limite | `4000m` |
| `resources.limits.memory` | Mémoire limite | `8Gi` |

### Probes

Les probes utilisent `pgrep -f HytaleServer.jar` pour vérifier que le processus est en cours d'exécution.

| Paramètre | Description | Défaut |
|-----------|-------------|--------|
| `livenessProbe.initialDelaySeconds` | Délai initial | `120` |
| `readinessProbe.initialDelaySeconds` | Délai initial | `60` |
| `startupProbe.failureThreshold` | Seuil d'échec | `30` |

## Exemples de déploiement

### Serveur de développement

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

### Serveur de production

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

### Avec Gateway API

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

## Structure des données

Le volume `/server` contient :

```
/server/
├── universe/          # Données du monde
├── logs/              # Logs du serveur
├── config/            # Configuration
├── auth.enc           # Authentification chiffrée
└── backups/           # Sauvegardes (si activées)
```

## Dépannage

### Le pod ne démarre pas

```bash
# Vérifier les événements
kubectl describe pod -l app.kubernetes.io/name=hytale

# Vérifier les logs
kubectl logs -l app.kubernetes.io/name=hytale --previous
```

### Problèmes de mémoire

Assurez-vous que `jvm.maxMemory` correspond aux `resources.limits.memory` :

```yaml
jvm:
  maxMemory: "8G"
resources:
  limits:
    memory: 10Gi  # Légèrement plus que maxMemory
```

### Authentification échoue

Si l'authentification chiffrée ne fonctionne pas :

1. Montez le machine-id : `machineId.mountFromHost: true`
2. Ou utilisez la persistence en mémoire :
   ```
   /auth persistence Memory
   ```

## Désinstallation

```bash
helm uninstall my-hytale
```

⚠️ Le PVC n'est pas supprimé automatiquement. Pour supprimer les données :

```bash
kubectl delete pvc my-hytale
```

## Licence

MIT License - voir [LICENSE](../../LICENSE)
