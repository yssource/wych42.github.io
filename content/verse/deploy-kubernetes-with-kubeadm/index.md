+++
title = "用 kubeadm 部署简易 kubernetes 集群"
author = ["chi"]
date = 2019-12-04T11:34:00+08:00
lastmod = 2019-12-04T18:54:57+08:00
tags = ["kubernetes"]
categories = ["烂笔头"]
draft = false
toc = true
+++

## 准备 {#准备}

-   这次部署能用到的设备都是小型号得 vps，零散在不同得公网区域，所以要部署一个跨公网集群。
-   各项配置都使用最简化得模式，比如单主节点，主要目的是自用+测试。
-   通读和参考文档： [Bootstrapping clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm)。


### 设备和网络 {#设备和网络}

三台设备：

-   控制节点（同时也当做worker用）: 赵云, 2 Core, 4G(x1)
-   worker: 赵云，1 Core, 1G(x2)
-   操作系统: debian 9

网络：三台设备在三个地区，各有公网，内网不通。


### 安装依赖 {#安装依赖}


#### 安装 docker {#安装-docker}

```bash
apt-get install -y \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg2 \
        software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

apt-get update && apt-get install -y docker-ce docker-ce-cli containerd.io
```

docker 使用 systemd 作为 [cgroupdriver](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/#kubeadm-blocks-waiting-for-control-plane-during-installation):

```bash
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
systemctl restart docker
```


#### 安装 kubeadm/kubelet/kubectl {#安装-kubeadm-kubelet-kubectl}

```bash
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
```


## 部署集群 {#部署集群}


### 初始化控制节点 {#初始化控制节点}

```bash
kubeadm init --control-plane-endpoint=<your-endpoint-fqdn> --pod-network-cidr=192.168.0.0/16 --image-repository=registry.aliyuncs.com/google_containers  --upload-certs
```

-   <your\_endpoint\_fqdn> 替换为集群的控制节点 IP 地址或者域名。
-   cidr 按照 flannel 的[文档](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network)设置。
-   国内的机器初始化时，image repository 替换为阿里云得镜像。

根据屏幕输出，配置好 kubeconfig

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### 启用 flannel {#启用-flannel}

```bash
sysctl net.bridge.bridge-nf-call-iptables=1
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```


### worker 加入集群 {#worker-加入集群}

在 kubeadm init 结束时，终端输出的命令，拷贝到 worker 节点上执行。

```bash
 kubeadm join k8s.xiatiao.io:6443 --token <your-token> \
--discovery-token-ca-cert-hash sha256:<your-hash>
```

如果错过了这段输出，可以用下面得命令生成：

```bash
kubeadm token create --print-join-command
```


### 控制节点兼任 worker {#控制节点兼任-worker}

想在控制节点上也跑一些任务，需要解除限制:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```


### 节点打标签 {#节点打标签}

三个节点，一个设置为 forwarder， 两个设置为 backend:

```bash
kubectl lable nodes <nodename> role=<role>
```


## 善后 {#善后}

至此，一个单节点的 kubernetes 集群就配置完成了，但是，由于三个设备在三个公网区域里，内网互不相同。虽然 api-server-address 是公网可用的，但是比如 `kubectl logs` 之类的命令，会尝试直接连接 worker 节点的内网地址，显然是不通的。这就导致了，虽然 pods 能被调度到各个节点上，但是集群「内部」的网络是不通的，services 就用不了。

解决方案有几种：

-   `--advertise-address` 设置为设备的公网地址, 通过 kubeadm 执行的话，要求这个地址在设备上能看到，类似某赵云用了 elastic ip 就不行。
-   配置 nat 转发
    -   在控制节点上: `iptables -t nat -A OUTPUT -d <worker private ip> -j DNAT --to-destination <worker public ip>`
    -   在 worker 节点上： `iptables -t nat -A OUTPUT -d <master private ip> -j DNAT --to-destination <master public ip>`
-   用类似 wireguard, slack/nebula 之类的工具，先把各个设备组一个 overlay network，再部署 kubernetes 集群。


## 命令备忘 {#命令备忘}

-   一键毁灭集群：

    ```bash
    for i in $(kubectl get nodes | tail -n +2 | awk '{print $1}' ); do kubectl drain $i --delete-local-data --force --ignore-daemonsets; kubectl delete node $i; done
    kubeadm reset
    iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
    ```
