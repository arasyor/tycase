# 1.1 Kubernetes Installation
## Updating system
```
yum update -y
reboot
```

## Installing docker-ce
```
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl enable --now docker
```

## Installing kubeadm, kubelet and kubectl
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```

## Add master node IP to hosts file on all nodes
```
echo "192.168.6.41 k8smaster" >> /etc/hosts
```

## Cluster configuration yaml file
```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: 1.21.1
controlPlaneEndpoint: "k8smaster:6443"
networking:
  podSubnet: 10.244.0.0/16
clusterName: "case-aras-yorganci.abc"
```

## Master node initialization
```
kubeadm init --config=kubeadm-config.yaml --upload-certs | tee kubeadm-init.out

mkdir -p $HOME/.kube                                    
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config         
```

## Configure Pod Network with Flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 
```

## Join second node
```
kubeadm join k8smaster:6443 --token l6i3ua.xj7lhkryigvzqvy4 \
        --discovery-token-ca-cert-hash sha256:6267422a3833ace5034d587227f55888f3675faaea70ae9c9eaec482c941f6a4
```

## Remove taint from master for scheduling
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

# 1.1 Prometheus Server Installation
## Taint a node work prometheus only
```
kubectl taint node server-aras-yorganci-1 monitoring:PreferNoSchedule
```

## Run prometheus