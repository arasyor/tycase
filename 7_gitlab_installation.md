# 6. Gitlab Installation
## Download repo and install Gitlab repo
```
curl -O https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh 
sh script.rpm.sh 

yum -y install gitlab-ce

gitlab-ctl reconfigure 
```

## Default user and password
```
Default admin account has been configured with following details:
Username: root
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.
```

## Install Gitlab runner and register for server-2
Registritation code can be obtained from http://server-aras-yorganci-3/root/tycase/-/settings/ci_cd#js-runners-settings
```
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash

yum install -y gitlab-runner

gitlab-runner register

Runtime platform                                    arch=amd64 os=linux pid=18712 revision=c1edb478 version=14.0.1
Running in system-mode.

Enter the GitLab instance URL (for example, https://gitlab.com/):
http://server-aras-yorganci-3/
Enter the registration token:
37fGG9UQJPqpJFp52gUr
Enter a description for the runner:
[server-aras-yorganci-3]:
Enter tags for the runner (comma-separated):

Registering runner... succeeded                     runner=37fGG9UQ
Enter an executor: docker, shell, docker+machine, docker-ssh+machine, custom, docker-ssh, parallels, ssh, virtualbox, kubernetes:
ssh
Enter the SSH server address (for example, my.server.com):
server-aras-yorganci-2
Enter the SSH server port (for example, 22):
22
Enter the SSH user (for example, root):
root
Enter the SSH password (for example, docker.io):
docker.io
Enter the path to the SSH identity file (for example, /home/user/.ssh/id_rsa):
/root/.ssh/Tycase.pem
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

## Server-2 IP and private key should be on server-3
```
echo "192.168.6.44    server-aras-yorganci-2  server-aras-yorganci-2" >> /etc/hosts

ls -l $HOME/.ssh/Tycase.pem
-rw------- 1 root root 1679 Jun 29 03:09 /root/.ssh/Tycase.pem

chmod 600 $HOME/.ssh/Tycase.pem
```

## Server-3 IP should be on server-2
```
echo "192.168.6.43    server-aras-yorganci-3  server-aras-yorganci-3" >> /etc/hosts
```

## Gitlab pipeline script below
```
cat .gitlab-ci.yml

variables:
  K8S_CLUSTER: "server-11"

stages:
  - prometheus-consul-registry
  - alertmanager-pod-restart

consul-job:       
  stage: prometheus-consul-registry
  script:
    - |
      cat <<EOF | tee /etc/consul.d/prometheus-$K8S_CLUSTER.json
      {
        "service": {
        "address": "$K8S_CLUSTER",
        "name": "prometheus-$K8S_CLUSTER",
        "tags": [
          "prometheus-$K8S_CLUSTER"
        ],
        "port": 80
        }
      }
      EOF
    - consul reload

alertmanager-job:
  stage: alertmanager-pod-restart
  script:
    - cd $HOME/prometheus-2.28.0.linux-amd64
    - |
      cat <<EOF | tee $K8S_CLUSTER-k8s.rules
      groups:
      - name: Pod Alerts - $K8S_CLUSTER
        rules:
        - alert: Pod Restart Alert - $K8S_CLUSTER
          expr: rate(kube_pod_container_status_restarts_total[1h]) * 3600 > 1
          for: 1m
          labels:
            severity: "critical"
          annotations:
            summary: "Pod {{$labels.namespace}}/{{$labels.pod}} restarting more than once during last one hours."
      EOF
    - curl -X POST http://localhost:9090/-/reload
```
Pipeline can be accessed from http://server-aras-yorganci-3/root/tycase/-/pipelines with K8S_CLUSTER parameter for cluster address.