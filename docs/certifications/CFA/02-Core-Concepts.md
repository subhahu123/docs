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
  * either
    * process with settings in systemd service
    * or pod with settings in `/etc/kubernetes/manifests/kube-apiserver.yaml` (kubeadm)
* kube-scheduler
  * Schedules pods on workers, updates etcd
  * decides which pod goes where based on requirements
* kubelet
  * Makes changes on worker
  * does EVERYTHING on node, communicates with api-server
  * Need to run on worker as service
* Controller-Manager (brain of k8s)
  * Manages controllers (processes that monitor status of components, nodes etc)
  * Controllers are inside Controller-Manager process
* kube-proxy
  * Deals with communications
  * Internal IPs can change on nodes, we use services instead of pod IPs
  * kube-proxy runs on each node and creates rules based on services so pod is accessible

### Pods

* We can create pods with `yaml`
* Several keys required in yaml

Required:

```yaml
apiVersion:
kind:
metadata:
spec:
```

Typical pod values:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
    containers:
        - name: nginx-container
          image: nginx
```

```shell
kubectl create -f $FILE.yaml
kubectl describe myapp-pod
```

For viewing state:

```shell
kubectl describe pod webapp
kubectl get pod webapp -o yaml
```

Checking where pod is located:

```shell
kubectl get pods -o wide
```

Modifying current pods:

```shell

```
