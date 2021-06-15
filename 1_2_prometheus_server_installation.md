# 1.2 Prometheus Server Installation
## Taint a node work prometheus only
```
kubectl taint node server-aras-yorganci-1 monitoring:PreferNoSchedule
```

## Create namespace for monitoring
```
kubectl create namespace monitoring
```

## Download helm binary
```
wget https://get.helm.sh/helm-v3.6.0-linux-amd64.tar.gz
tar -xf helm-v3.6.0-linux-amd64.tar.gz
cp -a linux-amd64/helm /usr/bin/helm
```

## Add helm repo and download chart for prometheus
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm pull prometheus-community/prometheus
tar -xf prometheus-14.1.1.tgz
```

## Configure prometheus helm chart values
```
cd $HOME/prometheus/
cp -a values.yaml values-orig.yaml

diff values.yaml values-orig.yaml
33c33
<   enabled: false
---
>   enabled: true
348c348
<     enabled: false
---
>     enabled: true
425c425
<   enabled: false
---
>   enabled: true
792,795c792,796
<   tolerations:
<     - key: "monitoring"
<       operator: "Exists"
<       effect: "PreferNoSchedule"
---
>   tolerations: []
>     # - key: "key"
>     #   operator: "Equal|Exists"
>     #   value: "value"
>     #   effect: "NoSchedule|PreferNoSchedule|NoExecute(1.6 only)"
822c823
<     enabled: false
---
>     enabled: true
1032c1033
<   enabled: false
---
>   enabled: true
```

## Install prometheus helm chart with new values
```
helm install -f $HOME/prometheus/values.yaml prometheus $HOME/prometheus --namespace monitoring
```

