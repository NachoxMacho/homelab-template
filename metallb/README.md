# MetalLB

This guid will walk you through installing MetalLB on your cluster.

## Prerequisites

- A Kubernetes cluster running (I'm using Proxmox to run my kubernetes cluster)
- At least one IP address that is not in use on your network (Not used by any other VMs, kubernetes API server, etc.)

## Installation

### Configuring the metallb files

The metallb needs to be configured to use the IP address you want to use.
This is done by editing the `ip-assignment.yaml`, `values.yaml`, and `namespace.yaml` files.

You will need to change the following values:

#### ip-assignment.yaml

- `<ingress-ip>`: This is the IP address you want to use for your cluster ingress.
  - This IP address must be unused on your network and not match any of the virtual machines you will be building.
  - If you want to allow multiple ingress destinations, you can use a CIDR range, like `10.0.0.10/31`.
- `<metallb-namespace>`: This is the namespace where you will install metallb.

#### namespace.yaml

- `<metallb-namespace>`: This is the namespace where you will install metallb.

### Creating the metallb namespace

Before you can install metallb, you need to create the namespace where you will install it.
You can do this by running the following command:

```bash
kubectl apply -f namespace.yaml
```

### Installing the metallb helm chart

Once you have configured the `ip-assignment.yaml` and `values.yaml` files, you can install the metallb using the following commands:

```bash
kubectl create namespace <metallb-namespace>
kubectl apply -f ip-assignment.yaml -n <metallb-namespace>
helm install metallb metallb/metallb -n <metallb-namespace> --values values.yaml
```

Assuming everything is configured correctly you should see pods start running in the namespace you specified.

### Creating the metallb IP address pool

You will need to create an IP address pool for the metallb to use.
This is done by running the following command:

```bash
kubectl apply -f ip-assignment.yaml -n <metallb-namespace>
```

This will create an IP address pool for the metallb to expose.


