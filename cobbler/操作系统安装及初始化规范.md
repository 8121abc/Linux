# 操作系统安装及初始化规范
   
> v1.0


## 操作系统安装流程
### 1、	服务器采购
### 2、	服务器验收并设置raid
### 3、	服务商提供验收单，运维验收负责人签字
### 4、	服务器上架
### 5、	资产录入
### 6、	开始自动化安装。
### 7、	将新服务器划入装机vlan
### 8、	根据资产清单上的mac地址，自定义安装
> * 1、机房；
> * 2、机房区域；
> * 3、机柜；
> * 4、服务器位置；
> * 5、服务器网线接入端口；
> * 6、改端口的mac地址；
> * 7、profile 操作系统 分区等 预分配的IP地址 主机名 子网 网关 dns 角色
### 3、	自动化平台装机。
> - MAC:	08:00:27:4F:EA:8F
> - IP:		192.168.100.100	
> - HOSTNAME:	linux-node2.tengyaods.cn 
> - NETMAST:	255.255.255.0 
> - GATEWAY:	192.168.100.1
> - DNS:		192.168.100.1

<pre> 
cobbler system list
</pre>
<pre>
cobbler system add –name=linux-node2.tengyaods.cn –mac=08:00:27:4F:EA:8F –proifle=CentOS7-x86_64 –ip-address=192.168.100.100 –subnet=255.255.255.0 –gateway=192.168.100.1 –interface=eth0 –static=1 –hostname=linux-node2.tengyaods.cn –name-servers”192.168.100.1” –kictstart=/var/lib/cobbler/kictstarts/CentOS7-x86_64.cfg
</pre>

## 操作系统安装规范

   1. 当前我公司使用操作系统为CentOS 6和CentOS 7，均使用x86_64位系统，需要用公司cobbler进行自动化安装，禁止自定义设置。
   2. 版本选择，数据库统一使用cobbler上centos7-db这个专用的profile，其他web应用统一使用cobbler上Centos-7-web。


## 系统初始化规范
### 初始化操作
* 设置DNS 192.168.100.1
* 安装zabbix-agent：server 192.168.100.1
* 安装salt stack minion: salt stack master：192.168.100.1
* history记录时间
<pre>
   export HISTTIMEFORMAT="%F %T `whoami`"
</pre>
* 日志记录操作
<pre>
export PROMPT_COMMAND='{ msg=$(history 1 | { read x y; echo $y; });logger "[euid=$(whoami)]":$(who am i):[`pwd`]"$msg"; }'
</pre>
* 内核参数优化
* yum仓库



### 目录规范
* 脚本放置目录： /opt/shell
* 脚本日志目录：/opt/shell/log
* 脚本锁文件目录：/opt/shell/lock

### 程序服务安装规范

* 1.源码安装路径 /usr/local/appname.version	
* 2.创建软连接 ln -s /usr/local/appname.version /usr/local/appname

### 主机名命令规范
  **机房名称--项目--角色--服务--集群--节点.域名**

例子：
	idc01-xxshop-api-nginx-bj-node1.shop.com

### 服务启动用户规范

所有服务，同意使用www用户，uid为666，除负载均衡需要监听80端口使用root启动外，所有服务必须使用www用户启动，使用大于1024的端口。

