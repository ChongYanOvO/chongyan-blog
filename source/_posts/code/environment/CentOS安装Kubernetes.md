---
title: CentOS安装Kubernetes
categories: 环境配置
tags:
  - Kubernetes
  - CentOS
ai: true
cover: 'https://bu.dusays.com/2023/08/09/64d36176c3f02.png'
abbrlink: dac5bf30
date: 2023-08-09 17:46:53
---
# Kubernetes安装

## Docker安装

### **移除以前docker相关包**

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### **配置yum源**

```bash
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### **安装docker**

```bash
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6
```

### **启动**

```bash
systemctl enable docker --now
```

### **配置加速**

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Kubernetes安装

### 基础环境安装

```bash
# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### **安装kubelet、kubeadm、kubectl**

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF


sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

### **下载各个机器需要的镜像**

```bash
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
EOF
   
chmod +x ./images.sh && ./images.sh
```

### **初始化主节点**

**注意**: 这里`apiserver-advertise-address`和`control-plane-endpoint`要填 master 节点的 IP 和域名映射

```bash
kubeadm init \
--apiserver-advertise-address=10.211.55.60 \
--control-plane-endpoint=master \
--image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
```

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join master:6443 --token vixrtg.vwk0b82x13274in2 \
    --discovery-token-ca-cert-hash sha256:7e429a113c382ed553c0b865953cc767f9af9c2cfe0be0f8170c5572cb12f10a \
    --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join master:6443 --token vixrtg.vwk0b82x13274in2 \
    --discovery-token-ca-cert-hash sha256:7e429a113c382ed553c0b865953cc767f9af9c2cfe0be0f8170c5572cb12f10a
```

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### **安装网络组件**

```bash
curl https://docs.projectcalico.org/v3.20/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

### **加入node节点**

```bash
kubeadm join master:6443 --token vixrtg.vwk0b82x13274in2 \
    --discovery-token-ca-cert-hash sha256:7e429a113c382ed553c0b865953cc767f9af9c2cfe0be0f8170c5572cb12f10a
```

### 创建新令牌

```bash
kubeadm token create --print-join-command
```

### 查看安装结果

```shell
[root@kubernetes-master ~]# kubectl get pods -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-577f77cb5c-825qj    1/1     Running   0          12m
kube-system   calico-node-fsbbk                           1/1     Running   0          30m
kube-system   calico-node-n7jps                           1/1     Running   0          10m
kube-system   calico-node-w5jvg                           1/1     Running   0          10m
kube-system   coredns-54d67798b7-65mbk                    1/1     Running   0          55m
kube-system   coredns-54d67798b7-vxsns                    1/1     Running   0          55m
kube-system   etcd-kubernetes-master                      1/1     Running   0          55m
kube-system   kube-apiserver-kubernetes-master            1/1     Running   0          55m
kube-system   kube-controller-manager-kubernetes-master   1/1     Running   0          55m
kube-system   kube-proxy-cmkz6                            1/1     Running   0          55m
kube-system   kube-proxy-ftf52                            1/1     Running   0          10m
kube-system   kube-proxy-xxctv                            1/1     Running   0          10m
kube-system   kube-scheduler-kubernetes-master            1/1     Running   0          55m

[root@kubernetes-master ~]# kubectl get nodes
NAME                STATUS   ROLES                  AGE   VERSION
kubernetes-master   Ready    control-plane,master   55m   v1.20.9
kubernetes-slave1   Ready    <none>                 10m   v1.20.9
kubernetes-slave2   Ready    <none>                 10m   v1.20.9
```

