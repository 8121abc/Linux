# Centos7安装zabbix
Zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。zabbix能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。

## 一、环境准备
### 1.1 操作系统环境
#### 1.1.1 系统版本
	[root@localhost ~]# cat /etc/redhat-release 
	CentOS Linux release 7.2.1511 (Core) 
#### 1.1.2 关闭selinux
	临时关闭selinux
	[root@localhost ~]# setenforce 0
	setenforce: SELinux is disabled
	[root@localhost ~]# getenforce 
	Disabled
	永久关闭selinux
	sed -i 's/=enforcing/=disabled/g' /etc/selinux/config
#### 1.1.3 关闭防火墙
	[root@localhost ~]# systemctl status firewalld
	● firewalld.service - firewalld - dynamic firewall daemon
	   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
	   Active: active (running) since Tue 2019-01-15 22:26:36 CST; 28s ago
	 Main PID: 33417 (firewalld)
	   CGroup: /system.slice/firewalld.service
	           └─33417 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
	
	Jan 15 22:26:35 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewal.....
	Jan 15 22:26:36 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall...n.
	Hint: Some lines were ellipsized, use -l to show in full.
	[root@localhost ~]# systemctl stop firewalld
	[root@localhost ~]# systemctl disable firewlld
	[root@localhost ~]# systemctl status firewalld
	● firewalld.service - firewalld - dynamic firewall daemon
	   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
	   Active: inactive (dead)
	
	Jan 15 22:26:35 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewal.....
	Jan 15 22:26:36 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall...n.
	Jan 15 22:27:11 localhost.localdomain systemd[1]: Stopping firewalld - dynamic firewal.....
	Jan 15 22:27:12 localhost.localdomain systemd[1]: Stopped firewalld - dynamic firewall...n.
	Hint: Some lines were ellipsized, use -l to show in full.
### 1.2 网络环境
	[root@localhost ~]# hostname -I
	192.168.229.133 
### 1.3 zabbix版本选择
	[root@localhost ~]# rpm -aq|grep zabbix-release
	zabbix-release-3.0-1.el7.noarch

## 二、安装zabbix rpm包
### 2.1 安装zabbix3.0的源
	rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
### 2.2 安装zabbix3.0的相关rpm包
	[root@localhost ~]# yum install -y zabbix-server-mysql zabbix-web-mysql

## 三、安装并启动mariadb
### 3.1 安装lamp环境
	yum -y install mariadb mariadb-server php php-mysql httpd
### 3.2 初始化数据库
	启动数据库
	systemctl start mariadb
	设置开机自启动
	systemctl enable mariadb
	初始化数据库
	mysql_secure_installation
