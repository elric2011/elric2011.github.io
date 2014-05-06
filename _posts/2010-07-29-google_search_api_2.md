---
layout: post
title: "应用程序获取Google搜索结果（二）"
permalink: /a/google_search_api_2.html
description: "介绍如果通过程序去自动获取Google搜索结果，介绍Google Ajax API的使用方法，以及相应的Java程序示例。"
category: 技术原创
tags: [搜索, google, search, Java, Ajax]
---
{% include JB/setup %}

[上篇](/a/google_search_api_1.html)简单介绍了一下利用Google Ajax Search API实现在Java应用程序中访问Google搜索并获取搜索结果的方法原理，这篇将主要贴出几段代码，俩目的：

一，作为上一篇的那些理论知识的一个实例，直观地展现一下到底怎么玩；

二，通过面向对象的思想（吾尚为菜鸟级人物，还谈不上思想，求原谅），将这个API进行了简单的封装，实现了一个GoogleSearcher类，以屏蔽URLConnection连接网络、解析JSON字符串这些复杂的细节。（小说一下，这些代码谈不上任何版权问题，你要是觉得还方便就随便拿着用）

GoogleSearcher类：

    /**
     * @author Elric
     */
    package google;

    import java.io.*;
    import java.net.*;

    // import org.json.JSONException;
    import org.json.JSONObject;
    import org.json.JSONArray;

    public class GoogleSearcher {

        public GoogleSearcher() {
            this(4);
        }

        public GoogleSearcher(int amount) {
            this.amountOfResults = amount;
            this.searchResults = null;
        }

        public void search(String keyword) {
            this.keyword = keyword;
            searchResults = new GoogleSearchResult[amountOfResults];
            URL url = null;
            String urlString = generalUrl + keyword.replaceAll("\\s+", "%20");
            try {
                JSONArray jsonArr = null;
                int index = 0;

                for (int i=0; i<amountOfResults; ++i) {
                    if (jsonArr == null || index >= jsonArr.length()) {
                        url = new URL(urlString + "&start=" + i);
                        // System.out.println(url);
                        URLConnection conn = url.openConnection();
                        conn.addRequestProperty("Referer",
                                "http://www.informatik.uni-trier.de/~ley/db/");

                        StringBuilder builder = new StringBuilder();
                        BufferedReader br = new BufferedReader(
                                new InputStreamReader(conn.getInputStream(), "utf-8"));
                        String line;
                        while ((line = br.readLine()) != null) {
                            builder.append(line);
                        }
                        br.close();
                        // System.out.println(builder.toString());
                        jsonArr = new JSONObject(builder.toString()).
                                getJSONObject("responseData").getJSONArray("results");
                        index = 0;
                    }
                    JSONObject jsonObj = jsonArr.getJSONObject(index);
                    GoogleSearchResult result = new GoogleSearchResult();
                    result.setGSearchResultClass(
                            jsonObj.getString("GsearchResultClass"));
                    result.setUnescapedUrl(jsonObj.getString("unescapedUrl"));
                    result.setUrl(jsonObj.getString("url"));
                    result.setVisibleUrl(jsonObj.getString("visibleUrl"));
                    result.setCacheUrl(jsonObj.getString("cacheUrl"));
                    result.setTitle(jsonObj.getString("title"));
                    result.setTitleNoFormatting(
                            jsonObj.getString("titleNoFormatting"));
                    result.setContent(jsonObj.getString("content"));
                    searchResults[i] = result;

                    ++index;
                }
            } catch (Exception e) {
                searchResults = null;
                e.printStackTrace();
            }
        }

        /* public static void main(String[] args) {
            GoogleSearcher gs = new GoogleSearcher(16);
            gs.search("Paris Hilton");
            GoogleSearchResult[] rs = gs.getSearchResults();
            for (int i=0; i<rs.length; ++i) {
                System.out.println(rs[i].getUnescapedUrl());
            }
        } */

        public String getKeyword() {
            return keyword;
        }
        public int getAmountOfResults() {
            return amountOfResults;
        }
        public void setAmountOfResults(int amountOfResults) {
            this.amountOfResults = amountOfResults;
            this.searchResults = null;
        }
        public GoogleSearchResult[] getSearchResults() {
            return searchResults;
        }

        protected String keyword;
        protected int amountOfResults;
        protected GoogleSearchResult[] searchResults;
        protected final String generalUrl =
                "http://ajax.googleapis.com/ajax/services/search/web?v=1.0&rsz=large&q=";
    }
 

