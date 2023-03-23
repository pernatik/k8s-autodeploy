## Что я делаю
Подготавливаю ВМ с Ubuntu 22.04.xx для k8s (на данный моментставится версия 1.26.3). 


## Пререквизиты:

1. Со стороны инфроструктуры
* Должны быть записи в DNS для ВМ. 

2. На ВМ, с которой выполняется установка
* должен стоять ansible 
```
apt install ansible
```
* должен стоять sshpass
```
apt install sshpass
```
3. На ВМ, где будем развертывать k8s
* Должен быть пользователь с домашней папкой и правами SUDO
```
useradd -m ansible --shell /bin/bash
passwd ansible
usermod -aG sudo ansible
```

## My infrastructure:
```
lb1.k8s.lo     192.168.0.140
cp1.k8s.lo     129.168.0.141
cp2.k8s.lo     192.168.0.142
cp3.k8s.lo     192.168.0.143
wn1.k8s.lo     192.168.0.144
wn2.k8s.lo     192.168.0.145
wn3.k8s.lo     192.168.0.146
```
Где,

lb - load balancer,
cp - control plane,
wn - worker node.



## Запуск 

Редактируем inventory под себя

Затем запускаем на ВМ с ansible
```
ansible-playbook install.yaml -i inventory \
--extra-vars "ansible_user=ansible ansible_password=<пароль> ansible_sudo_pass=<пароль>"
```
Как ansible отработает, ВМ перезагрузятся. После этого можно запускать kubeadm init

## Создание кластера

Запускаем на cp1:
```
kubeadm init --control-plane-endpoint "cp1.k8s.lo:6443" --upload-certs
```

Получем вывод:

```
To start using your cluster, you need to run the following as a regular user:
 
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
 
Alternatively, if you are the root user, you can run:
 
  export KUBECONFIG=/etc/kubernetes/admin.conf
 
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
 
You can now join any number of the control-plane node running the following command on each as root:
 
  kubeadm join cp1.k8s.lo:6443 --token 79itry.l0f4yaedzzmf35j3 \
        --discovery-token-ca-cert-hash sha256:cf99d19f6696bfae7db18d442a6465206d054fc0fafe63831cb91bcd47e50480 \
        --control-plane --certificate-key f0a532e256c3cead1dcf75797984347013df8b0477d17a4df5819973048630e8
 
Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
 
Then you can join any number of worker nodes by running the following on each as root:
 
kubeadm join cp1.k8s.lo:6443 --token 79itry.l0f4yaedzzmf35j3 \
        --discovery-token-ca-cert-hash sha256:cf99d19f6696bfae7db18d442a6465206d054fc0fafe63831cb91bcd47e50480
```
## lb1-k8s.lo
Для "отказоустойчивости" можно доустановить балансировщик lb1-k8s.lo для контрол плейнов:

```
apt install -y nginx
systemctl enable nginx
systemctl status nginx
mkdir -p /etc/nginx/tcpconf.d
nano /etc/nginx/nginx.conf
--- добавть строчку >
include /etc/nginx/tcpconf.d/*;
--->

```
add nginx conf --> /etc/nginx/tcpconf.d/kubernates.conf
```
stream { 
  upstream kubernates { 
    server 192.168.0.141:6443; 
    server 192.168.0.142:6443; 
    server 192.168.0.143:6443; 
  } 
  server { 
    listen 6443; 
    proxy_pass kubernates; 
  } 
}
```
рестарт nginx:
```
nginx -s reload
```
