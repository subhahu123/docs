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

## ReplicaSets

* A controller
* Lets is run multiple pods for HA
* Enforces number of pods
* Also used for load scaling
* Controller with balance pods across multiple nodes

ReplicaSet replaces depreciated Replication Controller

Depreciated **Replication Controller**:

Create:

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
    name: myapp-pod
    labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```

So spec.template is children

```shell
kubectl create -f $FILE.yml
kubectl get replicationcontroller
```

**ReplicaSet:**

selector is main difference, its required and takes children labels

Create:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
    name: myapp-pod
    labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```shell
kubectl create -f $FILE.yml
kubectl get replicaset
```

ReplicaSet monitors and keeps pods up based on labels and selectors.

## Scaling

Several options for scaling.

```shell
kubectl replace -f $FILE.yml # With updated replicas
kubectl scale --replicas=6 -f $DEFINITION.yml
kubectl scale --replicas=6 replicaset myapp-replicaset # By name
```

## Deployments

Used for rolling updates and scaling.

Deployments are a superset of other objects like ReplicaSet

Compared to ReplicaSet only `kind: Deployment` needs changing:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
    name: myapp-pod
    labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

```shell
kubectl create -f $FILE.yml
kubectl get deployments
kubectl get all # show all (pods, replicasets, deployments)
```

## Creating YAML in CKA

Using the `kubectl run` command can help in generating a YAML template. And sometimes, you can even get away with just the `kubectl run` command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with a specific name and image, you can simply run the `kubectl run` command.

* [Conventions](https://kubernetes.io/docs/reference/kubectl/conventions/)

Create an NGINX Pod

```shell
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file (`-o yaml`). Don’t create it(–dry-run)

```shell
kubectl run nginx --image=nginx --dry-run=client -o yaml
```

Create a deployment

```shell
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file (`-o yaml`). Don’t create it(`--dry-run`)

```shell
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
```

Generate Deployment YAML file (`-o yaml`). Don’t create it (`--dry-run`) and save it to a file.

```shell
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

```shell
kubectl create -f nginx-deployment.yaml
```

OR

In k8s version 1.19+, we can specify the –replicas option to create a deployment with 4 replicas.

```shell
kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

## Services

Help with establishing connections.

Pods are on private net, we need to expose services within them

Service is an object that:
* NodePort: forwards ports from node to pod
* ClusterIP: Creates virtual IP for internal communication
* LoadBalance: Distributes traffic

### NodePort

* TargetPort: pod port
* Port: port for Service to Pod
* NodePort: port on Node

![NodePort](./images/02/NodePort.png)
