---
title: PHP Memcache扩展的坑
date: '2013-03-22'
description:
categories: Tech
tags: [PHP,PECL,Memcached]
---
问题描述
-------

今天帮同事分析一段诡异的代码，代码本身写法很乱，再加上代码关闭了错误输出，所以开始一直没找对问题所在。后来慢慢定位到是Memcache扩展set函数flags参数设置问题，再后来结合扩展源码找出问题根本原因，最后终于开启了error_reporting并验证了问题原因。  
问题代码大概是这样的：

	<?php
	error_reporting(7);
	$mc = new Memcache();
	$mc->connect('127.0.0.1',11211);
	$mc->set('key', 'value', true, 60);
	sleep(1);
	var_dump($mc->get('key'));

最终输出为FALSE，不是预期的value string。这段代码有两个坑，一小一大。  
小坑是同事告诉我error\_reporting(7)与error\_reporting(E\_ALL)作用是相同的，这个坑我很快跳进去了，因为我一直使用后者，也忘记了E_ALL常量的值，加之不清楚有没其他特别写法，所以就相信他说的，于是就相信所有的信息将会被输出。 
大一点的坑就是set函数第3个参数的值，下面重点说这个。

Memcached的flags
---------------

Memcached在存储数据时除了可以指定过期时间外，还支持flags参数。`SET key expire flags value_length`，第3个参数就是flags。  
在Memcached 1.2.1之前为flags预留了16位，到了1.2.1以后预留了32位（4bytes）。对于服务器端而言，并不清楚你设置这些标记的作用，只是在你取数据的时候同时传回给客户端。因此客户端就可以利用这个值来标记数据是否是经过编码的、是压缩过的……PHP Memcached扩展就是这么干的。  

Memcache扩展的flags
------------------

Memcache扩展中定义了两个宏MMC_SERIALIZED和MMC_COMPRESSED，分别表示序列化和zlib压缩，但只有MMC_COMPRESSED对应值被同时定义为PHP常量。  

宏定义  
![宏定义](http://www.zhouxiongzhi.net/assets/media/2013-03/php-memcache-flags-define.png)

常量定义  
![常量定义](http://www.zhouxiongzhi.net/assets/media/2013-03/php-memcache-flags-constant.png)

看到这里，其实我们隐约可以感觉到Memcache扩展是不支持在PHP中启用数据序列化的，事实正是如此。  
当flags参数为MMC_COMPRESSED时，扩展会对数据进行zlib压缩后再发送给服务端；在取回数据后再根据flags参数，对数据解压并返回给PHP。判断使用位与（&）。  

存储压缩  
![存储压缩](http://www.zhouxiongzhi.net/assets/media/2013-03/php-memcache-compress-value.png)

取回解压  
![取回解压](http://www.zhouxiongzhi.net/assets/media/2013-03/php-memcache-uncompress-value.png)

但数据序列化却不是这样，**序列化不受flags参数控制**。只有值类型不是string、long、double、bool类型，即array、object等类型时，才会对值序列化，而无视flags参数，当然这里序列化后会更新flags值确保取回时反序列化。在取回数据时，扩展会严格根据flags值对数据反序列化。

存储序列化  
![存储序列化](http://www.zhouxiongzhi.net/assets/media/2013-03/php-memcache-serialize-value.png)

取回反序列化  
![取回反序列化](http://www.zhouxiongzhi.net/assets/media/2013-03/php-memcache-unserialize-value.png)

OK，问题来了：如果值类型为string、long、double、bool其中之一，并设置了flags为1、true、所有可以转换为1或`&1>0`的值时，扩展在存储时不会序列化，但在取回时尝试反序列化，这时会抛出一个"unable to unserialize data"的Notice。如果关闭了Notice输出或错误输出，那就像上面的代码一样，返回FALSE。

解决方案
-------

- 使用Memcached扩展（无flags参数）；
- 修改扩展源码，在没有序列化时`flags &= ~MMC_SERIALIZED`强制取消序列化标识；
- 封装或继承set函数，对上面4种类型判断自动修改flags值。

