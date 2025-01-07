# Security

## Authentication

* Users (humans)
* Service Accounts (Machines)

User access is via `kube-apiserver`

File-based:

* Static password file
* Static token file

Simple, but insecure

Static pass file:

```csv
password123,user1,u0001
password123,user2,u0002
```

Add to `kube-apiserver` command (likely in pod):

```
--basic-auth-file=/tmp/users/user-details.csv
```

Next create `Role` and `RoleBinding`

## TLS

Asymmetric Encryption:

Public "lock"
Private "key"

Lock a resource - eg `.ssh/authorized_keys`

We need a way to securely transfer keys to the server so we can "unlock"

We encrypt the private key before sending using the servers public key. Then we send and server can get the key

We need to "certify" server is who it says it is.

We use a CA for signing with a CSR

## TLS in Kubernetes

Thee kinds of certificate we will consider:

* Root
* Client
* Server

Naming conventions:

Usually Public keys are `.crt` and `.pem`

Private keys in `.key` and `-key.pem`

On k8s all servers and clients need client or server certificates (depending on function).

kube-api and etcd need server cert, rest client

We need a CA for creating these certs.

## Generating Certificates

If using openssl:

### Setup CA

* Generate Keys

  ```shell
  openssl genrsa -out ca.key 2048
  ```

* Generate CSR

  ```shell
  openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
  ```

* Sign certificates

  ```shell
  openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  ```

### Generating Client Certificates

#### Admin User Certificates

* Generate Keys

  ```shell
  openssl genrsa -out admin.key 2048
  ```

* Generate CSR (CN just for logs)

  ```shell
  openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr
  ```

* Sign certificates

  ```shell
  openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
  ```

* Certificate with admin privilages

  ```shell
  openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
  ```

Viewing certs:

```shell
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

[Cert info](https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools)

## Certificate API

Rather than manually signing new certs we can use API.

Use can request a cert signed.

EG, jane requests:

```shell
openssl genrsa -out jane.key 2048
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```

```yaml
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: jane
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request:
    <certificate-goes-here>
```

Then send to `kubectl` `base64 encode in file:

```shell
cat jane.csr | base64 -w 0 # single line
kubectl create -f jane.yaml
```

```shell
kubectl get csr
kubectl certificate approve jane
kubectl get csr jane -o yaml
echo "<certificate>" |base64 --decode
```

All this is completed by `controller-manager`

## KubeConfig

We need to use cert with `kubectl` on every call. Rathen than CLI, we put in KubeConfig:

Default: `$HOME/.kube/config`

Three sections:

* Clusters
* Contexts
* Users

Context groups Cluster and Context.

EG: user: `Admin`, cluster `AWS`, context: `Admin@AWS`

`current-context` is default.

Current config:

```shell
kubectl config view
```

```shell
kubectl config use-context user@cluster
```

We can put namespaces in context if we want.

[Configure Access to Multiple Clusters](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

## API Groups

The api has multiple API groups.

Core functionality is in `/api` (`/api/v1`) (secrets, pods, svc etc..)

Named groups are more heirarchical (in future API changes here)

Groups are shown in docs

EG: `/apis/apps/v1/{deployments, replicasets, statefulsets}`

Can use `curl` to see groups:

```shell
curl http://localhost:6443 -k \
    --key=admin.key \
    --cert=admin.crt \
    --cacert=ca.crt
```

Can use:

```shell
kubectl proxy
```

To avoid needing to specify certs (sets up listener with auth)

## Authorization

_What can I do with access?_

We create accounts, then authorize certain things. Usually done via namespaces.

### Authorization types

**Node Authorizer:**

Used by `kubelets`

User in this group by adding cert `system:node` prefix on cert


**ABAC Authorizer:**

Associate user with a permission:

eg "view pod"

Requires restarting API server after perm change.

**RBAC Authorizer:**

Assign perms to a role

Associate users with the role.

**Webhook:** Outsource to third party (eg Open Policy Agent)

**AlwaysAllow:** The default.

**AlwaysDeny:** Does what is says.

If you specify multiple `--authorization-mode=Node,RBAC`

It tries all in order then allows if no match.

## RBAC

Create `kind: Role` with `apiVersion: rbac.authorization.k8s.io/v1`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "list", "update", "delete", "create"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```

Now create `kind: RoleBinding` and link user to role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

```shell
kubectl get roles
kubectl get rolebindings
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding
```

Can check `can-i`:

```shell
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
kubectl auth can-i create deployments --as dev-user
kubectl auth can-i create pods --as dev-user
kubectl auth can-i create pods --as dev-user --namespace test
```

When in a namespace we can further restrict via `resourceNames`

* [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

## Cluster Roles

Some resources are cluster-wide

We can do cluster-wide roles via `ClusterRole`, `ClusterRoleBinding`

Very similar to `Role`

Can view api groups via:

```shell
kubectl api-resources
```

## Service Accounts

Used for machine access (eg an application) to `kube-api`

```shell
kubectl create serviceaccount
```

Creates a token used to connect.

Token is a `secret`, use:

```shell
kubectl describe secret
```

Token can be used as a "Bearer" eg curl.

Can use RBAC with the service account.

If hosting an app on K8S we can expose the secret as a volume.
By default the default namespace secret is exposed (only basic k8s access).

`>=` v1.22 `TokenRequestAPI` creates token with expiry, bound to a pod.
See projected volume mount.

`>=` v1.24 service accounts dont create tokens by default. And have expiry.

To use legacy (no binding, no expiry) use [ServiceAccount token Secrets](https://kubernetes.io/docs/concepts/configuration/secret/#serviceaccount-token-secrets)

## Image Security

In `image` names, it has implicit account `library` on dockerhub if not specified.

`image: REGISTRY/USER/NAME`

We can use a private registry too. We need to pass auth to CRI.

```shell
kubectl create secret docker-registry regcred \
  --docker-server=private-registry.io \
  --docker-username=registry-user \
  --docker-password=registry-password \
  --docker-email=registry-user@org.com
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: regcred
```

## Security Contexts

We can set capabilities at container granularity.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```

securitycontext can be at pod granularity, but no capabilities.

* [Configure a Security Context for a Pod or Container](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/)

## Network Policies

Ingress: inbound
Egress: outbound

By defaults k8s has `AllAllow` on communication between pods.

We implement a network policy to restrict traffic.

We use labels and selectors for policies.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

Not all solutions support policies.

If there are multiple items in `from:`, matching 1 allows.

