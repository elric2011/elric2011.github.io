---
layout: post
title: "Mac Java开发环境搭建"
permalink: /a/setup_mac_java_env.html
description: ""
category: 技术原创
tags: [Java, idea]
---
{% include JB/setup %}

Java
------

Mac预装了Java6 和 JDK1.6，通过命令行如下命令查看版本：

	$ java -version
	$ javac -version

测试：

	public class Main {
		public static void main(String[] args) {
			System.out.println("Hello DianPing!\n你好点评！");
		}
	}

编译，执行：

	$ javac Main.java
	$ java Main

可能出现的问题：

1. javac/java命令的输出信息乱码（试试javac -help）

	因为Mac默认是utf-8，Java默认输出编码是GB18030，下载一个Terminal替换品[iTerm]()，`View->Edit Current Session->Terminal`，将Encoding设置为GB18030。

2. 编译提示`编码 EUC_CN 的不可映射字符`

	.java源文件保存的字符集编码（如utf-8）与javac的默认编码不一致，使用命令`javac -encoding utf8 Main.java`


安装 Xcode Command Line Tools
------

这个package提供了很多Mac下的命令行工具（如：LLVM compiler, linker, Make），建议开发人员都装上（不管你是不是个Xcoder），装上后git、svn、mvn等命令都有了。

如果你装了Xcode，通过 `Preferences->Downloads->Command Line Tools` 安装。

如果没有Xcode，也可以通过[苹果开发者网站](https://developer.apple.com/downloads/index.action)下载安装。

### maven配置 ###

	$ sudo mv /usr/share/maven/conf/settings.xml /usr/share/maven/conf/settings.xml.bak
	$ cp ./settings.xml /usr/share/maven/conf
	// 复制一份到 ~/.m2 目录，供ide使用
	$ cp ./settings.xml ~/.m2


安装配置Tomcat
------

Tomcat免安装，下载解压即可（假设解压到TOMCAT_HOME），[下载地址](http://tomcat.apache.org/download-60.cgi)

为脚本文件增加权限：

	$ chmod +x TOMCAT_HOME/bin/*.sh

解决中文编码问题，修改`TOMCAT_HOME/conf/server.xml`：

	<Connector port="8080" …> 修改成 <Connector port="8080" … URIEncoding="UTF-8"/>
	
	<Connector port="8009" …> 修改成 <Connector port="8009" … URIEncoding="UTF-8"/>

点评大部分项目的日志输出路径都设置在 `/data` 目录下，当在本地启动一个点评web应用后，应用需要往该目录下写文件，因此请确保改目录具有写权限！设置方法：

	$ sudo mkdir /data
	$ sudo chown ${yourloginname} /data

Eclipse
------

[下载地址](http://www.eclipse.org/)


Intellij IDEA
------

[下载地址](http://www.jetbrains.com/idea/download/)，安装过程中，插件选择全部安装

### 注册：###

idea有免费试用版，基本功能都可以用，但貌似不能集成Tomcat部署web应用。正式版收费，经济条件允许建议去官网购买正版，其他同学可以去网上搜搜～

### 设置 ###

- 设置编码：`Preferences->File Encoding`，IDE和Project的encoding都选择UTF-8
- 设置Tomcat Server：`Preferences->Application Servers->点“+”->Tomcat Server`，选择Tomcat的路径（如`~/Downloads/apache-tomcat-6.0.36`）


### 导入项目，运行 ###

1. `File->Import Project…->选中piccenter-web/pom.xml->一直next`
2. 在SDK选择页面，`点击“+”->JDK->选择“/System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home”`
3. `Run->Edit Configurations->点“+”->Tomcat Server->Local`
4. 输入`Name`，如：web
5. 弹出框下方会提示`Warning:No artifacts marked for deployment`，点击`Fix`，选择`piccenter-web:war explored`，点Ok
6. 启动Server，启动完成后浏览器输入：[http://localhost:8080/index.jsp](http://localhost:8080/index.jsp)

