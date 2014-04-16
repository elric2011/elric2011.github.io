---
layout: post
title: "Robot Framework学习笔记"
permalink: /a/robot_framework_study_note.html
description: ""
category: 
tags: [技术原创, 学习笔记]
---
{% include JB/setup %}

First Case
------

编写一个简单测试用例，保存为hello.txt

	*** Settings ***
	Library   Selenium2Library

	*** Test Cases ***
	Example
	   Open Browser  http://www.google.com
	   [Teardown]  Close Browser

Mac开发环境搭建
------
系统环境：

- Mac OS X 10.8.3
- 自带 Python 2.7.2

安装 pip, see [Installing the Package Tools](http://guide.python-distribute.org/installation.html)

	wget http://pypi.python.org/packages/source/p/pip/pip-0.7.2.tar.gz
	tar xzf pip-0.7.2.tar.gz
	cd pip-0.7.2
	sudo python setup.py install
	
	# install distribute
	wget http://python-distribute.org/distribute_setup.py
	python distribute_setup.py
	

安装 robotframework

	wget https://robotframework.googlecode.com/files/robotframework-2.7.7.tar.gz
	tar zxvf robotframework-2.7.7.tar.gz
	cd robotframework-2.7.7
	sudo python setup.py install
	# test installation
	pybot --version
	
	# install selenium2library
	sudo pip install robotframework-selenium2library
	
运行测试用例hello.txt

	pybot hello.txt
	
robot默认使用Firefox作为启动浏览器，如果要使用其他浏览器（如Chrome），需要安装相应的Driver

	wget https://chromedriver.googlecode.com/files/chromedriver2_mac32_0.8.zip
	unzip chromedriver2_mac32_0.8.zip
	sudo cp chromedriver2_mac32_0.8/chromedriver /usr/bin

通过Python测试Chromedriver：

	$ python
	>>> from selenium import webdriver
	>>> driver = webdriver.Chrome()
	>>> driver.get("http://www.google.com")

修改case，重新执行 pybot hello.txt（如果报浏览器找不到错误，尝试重启系统）

	Open Browser  http://www.google.com  Chrome


安装 RobotFramework IDE，see [Installation Instructions](https://github.com/robotframework/RIDE/wiki/Installation-Instructions)：
	
	wget https://robotframework-ride.googlecode.com/files/robotframework-ride-1.1.tar.gz
	tar zxvf robotframework-ride-1.1.tar.gz
	cd robotframework-ride-1.1
	sudo python setup.py install

Windows开发环境搭建
------

### 安装robot ###

1. 安装 python2.7-win32（假设安装到D:\tools\Python27）
2. 设置 PATH=$PATH;D:\tools\Python27;D:\tools\Python27\Script
3. 安装 setuptools
4. 安装 decorator-3.4.0，下载源代码解压，**管理员权限运行**：`python .\setup.py install`
5. 安装 selenium-2.32.0
6. 安装 robotframework-2.7.7
7. 安装 robotframework-selenium2library
8. 解压 chromedriver.exe到系统路径下，例如python安装目录

至此，已经可以运行hello.txt的case

### 安装robotide(RIDE) ###

1. 安装 wxPython2.8-win32-unicode-2.8.12.1-py27
2. 安装 robotframework-ride-1.1.win32

在运行 `python Script\ride.py` 时，可能报错提示“unable to open database file”，原因可能是系统的 AppData 目录路径包含非ANSI字符，解决方法：

修改robotide源码的settings.py模块（在Lib\site-packages\robotide\preferences下），找到下面这行：

	SETTINGS_DIRECTORY = os.path.join(os.environ['APPDATA'], 'RobotFramework', 'ride')

将 `os.environ['APPDATA']` 修改成不含非ANSI字符的路径（建议先备份），如下：

	SETTINGS_DIRECTORY = os.path.join('D:\data', 'RobotFramework', 'ride')

### 安装AutoItLibrary ###

AutoIt是Windows下的测试Windows窗口应用程序的框架，参考：[官方地址](http://www.autoitscript.com/site/autoit/)、[安装教程](http://cgmblog.sinaapp.com/html/260.html)

1. 安装 pywin32-218.win32-py2.7
2. 安装 AutoItLibrary
3. 64位python可能需要额外安装 AutoIt V3

安装完成后执行如下Case：

	*** Settings ***
	Library    AutoItLibrary
	
	*** Test Cases ***
	打开计算器
		Run  calc.exe
		Wait For Active Window   计算器


参考
------
- [BuiltIn Keywords](http://robotframework.googlecode.com/hg/doc/libraries/BuiltIn.html?r=2.6.1)
- [Selenium2Library Keywords](https://github.com/rtomac/robotframework-selenium2library/blob/master/doc/Selenium2Library.html)
- [Selenium2Library On GitHub](https://github.com/rtomac/robotframework-selenium2library)
- [RIDE Overview](http://blog.codecentric.de/en/2012/01/robot-framework-ide-ride-overview/)
- [AutoIt doc](http://www.jb51.net/shouce/autoit/index.html)

问题
------
1. Mac下运行ride.py后，总是异常退出，原因未查明
