---
layout: post
title: 今天研究抓取 feed 到本地的折腾记录
tags: []
status: draft
type: post
published: false
meta: {}
---
最近 Google Reader 的 feed 越来越多，有保存价值的资源已经存不过来了。故研究怎样自动存储这些东西。

目的
将一些知名网页、Logo、CG 展示博客的文章收集到本地，最好自动标签，外加缓存图片，还能输出 pdf。

涉及工具
<ul>
	<li>XAMPP</li>
	<li>WordPress</li>
	<li>WordPress 插件 WP-o-Matic</li>
</ul>
1，先在本地搭建 PHP 服务器。用本地服务器是因为这玩意搁网上性质就类似垃圾站。另一原因是收集完了能立马打包走人。这里拿 XAMPP 整就行了。目前是 <a href="http://www.apachefriends.org/en/xampp-windows.html">v1.7.3 for Win</a>。

2，然后安装 WordPress，目前有 WP 中文团队做的 <a href="http://wfans.org/blog/2010/06/wordpress-3-0-chinese-version-released/">v3.0 简体中文版</a>。

3，然后安装 WP-o-Matic，自动采集插件还有很多，选这个是因为介绍中有写缓存图片的功能。在 WP 的插件一项中搜索同名关键字后安装。这东西要在 Firefox / Chrome 中启用。

4，开始核心内容。WP-o-Matic 原理是抓取用户设置的 feed，把内容提交到 WP 里，同时把图片缓存到 \wp-content\plugins\wp-o-matic\cache 路径下。如本地没有这个路径，要自己手动建立。

在 WP-o-Matic 里选择 Add campaign ，填入要抓取网站的 feed url，比如一个专搞 Showcase 的设计网站 <a href="http://www.instantshift.com/">instantShift</a> ，找到他们的 feed 地址 http://feeds2.feedburner.com/ishift 填进去，选项里主要设一下抓到后放到什么分类（标签），搁多长时间更新一次，还有一次抓取多少条目（设为0则抓取全部，人间凶器啊）。

一般来说这时点 fetch ，就应该搞到这些文章了。如果网站是拿 <a href="http://feedburner.google.com/">FeedBurner</a> 烧的 RSS ，会发现 WP-o-Matic 报一个 cURL 的错误，原因不明。猜测与 FeedBurner 被墙有关？但是我不会怎么在 XAMPP 的本地环境上再加一个代理翻出去，所以也没法证实。。

另一推测是 FeedBurner 封掉了常见的网页采集程序？同样没法证明。遂换用了 <a href="http://pipes.yahoo.com/pipes/">Yahoo Pipes</a> 从 FeedBurner 转烧 RSS，结果同样报错。那么换用 <a href="http://feed.feedsky.com/">FeedSky</a> 呢？发现 FeedBurner 封了 FeedSky 的爬取。。

那么好吧，用 Yahoo Pipes 转录 FeedBurner 的 RSS ，再拿 FeedSky 转录 Yahoo Pipes，总算折腾出来啦。

完之后，WP-o-Matic 的图片目录就开始哗啦哗啦的生成文件了。先到这，接下来研究怎么自动取得源网站的文章标签，还有怎么转换为pdf……
