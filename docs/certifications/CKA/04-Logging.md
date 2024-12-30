# Logging

## Monitoring

Can have one metrics server per cluster (built-in in memory)

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

* Will poll and record metrics

```shell
kubectl top node
kubectl top pod
```

or

Use alternatives like prometheus etc

## Application Logs

```
kubectl logs -f PODNAME [ CONTAINERNAME ]
```

