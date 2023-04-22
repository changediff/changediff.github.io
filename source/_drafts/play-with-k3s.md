---
title: play-with-k3s
tags: ["kubernetes", "k8s", "rpi", "k3s"]
---
#

## 选型

k3s microk8s minikube


## k3s

sudo apt install linux-modules-extra-raspi

https://docs.k3s.io/quick-start

### install
#### master
```
curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=stable sh -s - --prefer-bundled-bin
```

INSTALL_K3S_VERSION
#### node
/var/lib/rancher/k3s/server/node-token
```
token=xxx
curl -sfL https://get.k3s.io | K3S_URL=https://k3s.lan.com:6443 K3S_TOKEN=$token INSTALL_K3S_CHANNEL=v1.26 sh -s - --prefer-bundled-bin
```


#### samba storage class

wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_arm64 -O /usr/bin/yq &&    chmod +x /usr/bin/yq

git clone 

#### nfs

/export 做软链解决root为overlay无法使用nfs的问题


https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

$ helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
$ helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=r5s.lan.com \
    --set nfs.path=/nfs


https://github.com/kubernetes-csi/csi-driver-nfs
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version v4.1.0

helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --set driver.name="nfs.csi.k8s.io" --set controller.name="csi-nfs-controller" --set rbac.name=nfs --set serviceAccount.controller=csi-nfs-controller-sa --set serviceAccount.node=csi-nfs-node-sa --set node.name=csi-nfs -node --set node.livenessProbe.healthPort=39653


https://www.cnblogs.com/yangyuliufeng/p/14227590.html


root@rpi1:/etc/rancher/k3s# cat /etc/rancher/k3s/config.yaml
data-dir: "/mnt/ssd/server/k3s"

## 内网服务

