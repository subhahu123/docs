# Core Concepts

## Cluster Architecture

* Kubelet listens for commanda (on each node)
* Kube proxy manages communication between workers (on each node)

### Containers

CRI - lets different solutions for running containers work (containerd etc)

Imagespec - how container images are setup
Runtimespec - how containers run

### ContainerD

For debugging `ctr` official tool

Alt tool: `nerdctl` - more user friendly, similar to `docker` cli

`crictl` works across all CRI runtimes, good for debugging

Very similar to `docker`

### etcd

* KV store
* 2 main APIs (v2, and v3), significant API change
* All k8s changes modify etcd

### Components

* kube-apiserver
  * Who you talk to with `kubectl`
  * Only think that talks to `etcd`
* kube-scheduler
  * Schedules pods on workers, updates etcd
* kubelet
  * Makes changes on worker
* Controller-Manager (brain of k8s)
  * Manages controllers (processes that monitor status of components, nodes etc)
  * Controllers are inside Controller-Manager process

kube-apiserver either process with settings in systemd service
or pod with settings in `/etc/kubernetes/manifests/kube-apiserver.yaml` (kubeadm)


