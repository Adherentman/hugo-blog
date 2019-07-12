---
title: "部署 Kubernetes 集群"
date: 2019-07-12T019:01:58+05:30
description: "搭建kubernetes的集群初级教程，权当尝试k8s"
tags: [docker, Kubernetes]
---

## 禁用 swap

```shell
sudo swapoff -a
sudo vim /etc/fstab
注释掉swap那一行
```

## 安装

默认安装了docker-ce=18.06.0~ce~3-0~ubuntu，
以下是安装 kubeadm kubelet kubectl 流程：

```shell
apt-get update && apt-get install -y apt-transport-https ca-certificates curl
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

## 初始化

Kubernetes 集群的初始化可以分为三个步骤

- 在 Master 上运行控制平面 (Control Plane)
- 将 Node 加入到集群中
- 安装 Pod 网络的附加组件

### Master

kubeadm 1.12 使用的是 v1alpha3 API，这里的定义包含了 InitConfiguration 和 ClusterConfiguration 两部分。

**kubeadmInit.yml**:

```yaml
apiVersion: kubeadm.k8s.io/v1alpha3
kind: InitConfiguration
nodeRegistration:
  kubeletExtraArgs:
    # 从 Aliyun Registry 拉取基础镜像
    pod-infra-container-image: registry.aliyuncs.com/google_containers/pause-amd64:3.1
---
apiVersion: kubeadm.k8s.io/v1alpha3
kind: ClusterConfiguration
# 从 Aliyun Registry 拉取 Control Plane 镜像
imageRepository: registry.aliyuncs.com/google_containers
# 使用确定的 Kubernetes 版本，避免初始化时从 https://dl.k8s.io/release/stable-1.12.txt 读取
kubernetesVersion: v1.12.2
networking:
  # 如使用 flannel 组件应增加如下配置
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
```

kubeadm 将自动下载镜像，以 Static Pod 方式运行 Control Plane。

```shell
kubeadm init --config kubeadmInit.yml
```

![png](https://1mu-test.oss-cn-hangzhou.aliyuncs.com/1mu-test/kubeadmInitRes.png)

这就是初始化成功之后的log，并且界面上会输出一条 kubeadm join 命令，将其记录到本地，稍后会用于 Node 加入集群。命令示例如下

```shell
kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
```

### 访问集群

可以参照 kubeadm 初始化后输出的说明，增加使用 kubectl 访问 Kubernetes 集群所需的配置。

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

这时可以使用 `kubectl -n kube-system get pods` 查看到已部署的 Control Plane Pod。

### Node

在Node机子上，我们需要去做安装工作(安装 kubeadm kubelet kubectl)。
之后用之前那个join后的参数去代替以下`<xxx>`
**NodeJoin.yml**:

```shell
apiVersion: kubeadm.k8s.io/v1alpha3
kind: JoinConfiguration
discoveryTokenAPIServers:
- <master-ip>:<master-port>
discoveryTokenCACertHashes:
- sha256:<hash>
nodeRegistration:
  kubeletExtraArgs:
    pod-infra-container-image: registry.aliyuncs.com/google_containers/pause-amd64:3.1
token: <token>
```

然后我们只用输入以下命令

```shell
kubeadm join --config NodeJoin.yml
```

运行成功后，我们可以在 Master 上使用 kubectl get nodes 查看到已加入集群的 Node。

### Pod Network

在完成前面两步后，通过 kubectl get nodes 我们可以看到 Master 和 Node 状态是 NotReady，这是因为缺少[Pod Network 附加组件](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network)。
现在部署的 Pod 不会被分配 IP，会维持在 `ContainerCreating`状态。比如通过 `kubectl -n kube-system get pods -l k8s-app=kube-dns`查看到的 CoreDNS Pod。
可以选择任意一款支持 CNI 的网络组件，这里使用 calico 作为示例，在 Master 上运行如下命令。

```shell
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.3/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
```

待网络组件安装完成后，再次运行 kubectl -n kube-system get pods -l k8s-app=kube-dns 可以看到 CoreDNS Pod 已经变为 Running 状态。

## 测试

### 验证 kube-apiserver, kube-controller-manager, kube-scheduler, pod network

```shell
# 部署一个 Nginx Deployment，包含两个 Pod
# https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
kubectl create deployment nginx --image=nginx:alpine
kubectl scale deployment nginx --replicas=2

# 待启动后，两个 Nginx Pod 应该是 Running 状态，并且各自分配有 10.244 开头的集群内 IP
kubectl get pods -l app=nginx -o wide
# 结果示例
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE
nginx-65d5c4f7cc-khzn8   1/1     Running   0          73m   192.168.1.2   zihao   <none>
nginx-65d5c4f7cc-v7x49   1/1     Running   0          72m   192.168.1.3   zihao   <none>
```

### 验证 kube-proxy

```shell
# 以 NodePort 方式对外提供服务
# https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
kubectl expose deployment nginx --port=80 --type=NodePort

# Nginx 服务应该得到一个 10.96 开头的集群内 IP，以及集群外可访问的 Port
kubectl get services nginx
# 结果示例
NAME    TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
nginx   NodePort   10.100.177.202   <none>        80:32551/TCP   96m

# 可以通过任意 NodeIP:Port 在集群外部访问这个服务
curl http://node-ip:32630
```

### 验证 dns, pod network

```shell
# 启动一个 Busybox 部署，并进入其内部
# 如果没有出现提示符，按下回车键
kubectl run -it curl --image=radial/busyboxplus:curl

# 输入命令 nslookup nginx 应可以正确解析出集群内的 IP，证明 DNS 服务正常
[ root@curl-5cc7b478b6-7zxlq:/ ]$ nslookup nginx
# 结果示例
# Server:    10.96.0.10
# Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
#
# Name:      nginx
# Address 1: 10.98.222.155 nginx.default.svc.cluster.local

# 输入命令 curl nginx 应可以正确返回 Nginx 首页，证明 kube-proxy 正常
[ root@curl-5cc7b478b6-7zxlq:/ ]$ curl http://nginx/
```
