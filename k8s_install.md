#	Kubernetes安装


##	软件环境

*	Centos:7.3
*	Kubernetes v1.8.2
*	Etcd v3.2.9
*	Calico v2.6.2
*	Docker v17.10.0-ce

##	服务器信息

*	k8s-master:192.168.10.188
*	k8s-minion-1:192.168.10.189
*	k8s-minion-2:192.168.10.190

##	基础环境安装

所有节点都需要安装

关闭防火墙，selinux

	systemctl stop firewalld && systemctl disable firewalld

	setenforce 0

添加hosts

	vim /etc/hosts

<pre>
192.168.10.188  k8s-master
192.168.10.189  k8s-minion-1
192.168.10.190  k8s-minion-2
</pre>

安装最新的docker版本

	curl -fsSL "https://get.docker.com/" | sh

启动并加入开机启动项

	systemctl enable docker && systemctl start docker

修改docker启动项

	vim /lib/systemd/system/docker.service

在“ExecStart=..”上面加入

<pre>
...
ExecStartPost=/sbin/iptables -I FORWARD -s 0.0.0.0/0 -j ACCEPT
ExecStart=..
...
</pre>

重启docker

	systemctl daemon-reload && systemctl restart docker

修改系统参数

	vim /etc/sysctl.d/k8s.conf

<pre>
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
</pre>

	sysctl -p /etc/sysctl.d/k8s.conf



##	Master配置

安装CFSSL工具，这将会用来建立 TLS certificates

	export CFSSL_URL="https://pkg.cfssl.org/R1.2"

	wget "${CFSSL_URL}/cfssl_linux-amd64" -O /usr/local/bin/cfssl

	wget "${CFSSL_URL}/cfssljson_linux-amd64" -O /usr/local/bin/cfssljson

	chmod +x /usr/local/bin/cfssl /usr/local/bin/cfssljson

创建集群 CA 与 Certificates
建立/etc/etcd/ssl文件夹，然后进入目录完成以下操作

	mkdir -p /etc/etcd/ssl && cd /etc/etcd/ssl

	vim etcd-ca-csr.json

<pre>
{"CN":"etcd","key":{"algo":"rsa","size":2048},"names":[{"C":"TW","ST":"Taipei","L":"Taipei","O":"etcd","OU":"Etcd Security"}]}
</pre>

	vim ca-config.json

