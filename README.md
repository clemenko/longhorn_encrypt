---
title: How to deploy Longhorn with encryption at rest.
author: Andy Clemenko, @clemenko, andy.clemenko@rancherfederal.com
---

# Deploy Longhorn with encryption at rest.

![logo](img/longhorn.jpg)

Data security is becoming an increasing importance with our customers. One of the great features of [Longhorn](https://longhorn.io) is the ability to encrypt the volumes at rest. Meaning the data on the nodes are encrypted. From the [docs](https://longhorn.io/docs/1.3.0/advanced-resources/security/volume-encryption/) : *An encrypted volume results in your data being encrypted while in transit as well as at rest, this also means that any backups taken from that volume are also encrypted.*

We will need a few tools for this guide. We will walk through how to install `helm` and `kubectl`.

Or [Watch the video](https://youtu.be/oM-6sd4KSmA).

---

> **Table of Contents**:
>
> * [Whoami](#whoami)
> * [Prerequisites](#prerequisites)
> * [Linux Servers](#linux-servers)
> * [RKE2 Install](#rke2-install)
>   * [RKE2 Server Install](#rke2-server-install)
>   * [RKE2 Agent Install](#rke2-agent-install)
> * [Rancher](#rancher)
>   * [Rancher Install](#rancher-install)
>   * [Rancher Gui](#rancher-gui)
> * [Longhorn](#longhorn)
>   * [Longhorn Install](#longhorn-install)
>   * [Longhorn Gui](#longhorn-gui)
> * [Automation](#automation)
> * [Conclusion](#conclusion)

---

## Whoami

Just a geek - Andy Clemenko - @clemenko - andy.clemenko@rancherfederal.com

## Prerequisites

The prerequisites are fairly simple. We need a kubernetes cluster with access to the internet. They can be bare metal, or in the cloud provider of your choice. I prefer [Digital Ocean](https://digitalocean.com). We need an `ssh` client to connect to the servers. And finally DNS to make things simple. Ideally we need a URL for the Rancher interface. For the purpose of the this guide let's use `longhorn.rfed.io`. We will need to point that name to the first server of the cluster. While we are at it, a wildcard DNS for your domain will help as well.

## Linux Servers and Kubernetes

For the sake of this guide we are going to use [Rocky Linux](https://rockylinux.org/). Honestly any OS will work. Our goal is a simple deployment. The recommended size of each node is 4 Cores and 8GB of memory with at least 60GB of storage. One of the nice things about [Longhorn](https://longhorn.io) is that we do not need to attach additional storage. Here is an example list of servers. Please keep in mind that your server names can be anything. Just keep in mind which ones are the "server" and "agents".

| name | ip | memory | core | disk | os |
|---| --- | --- | --- | --- | --- |
|rke1| 142.93.189.52  | 8192 | 4 | 160 | Rocky Linux RockyLinux 8.5 x64 |
|rke2| 68.183.150.214 | 8192 | 4 | 160 | Rocky Linux RockyLinux 8.5 x64 |
|rke3| 167.71.188.101 | 8192 | 4 | 160 | Rocky Linux RockyLinux 8.5 x64 |

As for Kubernetes, you can install any that you want. It is highly recommended to have an ingress controller as well. This will help getting to Longhorn's dashboard. We can use NodePort if needed.

## Longhorn

### Lognhorn Install

There are several methods for installing. Rancher has Chart built in.

![charts](img/charts.jpg)

Now for the good news, [Longhorn docs](https://longhorn.io/docs/1.3.0/deploy/install/) show two easy install methods. Helm and `kubectl`. Let's stick with `kubectl` for this guide.

```bash
# from https://longhorn.io/docs/1.3.0/deploy/install/install-with-kubectl/
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.3.0/deploy/longhorn.yaml
```

Here is what to expect.

```bash
$ kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/v1.3.0/deploy/longhorn.yaml
namespace/longhorn-system created
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/longhorn-psp created
serviceaccount/longhorn-service-account created
configmap/longhorn-default-setting created
configmap/longhorn-storageclass created
customresourcedefinition.apiextensions.k8s.io/backingimagedatasources.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/backingimagemanagers.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/backingimages.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/backups.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/backuptargets.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/backupvolumes.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/engineimages.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/engines.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/instancemanagers.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/nodes.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/orphans.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/recurringjobs.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/replicas.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/settings.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/sharemanagers.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/snapshots.longhorn.io created
customresourcedefinition.apiextensions.k8s.io/volumes.longhorn.io created
clusterrole.rbac.authorization.k8s.io/longhorn-role created
clusterrolebinding.rbac.authorization.k8s.io/longhorn-bind created
role.rbac.authorization.k8s.io/longhorn-psp-role created
rolebinding.rbac.authorization.k8s.io/longhorn-psp-binding created
service/longhorn-backend created
service/longhorn-frontend created
service/longhorn-conversion-webhook created
service/longhorn-admission-webhook created
service/longhorn-engine-manager created
service/longhorn-replica-manager created
daemonset.apps/longhorn-manager created
deployment.apps/longhorn-driver-deployer created
deployment.apps/longhorn-ui created
deployment.apps/longhorn-conversion-webhook created
deployment.apps/longhorn-admission-webhook created
```

Fairly easy right?

Make sure everything is up.

```bash
$ kubectl  get pod -n longhorn-system | grep -v Running
NAME                                          READY   STATUS    RESTARTS   AGE
```

### Longhorn GUI

This is going to be dependent upon your ingress controller. Personally I prefer [Traefik](https://traefik.io/). For the sake if simplicity we can use NodePort.

This brings up the Longhorn GUI.

![longhorn](img/longhorn.jpg)

One of the other benefits of this integration is that rke2 also knows it is installed. Run `kubectl get sc` to show the storage classes.

```text
root@rancher1:~# kubectl  get sc
NAME                 PROVISIONER          RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
longhorn (default)   driver.longhorn.io   Delete          Immediate           true                   3m58s
```

Now we have a default storage class for the cluster. This allows for the automatic creation of Physical Volumes (PVs) based on a Physical Volume Claim (PVC). The best part is that "it just works" using the existing, unused storage, on the three nodes. Take a look around in the gui. Notice the Volumes on the Nodes. For fun, here is a demo flask app that uses a PVC for Redis. `kubectl apply -f https://raw.githubusercontent.com/clemenko/k8s_yaml/master/flask_simple_nginx.yml`

### Enabling Encryption

### Using Encryption

## Automation

Yes we can automate all the things. Here is the repo I use automating the complete stack https://github.com/clemenko/rke2. This repo is for entertainment purposes only. There I use tools like pdsh to run parallel ssh into the nodes to complete a few tasks. Ansible would be a good choice for this. But I am old and like bash. Sorry the script is a beast. I need to clean it up.

## Conclusion

thanks!

![success](img/success.jpg)
