---
title: MySQL字符集笔记
date: '2013-08-01'
description:
categories: Tech
tags: [MySQL,字符集]
---
###字符集###

**系统变量**

- character_set_server：默认的内部操作字符集
- character_set_client：客户端来源数据使用的字符集
- character_set_connection：连接层字符集
- character_set_results：查询结果字符集
- character_set_database：当前选中数据库的默认字符集
- character_set_system：系统元数据(字段名等)字符集

查看当前字符集设置：`show variables like '%character%'`

**转换过程**

1. MySQL Server收到请求时将请求数据从character_set_client转换为character_set_connection；

2. 进行内部操作前将请求数据从character_set_connection转换为内部操作字符集，其确定方法如下：

	- 使用每个数据字段的CHARACTER SET设定值；
	- 若上述值不存在，则使用对应数据表的DEFAULT CHARACTER SET设定值(MySQL扩展，非SQL标准)；
	- 若上述值不存在，则使用对应数据库的DEFAULT CHARACTER SET设定值；
	- 若上述值不存在，则使用character_set_server设定值。

3. 将操作结果从内部操作字符集转换为character_set_results。

###字符序###

- 字符序(Collation)是指在同一字符集内字符之间的比较规则；
- 确定字符序后，才能在一个字符集上定义什么是等价的字符，以及字符之间的大小关系；
- 每个字符序唯一对应一种字符集，但一个字符集可以对应多种字符序，其中有一个是默认字符序(Default Collation)；
- MySQL中的字符序名称遵从命名惯例：以字符序对应的字符集名称开头；以\_ci(表示大小写不敏感)、\_cs(表示大小写敏感)或\_bin(表示按编码值比较)结尾。例如：在字符序“utf8\_general\_ci”下，字符“a”和“A”是等价的。

###笔记###

- `SET NAMES`会更新client、connection、results三处的字符集设置；
- 修改特定字符集设置使用`set character_set_results=utf8`类似语法；
- 客户端提交的数据会被当作client字符集解析，然后转换为connection字符集，最后转换为server字符集，查询时server字符集转换为results字符集；
- 默认除了system使用utf8，其他均为latin1。此时客户端用任何字符集都被当成latin1，且在client->connection->server整个过程中不转换。能存储所有类型字符集，只要保证在输入和输出的编码一致就不会产生乱码。但缺点是无法以字符为单位来进行SQL操作，一般情况下将数据库和连接字符集都置为utf8是较好的选择；
- 更新数据时，如果转码出现不兼容（比如：latin1<->gb），则会提示错误（鸟哥博客说这里会替换为‘?’，但在我本机MySQL 5.6.12报1366错误，插入失败），更新不成功，否则成功转换。读取数据时，如果转码出现不兼容（比如：latin1<->gb），则会替换为‘?’显示，否则成功显示。


参考：  
1. [深入Mysql字符集设置](http://www.laruence.com/2008/01/05/12.html)  
2. [MySQL字符集乱码总结](http://blog.csdn.net/sunboy_2050/article/details/5130789)  
3. [MySQL：字符集支持](https://dev.mysql.com/doc/refman/5.1/zh/charset.html)