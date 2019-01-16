### mbr自动应答文件：

	lang en_US
	keyboard us
	timezone Asia/Shanghai
	rootpw --iscrypted $default_password_crypted
	test
	install
	url --url=$tree
	bootloader --location=mbr
	zerombr
	clearpart --all --initlabel
	part /boot --fstype xfs --size 1024 --ondisk sda
	part swap --size 1024
	part / --fstype xfs --size 1 --grow --ondisk sda
	auth --useshadow --enablemd5
	$SNIPPET('network_config')
	reboot
	selinux --disabled
	firewall --disabled
	skipx
	
	%pre
	$SNIPPET('log_ks_pre')
	$SNIPPET('kickstart_start')
	$SNIPPET('pre_install_network_config')
	$SNIPPET('pre_anamon')
	%end
	
	%packages
	@ base
	@ core
	sysstat
	iptraf
	ntp
	lrzsz
	ncurses-devel
	openssl-devel
	zlib-devel
	OpenIPMI-tools
	nmap
	screen
	%end
	
	%post
	systemctl disable postfix.service
	%end
