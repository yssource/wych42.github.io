+++
title = "用 slack/nebula 在公网上部署 kubernetes 集群"
author = ["chi"]
date = 2019-12-04T18:52:00+08:00
lastmod = 2019-12-04T19:07:12+08:00
tags = ["kubernetes", "nebula"]
categories = ["烂笔头"]
draft = false
toc = true
+++

这是[上一篇](/deploy-kubernetes-with-kubeadm/)的后续，顺带解决更多需求：

-   在不同地区（云厂商）有多台设备, 甚至只在内网的设备（比如家里的 NAS）
-   用一个 kubernetes 集群管理这些设备上的运行的部分服务, 虽然不是好的实践，但是自己用真的方便
-   最好这些机器可以直接互联, 就像都在一个子网里一样

看起来我需要的就是一个搭建 overlay network 的方案，而且没有公网入口（但是有出口）的设备也可加入这个网络。

自然就想到了用过的 [wireguard](https://www.wireguard.com/quickstart/) 和 [tinc-vpn](https://www.tinc-vpn.org/)。然而，这俩配置和使用都很麻烦，也主要是用来作 vpn 的，于是就尝试下新项目 [slack/nebula](https://github.com/slackhq/nebula)。


## 安装和配置 nebula {#安装和配置-nebula}

三台设备：

-   a.host.com, 作为 lighthouse, 公网 IP 1.2.3.4, 组网 IP `192.168.100.1/24`
-   b.host.com, 组网 IP `192.168.100.2/24`
-   c.host.com, 组网 IP `192.168.100.3/24`


### 安装 nebula {#安装-nebula}

```bash
wget -qO- https://github.com/slackhq/nebula/releases/download/v1.0.0/nebula-linux-amd64.tar.gz | tar  xvf - -C /usr/local/bin
```


### 生成证书和配置文件 {#生成证书和配置文件}

```bash
./nebula-cert ca -name "calico"
./nebula-cert sign -name "a.host.com" -ip "192.168.100.1/24" -groups "lighthouse"
...
```

配置文件参考[文档](https://github.com/slackhq/nebula#5-configuration-files-for-each-host)设置就行，有几项需要注意：

所有设备上：

```yaml
# 都要注意把本机对应的证书和 *ca.crt* 拷贝到配置文件里写的路径：
pki:
  # The CAs that are accepted by this node. Must contain one or more certificates created by 'nebula-cert ca'
  ca: /etc/nebula/ca.crt
  cert: /etc/nebula/a.host.com.crt
  key: /etc/nebula/a.host.com.key
# 把 lighthouse 的公网 IP 映射加上
static_host_map:
  "192.168.100.1": ["1.2.3.4:4242"]
```

lighthouse 节点上，设置 `am_lighthouse: true`,以及 `lighthouse.hosts` 留空。

其他节点上: `lighthouse.hosts` 字段里填上 lighthouse 节点的组网 IP（即 192.168.100.1）。

接着把配置文件拷贝到各个节点的 `/etc/nebula/config.yaml`


### 运行 {#运行}

nebula 进程没有自带 daemon 模式，就用 supervisor 来运行吧。

```bash
apt-get install supervisor
cat <<EOF >/etc/supervisor/conf.d/nebula.conf
[program:nebula]
command=/usr/local/bin/nebula -config /etc/nebula/config.yaml
EOF
supervisorctl reload
```

可以再用 `supervisorctl status` 看下状态。


## 部署 kubernetes {#部署-kubernetes}

大体跟上一篇类似。不过这次用 [calico](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) 作 network add-on，因为我真的养了一只 calico-cat。


### 配置 calico {#配置-calico}

kubeadm 提供的 calico quickstart 命令需要调整，要把 [calico.yaml](https://docs.projectcalico.org/v3.8/manifests/calico.yaml) 下载到机器上修改后使用：

-   calico 默认使用了 `192.168.0.0/16` 网段，与 nebula 使用的网段重复，我将文件里的 CALICO\_IPV4POOL\_CIDR 修改成 `192.168.128.0/24`.
-   默认的 autodetection\_method 是 first-found，这样可能用不到 nebula 的 IP 地址，文件里没有这个字段，需要新加，修改后长这样：

    ```yaml
    ​- name: IP
      value: "autodetect"
    ​- name: IP_AUTODETECTION_METHOD
     value: "interface=nebula*"
    ```

在 `kubeadm init` 之后，执行 `kubectl apply -f calico.yaml` 即可。


### 调整 kubelet 的 node-ip {#调整-kubelet-的-node-ip}

在每个节点都完成了 `kubeadm join` 命令之后，需要调整下节点上 kubelet 的 node-ip 参数，修改为 nebula 的 IP 地址,以使 `kubectl logs` 之类的命令可以正常工作。

一键脚本,注意如果改了 nebula 的网卡名设置，脚本里也要对应的修改:

```bash
sed -i s,'KUBELET_KUBEADM_ARGS="[^"]*',"& --node-ip=$(ip addr show nebula1 | grep -Po 'inet \K[\d.]+')", /var/lib/kubelet/kubeadm-flags.env
systemctl restart kubelet
systemctl restart docker
```

在控制节点上执行 `kubectl -n kube-system get pods -o wide`,看 IP 那一列，如果显示 nebula 的地址，就表示成功了。
