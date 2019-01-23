## 安装php70w的源
	yum install epel-release
	# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm 
	yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
    yum install yum-utils
	yum-config-manager --enable remi-php70
	yum install php70w-gd php70w-curl php70w-cli php70w-mysql php70w-xml php70w-mbstring #官方推荐的php扩展模块

## 安装mariadb10.1
	cat /etc/yum.repos.d/mariadb.repo
	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.1/centos7-amd64
	gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck=1
	yum info Mariadb
	yum install MariaDB-server
