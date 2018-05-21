#	简介

saltstack和ansible一样都是可以进行配置标准化的，不用一条一条的命令执行，写成文件可复用。

SLS（代表SaLt State文件）是Salt State系统的核心。SLS描述了系统的目标状态，由格式简单的数据构成。这经常被称作配置管理

#	编写

##	top.sls

top.sls 是配置管理的入口文件，一切都是从这里开始，在master 主机上，默认存放在/srv/salt/目录。


top.sls 默认从 base 标签开始解析执行,下一级是操作的目标，可以通过正则，grain模块,或分组名,来进行匹配,再下一级是要执行的state文件，不包换扩展名。


- 通过正则进行匹配的示例

	vim /srv/salt/top.sls

<pre>
base:
  '*':
    - webserver
</pre>

- 通过分组名进行匹配的示例，必须要有 - match: nodegroup

	vim /srv/salt/top.sls

<pre>
base:
  group1:
    - match: nodegroup    
    - webserver
</pre>

- 通过grain模块匹配的示例，必须要有- match: grain

	vim /srv/salt/top.sls

<pre>
base:
  'os:Fedora':
    - match: grain
    - webserver
</pre>


准备好top.sls文件后，编写一个state文件

	vim /srv/salt/webserver.sls

<pre>
apache:                 # 标签定义
  pkg:                  # state declaration
    - installed         # function declaration
</pre>

注释：

*	第一行被称为（ID declaration） 标签定义，在这里被定义为安装包的名。注意：在不同发行版软件包命名不同,比如 fedora 中叫httpd的包 Debian/Ubuntu中叫apache2

*	第二行被称为（state declaration）状态定义， 在这里定义使用（pkg state module）

*	第三行被称为（function declaration）函数定义， 在这里定义使用（pkg state module）调用 installed 函数




#	参考文档

[http://blog.csdn.net/death_kada/article/details/48547271](http://blog.csdn.net/death_kada/article/details/48547271)