# Storage

## Volumes

Allows for persistent data.

Use `spec.containers[*].volumeMounts` and `spec.volumes`:

[Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

With basic `hostPath`, data is stored directly on EACH node, not shared.

Various volume types exist we can use.

## PersistentVolumes

Lets us store data centrally in a pool.

We then claim the data with a persistent volume claim (PVC)

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-vol1
spec:
  accessModes: [ "ReadWriteOnce" ]
  capacity:
   storage: 1Gi
  hostPath:
   path: /tmp/data
```

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

## Persistent Volume Claims

Looks for matching claims.

We can select one ourselves with labels and selectors.

PV and PVC are one to one

If one cannot be matched with the required resources the PVC will stay in pending state

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  resources:
   requests:
     storage: 1Gi
```

Once PVC deleted we can choose to automatically delete the underlying PV, retain it, or recycle it.

[Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes%5C)


## Storage Classes

Provisioner dynamically provisions when we need storage.

```yaml
sc-definition.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: google-storage
provisioner: kubernetes.io/gce-pd
```

Creates a pv for us automatically when we create a claim if we associate it:

```yaml
pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: google-storage
  resources:
   requests:
     storage: 500Mi
```

* [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)

