# Talos Cluster Installation


## Prerequisites

- Talos CLI installed: https://www.talos.dev/v1.8/talos-guides/install/talosctl/
- DNS record for the API server

## Configuration Generation

```bash
# Generate secrets for the cluster, store this file somewhere safe.
talosctl gen secrets --output-file secrets.yaml
# Generates the full cluster configuration files.
talosctl apply <name-of-cluster> <DNS-record-for-API-server> --config-patch-control-plane @talos/control-plane.yaml --config-patch-worker @talos/worker.yaml --with-secrets secrets.yaml --output tmp
```


## Setup Virtual Machines

Recommended VM Specs:

- CPU: 2 Cores
- Memory: 8GB
- Disk: 16GB

ISO can be found here: https://factory.talos.dev/?arch=amd64&cmdline-set=true&extensions=-&extensions=siderolabs%2Fqemu-guest-agent&platform=metal&secureboot=true&target=metal&version=1.8.2

Note that for secure boot, you may need to manually add the key to the VM.
