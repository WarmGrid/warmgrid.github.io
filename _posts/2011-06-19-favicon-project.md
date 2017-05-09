---
layout: post
title: Favicon项目
tags: []
status: draft
type: post
published: false
meta: {}
---
<div>
<h2>显摆一下拿 css3 做的标签样式</h2>
08月 12, 2010 作者：<a title="probe301 发布" href="http://probe301.wordpress.com/author/probe301/">probe301</a>

</div>
<div>
<div id="msgcns!5E902A513D05D6FA!675">html部分只用一个span元素，如
&lt;span class=’iconTag tagGreen’&gt;green&lt;/span&gt;
所有样式都以css3实现！<img src="http://lh6.ggpht.com/_Rqi-iDrPknA/TGP08tscevI/AAAAAAAAAFg/jHvu6pHzbw0/s800/tag%20snapshot.png" alt="" />

圆点拿 :before :after 伪元素模拟的，灵感来自
<a href="http://www.ruanyifeng.com/blog/2010/04/css_speech_bubbles.html">http://www.ruanyifeng.com/blog/2010/04/css_speech_bubbles.html</a>

做这个的原因是，最近在研究怎样批量收集设计资源博客的 Showcase 文章，
所以要先去研究怎样拿 Yahoo Pipes 合烧相关主题的文章 RSS，
但测试时发现 Logo 分类误打误撞烧进来一篇 Favicon (网站地址栏的16x小图标) 的展示文章，
然后觉得这个很好啊，可以整一个全的啊，
因此打算先搞一个 Favicon 的资源展示网页，
但发现没有系统的分类不好管理如此多的数据，
所以要整一个标签的模式，拿颜色，形状等 tag 从不同方面描述之，同时可以让用户无刷新做一些过滤，随机载入等操作，
所以就把 tag 的样式先鼓捣出来了…………
底下是预备整的样子，已收集 300+，数字代表标签数量，悬停则展开标签和网址链接。
一坨叉叉是因为图标从原网站动态载入的，有些站已经挂了。。所以正在整一个本地的缓存，先从本地指定文件夹载入，没有再跑原始站点去找。

<img src="http://lh5.ggpht.com/_Rqi-iDrPknA/TGP08sfHtZI/AAAAAAAAAFk/4VwHck9GATc/s800/favicon%20snapshot.png" alt="" />

<img src="http://lh3.ggpht.com/_Rqi-iDrPknA/TGP08qa1S-I/AAAAAAAAAFo/o83EgK2vn2M/s800/favicon1%20snapshot.png" alt="" />

还有些可以折腾的地方包括
0，把 Delicious 收藏夹的批量弄出来
1，借 Flickr 的照片颜色分析 API 自动加颜色标签
2，用户编辑和上传
3，浏览器端，弄一个“图标 – 网址”的连线游戏
<div>
<h2>favicon 展示有个雏形了</h2>
08月 17, 2010 作者：<a title="probe301 发布" href="http://probe301.wordpress.com/author/probe301/">probe301</a>

</div>
<div>
<div id="msgcns!5E902A513D05D6FA!723"><a href="http://www.astralwind.org/project/faviconWall/">http://www.astralwind.org/project/faviconWall/</a>花了最近10天大部分边角碎料时间。

目前有如下功能，
兼容Firefox和Chrome，
提供多组favicon来载入，
缩略图模式和列表模式转换，
简单的状态统计，
列表模式可编辑Tag，
编辑完了点生成txt，如果人肉把alert的内容存下来，可以当载入的txt数据用……

一直在本地测试的，上传后发现服务器在同一时间内有请求数量限制，结果白把图标缓存了，还是从来源网站获得。
另一问题是，因为有些著名Web2.0网站的url，所以时不时的撞墙撞墙撞墙，囧。

下一步计划，整理网页风格样式，以及编辑txt数据……

<img src="http://www.astralwind.org/project/faviconWall/preview.png" alt="" />
<div>
<h2>为完成 favicon 计划，研究了一下如何展示关联内容</h2>
08月 25, 2010 作者：<a title="probe301 发布" href="http://probe301.wordpress.com/author/probe301/">probe301</a>

</div>
<div>
<div id="msgcns!5E902A513D05D6FA!747">嗯嗯，其实是想把 favicon 展示页做成一面可以不断拖动的墙，
在边界处依据一个favicon的标签或分类找出相关度最高的其他favicon，动态填到画布上。所以又一次为了半途中的目的启动了另一个计划。。
初衷是用Ajax探索和动态添加节点，这是一直有的想法，
后来看到google搜索侧栏里的那个“神奇罗盘”，十分觊觎外加羡慕嫉妒恨这个功能，也明白大概要怎么做了。
这两天拿百度贴吧试了一下。

