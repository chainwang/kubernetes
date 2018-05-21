#	简介

部署单机kubernetes进行学习使用


#	搭建

##	下载yum源

	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo


	yum makecache

	yum  -y install *rhsm*

##	关闭防火墙服务

centos7 默认使用firewall为防火墙，而Kubernetes的Master与工作Node之间会有大量的网络通信，安全的做法是在防火墙上配置各种需要相互通讯的端口号，在一个安全的内部网络环境中可以关闭防火墙服务；

	systemctl disable firewalld.service

	systemctl stop firewalld.service

##	安装iptables

	yum install -y iptables-services

	systemctl start iptables.service

	systemctl enable iptables.service

##	安装etcd和Kubernetes

docker会在安装kubernetes的过程中被安装

	yum install -y etcd kubernetes

配置修改

	vim /etc/sysconfig/docker

<pre>
...
OPTIONS='--selinux-enabled=false --insecure-registry gcr.io'
...
</pre>

	vim /etc/kubernetes/apiserver

<pre>
...
KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
...
</pre>


##	启动

<pre>
systemctl start etcd
systemctl start docker
systemctl start kube-apiserver.service
systemctl start kube-controller-manager.service
systemctl start kube-scheduler.service
systemctl start kubelet.service
systemctl start kube-proxy.service
</pre>



#	使用

















#	参考文档

[http://lihaoquan.me/2017/2/25/create-kubernetes-single-node-mode.html](http://lihaoquan.me/2017/2/25/create-kubernetes-single-node-mode.html)