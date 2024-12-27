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

