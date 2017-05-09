---
layout: post
title: CSS 中的 Line-Height 行高学习笔记
tags:
- CSS
- 设计
- Note
status: publish
type: post
published: true
meta: {}
---
本文是从SlideShare找到的文档《Line-Height 行高中文版》的学习笔记，原演示稿的内容质量和设计编排都属上乘，不过觉得99张幻灯片的篇幅有点过多了，因此总结一下。
<div id="__ss_2470819" style="width:425px;">

<strong><a title="Line Height (中文版)" href="http://www.slideshare.net/daemao/line-height-2470819" target="_blank">Line Height (中文版)</a></strong> [slideshare id=2470819&amp;w=425&amp;h=355&amp;sc=no]
<div style="padding:5px 0 12px;">View more <a href="http://www.slideshare.net/" target="_blank">presentations</a> from <a href="http://www.slideshare.net/daemao" target="_blank">bigCat Mao</a></div>
</div>
<strong>行高的基本知识</strong>

在CSS中设定line-height的值，共有五种定义方式：
<ul type="disc">
	<li>line-height:normal 普通行高，此值据浏览器不同在1.2倍文字大小左右</li>
	<li>line-height:inherit 继承父级行高</li>
	<li>line-height:120% 百分比率，此时按照font-size求得数值</li>
	<li>line-height:20px 绝对长度</li>
	<li>line-height:1.2 纯数字比率</li>
</ul>
line-height可以缩写到font属性中，紧跟在font-size属性后面，用斜杠分割
<blockquote>p { font: 12pt/18pt 宋体; }</blockquote>
<!--more-->line-height减去font-size即为行间距，实质是文字上下各分配等大的半行间距。如line-height:20px，font-size:16px时：

<img src="http://pic.yupoo.com/probeprobe_v/9782491ab3a8/big.jpg" alt="" />

<strong>行高的继承</strong>

在父级元素（如body）中设置行高想传递给子级时，建议使用纯数字，即上述的第五种定义方法。

原因是：若在父级设置绝对长度的行高，则子级元素继承此固定数值，不随子级不同文字大小做自适应。父级设置百分比行高道理相同，会以父级font-size乘以百分比求出行高，仍以固定数值继承给子级。若在父级设置行高为normal，可以适应子级元素的不同文字大小，但normal的值在浏览器中解释有些许区别。

推荐方案：正文行高1.5，标题行高1.2（WCAG2.0规定段落行高至少1.5倍）。

<strong>行高的相关盒模型</strong>

与行高相关的四种盒模型是
<ul type="disc">
	<li>containing box</li>
	<li>line box</li>
	<li>inline box</li>
	<li>content area</li>
</ul>
以一个带有多个换行的段落为例：

1，段落整体构成一个containing box

<img src="http://pic.yupoo.com/probeprobe_v/5684491ab3a9/big.jpg" alt="" />

2，containing box中，每一行文字是一个line box

<img src="http://pic.yupoo.com/probeprobe_v/5025891ab3ac/big.jpg" alt="" />

3，回到整个段落中，依据元素标签划分为一系列inline box

<img src="http://pic.yupoo.com/probeprobe_v/1053091ab3ac/big.jpg" alt="" />

inline box有可能跨行，且分为有标签的inline box（如上图中的emphasis）和匿名inline box（如普通文字）

4，每个inline box中含有一个content area

<img src="http://pic.yupoo.com/probeprobe_v/6507791ab3ad/big.jpg" alt="" />

<strong>不同盒子的高度细节</strong>

1，content area的高度对应font-size

2，inline box的高度对应line-height，若font-size &gt; line-height时，inline box的高度仍使用行高（而非文字高）

3，line box高度取决其内部最高的inline box或替代元素（图片之类）

<img src="http://pic.yupoo.com/probeprobe_v/4832391ab3ae/big.jpg" alt="" />

4，多个line box无缝堆叠，构成整个containing box的高度

<strong>实际运用中的特殊情况</strong>

上标下标可能导致行高异常，可将该元素的行高归零解决
<blockquote>
<p lang="en-US">sup, sub {line-height:0; }</p>
</blockquote>
<img src="http://pic.yupoo.com/probeprobe_v/0285691ab3ae/big.jpg" alt="" />

文字中的图片也会造成行高异常

<img src="http://pic.yupoo.com/probeprobe_v/8360791ab3b1/big.jpg" alt="" />

之前在博客里为外站链接加了<a href="http://blog.xiao3.info/google-favicon-cache.html">Google缓存的Favicon</a>，结果在几个主流浏览器中一番折腾，仍旧是各种对不齐，所以研究一下行高的事，准备继续鼓捣，嗯嗯。
