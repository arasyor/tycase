# 5. Alertmanager Installation
## Download prometheus/alertmanager and extract
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.22.2/alertmanager-0.22.2.linux-amd64.tar.gz
tar -xf alertmanager-0.22.2.linux-amd64.tar.gz
```

## Run alertmanager
```
nohup ./alertmanager &
```

## Add pod restart rule for server-0 cluster
```
cd $HOME/prometheus-2.28.0.linux-amd64

cat<<EOF | tee server-0-k8s.rules
groups:
- name: Pod Alerts
  rules:
  - alert: Pod Restart Alert
    expr: rate(kube_pod_container_status_restarts_total[1h]) * 3600 > 1
    for: 1m
    labels:
      severity: "critical"
    annotations:
      summary: "Pod {{$labels.namespace}}/{{$labels.pod}} restarting more than once during last one hours."
EOF
```

## Reload prometheus on server-2
```
curl -X POST http://localhost:9090/-/reload
```