只有截图没测试链接，因为受同源策略限制，没法在自己页面跨域调用Baidu的页面，只好嵌入贴吧的页面中。

每个圈是一个百度贴吧，帖子数越多则圈越大，
每个贴吧点击后探索出友情贴吧，如是反复。
<div>（起始）
<img src="http://lh3.ggpht.com/_Rqi-iDrPknA/THUL9b5cSZI/AAAAAAAAAGA/NSxZVMuqp2Q/s800/2010-08-25_200441.png" alt="" />
（伊苏吧的友情贴吧）
<img src="http://lh3.ggpht.com/_Rqi-iDrPknA/THUL9TecjPI/AAAAAAAAAGE/wpbv_DwrCzc/s800/2010-08-25_200451.png" alt="" />
（FALCOM的友情贴吧）
<img src="http://lh4.ggpht.com/_Rqi-iDrPknA/THUL9fn1xmI/AAAAAAAAAGI/mKuoXF3NTQ8/s800/2010-08-25_200502.png" alt="" />当然，目标是做成带箭头的，完全展示贴吧之间follow和被follow的信息。
只是箭头太麻烦，先放着，弄完favicon再说。
<img src="http://lh3.ggpht.com/_Rqi-iDrPknA/THUL9qZTyWI/AAAAAAAAAGM/mNBwPKRhH4s/s800/2010-08-25_200502a.png" alt="" />

多点几次就出现一坨东西
<img src="http://lh6.ggpht.com/_Rqi-iDrPknA/THUMoQMj9nI/AAAAAAAAAGQ/3ebObBsa3v4/s800/2010-08-25_201702.png" alt="" />

圈的尺寸表示这个贴吧的水份
<img src="http://lh4.ggpht.com/_Rqi-iDrPknA/THUMoT3Rt9I/AAAAAAAAAGU/P-Bi2JHZVq0/s800/2010-08-25_201827.png" alt="" />

</div>
<div>

illi同学顺便讨论下 favicon 页面要什么样式吧？

方案1，白色，简洁，单栏，圆矩，Web2.0风格

主题突出排布整齐的一小撮图标，每次大约40个即可
所有功能都不刷新完成
精美的UI，细致的表单按钮
大量动画，
透明，渐变的Logo

这是原本的方案，我看了很多单页模板，结果发现这类布局大多需要好看的配图，纯文字太单调。
而加了配图，展示icon的区域又被抢眼球。目前没太理想的页面样式。

方案2，favicon wall 改名 favicon minerals，圆形直线交错的背景修饰，缺角矩形，淡蓝和明艳蓝色，科幻风格

中间弄一坨水晶，每个水晶代表一个分类，每凿一下水晶掉下一些碎屑，每个碎屑动画过后展示一个 favicon，
如果水晶凿完了，还可以配音说“not enough minerals”，
展示全部 favicon 的控制台秘籍则是“show me the money”

</div>
</div>
</div>
</div>
</div>
<div>
<h2>Favicon项目进化到第二形态～</h2>
09月 6, 2010 作者：<a title="probe301 发布" href="http://probe301.wordpress.com/author/probe301/">probe301</a>

</div>
<div>
<div id="msgcns!5E902A513D05D6FA!751">

<a href="http://www.astralwind.org/project/favicon-minerals/">http://www.astralwind.org/project/favicon-minerals/</a>

在illi同学的全力协助下，居然拥有后台数据库了。
之前一直拿前台脚本在折腾啊，
不禁内牛啊～

请在Firefox里测试
最上面的下拉框是从一些txt文件载入数据，

载入完后可以编辑一下，写到illi的数据库里，完后再按分类-标签查询，就是从数据库返回结果了。

如果报错的话，多试几次就好了。。。。

<img src="http://lh6.ggpht.com/_Rqi-iDrPknA/TITwT5A6III/AAAAAAAAAG4/a2eXhxGeZnc/s800/2010-09-06_213300.png" alt="" />

成功载入 分类Application + 标签blue 留念。基本点10次成功一回。。
<img src="http://lh3.ggpht.com/_Rqi-iDrPknA/TITwT1574RI/AAAAAAAAAG0/GycEUmO4Dow/s800/2010-09-06_213234.png" alt="" />

</div>
</div>
</div>
</div>
