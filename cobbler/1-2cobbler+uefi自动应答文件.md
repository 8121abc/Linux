**由于越来越多的机器支持UEFI模式，以下是UEFI的启动配置模板。**
		
UEFI启动配置文件：

		auth --enableshadow --passalgo=sha512
		install
		url --url=$tree
		text
		firstboot --disable
		firewall --disabled
		ignoredisk --only-use=sda
		selinux --disabled
		keyboard --vckeymap=us --xlayouts='us'
		lang en_US.UTF-8
		network  --bootproto=dhcp --hostname=NodeA --device=eth0  --activate
		rootpw --plaintext 123456
		reboot
		services --disabled="chronyd"
		skipx
		timezone Asia/Shanghai
		bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
		zerombr
		clearpart --none --initlabel
		
		part /boot/efi --fstype="efi" --ondisk=sda --size=1024 --fsoptions="defaults,uid=0,gid=0,umask=0077,shortname=winnt"
		part /app --fstype="xfs" --size=2048
		part swap --fstype="swap" --size=1024
		part / --fstype="xfs" --size=10240
		part /boot --fstype="ext4" --size=1024
		
		%post
		systemctl stop postfix.service
		%end		


		%pre
		parted -s /dev/sda mklabel gpt
		%end
		
		%packages
		chrony
		kexec-tools
		%end