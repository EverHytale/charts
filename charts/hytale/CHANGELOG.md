# Changelog

All notable changes to the Hytale Helm chart will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-01-12

### Added

- Initial release of the Hytale Helm chart
- Deployment with Recreate strategy for safe volume access
- Persistent storage for world data
- ConfigMap for server.properties configuration
- Secret for RCON password
- Service with TCP/UDP support for game traffic
- RCON support for remote administration
- Gateway API TCPRoute support
- TCP probes for liveness/readiness checks
- Startup probe for initial world generation
- Comprehensive documentation

### Configuration Options

- Server configuration (name, MOTD, max players, game mode, difficulty, etc.)
- RCON configuration with existing secret support
- Persistence with configurable storage class and size
- Resource limits and requests
- Node selector, tolerations, and affinity rules
