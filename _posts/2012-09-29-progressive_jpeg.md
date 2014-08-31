---
layout: post
title: "渐进JPEG图片研究"
permalink: /a/progressive_jpeg.html
description: "分析对比普通jpeg与渐进jpeg的优缺点"
category: 技术原创
tags: [渐进jpeg, 图像处理]
---
{% include JB/setup %}



渐进jpeg特点：
在装入图像时，开始只显示一个模糊的图像，随着数据的装入，图像逐步变得清晰。


效果比较：（从左到右是图片的载入过程）


   * 普通jpeg（baseline jpeg）,示例图片http://ticktick-test.stor.sinaapp.com/t_bl.jpg：




   * 渐进jpeg（progressive jpeg），示例图片http://ticktick-test.stor.sinaapp.com/t_pg.jpg：



大小比较：
Stoyan Stefanov的测试统计结果显示，


   * 当图片<10k时，baseline的压缩比更高些（压出的图片更小）
   * 当图片>10k时，progressive的压缩比更高

点评商户单张图片页显示的700x700的图片在100k左右。

beta随机测试了1000张700x700图片，progressive比baseline平均节省4.65k，平均节省4.08%

性能对比：
Stoyan Stefanov在PC（Windows XP, 2GHz dual CPU, 500Mb RAM）上单独做图片jpeg输出操作的测试结果，

   * ImageMagick baseline (7 images/s)
   * ImageMagick progressive (5.5 images/s)

由于我们网站图片除了做图片输出，还要做缩放、裁剪、打水印操作，所以总的性能差距会相对更小。

（TODO 网站两种模式性能对比测试）

其它差异对比：


   * 早期的ie版本下，baseline还是一块块显示，progressive则要等图片完全载完后显示


综合对比：


  1. 针对我们网站的图片（大部分都>10k），在相同尺寸、相同质量的前提下，渐进式jpeg压出的图片更小些
  2. 要输出成渐进式jpeg，会有少量的性能损失
  3. 两者最大的区别在于用户体验上，当网络较慢时，渐进jpeg可以让用户先看到图片的轮廓，随着图片载入可以逐渐看到越来越清晰的图片，相比于一块块载入获得一个更好的体验（表现在商户图片上，用户可以提前判断是跳到下一张，还是等图片载完仔细查看）。
  4. 然而，这只是我们的分析，究竟用户买哪个的帐，还有待验证。


目前使用这种模式图片的网站：QQ空间相册、人人相册等。

参考文章：http://www.yuiblog.com/blog/2008/12/05/imageopt-4/