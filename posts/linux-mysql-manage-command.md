---
title: MySQL管理命令
date: '2012-11-28'
description:
categories: Tech
tags: [MySQL,Linux]
---
###下载###

- 访问MySQL[官网下载页面][download_page]  
- 选择操作系统平台  
- 确定系统是32位还是64位  
- 选择一种文件类型点击Download

最简单的安装方式是使用已编译的二进制包，所以我下载了32位Linux通用版本“Linux - Generic 2.6 (x86, 32-bit), Compressed TAR Archive”，完成后得到[mysql-5.5.28-linux2.6-i686.tar.gz][mysql_55tgz]。

###Linux下安装###

安装命令在根目录的INSTALL-BINARY中有详细说明，直接Copy过来：

	To install and use a MySQL binary distribution, the basic command sequence looks like this:
	shell> groupadd mysql
	shell> useradd -r -g mysql mysql
	shell> cd /usr/local
	shell> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
	shell> ln -s full-path-to-mysql-VERSION-OS mysql
	shell> cd mysql
	shell> chown -R mysql .
	shell> chgrp -R mysql .
	shell> scripts/mysql_install_db --user=mysql
	shell> chown -R root .
	shell> chown -R mysql data
	# Next command is optional
	shell> cp support-files/my-medium.cnf /etc/my.cnf
	shell> bin/mysqld_safe --user=mysql &
	# Next command is optional
	shell> cp support-files/mysql.server /etc/init.d/mysql.server

###启动与停止###

**启动**

启动命令安装步骤有提及：
  
	bin/mysqld_safe --user=mysql &  

强制建议使用官方提供的mysqld\_safe脚本启动，不手动执行mysqld启动。因为mysqld_safe会完成下面这些有用的检测：

1. 检查系统和选项。
2. 检查MyISAM表。
3. 保持MySQL服务器窗口。
4. 启动并监视mysqld，如果因错误终止则重启。
5. 将mysqld的错误消息发送到数据目录中的host_name.err 文件。
6. 将mysqld\_safe的屏幕输出发送到数据目录中的host_name.safe文件。

**停止**

安全停止MySQL命令：

	bin/mysqladmin -uroot -p shutdown

另一种停止方式是发送TERM信号：KILL(-TERM)，不到万不得已不要使用KILL -9。

###修改密码###

正确命令：

	bin/mysqladmin -u root password 'yournewpassword'

另一种方式，直接修改系统表：

	use mysql;
	update user set password=password('yournewpassword') where user='root';
	flush privileges;

如果忘记原密码，加--skip-grant-tables参数启动，不用密码即可登录，再执行上面操作：
  
	bin/mysqld_safe --skip-grant-tables &

###开机启动###

Linux下让程序开机启动最简单的方式是将启动命令加到/etc/rc.local文件中，Linux在开机过程中会自动执行rc.local文件中所有的命令。  
这里就不能使用相对路径，必须加上完整路径，比如我的是：/usr/local/mysql/bin/mysqld_safe，因此可以执行：

	echo '/usr/local/mysql/bin/mysqld_safe --user=mysql &' >> /etc/rc.local

这种方式虽简单，但是不能设置关机自动停止MySQL，最佳方式请往下看。

###添加Linux服务###

关于Linux服务就不多介绍了，读者请自行Google。
 
要添加为Linux服务，需要准备一个特定的脚本，供系统service脚本调用。话说MySQL已经为我们提供了这个脚本：support-files/mysql.server，将这个脚本放到/etc/init.d目录下，文件名即为service名称，然后就没有然后了……

如果你仔细看过安装步骤，就会发现最后一步可选命令即是安装mysql.server这个服务滴。  
*注：如果MySQL安装目录不是/usr/local/mysql，需要在/etc/my.cnf中配置basedir和datadir*

既然已经是Linux服务，那么就有更多的事情可以做了。

**启动停止**

	service mysql.server start|stop|restart|...

**开机启动**

这次使用chkconfig命令，mysql.server脚本中已经配置chkconfig需要的相关参数。默认是运行级别为2、3、4、5时开启，开机启动优先级为64，关机停止优先级为36：

	# Comments to support chkconfig on RedHat Linux
	# chkconfig: 2345 64 36

所以你要做的又只有一行（有没觉得讲了半天废话TT)：

	chkconfig --add mysql.server

OK，现在MySQL Server能在开机时自动启动并在关机时自动停止了（注意只有以2、3、4、5运行级别启动时才会，查看当前系统运行级别：more /etc/inittab），而且可以通过优雅的service命令来管理啦^_^

###卸载###

卸载就更简单了，不多说：

	chkconfig --del mysql.server
	rm /init.d/mysql.server
	rm /etc/my.cnf
	rm -rf /usr/local/mysql

强烈建议安装完成后将MySQL数据目录移到独立数据分区下，并修改datadir。谨慎删除MySQL目录，后果概不负责~


[download_page]: http://dev.mysql.com/downloads/mysql/#downloads
[mysql_55tgz]: http://cdn.mysql.com/Downloads/MySQL-5.5/mysql-5.5.28-linux2.6-i686.tar.gz