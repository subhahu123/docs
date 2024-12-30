# Application Lifecycle Management

## Rolling updates and Rollbacks

Deployments trigger "rollouts", marking new "revisions"

```shell
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
```

Deployments rolling cause no downtime due to rolling strategy.

Modify yaml, then apply, causing new rollout and revision.

Upgrades in deployments create new replicaset and remove pods from old, add to new

Useful:

```shell
kubectl create -f deployment-definition.yaml
kubectl get deployments
kubectl apply -f deployment-definition.yaml
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
```

## Commands and Arguments

Override commands and arguments via:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
    containers:
        - name: ubuntu-sleeper
          image: ubuntu-sleeper
          command: [ "sleep2.0" ]
          args: [ "10" ]
```

## Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
   - containerPort: 8080
   env:
   - name: APP_COLOR
     value: pink
```

## Configmaps

Lets is define kv pairs

Imperative:

```shell
kubectl create configmap CONFIG_NAME --from-literal=KEY=VALUE
kubectl create configmap app-config \
    --from-literal=APP_COLOR=blue \
    --from-literal=APP_MODE=prod
```

File:

ConfigMap:

```
APP_COLOR: blue
APP_MODE: prod
```_

```shell
kubectl create configmap CONFIG_NAME --from-file=CONFIG_FILE
kubectl create configmap app-config --from-file=app_config.properties
```

Declarative:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
   - containerPort: 8080
   envFrom:
   - configMapRef:
       name: app-config
```

```shell
kubectl get configmaps
kubectl get configmap CONFIG_MAP
kubectl describe configmap CONFIG_MAP
```

## Secrets

Same as ConfigMap, but encoded (NOT ENCRYPTED).

```shell
kubectl create secret
```

We encode as `base64` in yaml:

```shell
echo -n VALUE | base64
echo -n ENOCED_VALUE | base64 --decode
```

Secret:

```
APP_COLOR: BASE64_ENCODED
APP_MODE: BASE64_ENCODED
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```shell
kubectl get secrets
kubectl get secret CONFIG_MAP
kubectl describe secret generic CONFIG_MAP
```

Just like configmap use envFrom:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
 containers:
 - name: simple-webapp-color
   image: simple-webapp-color
   ports:
   - containerPort: 8080
   envFrom:
   - secretKeyRef:
       name: app-config
```

Can use `EncryptionConfiguration` to encrypt secrets at rest (stall accessible by users with access to pods)

## Encrypting

We can encrypt at rest in `etcd`

We can query `etcd` with `etcdctl`:

[Encrypting Confidential Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

Check if `--encryption-provider-config` set in `kube-apiserver`:

(kubeadm):

```shell
less /etc/kubernetes/manifests/kube-apiserver.yaml
```

Create `EncryptionConfiguration` (see docs), and pass via `--encryption-provider-config`

## Init Containers

If you only wish to run something at initialization in a multi-container pod, use an `initContainer`, they work just like regular containers but exit.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone  ;']
```

`initContainer`s must run to completion before the other container start. They run in sequential order.