<pre>
{"signing":{"default":{"expiry":"87600h"},"profiles":{"kubernetes":{"usages":["signing","key encipherment","server auth","client auth"],"expiry":"87600h"}}}
</pre>

	cfssl gencert -initca etcd-ca-csr.json | cfssljson -bare etcd-ca

查看是否创建成功

	ls  etcd-ca*.pem

<pre>
etcd-ca-key.pem  etcd-ca.pem
</pre>

产生 kube-apiserver certificate 证书

	vim etcd-csr.json

根据master的ip修改对应的ip

<pre>
{"CN":"etcd","hosts":["127.0.0.1","192.168.10.188"],"key":{"algo":"rsa","size":2048},"names":[{"C":"TW","ST":"Taipei","L":"Taipei","O":"etcd","OU":"Etcd Security"}]
</pre>

生成证书

<pre>
cfssl gencert \
  -ca=etcd-ca.pem \
  -ca-key=etcd-ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  etcd-csr.json | cfssljson -bare etcd
</pre>

查看是否生成成功

	ls  etcd*.pem

<pre>
etcd-ca-key.pem  etcd-ca.pem  etcd-key.pem  etcd.pe
</pre>

	rm -rf *.json

确认/etc/etcd/ssl有以下文件：

	ls /etc/etcd/ssl

<pre>
etcd-ca.csr  etcd-ca-key.pem  etcd-ca.pem  etcd.csr  etcd-key.pem  etcd.pem
</pre>


Etcd 安装与设定
打开[https://github.com/coreos/etcd/releases/](https://github.com/coreos/etcd/releases/),找到最新的etcd(etcd-v3.2.9-linux-amd64.tar.gz),下载下来

	tar -xf etcd-v3.2.9-linux-amd64.tar.gz

	mv etcd-v3.2.9-linux-amd64/etcd* /usr/local/bin/ && rm -rf etcd-v3.2.9-linux-amd64

创建etcd用户和组

	groupadd etcd && useradd -c "Etcd user" -g etcd -s /sbin/nologin -r etcd

编写etcd配置文件

	vim /etc/etcd/etcd.conf

需要修改对应的master ip

<pre>
# [member]
ETCD_NAME=master1
ETCD_DATA_DIR=/var/lib/etcd
ETCD_LISTEN_PEER_URLS=https://0.0.0.0:2380
ETCD_LISTEN_CLIENT_URLS=https://0.0.0.0:2379
ETCD_PROXY=off

# [cluster]
ETCD_ADVERTISE_CLIENT_URLS=https://192.168.10.188:2379
ETCD_INITIAL_ADVERTISE_PEER_URLS=https://192.168.10.188:2380
ETCD_INITIAL_CLUSTER=master1=https://192.168.10.188:2380
ETCD_INITIAL_CLUSTER_STATE=new
ETCD_INITIAL_CLUSTER_TOKEN=etcd-k8s-cluster

# [security]
ETCD_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_CLIENT_CERT_AUTH="true"
ETCD_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_AUTO_TLS="true"
ETCD_PEER_CERT_FILE="/etc/etcd/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/ssl/etcd-key.pem"
ETCD_PEER_CLIENT_CERT_AUTH="true"
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ssl/etcd-ca.pem"
ETCD_PEER_AUTO_TLS="true"
</pre>

编写etcd启动文件

	vim /lib/systemd/system/etcd.service

<pre>
[Unit]
Description=Etcd Service
After=network.target

[Service]
Environment=ETCD_DATA_DIR=/var/lib/etcd/default
EnvironmentFile=-/etc/etcd/etcd.conf
Type=notify
User=etcd
PermissionsStartOnly=true
ExecStart=/usr/local/bin/etcd
Restart=on-failure
RestartSec=10
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
</pre>

建立 var 存放信息，然后启动 Etcd 服务:

	mkdir -p /var/lib/etcd && chown etcd:etcd -R /var/lib/etcd /etc/etcd

	systemctl enable etcd.service && systemctl start etcd.service

验证etcd是否成功

	export CA="/etc/etcd/ssl"

	ETCDCTL_API=3  /usr/local/bin/etcdctl --cacert=${CA}/etcd-ca.pem     --cert=${CA}/etcd.pem     --key=${CA}/etcd-key.pem     --endp379" endpoint health

出现下面信息就是安装成功

<pre>
https://192.168.10.188:2379 is healthy: successfully committed proposal: took = 2.570465ms
</pre>


安装kubernetes

下载Kubernetes 组件

	export KUBE_URL="https://storage.googleapis.com/kubernetes-release/release/v1.8.2/bin/linux/amd64"

	wget "${KUBE_URL}/kubelet" -O /usr/local/bin/kubelet

	wget "${KUBE_URL}/kubectl" -O /usr/local/bin/kubectl

	chmod +x /usr/local/bin/kubelet /usr/local/bin/kubectl



下载CNI

打开[https://github.com/containernetworking/plugins/releases](https://github.com/containernetworking/plugins/releases),找到最新的cni(cni-plugins-amd64-v0.6.0.tgz),下载下来

	
	mkdir -p /opt/cni/bin && cd /opt/cni/bin

	tar -xf cni-plugins-amd64-v0.6.0.tgz

创建集群 CA 与 Certificates
在这部分，将会需要生成 client 与 server 的各组件 certificates，并且替 Kubernetes admin user 生成 client 证书

创建pki文件夹

	mkdir -p /etc/kubernetes/pki && cd /etc/kubernetes/pki



#	参考文档

[https://www.kubernetes.org.cn/3096.html](https://www.kubernetes.org.cn/3096.html)	