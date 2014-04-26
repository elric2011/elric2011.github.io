---
layout: post
title: "mysql federated引擎实现跨库数据处理"
permalink: /a/mysql_federated_engine.html
description: ""
category: 技术原创
tags: [mysql, federated]
---
{% include JB/setup %}

需求背景： 
-----
mysql涉及到跨服务器实例的关联数据处理，一般的做法是将A库上的某几张表导到B库临时表，然后在B库join。在数据量很大时，导数据很浪费时间，而且当还要将B上的结果写回A时，这个方案就显得很复杂了。

federated引擎介绍： 
-----
federated引擎允许在A库上创建一个B库上某个表T的镜像表FED_T，在FED_T上的select、insert、update、delete等操作，都实际作用B库上的表T。实际上，在A库服务器的数据库目录只创建一个表定义文件（.frm扩展名），而无数据文件，因为数据都存在B库上。

官方描述：[http://dev.mysql.com/doc/refman/5.5/en/federated-storage-engine.html](http://dev.mysql.com/doc/refman/5.5/en/federated-storage-engine.html)

安装配置federated引擎： 
-----

### 1.检查是否该引擎 ###

	mysql>show engines;
	
	Engine | Support
	CSV | YES
	FEDERATED | YES
	MyISAM | YES
	InnoDB | DEFAULT

如果无FEDERATED一行，说明安装mysql时没安装该引擎，在安装mysql时应加上configure参数`--with-plugins=federated`
（可以在不重装mysql前提下添加该引擎，方法参考：[http://cau99.blog.51cto.com/1855224/425566](http://cau99.blog.51cto.com/1855224/425566)）

如果Support=NO，则应在my.cnf文件里加上fedrated配置重启mysql

	[mysqld]
	federated

### 2.新建federated引擎的表 ###

	CREATE TABLE `FED_T` (
	`PicID` INT(11) NOT NULL AUTO_INCREMENT,
	`Url` VARCHAR(255) NOT NULL DEFAULT '' COMMENT '图片相对路径',
	....
	PRIMARY KEY (`PicID`)
	) ENGINE=FEDERATED DEFAULT CHARSET=utf8 CONNECTION='mysql://root:xxx@192.168.33.11:3306/dianping/pc_picture';

做法是把B库上的T表的定义语句copy过来

* ENGINE=FEDERATED：指定使用federated引擎
* CONNECTION：指定目标数据库表的连接字符串，格式：`mysql://username[:password]@hostname[:port]/database/tablename`（username是目标服务器上的账号）

### 3.测试 ###

	SELECT * FROM Fed_PC_Picture LIMIT 1;

如果报如下错误，请检查连接字符串是否正确，以及目标服务器该账号是否对A服务器授权

	Error Code: 1429
	Unable to connect to foreign data source

接下来，B库上的T表 和 A库的表就犹如在一个库，导数据方便很多~