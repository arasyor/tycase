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
```

## Single node cluster configurtion for consul
```
cp -a /etc/consul.d/consul.hcl{,.orig}

diff /etc/consul.d/consul.hcl{,.orig}
40c40
< server = true
---
> #server = true
49c49
< bootstrap_expect=1
---
> #bootstrap_expect=3

systemctl enable --now consul
```