
系统时centos7上 关闭防火墙 systemctl stop firewalld.service 关闭selinux vi /etc/selinux/comfig

Kubernetes 集群中主要存在两种类型的节点，分别是 master 节点，以及 minion 节点。
Minion 节点是实际运行 Docker 容器的节点，负责和节点上运行的 Docker 进行交互，并且提供了代理功能。
Master 节点负责对外提供一系列管理集群的 API 接口，并且通过和 Minion 节点交互来实现对集群的操作管理。

apiserver：用户和 kubernetes 集群交互的入口，封装了核心对象的增删改查操作，提供了 RESTFul 风格的 API 接口，通过 etcd 来实现持久化并维护对象的一致性。

scheduler：负责集群资源的调度和管理，例如当有 pod 异常退出需要重新分配机器时，scheduler 通过一定的调度算法从而找到最合适的节点。

controller-manager：主要是用于保证 replicationController 定义的复制数量和实际运行的 pod 数量一致，另外还保证了从 service 到 pod 的映射关系总是最新的。

kubelet：运行在 minion 节点，负责和节点上的 Docker 交互，例如启停容器，监控运行状态等。

proxy：运行在 minion 节点，负责为 pod 提供代理功能，会定期从 etcd 获取 service 信息，并根据 service 信息通过修改 iptables 来实现流量转发（最初的版本是直接通过程序提供转发功能，效率较低。），将流量转发到要访问的 pod 所在的节点上去。

etcd：key-value键值存储数据库，用来存储kubernetes的信息的。

flannel：Flannel 是 CoreOS 团队针对 Kubernetes 设计的一个覆盖网络（Overlay Network）工具，需要另外下载部署。我们知道当我们启动 Docker 后会有一个用于和容器进行交互的 IP 地址，如果不去管理的话可能这个 IP 地址在各个机器上是一样的，并且仅限于在本机上进行通信，无法访问到其他机器上的 Docker 容器。Flannel 的目的就是为集群中的所有节点重新规划 IP 地址的使用规则，从而使得不同节点上的容器能够获得同属一个内网且不重复的 IP 地址，并让属于不同节点上的容器能够直接通过内网 IP 通信。




#	node节点

安装yum扩展


	yum install -y yum-utils

配置yum源


	yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

	yum-config-manager --enable docker-ce-edge

添加缓存


	yum makecache fast

更新


	yum update

安装docker-ce版本


	yum install docker-ce

查看docker信息


	yum list docker-ce.x86_64  --showduplicates |sort -r

# master

