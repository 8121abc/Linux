## 安装php70源
	yum install epel-release
	# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm 
	yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
    yum install yum-utils
	yum-config-manager --enable remi-php70
	yum install php70-php-gd php70-php-curl php70-php-cli php70-php-mysql php70-php-xml php70-php-mbstring #官方推荐的php扩展模块，系统自动加载相应的扩展模块，不用指定php70-gd或者php70w-gd

## 安装mariadb10.1
	cat /etc/yum.repos.d/mariadb.repo
	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.1/centos7-amd64
	gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck=1
	安装数据库
	yum info Mariadb
	yum install MariaDB-server

## 创建相应的数据库
	mysql -uroot -p
	create database piwik;
	create user 'piwik'@'localhost' identified by '123456';
	grant select,insert,update,delete,create,drop,alter,create temporary tables,lock tables on piwik.* to 'piwik'@'localhost';
	grant file on *.* to 'piwik'@'localhost'



用yum快速搭建LAMP平台
 
实验环境：
[root@nmserver-7 html]# cat  /etc/redhat-release 
CentOS release 7.3.1611 (AltArch) 
[root@nmserver-7 html]# uname -a
Linux nmserver-7.test.com 3.10.0-514.el7.centos.plus.i686 #1 SMP Wed Jan 25 12:55:04 UTC 2017 i686 i686 i386 GNU/Linux
1、安装apache
　　1.1 安装apache
[root@nmserver-7 ~]# yum install httpd httpd-devel
　　1.2 启动apache服务
[root@nmserver-7 ~]# systemctl start  httpd
　　1.3 设置httpd服务开机启动
[root@nmserver-7 ~]# systemctl enable  httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
　　1.4 查看服务状态
复制代码
[root@nmserver-7 ~]# systemctl status httpd
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since 五 2017-07-21 17:21:37 CST; 6min ago
     Docs: man:httpd(8)
           man:apachectl(8)
 Main PID: 2449 (httpd)
   Status: "Total requests: 11; Current requests/sec: 0; Current traffic:   0 B/sec"
   CGroup: /system.slice/httpd.service
           ├─2449 /usr/sbin/httpd -DFOREGROUND
           ├─2450 /usr/sbin/httpd -DFOREGROUND
           ├─2451 /usr/sbin/httpd -DFOREGROUND
           ├─2452 /usr/sbin/httpd -DFOREGROUND
           ├─2453 /usr/sbin/httpd -DFOREGROUND
           ├─2454 /usr/sbin/httpd -DFOREGROUND
           ├─2493 /usr/sbin/httpd -DFOREGROUND
           ├─2494 /usr/sbin/httpd -DFOREGROUND
           └─2495 /usr/sbin/httpd -DFOREGROUND

7月 21 17:21:35 nmserver-7.test.com systemd[1]: Starting The Apache HTTP Server...
7月 21 17:21:36 nmserver-7.test.com httpd[2449]: AH00558: httpd: Could not reliably determine the server's fully q...ssage
7月 21 17:21:37 nmserver-7.test.com systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
复制代码
　　1.5 防火墙设置开启80端口
[root@nmserver-7 ~]# firewall-cmd --permanent --zone=public  --add-service=http
success
[root@nmserver-7 ~]# firewall-cmd --permanent --zone=public  --add-service=https
success
[root@nmserver-7 ~]# firewall-cmd --reload
success
　　1.6确认80端口监听中
复制代码
[root@nmserver-7 ~]# netstat -tulp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      1084/sshd           
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN      1486/master         
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN      1084/sshd           
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN      1486/master         
tcp6       0      0 [::]:http               [::]:*                  LISTEN      2449/httpd          
udp        0      0 localhost:323           0.0.0.0:*                           592/chronyd         
udp6       0      0 localhost:323           [::]:*                              592/chronyd   
复制代码
　　1.8 查服务器IP
复制代码
[root@nmserver-7 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:56:bc:cf brd ff:ff:ff:ff:ff:ff
    inet 192.168.8.9/24 brd 192.168.8.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe56:bccf/64 scope link 
       valid_lft forever preferred_lft forever
3: bridge0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether ea:89:d5:c7:32:73 brd ff:ff:ff:ff:ff:ff
复制代码
　　1.9 浏览器登陆


 

2、安装mysql
　　2.1安装mysql
[root@nmserver-7 ~]# yum install mariadb mariadb-server mariadb-libs mariadb-devel
 

root@nmserver-7 ~]# rpm -qa |grep maria
mariadb-libs-5.5.52-1.el7.i686
mariadb-5.5.52-1.el7.i686
mariadb-server-5.5.52-1.el7.i686
mariadb-devel-5.5.52-1.el7.i686
 

