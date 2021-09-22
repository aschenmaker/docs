# 在Ubuntu20.04集群上安装Kubernetes学习集群

在公司实习的时候，有免费的Kubernetes集群可以使用。最近回到了学校，想要继续进行学习，考虑到成本，不可能去云厂商租一个集群。

刚好，实验室，有五台闲置的机器，于是上手，搭建一个集群。

## 01环境介绍

- 系统：[ubuntu-20.04.3-live-server-amd64](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/20.04.3/ubuntu-20.04.3-live-server-amd64.iso)
- 内存：4G
- CPU：

其中`k-5`节点被我拿来跑旁路由了，所以这里采用了一主四从的形式进行部署。

| 角色   | 主机名 | IP地址        |
| ------ | ------ | ------------- |
| Master | k-1    | 192.168.1.151 |
| Slave  | k-2    | 192.168.1.152 |
| Slave  | k-4    | 192.168.1.154 |
| Slave  | k-6    | 192.168.1.156 |

如果是自己在实验室进行部署的话，需要对IP地址进行固定，以及换源等等。

## 02准备工作

### 禁用交换空间swap

为什么需要禁止swap？

1. 首先，swap会严重影响性能，虚拟内存在磁盘上进行访问的效率会大大下降（内存、IO）。

2. 另外，开启了swap，通过cgroup进行切分的内存就会失效

可以参考这个Issue:  <https://github.com/kubernetes/kubernetes/issues/53533>

通过`swapon --show`可以产看当前的swap分区

```shell
sys-2@sys-2:~$ swapon --show
NAME      TYPE SIZE USED PRIO
/swap.img file 3.8G   0B   -2

sys-2@sys-2:~$  sudo swapoff -a
```

编辑`/etc/fstab`文件，注释掉引用swap的行，保存并重启后输入`sudo swapoff -a`即可。

### 配置hosts

```shell
sys-2@sys-2:~$ cat /etc/hosts
# for kubernetes hosts
192.168.1.151 k-master
192.168.1.152 k-2
192.168.1.154 k-3
192.168.1.155 k-4
192.168.1.156 k-6
```

### 换源

这里使用阿里镜像站的源，这个并不重要

```shell
sys-2@sys-2:~$  cat /etc/apt/sources.list
# 系统源
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse

```

### 添加k8s镜像 

访问：https://mirrors.aliyun.com/kubernetes/

```shell
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - 
cat /etc/apt/sources.list.d/kubernetes.list<<EOF
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF  
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

这里安装三个工具： sudo apt-get install -y `kubelet` `kubeadm` ` kubectl`

### docker

#### 安装docker

首先需要安装docker

```shell
sudo apt  install docker.io
```

#### docker换源

使用七牛云作为镜像源

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://reg-mirror.qiniu.com/"],
  "exec-opts": ["native.cgroupdriver=systemd"] # 这一行是
}
EOF	
```

#### docker代理

配置镜像pull代理，镜像拉去需要配置代理以提高速度。这里我是讲代理服务器配置到了我的局域网旁路由里。

在执行`docker pull`时，是由守护进程`dockerd`来执行。 因此，代理需要配在`dockerd`的环境中。 而这个环境，则是受`systemd`所管控，因此实际是`systemd`的配置。

```sh
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf

sudo vim /etc/systemd/system/docker.service.d/proxy.conf
```

在这个`proxy.conf`文件（可以是任意`*.conf`的形式）中，添加以下内容：

```ini
[Service]
Environment="HTTP_PROXY=http://192.168.1.191:7893/"
Environment="HTTPS_PROXY=http://192.168.1.191:7893/"
```

#### 解决kubelet 和docker 资源隔离不一致

kubelet默认的隔离驱动是systemd，但是docker默认的驱动为cgroups，所以我们需要进行指定。

继续编辑dockerd的配置，讲cgroup的隔离驱动设置为`systemd`

`sudo vim /etc/docker/daemon.json`

