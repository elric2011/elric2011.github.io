---
layout: post
title: "应用程序获取Google搜索结果（一）"
permalink: /a/google_search_api_1.html
description: "介绍如果通过程序去自动获取Google搜索结果，介绍Google Ajax API的使用方法，以及相应的Java程序示例。"
category: 技术原创
tags: [搜索, google, search, Java, Ajax]
---
{% include JB/setup %}

> 文章是大学时候写的，现在读起来就俩字：真啰嗦

当我们需要了解一些信息是，可以通过Google、百度等搜索引擎来找到我们想要的信息，有时候还有意外的惊喜。同样，我们的应用程序有时候也需要从互联网上获取一些信息，这时候我们同样可以利用Google。

Google提供给我们Ajax Search API，其主要目的是让建站人员能够利用JavaScript将Google搜索嵌入到自己的网页中，以便用户在访问我们的网页的同时也能耍耍他们的玩意。这个API也提供Java和Flash等支持，本文主要介绍在Java应用程序中如何利用Ajax API来获取Google搜索结果。有关Google Ajax Search API的详细内容，可以查看[官方文档](http://code.google.com/apis/ajaxsearch/documentation/)。

所谓搜索，无非就是我们提交给Google一个关键词，然后Google服务器给我们返回它搜索的结果。因此我们只要解决两个问题，一是如何将我们要搜索的关键词告诉Google，另一个则是如何获取服务器的搜索结果，并分析这个结果，从而得到我们想要的信息。

很显然，我们要先建立到服务器的网络连接，通过这个连接将关键词传送给服务器，同样是通过这个连接，获取服务器的响应，即搜索结果。Java提供了URLConnection类来实现到web服务器的连接，我们可以通过它来连接到一个URL，并且获取服务器返回的结果（包括静态的资源及服务器端动态处理结果）。关键是我们需要连接到谁呢？这就是Ajax Search API提供给我们的东东了。Ajax API提供给我们一个URL，这个URL有一些参数，我们只要将要搜索的关键词附加到这些参数中，然后建立到这个URL的连接，这是我们就向服务器发送了一个HTTP请求，服务器经处理后给出响应，响应的就是搜索结果了。这个URL的格式是：`http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=XXX`。你只要将"XXX”替换为你要搜索的关键词既可。

再看第二个问题，如何获取搜索结果，如何分析这个结果以获取想要的信息。URLConnection提供给了我们如何获取服务器响应的方法（通过流来读取响应数据），通过这些方法我们可以读取响应结果，或得了响应结果后，我们如何“读懂”这个结果呢？这就牵涉到服务器是以什么样的形式为我们传送响应结果的。Ajax API返回的响应是一个JSON字符串，所谓JSON字符串，就是一个用于表达、传输、存储数据的一个格式化字符串，它是由一系列“名-值”对组成，其中“值”又可以是一系列“名-值”对。通过下面的例子你就能看懂JSON字符串是个什么玩意儿了：

	{
		"firstName": "John",
		"lastName": "Smith",
		"age": 25,
		"address": {
			"streetAddress": "21 2nd Street",
			"city": "New York",
			"state": "NY",
			"postalCode": "10021"
		},
		"phoneNumber": [
			{ "type": "home", "number": "212 555-1234" },
			{ "type": "fax", "number": "646 555-4567" }
		]
	}
 

Ajax API提供给我们的JSON字符串形式如下：

	{
		"responseData": {
			"results": [
				{
					"GsearchResultClass": "GwebSearch",
					"unescapedUrl": "http://en.wikipedia.org/wiki/Paris_Hilton",
					"url": "http://en.wikipedia.org/wiki/Paris_Hilton",
					"visibleUrl": "en.wikipedia.org",
					"cacheUrl": "http://www.google.com/search?q\u003dcache:TwrPfhd22hYJ:en.wikipedia.org",
					"title": "\u003cb\u003eParis Hilton\u003c/b\u003e - Wikipedia, the free encyclopedia",
					"titleNoFormatting": "Paris Hilton - Wikipedia, the free encyclopedia",
					"content": "\[1\] In 2006, she released her debut album..."
				},
				{
					"GsearchResultClass": "GwebSearch",
					"unescapedUrl": "http://www.imdb.com/name/nm0385296/",
					"url": "http://www.imdb.com/name/nm0385296/",
					"visibleUrl": "www.imdb.com",
					"cacheUrl": "http://www.google.com/search?q\u003dcache:1i34KkqnsooJ:www.imdb.com",
					"title": "\u003cb\u003eParis Hilton\u003c/b\u003e",
					"titleNoFormatting": "Paris Hilton",
					"content": "Self: Zoolander. Socialite \u003cb\u003eParis Hilton\u003c/b\u003e..."
				},
				...
			],
			"cursor": {
				"pages": [
					{ "start": "0", "label": 1 },
					{ "start": "4", "label": 2 },
					{ "start": "8", "label": 3 },
					{ "start": "12","label": 4 }
				],
				"estimatedResultCount": "59600000",
				"currentPageIndex": 0,
				"moreResultsUrl": "http://www.google.com/search?oe\u003dutf8\u0026ie\u003dutf8..."
			}
		},
		"responseDetails": null,
		"responseStatus": 200
	}

相信从上面JSON字符串，你已经猜到一些眉目了，这里面就包含了我们平时通过浏览器所看到的Google搜索结果中的内容。

如何分析这个字符串呢？如何从里面提取出想要的信息呢？当然是利用现成的工具了，既然JSON提供了这种形式的表达数据的方法，那么它肯定提供了相应的parser（解析器）。相关类可以参考：[http://www.json.org/java/index.html](http://www.json.org/java/index.html)。

至此我们便得到了我们想要的搜索信息了，总结一下，大致流程是，通过URLConnection建立到`http://ajax.googleapis.com/ajax/services/search/web?v=1.0&q=XXX`（XXX指代你要搜索的关键词）的连接，然后获取响应结果，再用JSON的parser解析响应结果，提取所需信息。

下一章[应用程序获取Google搜索结果（二）](/a/google_search_api_2.html)给出Java实现的代码。