　　2.2 开启mysql服务，并设置开机启动，检查mysql状态
复制代码
[root@nmserver-7 ~]# systemctl start  mariadb 
[root@nmserver-7 ~]# systemctl enable  mariadb 
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@nmserver-7 ~]# systemctl status  mariadb 
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since 六 2017-07-22 21:19:20 CST; 21s ago
 Main PID: 9603 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─9603 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─9760 /usr/libexec/mysqld --basedir=/usr --datadir=/v...

7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:15 nmserver-7.test.com mariadb-prepare-db-dir[9524]: ...
7月 22 21:19:16 nmserver-7.test.com mysqld_safe[9603]: 170722 21...
7月 22 21:19:16 nmserver-7.test.com mysqld_safe[9603]: 170722 21...
7月 22 21:19:20 nmserver-7.test.com systemd[1]: Started MariaDB ...
复制代码
 

复制代码
[root@nmserver-7 ~]# netstat -tulp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      1084/sshd           
tcp        0      0 0.0.0.0:mysql           0.0.0.0:*               LISTEN      9760/mysqld         
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN      1084/sshd           
tcp6       0      0 [::]:http               [::]:*                  LISTEN      2449/httpd          
udp        0      0 localhost:323           0.0.0.0:*                           592/chronyd         
udp6       0      0 localhost:323           [::]:*                              592/chronyd 
复制代码
　　2.3 数据库安全设置
复制代码
[root@nmserver-7 ~]# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
复制代码
 

　　2.4 登陆数据库测试
复制代码
[root@nmserver-7 ~]# mysql -uroot -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.52-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.02 sec)

MariaDB [(none)]> 
复制代码
 

3、安装PHP
　　3.1 安装php
[root@nmserver-7 ~]# yum -y install php
[root@nmserver-7 ~]# rpm -ql php
/etc/httpd/conf.d/php.conf
/etc/httpd/conf.modules.d/10-php.conf
/usr/lib/httpd/modules/libphp5.so
/usr/share/httpd/icons/php.gif
/var/lib/php/session
 

　　3.2 将php与mysql关联起来
复制代码
[root@nmserver-7 ~]# yum install php-mysql
[root@nmserver-7 ~]# rpm -ql php-mysql
/etc/php.d/mysql.ini
/etc/php.d/mysqli.ini
/etc/php.d/pdo_mysql.ini
/usr/lib/php/modules/mysql.so
/usr/lib/php/modules/mysqli.so
/usr/lib/php/modules/pdo_mysql.so
复制代码
 

　　3.3 安装常用PHP模块
[root@nmserver-7 ~]# yum install -y php-gd php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-snmp php-soap curl curl-devel php-bcmath
 

　　3.4 测试PHP
复制代码
[root@nmserver-7 ~]# cd  /var/www/html/
[root@nmserver-7 html]# ls
[root@nmserver-7 html]# pwd
/var/www/html
[root@nmserver-7 html]# vi info.php

<?php
        phpinfo();
?>
~                                                                                        
~                                                                                        
                                                                  
:wq
复制代码
 

　　3.5重启apache服务器
[root@nmserver-7 html]# systemctl restart http
　　3.6测试PHP
　　在自己电脑浏览器输入 192.168.8.9/info.php，你可以看到已经安装的模块；

升级php
history命令历史
    8  yum provides php   #自带的只有5.4版本
    9   rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm         #更新源
   10   rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
   11  yum remove php-common -y     #移除系统自带的php-common
   12  yum install -y php56w php56w-opcache php56w-xml php56w-mcrypt php56w-gd php56w-devel php56w-mysql php56w-intl php56w-mbstring         #安装依赖包
   13  php -v                    #版本变为5.6
   14  yum provides php-fpm      #因为我是准备搭建lnmp，所以安装php-fpm，这里会提示多个安装源，选择5.6版本的安装就可以了
   15  yum install php56w-fpm-5.6.31-1.w7.x86_64 -y

最快的方式 56升级到安装PHP71
yum install yum-plugin-replace
yum replace php56w-common --replace-with=php71w-common
