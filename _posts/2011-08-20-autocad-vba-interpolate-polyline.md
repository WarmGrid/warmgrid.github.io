---
layout: post
title: AutoCAD VBA 加密多段线
tags:
- AutoCAD
- VBA
- 创作
status: publish
type: post
published: true
meta: {}
---
最近学习 AutoCAD VBA 编程，前几天想做加密多段线的没弄出来，现在学会了。

<strong>功能</strong>

加密 AutoCAD 中的二维多段线。按照设定的距离阈值处理圆弧，将其转换为内接于圆弧的多段线。

<strong>使用</strong>

此为 VBA 宏脚本，在 AutoCAD 中按”Alt+F8″，填写名字后新建宏，粘贴脚本到编辑框中，按”F5″执行。具体流程可自行搜索。

<!--more-->

代码如下：
===================
===================
<pre><span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">多段线插值加密</span>
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">功能：多段线炸开，标记每段直线或圆弧的起点，加密圆弧，删除原圆弧，生成面积报告</span>
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">需要：完成后再拼合回多段线</span> <span style="color:#008000;font-family:'Times New Roman';">只会拼合为面域</span> <span style="color:#008000;font-family:'Times New Roman';">拼合多段线</span> <span style="color:#008000;">done!</span>
<span style="color:#0000ff;">Sub</span> InterpolatePolyline<strong>()</strong>
<span style="color:#0000ff;">Dim</span> pl <span style="color:#0000ff;">As</span> AcadLWPolyline
ThisDrawing.Utility.GetEntity pl<strong>,</strong> Pnt<strong>,</strong> <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">指定将被加密的多段线：</span><span style="color:#808080;">"</span>

<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">获取加密距离，正实数，按空格默认为</span><span style="color:#008000;">0.5</span>
ThisDrawing.Utility.InitializeUserInput <span style="color:#ff0000;"><strong>6</strong></span>
<span style="color:#0000ff;">Dim</span> promptStr <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">String</span>
promptStr <strong>=</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">指定加密距离</span><span style="color:#808080;">&lt;0.5&gt;):"</span>
<span style="color:#0000ff;">On</span> <span style="color:#0000ff;">Error</span> <span style="color:#0000ff;">Resume</span> <span style="color:#0000ff;">Next</span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">这句如不写，则按空格会报错</span>
threshold <strong>=</strong> ThisDrawing.Utility.GetReal<strong>(</strong>promptStr<strong>)</strong>
<span style="color:#0000ff;">If</span> threshold <strong>=</strong> <span style="color:#ff0000;"><strong>0</strong></span> <span style="color:#0000ff;">Then</span> threshold <strong>=</strong> <span style="color:#ff0000;"><strong>0.5</strong></span>

<span style="color:#0000ff;">Dim</span> plarea <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> plarea <strong>=</strong> pl.Area <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">在炸开前取得</span><span style="color:#008000;">polyline</span><span style="color:#008000;font-family:'Times New Roman';">的信息</span>
pl.Copy <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">备份一个原</span><span style="color:#008000;">polyline</span>

<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">炸开</span>
<span style="color:#0000ff;">Dim</span> ss <span style="color:#0000ff;">As</span> AcadSelectionSet
ThisDrawing.ActiveSelectionSet.Clear
ThisDrawing.SendCommand <span style="color:#808080;">"Explode"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"(handent "</span> <strong>&amp;</strong> Chr<strong>(</strong><span style="color:#ff0000;"><strong>34</strong></span><strong>)</strong> _
<strong>&amp;</strong> pl.Handle <strong>&amp;</strong> Chr<strong>(</strong><span style="color:#ff0000;"><strong>34</strong></span><strong>)</strong> <strong>&amp;</strong> <span style="color:#808080;">")"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> vbCr
<span style="color:#0000ff;">Set</span> ss <strong>=</strong> ThisDrawing.ActiveSelectionSet
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">清空当前选择集，然后</span> <span style="color:#008000;">sendcommand x</span> <span style="color:#008000;font-family:'Times New Roman';">碎片自动被收集到当前选择集？</span>

