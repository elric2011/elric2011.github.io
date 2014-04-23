---
layout: post
title: "Nginx Location使用详解"
permalink: /a/nginx_location.html
description: ""
category: 
tags: [技术原创, nginx]
---
{% include JB/setup %}

官方文档
-----

文档链接：[http://wiki.nginx.org/NginxChsHttpCoreModule#location](http://wiki.nginx.org/NginxChsHttpCoreModule#location)

语法：`location [=|~|~*|^~] /uri/ {...}`

### 匹配逻辑：

1. Directives with the = prefix that match the query exactly. If found, searching stops.
2. All remaining directives with conventional strings. If this match used the ^~ prefix, searching stops.
3. Regular expressions, in the order they are defined in the configuration file.
4. If #3 yielded a match, that result is used. Otherwise, the match from #2 is used.

测试方法
-----

### 1. 写个test.php，用来输出规则到浏览器

	<?php
		echo $_GET['req'].’ -> '.$_GET['rule'];
	?>

### 2. 配置nginx，将规则传到test.php，显示

	location /uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/ last;
	}

	location ~* ^/test.php$ {
		include fastcgi.conf;
		fastcgi_pass 127.0.0.1:9000;
		fastcgi_param  SCRIPT_FILENAME $document_root/test.php;
	}

测试用例与结果
-----

### 1. 字符串匹配

* 匹配“/uri/”打头的url，大小写敏感
* 示例：`location / {...}` 或 `location /uri/ {...}`
* 多条规则间，取最长匹配的规则

测试用例：

	nginx rules：
	location / {
		rewrite (/.*) /test.php?req=$1&rule=[str]/ last;
	}
	location /uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/ last;
	}
	location /uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/sub/ last;
	}	

	test results：
	/uri/ss -> [str]/uri/
	/uri/sub/ss -> [str]/uri/sub/

### 2. 正则匹配

* ~ 表示大小写敏感正则，~* 表示大小写不敏感正则
* 示例：`location ~ ^/test/(.*)$ {...}` 或 `location ~* ^/uri/ {...}`
* 多条正则规则间，取最先匹配的规则
* 优先级高于字符串匹配

测试用例1：

	nginx rules：
	location ~* ^/uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[reg]/uri/ last;
	}
	location ~* ^/uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[reg]/uri/sub/ last;
	}	
	
	test results：
	/uri/xx -> [reg]/uri/
	/uri/sub/xx -> [reg]/uri/

测试用例2：

	nginx rules:
	location /uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/sub/ last;
	}
	location ~* ^/uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[reg]/uri/ last;
	}
	
	test results：
	/uri/sub/xx -> [reg]/uri/

### 3. ^~ 前缀

* 优先级提升的字符串匹配
* 示例：`location ^~ /uri/ {...}`
* 多条规则间，取最长匹配的规则
* 匹配成功后，则不再匹配正则

测试用例1：

	nginx rules:
	location /uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/ last;
	}
	location ^~ /uri/sub/1/ {
		rewrite (/.*) /test.php?req=$1&rule=[^~str]/uri/sub/1/ last;
	}
	location ~* ^/uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[reg]/uri/sub/ last;
	}
	
	test results:
	/uri/xx -> [str]/uri/
	/uri/sub/xx -> [reg]/uri/sub/
	/uri/sub/1/xx -> [^~str]/uri/sub/1/

测试用例2：

	nginx rules:
	location ^~ /uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[^~str]/uri/ last;
	}
	location  /uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/sub/ last;
	}
	location ~* ^/uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[reg]/uri/ last;
	}

	test results:
	/uri/xx -> [^~str]/uri/
	/uri/sub/xx -> [reg]/uri/

测试用例3：

	nginx rules:
	location /uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/ last;
	}
	location ^~ /uri/ {
		rewrite (/.*) /test.php?req=$1&rule=[^~str]/uri/ last;
	}
	
	test results:
	nginx启动报错：duplicate location

### 4. = 前缀

* 全等匹配
* 示例：`location = /crossdomain.xml {...}`
* 优先级最高

测试用例：

	nginx rules:
	location ^~ /uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[str]/uri/sub/ last;
	}
	location ~* ^/uri/sub/ {
		rewrite (/.*) /test.php?req=$1&rule=[reg]/uri/sub/ last;
	}
	location = /uri/sub/equal {
		rewrite (/.*) /test.php?req=$1&rule=[equal]/uri/sub/equal last;
	}	
	
	test results:
	/uri/sub/xx -> [str]/uri/sub/
	/uri/sub/equal -> [equal]/uri/sub/equal

总结
-----

### 再看官方文档：

1. Directives with the = prefix that match the query exactly. If found, searching stops.
2. All remaining directives with conventional strings. If this match used the ^~ prefix, searching stops.
3. Regular expressions, in the order they are defined in the configuration file.
4. If #3 yielded a match, that result is used. Otherwise, the match from #2 is used.

### 解读：

1. 先匹配 = 前缀，如果匹配到，则停止匹配，取该location
2. 匹配所有字符串规则（包括无前缀 和 ^~ 前缀），从中取出最长匹配的一条（可以理解有个变量，依次扫描规则，遇到匹配的则与变量中规则对比，如果长度更大，则顶替变量中的规则。最终变量里的规则就是第 2 步的匹配结果）。这一步中，字符串规则不允许重复，否则启动nginx会报错，而且有无 ^~ 前缀是无影响的。例如同时有两个规则：`location /uri/sub/ {...}` 和 `location ^~ /uri/sub/ {...}`，启动会报错。
3. 如果第二步的结果是 ^~ 前缀的，则不再进行正则匹配，直接取该结果。否则，按规则定义的顺序依次匹配正则，一旦遇到匹配的，则停止匹配，取该正则的location。如果匹配下来没找到匹配的规则，则取 第二步 的结果。
（正则的匹配只有 match 和 not match 两种结果，没有长度的概念，所以是取最先匹配到的）

### 总结：

1. 尽可能地使用正则匹配，字符串匹配规则可以用正则替代，正则匹配应按粒度从细到粗配置；
2. 尽量避免 ^~ 前缀，它会使规则变得复杂，结果难以预测；
3. 一般预留一个 `location / {...}`，表示没有匹配到任何规则时的默认行为；
4. 对于唯一的路径，使用 = 前缀，它总能保证不被覆盖。

（完）