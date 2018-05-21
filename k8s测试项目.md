#	简介

为了更好的了解k8s，编写几个项目进行测试。

项目所有的yaml文件都在本目录下

#	创建redis服务（master）

创建RC

	kubectl  create -f redis-master-controller.yaml 

查看RC

	kubectl  get rcs

查看pods

	kubectl  get pods

创建Service

	kubectl  create -f redis-master-service.yaml

查看Service

	kubectl  get services

#	创建redis服务（slave）

创建RC

	kubectl  create -f redis-slave-controller.yaml

创建Service

	kubectl  create -f redis-slave-service.yaml

#	创建Guestbook

	kubectl  create -f frontend-controller.yaml

	kubectl  create -f frontend-service.yaml



 

