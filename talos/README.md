# Talos Cluster Installation


## Prerequisites

- Talos CLI installed: https://www.talos.dev/v1.8/talos-guides/install/talosctl/

### Creating a DNS record for the API server

Kubernetes needs a DNS record for the API server.
Thus you will need pick an IP address for the API server, and create a DNS record for it.
**IMPORTANT**: THIS IP ADDRESS MUST BE UNUSED ON YOUR NETWORK. USE A STATIC IP ADDRESS IF POSSIBLE.
That same IP address will be used for the Virtual IP of the control plane, and needs to be set in the `control-plane.yaml` file.

## Configuration Generation

Review the `control-plane.yaml` and `worker.yaml` files, and make any customizations you wish.


```bash
# Generate secrets for the cluster, store this file somewhere safe.
talosctl gen secrets --output-file secrets.yaml
# Generates the full cluster configuration files.
talosctl gen config <name-of-cluster> <DNS-record-for-API-server> --config-patch-control-plane @talos/control-plane.yaml --config-patch-worker @talos/worker.yaml --with-secrets secrets.yaml --output-dir tmp
```


## Setup Virtual Machines

ISO can be found here: https://factory.talos.dev/?arch=amd64&cmdline-set=true&extensions=-&extensions=siderolabs%2Fqemu-guest-agent&platform=metal&secureboot=true&target=metal&version=1.8.2

Recommended VM Specs:

- CPU: 2 Cores
- Memory: 8GB
- Disk: 16GB


Recommended VM Settings in Proxmox:

- QEMU Guest Agent: Enabled
- BIOS: OVMF (UEFI)
- Pre-Enroll keys: Disabled
- Add TPM: Enabled
- Disk/Discard: Enabled
- Disk/Cache: Write Through

Note that for secure boot, you may need to manually add the key to the VM using the following instructions:

1. Boot the VM
1. You should get a line at the top mentioning a "Security Violation"
1. Wait for Talos to enter the boot menu, and select the option to "Reboot into Firmware interface"
1. Select Device Manager
1. Select the "Secure Boot Configuration" option
1. Select "Secure Boot Mode" and select "Custom Mode"
1. Select Custom Secure Boot Options
1. Select PK Options
1. Select Enroll PK
1. Select "Enroll PK Using file"
1. Navigate to the key file for the ISO, it's usually at the path `EFI/EFI/keys/uki-signing-cert.der`
1. Select the `.der` file
1. Select "Commit Changes and Exit"
1. Navigate back to the main menu and select "Continue"
1. Let the VM reboot

[Thanks to Nyanchovy on the proxmox forums for the instructions](https://forum.proxmox.com/threads/enroll-custom-secureboot-keys.151443/post-690267)

You will know you have successfully enrolled the key if you see the following line at the top of the talos overview screen:

```
SECUREBOOT: True
```

If you do not see this line, you need to manually enroll the key, see the previous section for instructions.

You will need a minimum of 2 VMs, one for the control plane, and one for the worker.

It's recommended to run at least 3 VMs for the contol plane, and you can run as many as needed for worker nodes.

Once both VMs are running in maintenance mode, you can proceed to the next step to bootstrap your cluster.

## Bootstrapping the cluster

[Talos instructions available here](https://www.talos.dev/v1.8/talos-guides/install/virtualized-platforms/proxmox/#create-control-plane-node)

Run the following command to apply the configuration file to the control plane VM. Start with a single node at first, and once you have completed this section you can add additional nodes.

```bash
talosctl apply-config --nodes <IP-of-control-plane-VM> --config tmp/control-plane.yaml --insecure
```

This will reboot the VM and install the talos image to disk, as well as start the kubernetes installation process.
Once you see the following line appear in the talos VM logs, you can proceed to bootstrapping etcd.

```
<insert line about bootstrapping here>
```

You can then run the following command to bootstrap etcd:

```bash
export TALOSCONFIG=tmp/talosconfig
talosctl config endpoint <IP-of-control-plane-VM>
talosctl bootstrap --nodes <IP-of-control-plane-VM>
```

or for those on Windows:

```powershell
$env:TALOSCONFIG="tmp/talosconfig"
talosctl config endpoint <IP-of-control-plane-VM>
talosctl bootstrap --nodes <IP-of-control-plane-VM>
```

## Adding extra nodes

Before you run any other talosctl commands against your cluster in the future, you need to config the talosctl client to use the correct configuration.

```bash
export TALOSCONFIG=tmp/talosconfig
talosctl config endpoint <IP-of-control-plane-VM>
```

or for those on Windows:

```powershell
$env:TALOSCONFIG="tmp/talosconfig"
talosctl config endpoint <IP-of-control-plane-VM>
```

