---
title: "通过kubeadm安装k8s"
date: 2021-01-16T18:38:09+08:00
lastmod: 2021-01-16T18:38:09+08:00
draft: true
keywords: ["k8s"]
description: "install k8s with kubeadm"
tags: ["sre","k8s","kubeadm"]
categories: ["笔记"]
author: ""

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: false
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---



> 如何在国内公有云服务器通过`kubeadm`部署`kubernetes`

通过kubeadm我们可以很方便的创建GA的kubernetes集群，国内通过该方式安装的过程中我们通常会因为网络问题导致一些镜像包无法下载。此处记录一下如何通过修改镜像源地址来正常安装。本次测试我们适用国内任一云厂商的三台虚拟机，操作系统为Ubuntu20.04，安装kubernetes 1.18.15
<!--more-->

#准备工作
国内云厂商直接申请三台虚拟机，这些机器需要满足以下几个条件即可
1. 满足安装Docker项目的需求，比如64未Linux操作系统、3.10及以上的内核版本
2. x86或者ARM架构均可
3. 机器之间网络互通，这是将来容器之间网络互通的前提（虚拟机处于同一个vpc中，设置正确的的安全组策略）
4. 有外部网络访问权限，需要拉取镜像
5. 单机可用资源建议2核CPU、8GB内存
6. 30GB或以上可用磁盘空间

本次部署中，准备机器配置如下
1. 4核心CPU、8GB内存  数量3
2. 80GB磁盘

4. 内网互通
5. 外网访问权限不受限制

本次部署中，系统及软件版本为
1. 系统 Ubuntu20.04
2. kubeadm 1.18.15-00
3. kubernetes 1.18.15
4. docker 19.03.14


本次部署，机器网络IP与host为
10.0.5.100  master
10.0.5.101  worker1
10.0.5.102  worker2

此次安装部署的步骤
1. 在所有节点安装 Docker 与 kubeadm
2. 部署 kubernetes Master
3. 部署容器网络插件
4. 部署 Kubernetes Worker
5. 部署 Dashboard 可视化插件
6. 部署容器存储插件
7. 部署 rancher


# 安装kubeadm 和 Docker

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system

apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors": [
		"https://hub-mirror.c.163.com",
		"https://mirror.baidubce.com"
	]
}
EOF
systemctl daemon-reload
systemctl restart docker

apt-get update
sudo apt-get install -qy kubelet=1.18.15-00 kubectl=1.18.15-00 kubeadm=1.18.15-00
sudo apt-mark hold kubelet=1.18.15-00 kubectl=1.18.15-00 kubeadm=1.18.15-00
```

在上述的安装过程中，我们安装了 kubeadm 和 kubelet、kubectl、kubernetes-cni几个二进制文件，同时也安装了docker

# 部署 Kubernetes 的 Master 节点

此处我们自定义一个kubeadm的初始化配置文件： kubeadm.yaml

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
controllerManager:
  extraArgs:
    horizontal-pod-autoscaler-use-rest-clients: "true"
    horizontal-pod-autoscaler-sync-period: "10s"
    node-monitor-grace-period: "10s"
apiServer:
  timeoutForControlPlane: 4m0s
  extraArgs:
    runtime-config: "api/all=true"
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: "stable-1.18"
```

然后，我们需要执行一句安装指令

```shell
kubeadm init --config kubeadm.yaml
```
此时，就完成了kubernetes master 的部署，部署完成之后，kubeadm 生成一条指令

```shell
kubeadm join 10.0.5.100:6443 --token 0cjl0p.ints413j578iume0 \
--discovery-token-ca-cert-hash sha256:40054faa2be39c3e1932a2bba5e72643b5818413ff214faa3acf5c5ef5159896
```

这个 kubeadm join 指令，用于给master节点加入更多工作节点。
此外，Kubeadm提示我们第一次使用 kubernetes 集群需要的配置命令


```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

# 部署网络插件
部署网络插件非常方便，我们通过kubectl apply来部署Weave网络插件

```shell
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

部署 Kubernetes的Worker节点
第一步，在Worker节点执行安装kubeadm与Docker
第二步，执行部署 Master节点的kubeadm join指令

```shell
kubeadm join 10.0.5.100:6443 --token 0cjl0p.ints41asgaj578iume0 \
--discovery-token-ca-cert-hash sha256:40054faada2be39cag33e3231a2bba5e72643b581dagasgd14faa3acf5c5ef515989
```

# 将master节点也作为工作节点

```shell
kubectl taint nodes --all node-role.kubernetes.io/master-
```


# 参考
## 查看系统支持的kubeadm版本

```shell
apt list kubeadm -a
```
可以看到提示可支持以下版本的kubeadm

