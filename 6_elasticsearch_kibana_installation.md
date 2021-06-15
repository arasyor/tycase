# 6. Elasticsearch & Kibana Installation
## Updating system
```
yum update -y
reboot
```

## Add erepos and install packages
```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch

cat <<EOF | tee /etc/yum.repos.d/elasticsearch.repo
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
EOF

cat <<EOF | tee /etc/yum.repos.d/kibana.repo
[kibana-7.x]
name=Kibana repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF

yum install -y elasticsearch kibana
```

## Configure elasticsearch and start service
```
cp -a /etc/elasticsearch/elasticsearch{.yml,orig.yml}

diff /etc/elasticsearch/elasticsearch{.yml,orig.yml}
23c23
< node.name: server-aras-yorganci-3
---
> #node.name: node-1
56c56
< network.host: 0.0.0.0
---
> #network.host: 192.168.0.1
74c74
< cluster.initial_master_nodes: ["server-aras-yorganci-3"]
---
> #cluster.initial_master_nodes: ["node-1", "node-2"]

systemctl enable --now elasticsearch
```

## Configure kibana and start service
```
cp -a /etc/kibana/kibana{.yml,-orig.yml}

diff /etc/kibana/kibana{.yml,-orig.yml}
7c7
< server.host: "0.0.0.0"
---
> #server.host: "localhost"
29c29
< server.name: "server-aras-yorganci-3"
---
> #server.name: "your-hostname"

systemctl enable --now kibana
```

## Apply filebeat daemonset on k8s cluster for kubernetes pod logs
```
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.13/deploy/kubernetes/filebeat-kubernetes.yaml

cp -a filebeat-kubernetes.yaml filebeat-kubernetes-orig.yaml

diff filebeat-kubernetes.yaml filebeat-kubernetes-orig.yaml
74c74
<           value: 192.168.6.43
---
>           value: elasticsearch

kubectl apply -f filebeat-kubernetes.yaml
```

## Kibana dashboard config for kubernetes logs
```
Stack Management -> Index Patterns -> filebeat-*
```