GoogleSearchResult类：

    /**
     * @author Elric
     */
    package google;

    public class GoogleSearchResult {

        public String getSearchResultClass() {
            return gSearchResultClass;
        }
        public void setGSearchResultClass(String searchResultClass) {
            this.gSearchResultClass = searchResultClass;
        }
        public String getUnescapedUrl() {
            return unescapedUrl;
        }
        public void setUnescapedUrl(String unescapedUrl) {
            this.unescapedUrl = unescapedUrl;
        }
        public String getUrl() {
            return url;
        }
        public void setUrl(String url) {
            this.url = url;
        }
        public String getVisibleUrl() {
            return visibleUrl;
        }
        public void setVisibleUrl(String visibleUrl) {
            this.visibleUrl = visibleUrl;
        }
        public String getCacheUrl() {
            return cacheUrl;
        }
        public void setCacheUrl(String cacheUrl) {
            this.cacheUrl = cacheUrl;
        }
        public String getTitle() {
            return title;
        }
        public void setTitle(String title) {
            this.title = title;
        }
        public String getTitleNoFormatting() {
            return titleNoFormatting;
        }
        public void setTitleNoFormatting(String titleNoFormatting) {
            this.titleNoFormatting = titleNoFormatting;
        }
        public String getContent() {
            return content;
        }
        public void setContent(String content) {
            this.content = content;
        }

        protected String gSearchResultClass;
        protected String unescapedUrl;
        protected String url;
        protected String visibleUrl;
        protected String cacheUrl;
        protected String title;
        protected String titleNoFormatting;
        protected String content;
    }
 

代码说明：

GoogleSearchResult类，这个类定义了很多字段，这些字段分别对应JSON字符串中每条搜索记录的各个属性（名字都一样），然后定义了各字段的getter和setter，仅此而已，因此这个类就是将每条搜索记录封装了起来，每个GoogleSearchResult对象就对应一条搜索记录。

GoogleSearcher类，我们从字段着手，先看generalUrl字符串，这就是Ajax API提供给我们的URL，只不过关键词这个属性我们还没有填，等实际运行的时候游用户动态添加，另外这个URL里多了个rsz属性，这个属性的作用我们待会再解释。keyword字段，很显然，就是用户输入的关键词。amountOfResults，这是干什么的呢？当我们在浏览器中Google一个关键词后，Google返回的肯定不止一条搜索结果，这个API也一样，也是返回多条记录，而amountOfResults这个字段就是用于设置记录的条数的。这也解释了为什么searchResults字段是一个GoogleSearchResult数组，而不是一个GoogleSearchResult对象。这个数组的长度就是amountOfResults。

再看构造器，构造器里面初始化了amountOfResults字段，默认为4条。注意，这里并没有在构造器中初始化keyword字段，因为我希望将这个GoogleSearcher类设计成一个Searcher可以进行多次搜索，搜索多个关键词，所以我并没有选择在初始化的时候设置keyword。

接下来就是我们的主体，search()方法，这个方法就是我们要用的搜索方法了，参数keyword就是要搜索的关键词。这个方法返回的是void，在调用该方法后需要调用GoogleSearchResult()方法来获取搜索结果。我们进入search()来看一下有哪些需要注意一下。

1. 第28行，将关键词中的空格替换成"%20”，在URL中空格需要用转义字符"%20”表示；
2. 第33行，for循环，很显然是获取amountOfResults条搜索结果。下面的if是做什么的呢？可以看到，连接到网络以及从网络获取搜索结果的部分都是在if中，而且在for循环中可能需要多次访问网络。这是因为Ajax API是以页的形式返回搜索结果的，每一页中只包含若干条搜索记录，如果需要更多的搜索记录，就需要多次连接网络。如何使这些不同的连接能获得不同的记录呢？通过URL中"start”属性设置。这个属性用于设置当前获取的页是从第几条记录开始的。例如我们获取了一页，这一页里有第0~3条记录，现在要获取第4条记录，就需要再访问一次网络，并将URL中start设为4，我们会再获得一页，其中包含了第4~7条记录，以此类推。这时候就可以解释前面提到的rsz属性了，这个属性就是设置一页中记录的条数，large表示每页8条记录。更多关于URL中可用属性的说明可访问[http://code.google.com/apis/ajaxsearch/documentation/reference.html#_intro_fonje](http://code.google.com/apis/ajaxsearch/documentation/reference.html#_intro_fonje)。
3. 第50行，解析JSON字符串，获取其中的results部分，这部分是一个包含多条搜索记录的JSON数组，第54行是获取JSON数组中代表每一条搜索记录的JSON对象，后面几行就是提取JSON对象中的信息，并封装到一个GoogleSearchResult对象中，形成一个搜索记录。第66行，将这条搜索记录存放到searchResults数组中，以便通过调用GoogleSearchResult()方法来获取这amountOfResults条搜索记录。


补充说明：

* 这些代码中只给出了一个基本的搜索功能的实现，Ajax API提供的URL还有一些属性可以设置，同时返回的JSON字符串中除搜索结果以外还包含其它信息，如果你的应用程序需要用到这些属性或者这些信息，可以参考Google提供的文档。前文中包含了部分文档页的链接。
* 要知道搜索只是第一步，我们通过浏览器Google一个关键词时是需要打开搜索结果中链接的页面，从中找出我们想要的信息。如何让我们的应用程序从搜索结果所指向的页面中获取我们需要的信息（有时我们还需要从搜索结果中挑选出哪一条记录才是指向我们想要的页面），这些才是需要大学问的。