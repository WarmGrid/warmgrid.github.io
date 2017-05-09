---
layout: post
title: AutoCAD VBA 抽稀多段线
tags:
- AutoCAD
- Code
- VBA
- 创作
status: publish
type: post
published: true
meta: {}
---
最近迫于工作压力开始学 AutoCAD VBA 编程。有好多打算弄成自动处理的工作，预备一点一点搞定。对着手册折腾一番后，研究出了将多段线抽稀的 VBA 宏（其实我是想做多段线加密的，但暂时没能搞出来……）。

<strong>功能</strong>

抽稀（优化）AutoCAD 中的二维多段线。依次计算线上原有点之间的距离，合并在距离阈值之内的点。

<strong>使用</strong>

此为 VBA 宏脚本，在 AutoCAD 中按"Alt+F8"，填写名字后新建宏，粘贴脚本到编辑框中，按"F5"执行。具体流程可自行搜索。

<!--more-->
代码如下：
===================
===================
<pre><span style="color:#008000;">'抽稀多段线</span>
<span style="color:#008000;">'利用Polyline原有的点优化线条，凡阈值内的点都被归并为一处，保留起止点</span>
<span style="color:#0000ff;">Sub</span> RefineLine<strong>()</strong>
<span style="color:#0000ff;">Dim</span> pl <span style="color:#0000ff;">As</span> AcadLWPolyline
ThisDrawing.Utility.GetEntity pl<strong>,</strong> Pnt<strong>,</strong> <span style="color:#808080;">"指定将被抽稀的多段线："</span>

<span style="color:#008000;">'获取正实数，按空格默认为0.5</span>
ThisDrawing.Utility.InitializeUserInput <span style="color:#ff0000;"><strong>6</strong></span>
<span style="color:#008000;">' 1 不接受 NULL 输入 防止用户只按回车或空格来响应输入请求</span>
<span style="color:#008000;">' 2 不接受输入零值(0) 防止用户输入 0 来响应输入请求</span>
<span style="color:#008000;">' 4 不接受输入负值 防止用户输入负值来响应输入请求</span>
<span style="color:#0000ff;">Dim</span> promptStr <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">String</span>
promptStr <strong>=</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"指定阈值，此距离内的冗余节点将被合并&lt;0.5&gt;):"</span>
<span style="color:#0000ff;">On</span> <span style="color:#0000ff;">Error</span> <span style="color:#0000ff;">Resume</span> <span style="color:#0000ff;">Next</span> <span style="color:#008000;">'这句如不写，则按空格会报错</span>
threshold <strong>=</strong> ThisDrawing.Utility.GetReal<strong>(</strong>promptStr<strong>)</strong>
<span style="color:#0000ff;">If</span> threshold <strong>=</strong> <span style="color:#ff0000;"><strong>0</strong></span> <span style="color:#0000ff;">Then</span>
threshold <strong>=</strong> <span style="color:#ff0000;"><strong>0.5</strong></span>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">If</span>

<span style="color:#0000ff;">Dim</span> pin <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Variant</span> <span style="color:#008000;">'输入坐标组</span>
pin <strong>=</strong> pl.Coordinates
up <strong>=</strong> UBound<strong>(</strong>pin<strong>)</strong>

<span style="color:#0000ff;">Dim</span> pout<strong>()</strong> <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span> <span style="color:#008000;">'输出坐标组</span>
<span style="color:#0000ff;">ReDim</span> pout<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span> <span style="color:#0000ff;">To</span> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
<span style="color:#008000;">'动态数组真麻烦，必须这么定义，然后往里面写几个东西，否则没法直接扩充</span>

<span style="color:#0000ff;">Dim</span> comparePt<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span> <span style="color:#0000ff;">To</span> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span> <span style="color:#008000;">'比对点坐标组，就一组</span>

<span style="color:#008000;">'保持首端点不变</span>
pout<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong>
pout<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
comparePt<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong>
comparePt<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>

