# Scheduling

## Manual Scheduling

To manually schedule at creation - `nodeName`:

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
  name: nginx
spec:
 containers:
 - name: nginx
   image: nginx
   ports:
   - containerPort: 8080
 nodeName: node02
 ```

Or create a binding object:

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

## Labels and Selectors

Filter via selectors

Labels in metadata

Can use:

```shell
kubectl get pods --selector app=nginx
```

## Taints and Tolerations

* Taint: Tell pod "dont schedule here"
  * We taint nodes
* Toleration: "You can schedule here even with taint"
  * Tolerate taint=xyz

```shell
kubectl taint nodes
kubectl taint nodes <node-name> key=value:taint-effect
```

Taint effect defines what would happen to the pods if they do not tolerate the taint.

* NoSchedule
* PreferNoSchedule: Best effort
* NoExecute: Happens to nodes on existing nodes
  * Once taint takes effect, existing node evicts pod unless meets NoEvict

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: nginx-container
   image: nginx
 tolerations:
 - key: "app"
   operator: "Equal"
   value: "blue"
   effect: "NoSchedule"
```

Master nodes have NoSchedule

## Node Selectors

We can add `nodeSelectors` to a pod, which will help with scheduling:

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 nodeSelector:
  size: Large
```

To label nodes:

```shell
kubectl label nodes <node-name> <label-key>=<label-value>
kubectl label nodes node-1 size=Large
```

## Node Affinity

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
 containers:
 - name: data-processor
   image: data-processor
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
```

Other options:

```yaml
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: NotIn
            values:
            - Small
```

```yaml
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: Exists
```

Available
* `requiredDuringSchedulingIgnoredDuringExecution`
* `preferredDuringSchedulingIgnoredDuringExecution`

## Resource Requirements

* Can specify requirements with `resource.requests`
* Can specify limits with `resource.limits`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
    - containerPort:  8080
   resources:
     requests:
      memory: "1Gi"
      cpu: "1"
     limits:
       memory: "2Gi"
       cpu: "2"
```

Defaults is no limit, no requirements.

* If no request, but we have limit, request = limit
* Should atleast set `requests` to avoid starting a pod.

If pod uses too much RAM during usage, we will OOM kill.

We can set defaults for a namespace with `LimitRange`:

We can also set `ResourceQuota` request and limit for a namespace.

You cant adjust limits on pod without deletion, you can on deployment. Deployment will re-create.

## DaemonSets

Run one copy of pod on every node in cluster.

Matadata very similar to `ReplicaSet`:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
  labels:
    app: nginx
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
     labels:
       app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

Under the hood uses affinity.

## Static Pods

Kubelet can read from `/etc/kubernetes/manifests` instead of talking to `kube-api`

We can only use pods, no complex deployments.

Check `--pod-manifest-path` or (`--kubeconfig` for `staticPodPath:`)

We can view these by listing containers:

* `crictl ps`
* `nerdctl ps`
* `docker ps`

Cluster is aware of static pods, but we can't edit them outside manifests.

Kubeadm sets up some services this way.

## Multiple Schedulers

We can add custom schedulers.

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
```

If using process, name should match systemd service which points at `yaml` config with `--config`

If scheduler in pod, simply deploy as normal pod/deployment:

[Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/)

On pod creation, direct pod to use custom scheduler:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-custom-scheduler
```

```shell
kubectl get events -o wide
kubectl logs my-custom-scheduler -n kube-system
```

## Scheduler Profiles

Scheduling has various stages, each can have associated plugins:

* Scheduling queue
* Filtering
* Scoring
* Binding

To customize plugins for each phase we have extension points

We can set multiple profiles for one scheduler binary:

```yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
    plugins:
      score:
        disabled: []
        enabled: []
```
