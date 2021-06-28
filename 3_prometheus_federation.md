# 3. Prometheus Federation
## Download prometheus server and extract
```
wget https://github.com/prometheus/prometheus/releases/download/v2.28.0/prometheus-2.28.0.linux-amd64.tar.gz
tar -xf prometheus-2.28.0.linux-amd64.tar.gz
```

## Edit prometheus.yaml for federation with consul
```
cd $HOME/prometheus-2.28.0.linux-amd64
cp -a prometheus{.yml,-orig.yml}

cat <<EOF | tee prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
  - static_configs:
    - targets:
       - localhost:9093

rule_files:
  - ./*-k8s.rules

scrape_configs:
  - job_name: 'federate'
    scrape_interval: 15s

    honor_labels: true
    metrics_path: '/federate'

    params:
      'match[]':
        - '{job=~".+"}'

    consul_sd_configs:
    - server: server-aras-yorganci-2:8500
EOF
```

## Run prometheus server with lifecycle
```
nohup ./prometheus --web.enable-lifecycle &
```