<span style="color:#0000ff;">For</span> i <strong>=</strong> <span style="color:#ff0000;"><strong>2</strong></span> <span style="color:#0000ff;">To</span> up <strong>-</strong> <span style="color:#ff0000;"><strong>3</strong></span> <span style="color:#0000ff;">Step</span> <span style="color:#ff0000;"><strong>2</strong></span> <span style="color:#008000;">'依次分析处理第2个点直到倒数第二点</span>
dx <strong>=</strong> comparePt<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong> <strong>-</strong> pin<strong>(</strong>i<strong>)</strong>
dy <strong>=</strong> comparePt<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>-</strong> pin<strong>(</strong>i <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
dx2y2 <strong>=</strong> dx <strong>*</strong> dx <strong>+</strong> dy <strong>*</strong> dy <span style="color:#008000;">'该点距比对点的距离</span>
nextdx <strong>=</strong> comparePt<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong> <strong>-</strong> pin<strong>(</strong>i <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong>
nextdy <strong>=</strong> comparePt<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>-</strong> pin<strong>(</strong>i <strong>+</strong> <span style="color:#ff0000;"><strong>3</strong></span><strong>)</strong>
nextdx2y2 <strong>=</strong> nextdx <strong>*</strong> nextdx <strong>+</strong> nextdy <strong>*</strong> nextdy <span style="color:#008000;">'该点下一点距比对点的距离</span>
<span style="color:#008000;">'在该点超过阈值，或该点下一点超过阈值时，记录为有效的点，同时设此为新比对点</span>
<span style="color:#008000;">'不满足条件的是废点，忽略之</span>
<span style="color:#0000ff;">If</span> dx2y2 <strong>&gt;</strong> threshold <strong>*</strong> threshold <span style="color:#0000ff;">Or</span> nextdx2y2 <strong>&gt;</strong> threshold <strong>*</strong> threshold <span style="color:#0000ff;">Then</span> <span style="color:#008000;">'扩充output数组，写入新点</span>
leng <strong>=</strong> UBound<strong>(</strong>pout<strong>)</strong>
<span style="color:#0000ff;">ReDim</span> <span style="color:#0000ff;">Preserve</span> pout<strong>(</strong>leng <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong>
pout<strong>(</strong>leng <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong>i<strong>)</strong>
pout<strong>(</strong>leng <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong>i <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
comparePt<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong>i<strong>)</strong>
comparePt<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong>i <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">If</span>
<span style="color:#0000ff;">Next</span> i

leng <strong>=</strong> UBound<strong>(</strong>pout<strong>)</strong>
<span style="color:#0000ff;">ReDim</span> <span style="color:#0000ff;">Preserve</span> pout<strong>(</strong>leng <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong> <span style="color:#008000;">'处理末端点，照抄其坐标，不能改变</span>
pout<strong>(</strong>leng <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong>up <strong>-</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
pout<strong>(</strong>leng <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong> <strong>=</strong> pin<strong>(</strong>up<strong>)</strong>

<span style="color:#0000ff;">Set</span> newline <strong>=</strong> ThisDrawing.ModelSpace.AddLightWeightPolyline<strong>(</strong>pout<strong>)</strong>
<span style="color:#0000ff;">If</span> pl.Closed <span style="color:#0000ff;">Then</span> <span style="color:#008000;">'原线条若闭合，则同样闭合</span>
newline.Closed <strong>=</strong> <span style="color:#0000ff;">True</span>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">If</span>

<span style="color:#008000;">'统计面积</span>
pla <strong>=</strong> Format<strong>(</strong>pl.Area<strong>,</strong> <span style="color:#808080;">"0.000"</span><strong>)</strong>
nla <strong>=</strong> Format<strong>(</strong>newline.Area<strong>,</strong> <span style="color:#808080;">"0.000"</span><strong>)</strong>

MsgBox <span style="color:#808080;">"原节点数："</span> <strong>&amp;</strong> up <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"新节点数："</span> <strong>&amp;</strong> UBound<strong>(</strong>pout<strong>)</strong> <strong>&amp;</strong> vbCr _
<strong>&amp;</strong> <span style="color:#808080;">"阈值："</span> <strong>&amp;</strong> Format<strong>(</strong>threshold<strong>,</strong> <span style="color:#808080;">"0.00"</span><strong>)</strong> <strong>&amp;</strong> vbCr _
<strong>&amp;</strong> <span style="color:#808080;">"原面积："</span> <strong>&amp;</strong> pla <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"新面积："</span> <strong>&amp;</strong> nla <strong>&amp;</strong> vbCr _
<strong>&amp;</strong> <span style="color:#808080;">"面积相差："</span> <strong>&amp;</strong> Format<strong>((</strong>nla <strong>-</strong> pla<strong>),</strong> <span style="color:#808080;">"0.000"</span><strong>)</strong> _
<strong>&amp;</strong> <span style="color:#808080;">" ("</span> <strong>&amp;</strong> Format<strong>(((</strong>nla <strong>-</strong> pla<strong>)</strong> <strong>/</strong> pla <strong>*</strong> <span style="color:#ff0000;"><strong>1000</strong></span><strong>),</strong> <span style="color:#808080;">"0.000‰"</span> <strong>&amp;</strong> <span style="color:#808080;">")"</span><strong>)</strong>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">Sub</span></pre>
==================
==================
初学 VBA 许多地方都没有搞顺溜，今后逐步完善。

<img src="http://clip2net.com/clip/m22533/1313833499-79286-153kb.jpg" alt="" />

完
