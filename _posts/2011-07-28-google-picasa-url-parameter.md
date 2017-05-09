---
layout: post
title: Google Picasa图片外链的url参数
tags:
- ExtLink
- Google
- Photo
- Picasa
- URL
- 创作
status: publish
type: post
published: true
meta: {}
---
最近我重写了批量外链相册图片的Javascript工具，顺便研究了一下Google Picasa外链图片url的规律。

首先说明，Picasa相册链出图片的大小是由一小段url控制的，比如<a href="http://picasaweb.google.com/probeprobe/Summer_cocktail#5317452960761205394">这张图片</a>，其外链地址是
http://lh4.googleusercontent.com/-a0YNVIdgIyM/SctjS3QFUpI/AAAAAAAAA2M/7LtzZgRE6Y4/<span style="color:#ff0000;">s400/</span>Summer_cocktail_by_iuneWind.jpg
图片显示为
<img src="http://lh4.googleusercontent.com/-a0YNVIdgIyM/SctjS3QFUpI/AAAAAAAAA2M/7LtzZgRE6Y4/s400/Summer_cocktail_by_iuneWind.jpg" alt="" />
将这个url从红色的"s400/"处拆开：
"s400/"之前的部分是图片本身的路径，只有这段url时链出原图(不超过1600px)。
"s400/"用来控制图片大小，后面必须有斜杠。
"s400/"之后的部分仅描述了文件名，没实际作用。

目前发现的规律总结如下：<!--more-->

1，Picasa相册可以外链1600px以内的任何图幅。进入自己相册的单张图片页面，会看到有5种外链图幅，分别对应参数"s144" "s288" "s400" "s640" "s800"。实际上可以使用1600px之内的任何图幅，即便写上"s377"这么奇怪的参数也会生成对应的图。
<img src="http://lh3.googleusercontent.com/-NQMzyMNX0sc/TjFQLYPkaQI/AAAAAAAAAr8/_ArQzSsfJ0M/s600/picasa_album_url_01.png" alt="" />

2，若指定的尺寸比原图大，则链出原图。若指定的尺寸超过1600会报错。

3，指定的尺寸都是限制图片长宽中较大者。比如800x600的图片按"s600"外链，则缩小到600x450；400x600的图片按"s300"外链，则缩小到200x300。未发现仅控制长度(或宽度)的外链办法。

4，"-c"或"-p"表示裁切为正方形，这样使用："s128-c"或"s256-p"。"-c"与"-p"可能是cut和portrait的意思，没看出两者间的区别。如需要裁切但不想指定图幅，可直接使用"c"或"p"。
<img src="http://lh4.googleusercontent.com/-a0YNVIdgIyM/SctjS3QFUpI/AAAAAAAAA2M/7LtzZgRE6Y4/s160-c/Summer_cocktail_by_iuneWind.jpg" alt="" />

5，"-o"可以在图上加个视频播放按钮……如需要加按钮但不想指定图幅，可直接使用"o"。
<img src="http://lh4.googleusercontent.com/-a0YNVIdgIyM/SctjS3QFUpI/AAAAAAAAA2M/7LtzZgRE6Y4/s160-o/Summer_cocktail_by_iuneWind.jpg" alt="" />

6，形如"t***"的参数(以t开头，后加任意数字或字母)都可以链出，图幅为原图。

7，"-a" "-b" "-d" "-k" "-r" "-v"都可以组合在"s128"后面，或直接使用这个字母，链出图幅为原图。不知道Picasa预留这些字母是干什么用的。

完