```shell
Listing... Done
kubeadm/kubernetes-xenial 1.20.2-00 amd64 [upgradable from: 1.18.15-00]
kubeadm/kubernetes-xenial 1.20.1-00 amd64
kubeadm/kubernetes-xenial 1.20.0-00 amd64
kubeadm/kubernetes-xenial 1.19.7-00 amd64
kubeadm/kubernetes-xenial 1.19.6-00 amd64
kubeadm/kubernetes-xenial 1.19.5-00 amd64
kubeadm/kubernetes-xenial 1.19.4-00 amd64
kubeadm/kubernetes-xenial 1.19.3-00 amd64
kubeadm/kubernetes-xenial 1.19.2-00 amd64
kubeadm/kubernetes-xenial 1.19.1-00 amd64
kubeadm/kubernetes-xenial 1.19.0-00 amd64
kubeadm/kubernetes-xenial,now 1.18.15-00 amd64 [installed,upgradable to: 1.20.2-00]
kubeadm/kubernetes-xenial 1.18.14-00 amd64
kubeadm/kubernetes-xenial 1.18.13-00 amd64
kubeadm/kubernetes-xenial 1.18.12-00 amd64
kubeadm/kubernetes-xenial 1.18.10-00 amd64
kubeadm/kubernetes-xenial 1.18.9-00 amd64
kubeadm/kubernetes-xenial 1.18.8-00 amd64
kubeadm/kubernetes-xenial 1.18.6-00 amd64
kubeadm/kubernetes-xenial 1.18.5-00 amd64
kubeadm/kubernetes-xenial 1.18.4-01 amd64
kubeadm/kubernetes-xenial 1.18.4-00 amd64
kubeadm/kubernetes-xenial 1.18.3-00 amd64
kubeadm/kubernetes-xenial 1.18.2-00 amd64
kubeadm/kubernetes-xenial 1.18.1-00 amd64
kubeadm/kubernetes-xenial 1.18.0-00 amd64
kubeadm/kubernetes-xenial 1.17.17-00 amd64
kubeadm/kubernetes-xenial 1.17.16-00 amd64
kubeadm/kubernetes-xenial 1.17.15-00 amd64
kubeadm/kubernetes-xenial 1.17.14-00 amd64
kubeadm/kubernetes-xenial 1.17.13-00 amd64
kubeadm/kubernetes-xenial 1.17.12-00 amd64
kubeadm/kubernetes-xenial 1.17.11-00 amd64
kubeadm/kubernetes-xenial 1.17.9-00 amd64
kubeadm/kubernetes-xenial 1.17.8-00 amd64
kubeadm/kubernetes-xenial 1.17.7-01 amd64
kubeadm/kubernetes-xenial 1.17.7-00 amd64
kubeadm/kubernetes-xenial 1.17.6-00 amd64
kubeadm/kubernetes-xenial 1.17.5-00 amd64
kubeadm/kubernetes-xenial 1.17.4-00 amd64
kubeadm/kubernetes-xenial 1.17.3-00 amd64
kubeadm/kubernetes-xenial 1.17.2-00 amd64
kubeadm/kubernetes-xenial 1.17.1-00 amd64
kubeadm/kubernetes-xenial 1.17.0-00 amd64
kubeadm/kubernetes-xenial 1.16.15-00 amd64
kubeadm/kubernetes-xenial 1.16.14-00 amd64
kubeadm/kubernetes-xenial 1.16.13-00 amd64
kubeadm/kubernetes-xenial 1.16.12-00 amd64
kubeadm/kubernetes-xenial 1.16.11-01 amd64
kubeadm/kubernetes-xenial 1.16.11-00 amd64
kubeadm/kubernetes-xenial 1.16.10-00 amd64
kubeadm/kubernetes-xenial 1.16.9-00 amd64
kubeadm/kubernetes-xenial 1.16.8-00 amd64
kubeadm/kubernetes-xenial 1.16.7-00 amd64
kubeadm/kubernetes-xenial 1.16.6-00 amd64
kubeadm/kubernetes-xenial 1.16.5-00 amd64
kubeadm/kubernetes-xenial 1.16.4-00 amd64
kubeadm/kubernetes-xenial 1.16.3-00 amd64
kubeadm/kubernetes-xenial 1.16.2-00 amd64
kubeadm/kubernetes-xenial 1.16.1-00 amd64
kubeadm/kubernetes-xenial 1.16.0-00 amd64
kubeadm/kubernetes-xenial 1.15.12-00 amd64
kubeadm/kubernetes-xenial 1.15.11-00 amd64
kubeadm/kubernetes-xenial 1.15.10-00 amd64
kubeadm/kubernetes-xenial 1.15.9-00 amd64
kubeadm/kubernetes-xenial 1.15.8-00 amd64
kubeadm/kubernetes-xenial 1.15.7-00 amd64
kubeadm/kubernetes-xenial 1.15.6-00 amd64
kubeadm/kubernetes-xenial 1.15.5-00 amd64
kubeadm/kubernetes-xenial 1.15.4-00 amd64
kubeadm/kubernetes-xenial 1.15.3-00 amd64
kubeadm/kubernetes-xenial 1.15.2-00 amd64
kubeadm/kubernetes-xenial 1.15.1-00 amd64
kubeadm/kubernetes-xenial 1.15.0-00 amd64
kubeadm/kubernetes-xenial 1.14.10-00 amd64
kubeadm/kubernetes-xenial 1.14.9-00 amd64
kubeadm/kubernetes-xenial 1.14.8-00 amd64
kubeadm/kubernetes-xenial 1.14.7-00 amd64
kubeadm/kubernetes-xenial 1.14.6-00 amd64
kubeadm/kubernetes-xenial 1.14.5-00 amd64
kubeadm/kubernetes-xenial 1.14.4-00 amd64
kubeadm/kubernetes-xenial 1.14.3-00 amd64
kubeadm/kubernetes-xenial 1.14.2-00 amd64
kubeadm/kubernetes-xenial 1.14.1-00 amd64
kubeadm/kubernetes-xenial 1.14.0-00 amd64
kubeadm/kubernetes-xenial 1.13.12-00 amd64
kubeadm/kubernetes-xenial 1.13.11-00 amd64
kubeadm/kubernetes-xenial 1.13.10-00 amd64
kubeadm/kubernetes-xenial 1.13.9-00 amd64
kubeadm/kubernetes-xenial 1.13.8-00 amd64
kubeadm/kubernetes-xenial 1.13.7-00 amd64
kubeadm/kubernetes-xenial 1.13.6-00 amd64
kubeadm/kubernetes-xenial 1.13.5-00 amd64
kubeadm/kubernetes-xenial 1.13.4-00 amd64
kubeadm/kubernetes-xenial 1.13.3-00 amd64
kubeadm/kubernetes-xenial 1.13.2-00 amd64
kubeadm/kubernetes-xenial 1.13.1-00 amd64
kubeadm/kubernetes-xenial 1.13.0-00 amd64
kubeadm/kubernetes-xenial 1.12.10-00 amd64
kubeadm/kubernetes-xenial 1.12.9-00 amd64
kubeadm/kubernetes-xenial 1.12.8-00 amd64
kubeadm/kubernetes-xenial 1.12.7-00 amd64
kubeadm/kubernetes-xenial 1.12.6-00 amd64
kubeadm/kubernetes-xenial 1.12.5-00 amd64
kubeadm/kubernetes-xenial 1.12.4-00 amd64
kubeadm/kubernetes-xenial 1.12.3-00 amd64
kubeadm/kubernetes-xenial 1.12.2-00 amd64
kubeadm/kubernetes-xenial 1.12.1-00 amd64
kubeadm/kubernetes-xenial 1.12.0-00 amd64
kubeadm/kubernetes-xenial 1.11.10-00 amd64
kubeadm/kubernetes-xenial 1.11.9-00 amd64
kubeadm/kubernetes-xenial 1.11.8-00 amd64
kubeadm/kubernetes-xenial 1.11.7-00 amd64
kubeadm/kubernetes-xenial 1.11.6-00 amd64
kubeadm/kubernetes-xenial 1.11.5-00 amd64
kubeadm/kubernetes-xenial 1.11.4-00 amd64
kubeadm/kubernetes-xenial 1.11.3-00 amd64
kubeadm/kubernetes-xenial 1.11.2-00 amd64
kubeadm/kubernetes-xenial 1.11.1-00 amd64
kubeadm/kubernetes-xenial 1.11.0-00 amd64
kubeadm/kubernetes-xenial 1.10.13-00 amd64
kubeadm/kubernetes-xenial 1.10.12-00 amd64
kubeadm/kubernetes-xenial 1.10.11-00 amd64
kubeadm/kubernetes-xenial 1.10.10-00 amd64
kubeadm/kubernetes-xenial 1.10.9-00 amd64
kubeadm/kubernetes-xenial 1.10.8-00 amd64
kubeadm/kubernetes-xenial 1.10.7-00 amd64
kubeadm/kubernetes-xenial 1.10.6-00 amd64
kubeadm/kubernetes-xenial 1.10.5-00 amd64
kubeadm/kubernetes-xenial 1.10.4-00 amd64
kubeadm/kubernetes-xenial 1.10.3-00 amd64
kubeadm/kubernetes-xenial 1.10.2-00 amd64
kubeadm/kubernetes-xenial 1.10.1-00 amd64
kubeadm/kubernetes-xenial 1.10.0-00 amd64
kubeadm/kubernetes-xenial 1.9.11-00 amd64
kubeadm/kubernetes-xenial 1.9.10-00 amd64
kubeadm/kubernetes-xenial 1.9.9-00 amd64
kubeadm/kubernetes-xenial 1.9.8-00 amd64
kubeadm/kubernetes-xenial 1.9.7-00 amd64
kubeadm/kubernetes-xenial 1.9.6-00 amd64
kubeadm/kubernetes-xenial 1.9.5-00 amd64
kubeadm/kubernetes-xenial 1.9.4-00 amd64
kubeadm/kubernetes-xenial 1.9.3-00 amd64
kubeadm/kubernetes-xenial 1.9.2-00 amd64
kubeadm/kubernetes-xenial 1.9.1-00 amd64
kubeadm/kubernetes-xenial 1.9.0-00 amd64
kubeadm/kubernetes-xenial 1.8.15-00 amd64
kubeadm/kubernetes-xenial 1.8.14-00 amd64
kubeadm/kubernetes-xenial 1.8.13-00 amd64
kubeadm/kubernetes-xenial 1.8.12-00 amd64
kubeadm/kubernetes-xenial 1.8.11-00 amd64
kubeadm/kubernetes-xenial 1.8.10-00 amd64
kubeadm/kubernetes-xenial 1.8.9-00 amd64
kubeadm/kubernetes-xenial 1.8.8-00 amd64
kubeadm/kubernetes-xenial 1.8.7-00 amd64
kubeadm/kubernetes-xenial 1.8.6-00 amd64
kubeadm/kubernetes-xenial 1.8.5-00 amd64
kubeadm/kubernetes-xenial 1.8.4-00 amd64
kubeadm/kubernetes-xenial 1.8.3-00 amd64
kubeadm/kubernetes-xenial 1.8.2-00 amd64
kubeadm/kubernetes-xenial 1.8.1-01 amd64
kubeadm/kubernetes-xenial 1.8.0-01 amd64
kubeadm/kubernetes-xenial 1.8.0-00 amd64
kubeadm/kubernetes-xenial 1.7.16-00 amd64
kubeadm/kubernetes-xenial 1.7.15-00 amd64
kubeadm/kubernetes-xenial 1.7.14-00 amd64
kubeadm/kubernetes-xenial 1.7.11-00 amd64
kubeadm/kubernetes-xenial 1.7.10-00 amd64
kubeadm/kubernetes-xenial 1.7.9-00 amd64
kubeadm/kubernetes-xenial 1.7.8-00 amd64
kubeadm/kubernetes-xenial 1.7.7-00 amd64
kubeadm/kubernetes-xenial 1.7.6-00 amd64
kubeadm/kubernetes-xenial 1.7.5-00 amd64
kubeadm/kubernetes-xenial 1.7.4-00 amd64
kubeadm/kubernetes-xenial 1.7.3-01 amd64
kubeadm/kubernetes-xenial 1.7.2-00 amd64
kubeadm/kubernetes-xenial 1.7.1-00 amd64
kubeadm/kubernetes-xenial 1.7.0-00 amd64
kubeadm/kubernetes-xenial 1.6.13-00 amd64
kubeadm/kubernetes-xenial 1.6.12-00 amd64
kubeadm/kubernetes-xenial 1.6.11-01 amd64
kubeadm/kubernetes-xenial 1.6.10-00 amd64
kubeadm/kubernetes-xenial 1.6.9-00 amd64
kubeadm/kubernetes-xenial 1.6.8-00 amd64
kubeadm/kubernetes-xenial 1.6.7-00 amd64
kubeadm/kubernetes-xenial 1.6.6-00 amd64
kubeadm/kubernetes-xenial 1.6.5-00 amd64
kubeadm/kubernetes-xenial 1.6.4-00 amd64
kubeadm/kubernetes-xenial 1.6.3-00 amd64
kubeadm/kubernetes-xenial 1.6.2-00 amd64
kubeadm/kubernetes-xenial 1.6.1-00 amd64
kubeadm/kubernetes-xenial 1.5.7-00 amd64
```

## 查看kubeadm需要的镜像包

```shell
kubeadm config images list
```
可以看到相关的镜像
```shell
k8s.gcr.io/kube-apiserver:v1.18.15
k8s.gcr.io/kube-controller-manager:v1.18.15
k8s.gcr.io/kube-scheduler:v1.18.15
k8s.gcr.io/kube-proxy:v1.18.15
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
```
## 获取kubeadm的默认配置文件

```shell
kubeadm config print init-defaults
```
这样我们可以得到一份默认的配置
```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: node1
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```
