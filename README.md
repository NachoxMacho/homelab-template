# Homelab Template

This is a bunch of files that I use to set up my homelab.

## Overview

Here's an overview of each folder's contents, in the order I've deployed the services in.
I'd recommend starting with the [Core Services](#Core-Services), as that will get you to a base cluster that you can deploy other services on top of.
If you are just looking for fun/useful services I've deployed to my cluster, check out the [Additional Services](#Additional-Services) section.

## Core Services

This services are requirements for other services and covers the following categories:

- Cluster creation
- Networking
- Ingress
- Storage
- Secrets Management
- Certificate Management
- Databases

### Talos

The talos folder contains patch files for deploying a talos cluster.

Start by installing the talos CLI: https://www.talos.dev/v1.8/talos-guides/install/talosctl/

Then follow the instructions located in the [talos folder](https://github.com/NachoxMacho/homelab-template/tree/main/talos).

### Metallb

### Nginx-Ingress

### Ceph-CSI

The ceph-csi folder contains example files for deploying the cephfs-csi to the talos cluster.
This is the primary storage class in the cluster, but requires an existing Ceph cluster to be created.
If you don't have a Ceph cluster available, then you will have to substitue this step for something else, I'd recommend either [Longhorn]() or [Rook]().

Start with the README in the [ceph-csi folder](./ceph-csi/README.md).

### Infisical

The infisical folder contains example files for deploying [Infisical]() to the talos cluster.
This is the primary tool I use to store and sync sensitive values and secrets.
I will create a secret in the application and can then sync that secret to the kubernetes cluster, allowing me to easily rotate secrets as well as exclude them from various helm charts.
You can swap this with another secrets provider if you want to, just note that in future documentation I assume you have Infisical setup for secret syncing so you will have to modify the files/examples to your own provider.

Start with the README in the [infisical folder](./infisical/README.md).

### Cert-Manager

The cert-manager folder contains example files for deploying [Cert-Manager]() to the talos cluster.
This tool will
