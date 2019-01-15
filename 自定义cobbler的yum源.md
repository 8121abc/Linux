## 自定义cobbler的yum源

### 1. 添加repo

	cobber repo add --name=openstack-xx --mirror=http://mirrors.aliyun.com/centos/7/cloud/x86_64/openstack-xx

### 2. 同步下载repo的rpm包（比较耗时，根据网络情况）
	
	cobbler reposync

### 3. 添加repo到对应的profile文件
	
	cobbler profile edit --name=CentOS7.4-x86_64 --repos=http://mirrors.aliyun.com/centos7/cloud/x86_64/openstack-xx

### 4. 修改kickstart文件，添加到%post  %end之间

	vim /var/lib/cobbler/kickstart/CentOS7.4-x86_64.cfg
	
	%post
	$yum_config_stanza
	%end

### 5. 自动化同步cobbler yum 源

	ceho “1 3 * * * /usr/bin/cobbler reposync --tries 3 --no-fail” >> /var/spoon/cron/root