#	简介

k8s使用

#	常用命令

创建RC、Service

	kubectl create -f mysql-rc.yaml

查看RC

	kubectl get rc

创建RC会创建Pod,查看Pod的创建情况

	kubectl get pods


查看**的详细信息（log）

	kubectl  describe pod mysql

查看service的状态

	kubectl get svc

删除**

	kubectl  delete rc mysql