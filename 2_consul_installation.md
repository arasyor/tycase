# 2. Single Node Consul Installation
## Updating system
```
yum update -y
reboot
```

## Add Hashicorp repository to server and install Consul
```
yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y install consul
systemctl enable --now consul
```

