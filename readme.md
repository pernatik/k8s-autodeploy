Infrastructure:
```
k8s-lb01       192.168.0.130
k8s-master01   129.168.0.131
k8s-master02   192.168.0.132
k8s-master03   192.168.0.133
k8s-node01     192.168.0.134
k8s-node02     192.168.0.135
k8s-node03     192.168.0.136
```

Подготовка VM Ubuntu 20.04

```
apt update && apt upgrade -y

sudo mkdir -m 0755 -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg


echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

chmod a+r /etc/apt/keyrings/docker.gpg

sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

apt update

sudo apt-get install -y kubelet kubeadm kubectl docker-ce

```