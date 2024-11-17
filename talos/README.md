# Talos Cluster Installation


## Prerequisites

- Talos CLI installed: https://www.talos.dev/v1.8/talos-guides/install/talosctl/

### Creating a DNS record for the API server

Kubernetes needs a DNS record for the API server.
Thus you will need pick an IP address for the API server, and create a DNS record for it.

**IMPORTANT**: THIS IP ADDRESS MUST BE UNUSED ON YOUR NETWORK AND NOT MATCH ANY OF THE VIRTUAL MACHINES YOU WILL BE BUILDING. USE A STATIC IP ADDRESS IF POSSIBLE.

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
- Disk/Store: Local disk (not backed by ceph. [Related Problem](../troubleshooting-log.md#2024-11-14-etcd-leader-election-lost-pod-restarts))

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

This is when if you are using DHCP for node IPS, you can make sure that the IPs are assigned to the VMs.
They will show on the Talos overview screen in the top right by IP address, if the IPs are not correct you will need to modify the DHCP leases.

If you are using static IPs, refer the `control-plane.yaml` and `worker.yaml` files for setting the IPs.

## Bootstrapping the cluster

[Talos instructions available here](https://www.talos.dev/v1.8/talos-guides/install/virtualized-platforms/proxmox/#create-control-plane-node)

Run the following command to apply the configuration file to the control plane VM.
Start with a single control plane node at first, and once you have completed bootstrapping the cluster you can add additional nodes.

```bash
talosctl apply-config --nodes <IP-of-control-plane-VM> --file tmp/controlplane.yaml --insecure
```

This will reboot the VM and install the talos image to disk, as well as start the kubernetes installation process.
Once you see the following line appear in the talos VM logs, you can proceed to [configuring the talosctl client](#Configuring-the-talosctl-client) and [bootstrapping etcd](#Bootstrapping-etcd).

```
etcd is waiting to join the cluster, if this node is the first node in the cluster please run `talosctl bootstrap` against one of the following IPs
```

### Configuring the talosctl client

Before you run any other talosctl commands against your cluster in the future, you need to config the talosctl client to use the correct configuration.
You only need to do this once per bash/powershell session.

```bash
export TALOSCONFIG=tmp/talosconfig
talosctl config endpoint <IP-of-control-plane-VM>
```

or for those on Windows:

```powershell
$env:TALOSCONFIG="tmp/talosconfig"
talosctl config endpoint <IP-of-control-plane-VM>
```

## Bootstrapping etcd

You can then run the following command to bootstrap etcd:

```bash
talosctl config nodes <IP-of-control-plane-VM>
talosctl bootstrap
```

## Retrieving the kubeconfig file

You can retrieve the kubeconfig file for the cluster using the following command.

```bash
talosctl kubeconfig --merge
```

You can then verify that the cluster is running by running `kubectl get pods -A`.
You should see a list of pods running in the `kube-system` namespace, like the following:

```bash
$ kubectl get pods -A
NAMESPACE                       NAME                                            READY   STATUS    RESTARTS       AGE
kube-system                     coredns-68d75fd545-98clg                        0/1     Pending   0              117s
kube-system                     coredns-68d75fd545-p7v44                        0/1     Pending   0              117s
kube-system                     kube-controller-manager-c1-trueflame            1/1     Running   0              48s
kube-system                     kube-proxy-9mpzg                                1/1     Running   0              102s
kube-system                     kube-scheduler-c1-trueflame                     1/1     Running   1 (2m7s ago)   42s
kubelet-serving-cert-approver   kubelet-serving-cert-approver-f6995d4dc-qwts7   0/1     Pending   0              117s
```

## Add a worker node

Now that you have a control plane node, you can add your first worker node to run the kubelet cert approver.

```bash
talosctl apply-config --nodes <IP-of-worker-VM> --file tmp/worker.yaml --insecure
```

## Adding Cilium CNI

Cilium is a CNI that provides networking for Kubernetes.
You can add Cilium to your cluster by running the following command.
This is needed to allow your nodes to communicate with each other.

```bash
helm repo add cilium https://helm.cilium.io/
helm repo update
helm upgrade cilium cilium/cilium --namespace kube-system --values cilium/values.yaml --install
```


## Adding extra nodes

Once you have [configured the talosctl client](#Configuring-the-talosctl-client), you can use the following commands to add additional nodes to the cluster based on what type of node you want to add.

### Adding a control plane node

```bash
talosctl apply-config --nodes <IP-of-control-plane-VM> --file tmp/controlplane.yaml --insecure
```

### Adding a worker node

```bash
talosctl apply-config --nodes <IP-of-worker-VM> --file tmp/worker.yaml --insecure
```

## Updating a node

If you need to update a node's configuration, you can use the following command based on the type of node you want to update.

### Updating a control plane node

```bash
talosctl apply-config --nodes <IP-of-control-plane-VM> --file tmp/controlplane.yaml
```

### Updating a worker node

```bash
talosctl apply-config --nodes <IP-of-worker-VM> --file tmp/worker.yaml
```



## IF YOU FUCKED UP

If you messed up a node at any point, you can use the following command to reset the node.
Make sure to [configure the talosctl client](#Configuring-the-talosctl-client) before running this command.

```bash
talosctl reset --nodes <IP-of-control-plane-VM>
```
