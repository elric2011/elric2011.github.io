---
layout: post
title: "centos下mysql源代码安装"
permalink: /a/centos_install_mysql_serserver.html
description: ""
category: 技术原创
tags: [mysql]
---
{% include JB/setup %}

centos下可以通过 `yum -y install mysql-server` 安装mysql服务器，下面介绍通过源码安装的方法

安装mysql： 
-----

	$ wget "http://cdn.mysql.com/Downloads/MySQL-5.1/mysql-5.1.65.tar.gz"

	$ groupadd mysql
	$ useradd -g mysql mysql

	$ CFLAGS="-O3" CXX=gcc CXXFLAGS="-O3 -felide-constructors -fno-exceptions -fno-rtti -fomit-frame-pointer -ffixed-ebp"

	$ tar zxvf mysql-5.1.65.tar.gz
	$ cd mysql-5.1.65/

	$ ./configure --prefix=/usr/local/mysql --enable-assembler --with-mysqld-ldflags=-all-static --with-client-ldflags=-all-static --with-unix-socket-path=/usr/local/mysql/tmp/mysql.sock --with-charset=utf8 --with-collation=utf8_general_ci --with-extra-charsets=all --with-plugins=innobase,federated

	$ make & make install


配置mysql： 
-----

	$ cp /usr/local/mysql/share/mysql/my-innodb-heavy-4G.cnf /etc/my.cnf

修改 /etc/my.cnf

	[mysqld]
	default-storage-engine=INNODB
	default-character-set=utf8

	$ /usr/local/mysql/bin/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/mysql/data --user=mysql

	$ /usr/local/mysql/bin/mysqladmin -u root -p password "xxx"
	（提示输入旧密码，直接回车）


启动 mysqld： 
-----

	$ /usr/local/mysql/bin/mysqld_safe --user=mysql --datadir=/data/mysql/data &

	mysql -u root -p
	mysql>use mysql;
	mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY "xxx" WITH GRANT OPTION;
	mysql>CREATE USER abc IDENTIFIED BY 'xxx';
	mysql>GRANT ALL PRIVILEGES ON test.* TO 'abc'@'%' IDENTIFIED BY "xxx" WITH GRANT OPTION;


停止 mysqld： 
-----

	$ /usr/local/mysql/bin/mysqladmin -u root -p shutdown