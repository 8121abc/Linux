# cobbler安装步骤

> 操作系统版本CentOS7.4

> cobbler版本2.8.6


## 1. 安装依赖包
1. 安装epel源

		yum install epel-release -y
2. 安装需要的rpm包

		yum install httpd tftp dhcp xinetd cobbler cobbler-web pykickstart -y



## 2. 初始化cobbler
1. 启动httpd服务

		systemctl start httpd
		netstat -tlnp|grep 80
2. 启动cobblerd服务
		
		systemctl start cobblerd
		netstat -tlnp|grep 25151
3. 运行cobbler的检查命令
运行命令之前需要关闭firewalld防火墙和selinux。

		systemctl stop firewalld
		systemctl disable firewalld
		setenfoce 0 #临时关闭selinux，建议运行一下命令，永久关闭selinux，运行命令后重启服务器。
		sed -i 's/=enforcing/=disabled/g' /etc/selinux/configcat
	<pre>
	cobbler check
	[root@localhost ~]# ~]# cobbler check
	The following are potential configuration items that you may want to fix:
	
	1 : The 'server' field in /etc/cobbler/settings must be set to something other than an localhost, or, or kickstarting features will not work.  This should be a resolvable hostname or IP for the boot server as reachable by all machines that will use it. #设置能网卡启动的服务器IP地址或者主机名
	root# vim /etc/cobbler/setting #384行**
	2 : For PXE to be functional, the 'next_server' field in /etc/cobbler/settings must be set to something other than an 127.0.0.1, an, and should match the IP of the boot server on the PXE network.
	root# vim /etc/cobbler/setting #272行**
	3 : SELinux is enabled. Please review the following wiki page for details on ensuring cobbler works correctly in your SELinux environment:
	       https://github.com/cobbler/cobbler/wiki/Selinux
	4 : change 'disable' to 'no' in /etc/xinetd.d/tftp
	root# sed -i '14s/yes/no/g' /etc/xinetd.d/tftp**
	5 : Some network boot-loaders are missing from /var/lib/cobbler/loaders, you may run 'cobbler get-loaders' to download them, or, if you only want to handle x86/x86_64 netbooting, you may ensure that you have installed a *recent* version of the syslinux package installed and can ignore this message entirely.  Files in this directory, should you want to support all architectures, should include pxelinux.0, menu.c32, 2, elilo.efi, an, and yaboot. The 'cobbler get-loaders' command is the easiest way to resolve these requirements.
	root# cobbler get-loaders**
	6 : enable and start rsyncd.service wit with systemctl
	root# systemctl enable rsyncd**
	root# systemctl start rsyncd**
	7 : debmirror package is not installed, it will be required to manage debian deployments and repositories
	8 : ksvalidator was not found, install pykickstart
	9 : The default password used by the sample templates for newly installed machines (default_password_crypted in /etc/cobbler/settings) is still set to 'cobbler' and should be changed, try: "openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'" to generate new one
	root# openssl passwd -1 -salt 'random-phrase-here' 'your-password-here'**
	10 : fencing tools were not found, and are required to use the (optional) power management features. install cman or fence-agents to use them
	11 : /etc/cobbler/settings manage_dhcp: 0改为manage_dhcp: 1
	修改/etc/cobbler/dhcp.template文件的DHCP相关信息。注意IP地址范围一定要正确。
	
	Restart cobblerd and then run 'cobbler sync' to apply changes.
	
	成功完成述步骤之后，重启一下httpd和cobblerd服务，启动xinetd服务，然后运行cobbler sync同步一下相关信息。
	
	systemctl restart httpd
	systemctl restart cobblerd
	systemctl start xinetd
	cobbler sync
	</pre>

4. 导入镜像

		cobbler import --path=/mnt/ --name=CentOS7.4-x86_64 --arch=x86_64 
		looking for /var/www/cobbler/ks_mirror/CentOS7.4-x86_64/repodata/*comps*.xml 
		Keeping repodata as-is :/var/www/cobbler/ks_mirror/CentOS7.4-x86_64/repodata

5. 上传自定义的kickstart.cfg配置文件

		cobbler profile list
		cobbler profile repost --name=CentOS7.4-x86_64 
		cobbler profile edit --name=CentOS7.4-x86_64 --kickstart=/var/lib/cobbler/kickstart/CentOS7.4-x86_64.cfg
		cobbler profile edit --name=CentOS7.4-x86_64 --kopts='net.ifnames=0 biosname=0' #参数之间没有冒号！

## 3.安装系统

开启需要安装系统的服务器，设置网卡第一启动顺序，然后选择需要安装的系统，等待安装完成即可。

## 4.常见错误
	1. cobbler check报错
	解决办法：
		systemctl restart httpd
		systemctl restart cobblerd
	2. No space left on device
	原因是虚拟机内存少于2个G
	解决办法：
		将虚拟机的内存调整为大于2G即可。