# 4. Grafana Installation
## Add grafana repo and install on server-2
```
cat <<EOF | tee /etc/yum.repos.d/grafana.repo
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF

yum install -y grafana

systemctl enable --now grafana-server
```

## Add server-0 IP address to server-2 hosts file
```
echo "192.168.6.41    server-aras-yorganci-0  server-aras-yorganci-0" >> /etc/hosts
```

## Add prometheus data source to grafana
```
Configuration -> Add data source -> Prometheus -> URL http://server-aras-yorganci-0
```

## Add grafana dashboard for k8s cluster monitoring
```
Dashboards -> Manage -> Import -> Dashboard ID 10000 Load
https://grafana.com/grafana/dashboards/10000
```
