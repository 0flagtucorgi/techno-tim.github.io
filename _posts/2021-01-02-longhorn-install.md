---
layout: post
title: "Cloud Native Distributed Storage in Kubernetes with Longhorn"
date: 2021-01-02 09:00:00 -0500
categories: kubernetes rancher
tags: homelab rancher kubernetes longhorn k3s
image:
  path: /assets/img/headers/longhorn-cattle.jpg
---

Storage in Kubernetes is hard, complicated, and messy.  Configuring volumes, mounts, and persistent volumes claims and getting it right can be a challenge.  It's also challenging to manage that storage and replicate it across all your Kubernetes clusters.  It's also been very challenging to do this on bare metal, outside of a cloud provider.  That's where Longhorn comes.  Longhorn is an open source, a CNCF distributed block storage system for Kubernetes.  It comes with a UI, backups, snapshots, cluster disaster recovery, and it does all this with or without Rancher.  Rancher is NOT a requirement.

{% include embed/youtube.html id='eKBBHc0t7bc' %}

📺 [Watch Video](https://www.youtube.com/watch?v=eKBBHc0t7bc)

## Installation

### Additional Dependencies

There are some additional dependencies you might want to install on target nodes prior to configuring

```bash
sudo apt update
sudo apt install nfs-common open-iscsi
#start the service now and on reboot
sudo systemctl enable open-iscsi --now
```

## Install Methods

### Rancher app catalog

See the app catalog within Rancher

### Kubectl

```bash
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```

```bash
kubectl get pods \
--namespace longhorn-system \
--watch
```

See more at [https://longhorn.io/docs/1.0.0/deploy/install/install-with-kubectl](https://longhorn.io/docs/1.0.0/deploy/install/install-with-kubectl)

### Helm

helm3

```bash
kubectl create namespace longhorn-system
helm install longhorn ./longhorn/chart/ --namespace longhorn-system
```

```bash
kubectl -n longhorn-system get pod
```

## Taints

> This is not required, nor do I taint nodes anymore.  I allow Longhorn storage to use any available space on any node that is not running `etcd` / control plane.  You can simply skip this step and it will work like this.  If you're still convinced you need dedicated nodes, it's much easier doing it in the Longhorn UI after a node joins the cluster than with taints.
{: .prompt-info }

I ended up tainting my storage nodes using this command

```bash
kubectl taint nodes luna-01 luna-02 luna-03 luna-04 CriticalAddonsOnly=true:NoExecute
kubectl taint nodes luna-01 luna-02 luna-03 luna-04 StorageOnly=true:NoExecute
```

Then applying that toleration to Longhorn in settings

`StorageOnly=true:NoExecute;CriticalAddonsOnly=true:NoExecute`

This ensures that the storage nodes won't take on any general workloads and still allow Lonhorn to use these as storage.

## Links

⚙️ See all the hardware I recommend at <https://l.technotim.live/gear>

🚀 Don't forget to check out the [🚀Launchpad repo](https://l.technotim.live/quick-start) with all of the quick start source files
