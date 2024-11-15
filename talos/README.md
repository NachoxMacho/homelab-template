# Talos Cluster Installation


## Prerequisites

- Talos CLI installed: https://www.talos.dev/v1.8/talos-guides/install/talosctl/

### Creating a DNS record for the API server

Kubernetes needs a DNS record for the API server.
Thus you will need pick an IP address for the API server, and create a DNS record for it.
That same IP address will be used for the Virtual IP of the control plane, and needs to be set in the `control-plane.yaml` file.

## Configuration Generation

Review the `control-plane.yaml` and `worker.yaml` files, and make any customizations you wish.


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

Recommended VM Settings in Proxmox:

- QEMU Guest Agent: Enabled
- BIOS: OVMF (UEFI)
- Pre-Enroll keys: Disabled
- Add TPM: Enabled
- Disk/Discard: Enabled
- Disk/Cache: Write Through

Note that for secure boot, you may need to manually add the key to the VM.

