## 自定义cobbler system 实现操作系统全自动化安装

1. 自动化安装系统的命令

		cobbler system add --name=linux-node2.tengyaods.cn \
		  --mac=xx:xx:xx:xx:xx:xx --profile=CentOS7.4 \
		  --ip-address=192.168.220.140 --subnet=255.255.255.0 \
		  --gateway=192.168.220.1 --interface=eth0 \
		  --static=1 --hostname=linux-node2.tengyaods.cn \
		  --name-servers="192.168.220.1" \
		  --kickstart=/var/lib/cobbler/kickstart/CentOS7.4.cfg

	测试一下是否需要sync同步。

2. 使用api自定义安装

		#!/usr/bin/python
		import xmlrpclib
		server = xmlrpclib.Server("http://192.168.220.130/cobbler_api")	
		print server.get_distros()
		print server.get_profiles()
		print server.get_systems()
		print server.get_images()

		\#!/usr/bin/env python
		\# -*- coding: uf-8 -*-
		class CobblerAPI(object):
			def __init__(self,url,user,password):
				self.cobbler_user= user
				self.cobbler_pass= 123456
				self.cobbler_usr= url
			
			def add_system(self,hostname,ip_add,mac_add,profile):
				'''
				Add Cobbler Systemc Information
				'''
				ret = {
					"result": True,
					"comment": [],
				}
			remote = xmlrpclib.Server(self.cobbler_url)
			token = remote.login(self.cobbler_user,self.cobbler_pass)
			sytemc_id = remote.new_system(token)