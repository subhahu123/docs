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
