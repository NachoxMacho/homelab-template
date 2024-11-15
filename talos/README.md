# Talos Cluster Installation


## Prerequisites

- Talos CLI installed: https://www.talos.dev/v1.8/talos-guides/install/talosctl/
- DNS record for the API server

## Installation

```bash
talosctl gen secrets --output-file secrets.yaml # Generate secrets for the cluster, store this file somewhere safe.
```
