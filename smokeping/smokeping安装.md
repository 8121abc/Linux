### 遇到错误：

	smokeping_config.pod around line 81: alternative text 'the master/slave mode' contains non-escaped | or /
	POD document had syntax errors at /usr/bin/pod2man line 69.
	gmake[1]: *** [smokeping_config.5] Error 255

![错误截图](https://i.imgur.com/14qROeN.png)

### 原因分析：

可能是pod2man版本原因所致

### 解决方案：
	rm -rf /usr/bin/pod2man
    yum reinstall perl-podlators #pod2man由perl-podlators提供，重新安装一次，即可解决问题。