```
"exec-opts": ["native.cgroupdriver=systemd"]
```

最后，重启docker

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 配置grub

 修改 GRUB_CMDLINE_LINUX="" 修改成下述內容

```
GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
```

```shell
sudo vim /etc/default/grub
sudo update-grub
```

最后重启服务器

```shell
sudo reboot
```

## 03安装k8s master节点

到这里已经可以愉快的装Kubernetes了

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.1.151
```

你需要将`--apiserver-advertise-address`替换成你自己的

执行之后，输出如下：

```shell
sys-1@305-1:~$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.1.151
[init] Using Kubernetes version: v1.22.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [305-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.1.151]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [305-1 localhost] and IPs [192.168.1.151 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [305-1 localhost] and IPs [192.168.1.151 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 16.507819 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.22" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node 305-1 as control-plane by adding the labels: [node-role.kubernetes.io/master(deprecated) node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
[mark-control-plane] Marking the node 305-1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: wr4p75.xkkku659wiqkeh0f
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

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

Then you can join any number of worker nodes by running the following on each as root:
# 通过这里，讲从节点加入到
kubeadm join 192.168.1.151:6443 --token wr4p75.xkkku659wiqkeh0f \
	--discovery-token-ca-cert-hash sha256:9e90cfc7244e7834d660975f4730aca4b79087ef52e4accc0f4aa62a4f0c066e
```



## 04将 slave 从节点加入

**需要对每一个节点进行准备工作的设置**

```shell
sudo kubeadm join 192.168.1.151:6443 --token wr4p75.xkkku659wiqkeh0f --discovery-token-ca-cert-hash sha256:9e90cfc7244e7834d660975f4730aca4b79087ef52e4accc0f4aa62a4f0c066e
```

可以通过kubectl获取节点信息，发现节点还是没有就绪。

```shell
sys-1@305-1:~$ kubectl get nodes
NAME    STATUS     ROLES                  AGE     VERSION
305-1   NotReady   control-plane,master   81m     v1.22.2
305-4   NotReady   <none>                 6m33s   v1.22.2
305-6   NotReady   <none>                 10s     v1.22.2
sys-2   NotReady   <none>                 55m     v1.22.2
```

这是因为还没有构建网络。

## 05接入网络

可以部署`flannel`作为网络插件

```shell
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

成功部署之后，你会发现你coredns， 以及你的node都已就绪！

```shell
sys-1@305-1:~$ kubectl get pod -n kube-system -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP              NODE     NOMINATED NODE   READINESS GATES
coredns-78fcd69978-r8qr7        0/1     Pending   0          81m     <none>          <none>   <none>           <none>
coredns-78fcd69978-x2557        0/1     Pending   0          81m     <none>          <none>   <none>           <none>
etcd-305-1                      1/1     Running   1          82m     192.168.1.151   305-1    <none>           <none>
kube-apiserver-305-1            1/1     Running   1          82m     192.168.1.151   305-1    <none>           <none>
kube-controller-manager-305-1   1/1     Running   1          82m     192.168.1.151   305-1    <none>           <none>
kube-proxy-9psw2                1/1     Running   0          7m16s   192.168.1.154   305-4    <none>           <none>
kube-proxy-pbf6v                1/1     Running   0          81m     192.168.1.151   305-1    <none>           <none>
kube-proxy-tgmm9                1/1     Running   0          56m     192.168.1.152   sys-2    <none>           <none>
kube-proxy-xp42n                1/1     Running   0          53s     192.168.1.156   305-6    <none>           <none>
kube-scheduler-305-1            1/1     Running   1          82m     192.168.1.151   305-1    <none>           <none>
```

## 06至此

搭建完成，总结一下有几个方面

- 前置准备条件，需要对本地网络进行配置代理，确保能够告诉拉取镜像
- 配置不同子节点的过程基本一致，使用kubeadm基本没有任何心智负担

最后如果有安装上的问题，可以联系进行交流，免费答疑。

