## 安装php70w的源
	yum install epel-release
	# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm 
	yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
    yum install yum-utils
	yum-config-manager --enable remi-php70
	yum install php-gd php-curl php-cli php-mysql php-xml php-mbstring #官方推荐的php扩展模块，系统自动加载相应的扩展模块，不用指定php70-gd或者php70w-gd

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