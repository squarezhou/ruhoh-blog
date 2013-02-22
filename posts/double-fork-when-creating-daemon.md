---
title: Daemon进程为何需要两次fork
date: '2012-11-22'
description:
categories: 技术
---
Daemon进程
---------

守护进程（daemon）是指在UNIX或其他多任务操作系统中在后台执行的电脑程序，并不会接受电脑用户的直接操控。此类程序会被以进程的形式初始化。守护进程程序的名称通常以字母“d”结尾：例如，syslogd就是指管理系统日志的守护进程。（来自[维基百科][wikipedia]）

常见的Apache服务器（httpd），MySQL服务端（mysqld），作业规划进程（crond）都是以守护进程的方式运行于Linux系统中。

创建Daemon
---------

守护进程最重要的特性是常驻后台运行，因此必须要与当前的运行环境完全隔离，这些环境包括未关闭的文件描述符，控制终端，会话和进程组，工作目录以及文件创建掩模等。

关于如何创建Daemon进程可以参考这篇[daemon进程原理及实现][yungang]，这里就不再赘述。

为何两次Fork
----------

这是创建Daemon进程的大致步骤及作用：

- 第一次fork（产生父子进程，父进程退出，子进程与父进程控制终端脱离）  
- 子进程setsid（子进程成为会话组长，与父进程会话脱离）  
- **第二次fork**（产生子孙进程，子进程退出，孙进程成为Daemon最终进程）  
- 切换工作目录、设置文件创建掩模、关闭所有打开文件句柄等

原则上来说在第一次fork后，子进程即与父进程的控制终端脱离关系，父进程退出后，子进程被init接管，基本达到了Daemon进程要求。但还差了一点，子进程与父进程还在同一个会话组，因此子进程需要调用setsid以达到与父进程会话脱离。也就是这个setsid，使子进程成为了新的会话组组长，却导致了新的问题产生：在Linux中会话组长可以重新申请打开一个控制终端。为了彻底与控制终端断绝关系，我们需要一个非会话组长的进程，子进程的子进程正是我们要的。（The first process in the process group becomes the process group leader and the first process in the session becomes the session leader. Every session can have one TTY associated with it. Only a session leader can take control of a TTY. For a process to be truly daemonized (ran in the background) we should ensure that the session leader is killed so that there is no possibility of the session ever taking control of the TTY.）

参考：  
1. [维基百科 守护进程][wikipedia]  
2. [daemon进程原理及实现][yungang]  
3. [What is the reason for performing a double fork when creating a daemon?][stackoverflow]


[wikipedia]: http://zh.wikipedia.org/wiki/%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B
[yungang]: http://blog.163.com/yungang_z/blog/static/175153133201232462140622/
[stackoverflow]: http://stackoverflow.com/questions/881388/what-is-the-reason-for-performing-a-double-fork-when-creating-a-daemon