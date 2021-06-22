# 1.4 Ingress Controller
## Add helm repo and download chart for haproxy
```
helm repo add haproxytech https://haproxytech.github.io/helm-charts
helm repo update
helm pull haproxytech/kubernetes-ingress
tar -xf kubernetes-ingress-1.15.2.tgz
```

## Configure haproxy helm chart values
```
cd $HOME/kubernetes-ingress

55c55
<   kind: DaemonSet    # can be 'Deployment' or 'DaemonSet'
---
>   kind: Deployment    # can be 'Deployment' or 'DaemonSet'
130c130
<   ingressClass: haproxy   # typically "haproxy" or null to receive all events
---
>   ingressClass: null   # typically "haproxy" or null to receive all events
273c273
<     type: LoadBalancer    # can be 'NodePort' or 'LoadBalancer'
---
>     type: NodePort    # can be 'NodePort' or 'LoadBalancer'
354c354
<     useHostNetwork: true  # also modify dnsPolicy accordingly
---
>     useHostNetwork: false  # also modify dnsPolicy accordingly
```

## Install haproxy helm chart with new values
```
helm install -f $HOME/kubernetes-ingress/values.yaml \
kubernetes-ingress $HOME/kubernetes-ingress \
--create-namespace --namespace ingress-controller
```

## Apply prometheus ingress
```
cat prometheus-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: haproxy
  name: prometheus-ingress
  namespace: monitoring
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: prometheus-server
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
```
```
kubectl apply -f prometheus-ingress.yaml   
```
