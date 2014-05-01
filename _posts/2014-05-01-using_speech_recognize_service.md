---
layout: post
title: "使用语音识别服务"
permalink: /a/using_speech_recognize_service.html
description: ""
category: 技术原创
tags: [语音识别, Google Speech API, 科大讯飞]
---
{% include JB/setup %}

近期因为项目需要，探索了两家提供语音识别服务的效果，分别是Google（[参考文档](http://blog.csdn.net/dlangu0393/article/details/7214728)）和国内的[科大讯飞](http://open.voicecloud.cn/)，探索的维度主要包括服务的使用方法以及识别的准确性。

Google
-----
Google很早就有自己的语音识别技术了，只不过没有开放出服务，有技术大拿通过挖掘Chrome中H5语音输入API相关的源码，找到了语音识别服务的接口，地址如下：

    https://www.google.com/speech-api/v1/recognize

通过将编码的语音文件POST到这个API，带上相关参数，API就会返回识别结果。

使用API示例：

    wget -O "GoogleSpeechAPI.txt" --user-agent="Mozilla/5.0" --post-file=record_0001.wav --header="Content-Type: audio/L16; rate=16000" "http://www.google.com/speech-api/v1/recognize?xjerr=1&client=chromium&lang=zh-CN&maxresults=1"


参数说明：

    # HTTP头：
    Content-Type: audio/L16; rate=16000  # 指定音频编码方式和采样频率
    # 请求参数:
    xjerr=1   # 不详，猜测为错误的标准
    client=chromium   # 客户端类型，这里是Chromium，Chrome也可行
    lang=zh-CN  # 语言类型，其余参考：http://msdn.microsoft.com/en-us/library/ms533052.aspx
    maxresults=1  # 最大返回结果数量，多个结果在hypotheses列表中保存。

返回的结果：

    {
        "status":0,    /* 结果代码，详细见本文结尾 */
        "id":"c421dee91abe31d9b8457f2a80ebca91-1",    /* 识别编号 */
        "hypotheses":    /* 结果 */
        [
            {
                "utterance":"这是语音识别测试",    /* 话语 */
                "confidence":0.91828251    /* 信心，即准确度 */
            }
        ]
    }

测试的时候之前可能是因为音频实际采样频率不正确，识别的结果完全不搭嘎，后来找了一款android的录音软件，设置音频了采样频率，结果开始靠谱。既然有HTTP API，要在Java中实现调用不会困难。至于识别的准确率，参考后面的比较。

### 总结： ###

* API可用，对于简短的语句，识别的效果还算不错
* 支持多种语言（测试了中英文都可以），但需要我们告诉她是什么语言
* 支持WAV、Flac、SPEEX等多种音频格式
* 返回结果中给出了准确度信息，这样我们可以根据这个消息做差异化处理
* 存在一些可能的风险：
  * 这个服务没有开放使用，法务上是否有问题
  * 没有测试过是否有使用次数限制
  * GFW

科大讯飞语音云
-----

科大讯飞在国内做语音识别、语音合成等方面算得上领先的公司（上市公司），讯飞语音云（[http://open.voicecloud.cn/](http://open.voicecloud.cn/)）是其为开发者提供的语音识别/合成开放平台。

讯飞语音云提供了包括Android/iOS/Windows/Linux/Java等多个平台的SDK，SDK主要起到访问server端相应服务的客户端的作用，我们这次使用的是Java SDK。

使用SDK前需要先在web端申请注册应用并提交审核，审核通过前有500次/天的调用限制，通过后无次数限制。

Java SDK使用方法：

1. 访问科大讯飞语音云主页，注册账号并申请一个appid
2. 下载Java SDK，将提供的jar引入到项目classpath中，运行时将提供的dll/so文件引入到JVM本地库目录（可以通过java.library.path添加）
3. 编写代码，调用SDK中相应接口发送音频流，并注册Listener接受识别结果，接口对音频流有限制，要求是`采样率16KHZ或者8KHZ，单声道，采样精度16bit的PCM或者WAV格式的音频`

识别语音Java代码：

    private void execute() throws Exception {
        SpeechRecognizer speechRecognizer = SpeechRecognizer.createRecognizer("appid=xxx");
        FileInputStream inputStream = new FileInputStream(new File("Record.wav"));
        byte[] bytes = IOUtils.toByteArray(inputStream);
        speechRecognizer.recognizeAudio(new MyListener(), bytes, "sms", null, null);
        Thread.sleep(20*1000);
    }

    private class MyListener implements RecognizerListener {
        @Override
        public void onVolumeChanged(int i) {
        }
        @Override
        public void onBeginOfSpeech() {
        }
        @Override
        public void onEndOfSpeech() {
        }
        @Override
        public void onResults(ArrayList results, boolean b) {
            String text = "";
            for(int i = 0; i < results.size(); i++)
            {
                RecognizerResult result = (RecognizerResult)results.get(i);
                text += result.text;
            }
            System.out.println("result:" + text);
        }
        @Override
        public void onEnd(SpeechError speechError) {
        }
        @Override
        public void onCancel() {
        }
    }

### 总结： ###

* 提供了跨平台的SDK，Java SDK使用起来还算方便（除了要在运行时引入本地库）
* 同Google一样，在识别简短语句时准确率还不错，当语音较长或者说话者说话断断续续时（停下来思考如何点评），识别率降低很多
* 仅支持中文，对输入的音频编码格式要求较严格
* 稳定性没做过验证
* 技术支持很到位，在群里问的问题基本都有人会回答

语音识别测试效果对比
-----

| 原文 | Google | 科大讯飞 |
|:-----:|:-----:|:-----:|
| 这是语音识别测试 | 这是语音识别测试 | 这是语音识别测试 |
