# EverHytale Helm Charts

[![Lint Charts](https://github.com/everhytale/charts/workflows/Lint%20Charts/badge.svg)](https://github.com/everhytale/charts/actions)
[![Release Charts](https://github.com/everhytale/charts/workflows/Release%20Charts/badge.svg)](https://github.com/everhytale/charts/actions)

Helm charts for deploying Hytale game servers on Kubernetes.

## Usage

[Helm](https://helm.sh) must be installed to use the charts.
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

### Installation from OCI Registry (Recommended)

```bash
# Install directly from OCI registry
helm install my-hytale oci://ghcr.io/everhytale/hytale
```

### Installation from Helm Repository

```bash
# Add the repository
helm repo add everhytale https://everhytale.github.io/charts

# Update your repositories
helm repo update

# Search available charts
helm search repo everhytale

# Install a chart
helm install my-hytale everhytale/hytale
```

## Available Charts

| Chart | Description | Version |
|-------|-------------|---------|
| [hytale](./charts/hytale) | Hytale game server | 0.1.0 |

## Quick Start

### Basic Installation

```bash
helm install my-server oci://ghcr.io/everhytale/hytale
```

### Custom Configuration

```bash
helm install my-server oci://ghcr.io/everhytale/hytale \
  --set server.name="My Server" \
  --set server.maxPlayers=50 \
  --set persistence.size=20Gi
```

### With Values File

```bash
helm install my-server oci://ghcr.io/everhytale/hytale -f values.yaml
```

## Documentation

- [Hytale Chart Documentation](./charts/hytale/README.md)
- [Artifact Hub](https://artifacthub.io/packages/search?repo=everhytale-charts)

## Contributing

We welcome contributions! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

### Development

```bash
# Lint charts
helm lint charts/*

# Template locally
helm template my-release charts/hytale

# Install with debug
helm install my-release charts/hytale --debug --dry-run
```

## CI/CD

This repository uses GitHub Actions for:

- **Linting**: Runs on every pull request
- **Dev builds**: On push to main branch (only changed charts)
- **Release**: Triggered by tags per chart

### Tag Format

Tags follow the format `<chart-name>-v<version>`:

| Tag | Type | Example |
|-----|------|---------|
| `<chart>-v<semver>` | Release | `hytale-v0.1.0` |
| `<chart>-v<semver>-rc.<n>` | Release Candidate | `hytale-v0.1.0-rc.1` |
| `<chart>-v<semver>-alpha.<n>` | Alpha | `hytale-v0.2.0-alpha.1` |
| `<chart>-v<semver>-beta.<n>` | Beta | `hytale-v0.2.0-beta.1` |

### Publishing

Charts are published to:
- OCI Registry: `ghcr.io/everhytale/<chart-name>` (e.g., `ghcr.io/everhytale/hytale`)
- GitHub Pages: `https://everhytale.github.io/charts`

## License

MIT License - see [LICENSE](LICENSE) for details.
