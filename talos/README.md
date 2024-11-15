# Talos Cluster Installation


## Prerequisites

- Talos CLI installed: https://www.talos.dev/v1.8/talos-guides/install/talosctl/
- DNS record for the API server

## Installation

```bash
# Generate secrets for the cluster, store this file somewhere safe.
talosctl gen secrets --output-file secrets.yaml
# Generates the full cluster configuration files.
talosctl apply <name-of-cluster> <DNS-record-for-API-server> --config-patch-control-plane @talos/control-plane.yaml --config-patch-worker @talos/worker.yaml --with-secrets secrets.yaml --output tmp
```