<span style="color:#0000ff;">Dim</span> out<strong>()</strong> <span style="color:#0000ff;">As</span> AcadObject<strong>:</strong> <span style="color:#0000ff;">ReDim</span> out<strong>(</strong>ss.Count <strong>-</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">炸开碎片的数组</span>
<span style="color:#0000ff;">Dim</span> vtxcount <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Long</span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">统计节点数目</span>
<span style="color:#0000ff;">Dim</span> listent <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">String</span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">预备拼合之后的</span><span style="color:#008000;">polyline</span>

<span style="color:#0000ff;">For</span> i <strong>=</strong> <span style="color:#ff0000;"><strong>0</strong></span> <span style="color:#0000ff;">To</span> ss.Count <strong>-</strong> <span style="color:#ff0000;"><strong>1</strong></span>
<span style="color:#008000;">' temp = ThisDrawing.ModelSpace.AddPoint(ss(i).StartPoint)</span>
<span style="color:#0000ff;">If</span> ss<strong>(</strong>i<strong>).</strong>ObjectName <strong>=</strong> <span style="color:#808080;">"AcDbArc"</span> <span style="color:#0000ff;">Then</span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">加密碎片中的圆弧</span>
<span style="color:#0000ff;">Set</span> out<strong>(</strong>i<strong>)</strong> <strong>=</strong> InterpolateArc<strong>(</strong>ss<strong>(</strong>i<strong>),</strong> threshold<strong>)</strong>
vtxcount <strong>=</strong> vtxcount <strong>+</strong> <strong>(</strong>UBound<strong>(</strong>out<strong>(</strong>i<strong>).</strong>Coordinates<strong>)</strong> <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>\</strong> <span style="color:#ff0000;"><strong>2</strong></span> <strong>-</strong> <span style="color:#ff0000;"><strong>1</strong></span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">加密圆弧节点数</span>
<span style="color:#0000ff;">Else</span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">其余是直线，不去管它</span>
<span style="color:#0000ff;">Set</span> out<strong>(</strong>i<strong>)</strong> <strong>=</strong> ss<strong>(</strong>i<strong>)</strong>
vtxcount <strong>=</strong> vtxcount <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">直线节点数</span>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">If</span>
listent <strong>=</strong> listent <strong>+</strong> axEnt2lspEnt<strong>(</strong>out<strong>(</strong>i<strong>))</strong> <strong>+</strong> <span style="color:#808080;">" "</span>
<span style="color:#0000ff;">Next</span> i

ThisDrawing.SendCommand <span style="color:#808080;">"_pedit"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"m"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> listent <strong>&amp;</strong> vbCr <strong>&amp;</strong> _
<span style="color:#808080;">"y"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"j"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"0.00001"</span> <strong>&amp;</strong> vbCr <strong>&amp;</strong> vbCr

<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">输出</span><span style="color:#008000;">log</span>
ThisDrawing.SelectionSets.Item<strong>(</strong><span style="color:#808080;">"ss1"</span><strong>).</strong>Delete
<span style="color:#0000ff;">Set</span> ss <strong>=</strong> ThisDrawing.SelectionSets.Add<strong>(</strong><span style="color:#808080;">"ss1"</span><strong>)</strong>
ss.Select acSelectionSetLast <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">获取最后创建的对象</span>
plarea <strong>=</strong> Format<strong>(</strong>plarea<strong>,</strong> <span style="color:#808080;">"0.000"</span><strong>)</strong> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">原面积</span>
newplarea <strong>=</strong> Format<strong>(</strong>ss<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>).</strong>Area<strong>,</strong> <span style="color:#808080;">"0.000"</span><strong>)</strong> <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">新面积</span>

MsgBox <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">加密后节点数：</span><span style="color:#808080;">"</span> <strong>&amp;</strong> vtxcount <strong>&amp;</strong> vbCr _
<strong>&amp;</strong> <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">加密距离：</span><span style="color:#808080;">"</span> <strong>&amp;</strong> Format<strong>(</strong>threshold<strong>,</strong> <span style="color:#808080;">"0.00"</span><strong>)</strong> <strong>&amp;</strong> vbCr _
<strong>&amp;</strong> <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">原面积：</span><span style="color:#808080;">"</span> <strong>&amp;</strong> plarea <strong>&amp;</strong> vbCr <strong>&amp;</strong> <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">新面积：</span><span style="color:#808080;">"</span> <strong>&amp;</strong> newplarea <strong>&amp;</strong> vbCr _
<strong>&amp;</strong> <span style="color:#808080;">"</span><span style="color:#808080;font-family:'Times New Roman';">面积相差：</span><span style="color:#808080;">"</span> <strong>&amp;</strong> Format<strong>((</strong>newplarea <strong>-</strong> plarea<strong>),</strong> <span style="color:#808080;">"0.000"</span><strong>)</strong> _
<strong>&amp;</strong> <span style="color:#808080;">" ("</span> <strong>&amp;</strong> Format<strong>(((</strong>newplarea <strong>-</strong> plarea<strong>)</strong> <strong>/</strong> plarea <strong>*</strong> <span style="color:#ff0000;"><strong>1000</strong></span><strong>),</strong> <span style="color:#808080;">"0.000</span><span style="color:#808080;font-family:'Times New Roman';">‰</span><span style="color:#808080;">"</span> <strong>&amp;</strong> <span style="color:#808080;">")"</span><strong>)</strong>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">Sub</span>

<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">圆弧插值加密，</span><span style="color:#008000;">AcadArc</span><span style="color:#008000;font-family:'Times New Roman';">对象</span> <span style="color:#008000;">&gt;&gt;</span> <span style="color:#008000;font-family:'Times New Roman';">加密后的多段线并删除原圆弧</span>
<span style="color:#0000ff;">Public</span> <span style="color:#0000ff;">Function</span> InterpolateArc<strong>(</strong>arc <span style="color:#0000ff;">As</span> AcadArc<strong>,</strong> thres <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Variant</span><strong>)</strong> <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Variant</span>
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">内建</span><span style="color:#008000;">PI</span><span style="color:#008000;font-family:'Times New Roman';">，没法声明为常量</span>
<span style="color:#0000ff;">Dim</span> PI <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> PI <strong>=</strong> <span style="color:#ff0000;"><strong>4</strong></span> <strong>*</strong> Atn<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
<span style="color:#0000ff;">Dim</span> threshold <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> threshold <strong>=</strong> thres <span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">插值距离</span>
<span style="color:#0000ff;">Dim</span> cenx <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> cenx <strong>=</strong> arc.Center<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong>
<span style="color:#0000ff;">Dim</span> ceny <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> ceny <strong>=</strong> arc.Center<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
<span style="color:#0000ff;">Dim</span> radius <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> radius <strong>=</strong> arc.radius
<span style="color:#0000ff;">Dim</span> segment <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Long</span><strong>:</strong> segment <strong>=</strong> Int<strong>(</strong>arc.ArcLength <strong>/</strong> threshold<strong>)</strong> <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span>
<span style="color:#0000ff;">Dim</span> anglesegment <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span>
anglesegment <strong>=</strong> <strong>(</strong>arc.EndAngle <strong>-</strong> arc.StartAngle<strong>)</strong> <strong>/</strong> segment
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">圆弧永远是按逆时针定义角度</span>
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">处理</span><span style="color:#008000;">endangle</span><span style="color:#008000;font-family:'Times New Roman';">已经转过一圈，回到</span><span style="color:#008000;">stratpoint</span><span style="color:#008000;font-family:'Times New Roman';">之前的情况</span>
<span style="color:#0000ff;">If</span> anglesegment <strong>&lt;</strong> <span style="color:#ff0000;"><strong>0</strong></span> <span style="color:#0000ff;">Then</span> anglesegment <strong>=</strong> <strong>(</strong><span style="color:#ff0000;"><strong>2</strong></span> <strong>*</strong> PI <strong>+</strong> arc.EndAngle <strong>-</strong> arc.StartAngle<strong>)</strong> <strong>/</strong> segment
<span style="color:#0000ff;">Dim</span> point<strong>()</strong> <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span>
<span style="color:#0000ff;">ReDim</span> point<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span> <span style="color:#0000ff;">To</span> segment <strong>*</strong> <span style="color:#ff0000;"><strong>2</strong></span> <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong>
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">数组从</span><span style="color:#008000;">1</span><span style="color:#008000;font-family:'Times New Roman';">开始存储坐标，</span><span style="color:#008000;">1 2</span> <span style="color:#008000;font-family:'Times New Roman';">圆弧为起点，</span><span style="color:#008000;">segment+1 segment+2</span> <span style="color:#008000;font-family:'Times New Roman';">为终点</span>
<span style="color:#0000ff;">For</span> i <strong>=</strong> <span style="color:#ff0000;"><strong>3</strong></span> <span style="color:#0000ff;">To</span> segment <strong>*</strong> <span style="color:#ff0000;"><strong>2</strong></span> <span style="color:#0000ff;">Step</span> <span style="color:#ff0000;"><strong>2</strong></span>
<span style="color:#0000ff;">Dim</span> thisangle <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">Double</span><strong>:</strong> thisangle <strong>=</strong> arc.StartAngle <strong>+</strong> anglesegment <strong>*</strong> <strong>(</strong>i <strong>-</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>/</strong> <span style="color:#ff0000;"><strong>2</strong></span>
point<strong>(</strong>i<strong>)</strong> <strong>=</strong> cenx <strong>+</strong> radius <strong>*</strong> Cos<strong>(</strong>thisangle<strong>)</strong>
point<strong>(</strong>i <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> ceny <strong>+</strong> radius <strong>*</strong> Sin<strong>(</strong>thisangle<strong>)</strong>
<span style="color:#0000ff;">Next</span> i
<span style="color:#008000;">'</span><span style="color:#008000;font-family:'Times New Roman';">首尾端点，必须照抄圆弧的坐标</span>
point<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> arc.StartPoint<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong>
point<strong>(</strong><span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong> <strong>=</strong> arc.StartPoint<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>
point<strong>(</strong>segment <strong>*</strong> <span style="color:#ff0000;"><strong>2</strong></span> <strong>+</strong> <span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong> <strong>=</strong> arc.EndPoint<strong>(</strong><span style="color:#ff0000;"><strong>0</strong></span><strong>)</strong>
point<strong>(</strong>segment <strong>*</strong> <span style="color:#ff0000;"><strong>2</strong></span> <strong>+</strong> <span style="color:#ff0000;"><strong>2</strong></span><strong>)</strong> <strong>=</strong> arc.EndPoint<strong>(</strong><span style="color:#ff0000;"><strong>1</strong></span><strong>)</strong>

<span style="color:#0000ff;">Set</span> templ <strong>=</strong> ThisDrawing.ModelSpace.AddLightWeightPolyline<strong>(</strong>point<strong>)</strong>
arc.Delete
<span style="color:#0000ff;">Set</span> InterpolateArc <strong>=</strong> templ
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">Function</span>

<span style="color:#0000ff;">Public</span> <span style="color:#0000ff;">Function</span> axEnt2lspEnt<strong>(</strong>entObj <span style="color:#0000ff;">As</span> AcadEntity<strong>)</strong> <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">String</span>
<span style="color:#0000ff;">Dim</span> entHandle <span style="color:#0000ff;">As</span> <span style="color:#0000ff;">String</span>
entHandle <strong>=</strong> entObj.Handle
axEnt2lspEnt <strong>=</strong> <span style="color:#808080;">"(handent "</span> <strong>&amp;</strong> Chr<strong>(</strong><span style="color:#ff0000;"><strong>34</strong></span><strong>)</strong> <strong>&amp;</strong> entHandle <strong>&amp;</strong> Chr<strong>(</strong><span style="color:#ff0000;"><strong>34</strong></span><strong>)</strong> <strong>&amp;</strong> <span style="color:#808080;">")"</span>
<span style="color:#0000ff;">End</span> <span style="color:#0000ff;">Function</span></pre>
==================
==================

其中使用了两个函数：
InterpolateArc 处理针对单一圆弧的加密的函数
axEnt2lspEnt 将 VBA 中的图形对象转换为 lisp 中对象的函数，在将碎片联合为多段线时用到（这个函数是抄人家的）

<img src="http://clip2net.com/clip/m22533/1313833500-e3d45-153kb.jpg" alt="" />

完
