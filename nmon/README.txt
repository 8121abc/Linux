## 直接将nmon16g_x86_rhel72文件授予执行权限，然后./nmon16g_x86_rhel72即可运行nmon性能测试软件。

	chmod +x nmon16g_x86_rhel72
	./nmon16g_x86_rhel72 -c 10 -s 1- -f

将生成的文件拷贝到Windows系统，然后打开运行解压nmon_analyser_34a.zip里面的表格文件，点击分析数据，然后选择从Linux系统导出来的文件，即可实现分析。