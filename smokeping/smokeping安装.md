# 安装smokeping


## 一、安装epel源
### 1.1	更改系统默认源，推荐163源，速度会更快，更稳定
	cd /etc
	mv yum.repos.d/ yum.repos.d.ori
	mkdir yum.repos.d;cd yum.repos.d
	wget http://mirrors.163.com/.help/CentOS7-Base-163.repo

### 1.2 安装epel源
	yum install epel-release
	yum makecache

## 二、安装rrdtool与依赖库
### 2.1 安装依赖库
	yum -y install perl perl-Net-Telnet perl-Net-DNS perl-LDAP perl-libwww-perl perl-RadiusPerl perl-IO-Socket-SSL perl-Socket6 perl-CGI-SpeedyCGI perl-FCGI perl-CGI-SpeedCGI perl-Time-HiRes perl-ExtUtils-MakeMaker perl-RRD-Simple rrdtool rrdtool-perl curl fping  httpd httpd-devel gcc make  wget libxml2-devel libpng-devel glib pango pango-devel freetype freetype-devel fontconfig cairo cairo-devel libart_lgpl libart_lgpl-devel mod_fastcgi libidn-develpopt-devel perl-Sys-Syslog
### 2.2 编译安装echoping

	wget https://fossies.org/linux/misc/old/echoping-6.0.2.tar.gz
	tar xf echoping-6.0.2.tar.gz
	cd echoping-6.0.2
	./configure
	make -j 2 && make install 

## 三、安装smokeping

### 3.1 下载、解压、配置smokeping源码
	cd /usr/local/src
	wget http://oss.oetiker.ch/smokeping/pub/smokeping-2.6.9.tar.gz
	cd smokeping-2.6.9
	./configure #报错提示
![报错提示1](https://i.imgur.com/eB5ex1G.png)

	根据提示首先运行一下这个命令，该命令自动下载安装perl相关插件。
	./setup/build-perl-modules.sh /usr/local/smokeping/thirdparty

### 3.2 安装smokeping
	/usr/bin/gmake install
	
<font color=#FF0000 size=3 >gmake install 时会遇到报错：</font>

	smokeping_config.pod around line 81: alternative text 'the master/slave mode' contains non-escaped | or /
	POD document had syntax errors at /usr/bin/pod2man line 69.
	gmake[1]: *** [smokeping_config.5] Error 255

![错误截图](https://i.imgur.com/14qROeN.png)

#### 原因分析：

可能是pod2man版本原因所致

#### 解决方案：
	rm -rf /usr/bin/pod2man
    yum reinstall perl-podlators #pod2man由perl-podlators提供，重新安装一次，即可解决问题。
	/usr/bin/gmake install #再执行一次安装命令即可完成安装

## 四、安装后配置

### 4.1 安装完成后的目录结构
![完成后目录结构](https://i.imgur.com/bkUPAgc.png)

根据需要创建相应的目录，并做授权：

	cd /usr/local/smokeping
	1.创建cache data var 目录
	mkdir cache data var
	2.创建日志文件
	touch /var/log/smokeping.log
	3.授权
	chown apache.apache cache data var
	chown apache.apache /var/log/smokeping.log
	4.修改密码文件权限
	chmod 600 etc/smokeping_secrets.dist
	mv htdocs/smokeping.fcgi.dist htdocs/smokeping.fcgi
	5.复制配置文件
	mv etc/config.dist etc/config

创建相应的目录后：
![](https://i.imgur.com/wYO7a7K.png)

### 4.2 编辑etc/config文件，修改cgirul step参数
	
	vim etc/config
	修改一下内容：
	把some.url修改为你的ip或者域名
	cgiurl=http://some.rul/smokeping.cgi
 	*** Database ***
  	step = 300  #此处建议改为60 这是检测时间

### 4.3 编辑apache配置文件
	vim /etc/httpd/conf/httpd.conf
	添加如下内容：
	Alias /cache "/usr/local/smokeping/cache/"
	Alias /cropper "/usr/local/smokeping/htdocs/cropper/"
	Alias /smokeping "/usr/local/smokeping/htdocs/smokeping.fcgi"
	<Directory "/usr/local/smokeping">
	AllowOverride None
	Options All
	AddHandler cgi-script .fcgi .cgi
	Order allow,deny
	Allow from all
	DirectoryIndex smokeping.fcgi
	</Directory>

### 4.4 设置smokeping开机启动
	
	echo "/usr/local/smokeping/bin/smokeping --logfile=/var/log/smokeping.log 2>&1 &" >> /etc/rc.local
	chmod +x /etc/rc.local

### 4.5 启动httpd与smokeping
	/etc/init.d/httpd start
	/usr/local/smokeping/bin/smokeping --logfile=/var/log/smokeping.log 2>&1 &

### 4.6 smokeping登录加密

	<Directory "/usr/local/smokeping">
	AllowOverride None
	Options All
	AddHandler cgi-script .fcgi .cgi
	AllowOverride AuthConfig
	Order allow,deny
	Allow from all
	AuthName "Smokeping"
	AuthType Basic
	AuthUserFile /usr/local/smokeping/htdocs/htpasswd
	Require valid-user
	DirectoryIndex smokeping.fcgi
	</Directory>

	htpasswd -c /usr/local/smokeping/htdocs/htpasswd admin
	这个是设置登录账户为admin，密码在后面输入
	然后重启httpd就可以实现密码验证登录

### 4.7 web界面的中文支持

	yum -y install wqy-zenhei-fonts.noarch
	vim /usr/local/smokeping/etc/config
	*** Presentation *** 这行下面
	charset = UTF-8 #添加这行，解决出图乱码问题
	vi /usr/local/smokeping/lib/Smokeping/Graphs.pm
	#在第147行，下边插入这一行代码
	'--font TITLE:20:"WenQuanYi Zen Hei Mono"',

	if ($mode =~ /[anc]/){
	        my $val = 0;
        for my $host (@hosts){
            my ($graphret,$xs,$ys) = RRDs::graph
            ("dummy",
            '--start', $tasks[0][1],
            '--end', $tasks[0][2],
            "DEF:maxping=$cfg->{General}{datadir}${host}.rrd:median:AVERAGE",
            '--font TITLE:20:"WenQuanYi Zen Hei Mono"',#加在这里
            'PRINT:maxping:MAX:%le' );
            my $ERROR = RRDs::error();
            return "<div>RRDtool did not understand your input: $ERROR.</div>" if $ERROR;
            $val = $graphret->[0] if $val < $graphret->[0];
        }
        $val = 1e-6 if $val =~ /nan/i;
        $max = { $tasks[0][1] => $val * 1.5 };
    }

## 五、注意事项
1. 在/usr/local/smokeping/etc/config中添加;

2. smokeping就这点不好，添加节点不能在前台Web页面添加，一定要在后台的配置文件中添加;

3. 修改/usr/local/smokeping/etc/config后，必须重启smokeping程序，配置才会生效;

4. smokeping 会根据配置文件config 在/usr/local/smokeping/data 之下添加moniter文件夹，其下包含website子文件夹;

5. 用vmware workstation的虚拟机测试有一点好处，workstation下的虚拟网卡可以设置出入的丢包率，适合smokeping做丢包测试, 经过测试smokeping检测出的丢包率与vmware worksation虚拟网卡设置的丢包率基本相同,也就是说smokeping 能够反应网络的真实状况。
添加监控节点示例：注意+是第一层，++是第二层，+++ 是第三层