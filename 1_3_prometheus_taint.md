# 1.3 Prometheus Node Taint and Tolerations

With taints and tolerations at previous commands, there will be no other deployments run on server-1

```
kubectl describe nodes server-aras-yorganci-1
...
Taints:             monitoring:PreferNoSchedule
...
```

```
kubectl -n monitoring describe pod prometheus-server-7f45cbf8d5-vkkq7
...
Tolerations:                 monitoring:PreferNoSchedule op=Exists
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
...
```
