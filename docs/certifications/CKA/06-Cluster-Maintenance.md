# Cluster Maintenance

"drain" node and move pods:

 ```shell
kubectl drain node-1
 ```

This "cordons" a node, to uncordon:

 ```shell
kubectl uncordon node-1
 ```

cordon marks unschedulable but leaves existing nodes:

```shell
kubectl cordon node-1
```

## Cluster Upgrade Introduction

Components should be somewhat in synch.

kube-apiserver is main component, the controller manager and the kube scheduler should be less than or equal to the version, and be a maximum of one lower inversion. The kubelet and kube proxy should be a maximum of two versions lower than the API server and should not be greater than the version of the API server.

`kubectl` should be +-1

k8s supports last 3 minor versions.

Upgrades do master first (pods stay up meanwhile)

Nex we do workers, can do all at once or one node at a time.

Alternatively create new nodes with higher version and remove old

We need to upgrade `kubeadm` first with `apt`.

Then `kubelet` with `apt`

Upg master:

```shell
kubeadm upgrade plan
apt upgrade -y kubeadm=VERSION
kubectl get nodes
apt upgrade -y kubelet=VERSION
systemctl restart kubelet
kubectl get nodes
```

Upg workers:

```shell
kubectl drain NODE
apt upgrade -y kubeadm=VERSION
kubectl get nodes
apt upgrade -y kubelet=VERSION
systemctl restart kubelet
kubeadm upgrade node config --kubelet-version VERSION
kubectl uncordon NODE
```

[Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

## Backup and Restore

Can save all yaml for cluster via:

```shell
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

Can backup `etcd` via:

```shell
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

To restore:

```shell
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir=NEW_ETCD_DIR
```

[Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

Usually etcd is a static pod, so if we want to edit, edit manifests.

Look at pod:

```shell
kubectl describe ETCD_POD
```

Find ip, `trusted-ca-file`, `key-file` and `cert-file`, test via:

```shell
ETCDCTL_API=3 etcdctl --endpoints IP_ADDR:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  member list
```

Snapshot to `/opt/snapshot-pre-boot.db`:

```shell
ETCDCTL_API=3 etcdctl --endpoints IP_ADDR:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  snapshot save /opt/snapshot-pre-boot.db
```

Restore to `/etcd-backup`:

```shell
ETCDCTL_API=3 etcdctl --endpoints IP_ADDR:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --data-dir=/etcd-backup \
  snapshot restore /opt/snapshot-pre-boot.db
```

We will edit static pod. And point the etcd-data hostpath to new data directory.

## Multi-Cluster

List all:

```shell
kubectl config get-clusters
```

Swap:

```shell
kubectl config use-context CLUSTER
```
