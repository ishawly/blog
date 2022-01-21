---
title: 虚拟机配置 kubernetes 集群
date: 2022-01-12 21:00:00
description: 众所周知，kubernetes(文中简称k8s)已成为容器编排的首选工具，作为学习使用，便在虚拟机中通过`kubeadm`配置集群。本文主要参考官方文档及相关网络文章。
categories:
- 服务器
tags:
- kubernetes
- k8s
---

# 参考

- [Installing kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [Configuring a cgroup driver](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/)
- [Creating a cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
- [kubernetes安装（国内环境）](https://zhuanlan.zhihu.com/p/46341911)
- [在国内如何顺利地安装k8s](https://zhuanlan.zhihu.com/p/270952956)
- [Installing Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)
- [安装Kubernetes(k8s)保姆级教程---无坑版](https://www.cnblogs.com/Sunzz/p/15184167.html)，该文档中验证网络部分，若遇到不能 ping kubernetes.default 是因为 busybox 版本所致，可以参考：[dns can't resolve kubernetes.default and/or cluster.local](https://github.com/kubernetes/kubernetes/issues/66924)

# 环境配置

- VirtualBox 6.0
- Ubuntu 20.04(三台，一台做master，两台做node。后面称主机或节点)
- Kubernetes 1.23(该版本当前最新稳定版) 

# 安装 kubeadm

## 准备工作

- 内存 >= 2GB
- CPU核数 >= 2
- 节点之间完整的网络连接
- [每个虚拟机唯一的主机名，MAC地址，product_uuid。](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-mac-address)
- [虚拟机上特定的端口开发且能访问](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)
- 禁用swap（某些机器可能一开始就没开启）

### 禁用swap

1. 临时禁用`sudo swapoff -a`

2. 永久禁用，在`/etc/fstab`中注释swap相关行

## 让 iptables 能发现桥接的网络流量

使用`lsmod | grep br_netfilter` 命令确保`br_netfilter`模块已经加载。若无则执行`sudo modprobe br_netfilter`。

要让节点能发现桥接的网络流量，确保在`sysctl`的配置中已将`net.bridge.bridge-nf-call-iptables`设置为1。

```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```
 
## 安装运行时 

这里使用 Docker 作为 Pods 里容器的运行时，安装参考[https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)。

## 安装 kubeadm, kubelet, kubectl

由于网络问题将安装源改用阿里云，其他国内源亦可。

```
sudo apt-get update && sudo apt-get install -y apt-transport-https

curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/aliyun-archive-keyring.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/aliyun-archive-keyring.gpg] https://mirrors.aliyun.com/kubernetes/apt/ \
  kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null

# 注意 kubernetes-xenial main 不是 $(lsb_release -cs) stable

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

```

## 配置 cgroup driver

容器运行时和 kubelet 的 `cgroup driver` 需配置一致，否则将导致后续步骤失败。

发生的错误之一如下：

```
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
```

该问题修复如下：

```
# 修改 Docker 配置
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
    },
    "storage-driver": "overlay2"
}
EOF

# 重启服务
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 重置，若未执行后续的步骤，不执行该命令
sudo kubeadm reset
```

# 使用 kubeadm 创建集群

## master 节点初始化

```
sudo kubeadm init \
  --apiserver-advertise-address=0.0.0.0 \
  --service-cidr=10.96.0.0/16 \
  --pod-network-cidr=10.245.0.0/16 \
  --image-repository registry.aliyuncs.com/google_containers

# --kubernetes-version v1.18.8 指定版本
# --apiserver-advertise-address 为通告给其它组件的IP，一般应为master节点的IP地址
# --service-cidr 指定service网络，不能和node网络冲突
# --pod-network-cidr 指定pod网络，不能和node网络、service网络冲突
# --image-repository registry.aliyuncs.com/google_containers 指定镜像源，由于默认拉取镜像地址k8s.gcr.io国内无法访问，
#   这里指定阿里云镜像仓库地址。如果k8s版本比较新，可能阿里云没有对应的镜像，就需要自己从其它地方获取镜像了。
# --control-plane-endpoint 标志应该被设置成负载均衡器的地址或 DNS 和端口(可选) 


# 显示 Your Kubernetes control-plane has initialized successfully! 表示初始化成功。按照提示（最好保存下）执行后续步骤。
```

## 配置 kubectl

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 配置集群的 pod 网络

> 这里使用 Flannel，更多请参考[https://kubernetes.io/docs/concepts/cluster-administration/addons/](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

1. 下载 yaml 文件
```
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

2. 更改128行的网络配置，和上面`--pod-network-cidr`参数中保持一致

```
net-conf.json: |
    {
        "Network": "10.245.0.0/16",
        "Backend": {
        "Type": "vxlan"
        }
    }
```

3. 执行 yml 文件 `kubectl apply -f kube-flannel.yaml`，执行结果如下：

```
Warning: policy/v1beta1 PodSecurityPolicy is deprecated in v1.21+, unavailable in v1.25+
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created
```

4. 查看 Flannel 部署结果 `kubectl -n kube-system get pods -o wide`，当前只有一个节点，显示如下：

```
kube-flannel-ds-s4l5g       1/1     Running   0          2m44s   192.168.64.5   a      <none>           <none>
```

## 将其他节点加入集群

在其他节点执行初始化提示里的`kubeadm join`命令。

查看节点 `kubectl get node`显示如下：

```
ubuntu@a:~$ kubectl get node
NAME   STATUS   ROLES                  AGE   VERSION
a      Ready    control-plane,master   32m   v1.23.2
```

> 若只有一台机器来配置集群，需要让 control-plane 节点也能调度 Pods，则需执行如下命令

```
kubectl taint nodes --all node-role.kubernetes.io/master-

# node/a untainted
```

## 部署一个 Deployment

busybox.yml

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
spec:
  replicas: 2
  selector:
    matchLabels:
      name: busybox
  template:
    metadata:
      labels:
        name: busybox
    spec:
      containers:
      - name: busybox
        image: busybox  
        imagePullPolicy: IfNotPresent
        args:
        - /bin/sh
        - -c
        - sleep 1; touch /tmp/healthy; sleep 30000
        readinessProbe:   
          exec:
            command:
            - cat
            - /tmp/healthy
          initialDelaySeconds: 1
```

执行命令

```
kubectl apply -f busybox.yml

# deployment.apps/busybox created
```

查看 Pods

```
kubectl get pods

NAME                       READY   STATUS    RESTARTS   AGE
busybox-84d749f7cc-52mkq   1/1     Running   0          27s
busybox-84d749f7cc-g4dvl   1/1     Running   0          27s
```