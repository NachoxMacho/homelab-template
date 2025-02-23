# Ceph CSI

This will be a guide on how to install and configure the ceph csi for kubernetes.

## Prerequisites

- A Ceph cluster running (I'm using Proxmox to run my ceph cluster)
- A Ceph user with full permissions to the pool you want to use (I recommend creating a dedicated user and pool for the csi)
  - For instructions on how to create a user and pool, see the [ceph documentation](https://docs.ceph.com/en/latest/rados/operations/user-management/).
  - Optionally you can use [this link](https://devopstales.github.io/kubernetes/k8s-cephfs-storage-with-csi-driver/) to create a pool and user.

## Installation

### Installing the ceph CSI

This is lifted from the [ceph CSI installation guide](https://github.com/ceph/ceph-csi/tree/devel/charts/ceph-csi-cephfs).
Add the helm repository locally using the following commands.

```bash
helm repo add ceph-csi https://ceph.github.io/csi-charts
helm repo update
```

### Configuring the ceph CSI files

The ceph CSI needs to be configured to use the ceph cluster you want to use.
This is done by editing the `values.yaml`, `secret.yaml`, and `namespace.yaml` file.

You will need to change the following values:

#### values.yaml

- `csiConfig.clusterID`: This is the cluster ID of the ceph cluster you want to use. You can find this by running `ceph status` on a ceph monitor node.
- `csiConfig.monitors`: This is a list of the IPs your ceph monitors are reachable at. You can find this by running `ceph mon stat` on a ceph monitor node.
- `storageClass.clusterID`: This is the cluster ID of the ceph cluster you want to use, same as `csiConfig.ClusterID`.
- `storageClass.fsName`: This is the name of the ceph filesystem you want to use.
- `storageClass.pool`: This is the name of the ceph pool you want to use.
- `storageClass.provisionerSecretNamespace`: This is the namespace where you will install ceph.
- `storageClass.controllerExpandSecretNamespace`: This is the namespace where you will install ceph.
- `storageClass.nodeStageSecretNamespace`: This is the namespace where you will install ceph.

If you are running a smaller cluster (less than 3 worker nodes), you can reduce the number of replicas for the provisioner with the following configuration:

```yaml
provisioner:
  replicaCount: <number-of-nodes>
```

It's recommended to run multiple replicas of the provisioner to ensure that the ceph cluster is always available to create new volumes and for high availability.
In my testing, I've found the default of 3 replicas to be sufficient for my needs.

#### secret.yaml

This is where you will provider the user credentials for the ceph user the csi will act as.
For more information on setting up a user and pool, see the [prerequisites section](#Prerequisites).

- `data.adminID`: This is the admin ID of the ceph user you want to use.
- `data.adminKey`: This is the admin key of the ceph user you want to use.

#### namespace.yaml

This is where you will create the namespace where you will install the ceph CSI.

- `<ceph-namespace>`: This is the namespace where you will install the ceph CSI.

### Creating the namespace

Before you can install the ceph CSI helm chart, you need to create the namespace where you will install it.
You can do this by running the following command:

```bash
kubectl apply -f namespace.yaml
```

### Installing the ceph CSI

Once you have configured the `values.yaml` and `secret.yaml` files, you can install the ceph CSI using the following commands:

```bash
kubectl create namespace <ceph-namespace>
kubectl apply -f secret.yaml -n <ceph-namespace>
helm install ceph-csi ceph-csi/ceph-csi-cephfs -n <ceph-namespace> --values values.yaml
```

Assuming everything is configured correctly you should see pods start running in the namespace you specified.
