---
title: PHP实现Daemon
date: '2012-11-22'
description:
categories: Code
tags: [PHP,Daemon,Linux]
---
本来准备将此篇与上一篇《[Daemon进程为何需要两次fork][double-fork-when-creating-daemon]》写在一起的，话说本来上一篇是要详细介绍Daemon进程的，无奈语言能力太差，写不出大篇幅，只好写重点，于是本篇只能单独放出来了。

关于Daemon进程及如何创建Daemon进程可以继续参考这篇[daemon进程原理及实现][yungang]，这里也不啰嗦，只贴代码。Talk is cheap, show me the code.

PHP版Daemon进程代码：  

	<?php
	/**
	* a simple Daemon script via PHP
	* author: http://www.zhouxiongzhi.net/
	*/
	
	error_reporting(E_ALL);
	set_time_limit(0);
	
	// 检测系统环境
	if (strtolower(PHP_OS) != 'linux' && strtolower(PHP_OS) != 'freebsd') {
		exit("this program only works in linux or freebsd!\n");
	}
	
	if (!extension_loaded('pcntl')) {
		exit("this program needs support of pcntl extension!\n");
	}
	
	if (php_sapi_name() != 'cli') {
		exit("this program only works in Cli sapi!\n");
	}
	
	$pid = pcntl_fork();
	if ($pid == -1) {
		exit("Could not fork!\n");
	} else if ($pid) {
		echo "Grandparent Process Exit!\n";
		exit();
	} else {
		// 设置第一次fork的子进程为会话组组长
		posix_setsid();
		$pid2 = pcntl_fork();
		if ($pid2 == -1) {
			exit("Could not fork!\n");
		} else if ($pid2) {
			echo "Parent Process Exit!\n";
			exit();
		} else {
			// 切换工作目录到/
			chdir('/');
			// 设置文件创建掩模为0
			umask(0);
			
			// 这句输出必须要放在关闭标准输出前
			echo "Daemon Process Launched!\n";
			
			// 刷新输出缓冲
			flush();
			
			// 关闭标准输入、标准输出、标准错误输出，后面再也不要读取他们了
			fclose(STDIN);
			fclose(STDOUT);
			fclose(STDERR);
			
			// 这里添加处理逻辑
			for ($i=0;$i<60;$i++) {
				// 这里不能再echo，换成写日志吧
				error_log($i."\n", 3, '/tmp/daemon_simple.log');
				sleep(1);
			}
		}
	}
	?>


[double-fork-when-creating-daemon]: http://www.zhouxiongzhi.net/post/36286317417/double-fork-when-creating-daemon
[yungang]: http://blog.163.com/yungang_z/blog/static/175153133201232462140622/