![图片](https://images2018.cnblogs.com/blog/862626/201807/862626-20180706145937194-1806672696.png)

## 四、创建zabbix数据库以及zabbix账号
	[root@localhost ~]# mysql -uroot -p
	...
	mysql> create database zabbix default character set utf8 collate utf8_bin;
	Query OK, 1 row affected (0.00 sec)
	 
	mysql> grant all privileges on zabbix.* to 'root'@'localhost' identified by 'root-password';
	Query OK, 0 rows affected (0.00 sec)
	 
	mysql> grant all privileges on zabbix.* to 'zabbix'@'%' identified by 'zabbix';
	Query OK, 0 rows affected (0.00 sec)
	 
	mysql> flush privileges;
	Query OK, 0 rows affected (0.01 sec)

## 五、导入默认的zabbix数据库信息
	[root@localhost ~]# zcat /usr/share/doc/zabbix-server-mysql-3.0.19/create.sql.gz | mysql zabbix -uzabbix -pzabbix

## 六、修改zabbix_server.conf配置文件里面的数据库相关信息
	[root@localhost ~]# grep ^DB /etc/zabbix/zabbix_server.conf
	DBHost=localhost
	DBName=zabbix
	DBUser=zabbix
	DBPassword=zabbix

## 七、修改zabbix.conf配置文件，主要修改时区
	vim /etc/httpd/conf.d/zabbix.conf
	php_value max_execution_time 300
	php_value memory_limit 128M
	php_value post_max_size 16M
	php_value upload_max_filesize 2M
	php_value max_input_time 300
	php_value always_populate_raw_post_data -1
	php_value date.timezone Asia/Shanghai

## 八、启动httpd/zabbix-server服务
	[root@localhost ~]# systemctl start httpd
	[root@localhost ~]# systemctl enable httpd
	[root@localhost ~]# netstat -an |grep 80
	tcp6       0      0 :::80                   :::*                    LISTEN      32783/httpd  
	[root@localhost ~]# systemctl start zabbix-server
	[root@localhost ~]# systemctl enable zabbix-server
	[root@localhost ~]# netstat -tlnp|grep 10051
	tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      32511/zabbix_server 
	tcp6       0      0 :::10051                :::*                    LISTEN      32511/zabbix_server 
	
	查看zabbix-server的日志
	[root@localhost ~]# tailf  /var/log/zabbix/zabbix_server.log

	zabbix-server的web目录
	[root@localhost ~]# ls /usr/share/zabbix/
	actionconf.php                 app                 chart.php            hostgroups.php               index.php           popup_media.php    scripts_exec.php  tr_events.php
	adm.gui.php                    applications.php    charts.php           hostinventoriesoverview.php  items.php           popup.php          search.php        trigger_prototypes.php
	adm.housekeeper.php            audio               conf                 hostinventories.php          js                  popup_right.php    services.php      triggers.php
	adm.iconmapping.php            auditacts.php       conf.import.php      host_prototypes.php          jsLoader.php        popup_trexpr.php   setup.php.ori     tr_logform.php
	adm.images.php                 auditlogs.php       dashconf.php         host_screen.php              jsrpc.php           profile.php        slideconf.php     tr_status.php
	adm.macros.php                 authentication.php  discoveryconf.php    hosts.php                    latest.php          queue.php          slides.php        tr_testexpr.php
	adm.other.php                  browserwarning.php  disc_prototypes.php  httpconf.php                 local               report2.php        srv_status.php    usergrps.php
	adm.regexps.php                chart2.php          events.php           httpdetails.php              locale              report4.php        styles            users.php
	adm.triggerdisplayoptions.php  chart3.php          favicon.ico          image.php                    maintenance.php     robots.txt         sysmap.php        zabbix.php
	adm.triggerseverities.php      chart4.php          fonts                images                       map.import.php      screenconf.php     sysmaps.php
	adm.valuemapping.php           chart5.php          graphs.php           img                          map.php             screenedit.php     templates.php
	adm.workingtime.php            chart6.php          history.php          imgstore.php                 overview.php        screen.import.php  toptriggers.php
	api_jsonrpc.php                chart7.php          host_discovery.php   include   
## 九、网页安装zabbix
访问网站http://本机IP/zabbix/setup.php
过程基本都是点击下一步，需要设置的地方是第一步的数据库相关信息。其他保持默认直接下一步即可。

安装完成后可以将setup.php文件重命名来防止重装。

	[root@localhost ~]# pwd
	/usr/share/zabbix
	[root@localhost ~]# mv setup.php etup.php.bak
## 十、登录zabbix
访问网址http://192.168.229.133/zabbix

默认用户名：Admin

默认密码：zabbix #一定在登录系统后及时修改管理员密码
## 十一、安装zabbix客户端
	[root@localhost ~]# rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm #安装zabbix源，如果是另外的服务器
	[root@localhost ~]# yum install zabbix-agent -y
	保持默认配置，直接启动agent程序。
	[root@localhost ~]# systemctl start  zabbix-agent.service 
	#该操作只对zabbix server服务器本身有效，然后到网页端启动zabbix server的监控即可实现对自身的监控。
	#以后添加其他服务器，只需要把zabbix端的host name与该配置文件中的hostname对应即可。如下：

	Server=zabbix服务器ip地址
	ServerActive=zabbix服务器ip地址
	Hostname=客户端ip地址或者主机名 #在zabbix里面创建host时的name应和该内容保持一致
	Server被动连接 #服务器去连接客户端拉取监控数据
	ServerActive主动连接 #客户端主动连接服务端推送监控数据
	
	到此zabbix agent就已经安装完毕。

启动客户端

	[root@localhost ~]# zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
	
	[root@localhost ~]# systemctl start zabbix-agent
	
	[root@localhost ~]# systemctl enable zabbix-agent


zabbix agent的shell脚本

	[root@localhost ~]# cat zabbix_agent_add.sh
	#!/bin/sh
	systemctl stop firewalld.service
	systemctl disable firewalld.service 
	sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
	rpm -ivh http://mirrors.aliyun.com/epel/7/x86_64/Packages/e/epel/release-7-11.noarch.rpm
	rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
	sed -i "s/enabled=1/enabled=0/g" /etc/yum.repos.d/epel.repo
	yum clean all
	yum -y install zabbix-agent
	sed -i "s/Server=127.0.0.1/Server=192.168.3.132/g" /etc/zabbix/zabbix_agentd.conf
	sed -i "s/ServerActive=127.0.0.1/ServerActive=192.168.3.132/g" /etc/zabbix/zabbix_agentd.conf
	sed -i "s/Hostname=Zabbix server/Hostname=192.168.3.146/g" /etc/zabbix/zabbix_agentd.conf
	zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf
	systemctl start zabbix-agent
	systemctl restart zabbix-agent
	systemctl enable zabbix-agent
	reboot

## 十二、常见问题处理
### 12.1 访问http://ip/zabbix无反应，提示没有网页
原因分析：配置zabbix.conf后没有重启httpd服务来使修改生效。

	解决方案：
	[root@localhost ~]# systemctl restart httpd

### 12.2 中文乱码
原因分析：
字体原因造成

	解决方案：
	[root@localhost ~]# yum -y install wqy-microhei-fonts
	[root@localhost ~]# cp /usr/share/fonts/wqy-microhei/wqy-microhei.ttc /usr/share/fonts/dejavu/DejaVuSans.ttf
	[root@localhost ~]# systemctl restart zabbix-server
