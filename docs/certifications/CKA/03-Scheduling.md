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
* Toleration: "You can schedule here even with taint"


