---
layout: post
title: 在 wordpress.com 博客中使用半角引号
tags:
- Blog
- WordPress
- 创作
status: publish
type: post
published: true
summary: 不知现在还有人用 wordpress.com 写文章么？
---
插入代码时发现 WordPress 会自动把引号改成全角的，且网上的解决办法（修改formatting.php设置）只是针对自建 WordPress 博客的，托管在 wordpress.com 上的博客没法这么改动。

只好曲线救国，用 &lt;pre&gt; 实现半角，但之后发现又出现了不能换行的问题，结果再用 CSS 属性弥补之。折腾一小番，临时解决办法如下：文章编辑时切换进入 HTML 编辑模式，用 &lt;pre&gt; 包裹插入的代码并设好 white-space:pre-wrap 属性，像下面这样就可以了。
<pre style="font-family:'Courier New';white-space:pre-wrap;"><span style="color:#ff0000;">&lt;pre style="font-family: 'Courier New'; white-space: pre-wrap;"&gt;</span>&lt;span style="color: #ff8000;"&gt;6.8&lt;/span&gt;&lt;span style="color: #000080;"&gt;&lt;strong&gt;.&lt;/strong&gt;&lt;/span&gt; Functional Programming &lt;span style="color: #ff8000;"&gt;6.8&lt;/span&gt;&lt;span style="color: #000080;"&gt;&lt;strong&gt;.&lt;/strong&gt;&lt;/span&gt;&lt;span style="color: #ff8000;"&gt;1.&lt;/span&gt; 对一个Enumerable对象应用一个函数 Applying a Function to an Enumerable
...
...
...
<span style="color:#ff0000;">&lt;/pre&gt;</span></pre>
附：
<h1>CSS white-space 属性</h1>
<div>

<a title="CSS 参考手册" href="http://www.w3school.com.cn/css/css_reference.asp">CSS 参考手册</a>

</div>
<div>
<h2>定义和用法</h2>
white-space 属性设置如何处理元素内的空白。

这个属性声明建立布局过程中如何处理元素中的空白符。值 pre-wrap 和 pre-line 是 CSS 2.1 中新增的。
<table>
<tbody>
<tr>
<th>默认值：</th>
<td>normal</td>
</tr>
<tr>
<th>继承性：</th>
<td>yes</td>
</tr>
<tr>
<th>版本：</th>
<td>CSS1</td>
</tr>
<tr>
<th>JavaScript 语法：</th>
<td><em>object</em>.style.whiteSpace="pre"</td>
</tr>
</tbody>
</table>
</div>
<div>
<h2>浏览器支持</h2>
所有浏览器都支持 white-space 属性。

注释：任何的版本的 Internet Explorer （包括 IE8）都不支持属性值 "inherit"。

</div>
<div>
<h2>可能的值</h2>
<table>
<tbody>
<tr>
<th>值</th>
<th>描述</th>
</tr>
<tr>
<td>normal</td>
<td>默认。空白会被浏览器忽略。</td>
</tr>
<tr>
<td>pre</td>
<td>空白会被浏览器保留。其行为方式类似 HTML 中的 &lt;pre&gt; 标签。</td>
</tr>
<tr>
<td>nowrap</td>
<td>文本不会换行，文本会在在同一行上继续，直到遇到 &lt;br&gt; 标签为止。</td>
</tr>
<tr>
<td>pre-wrap</td>
<td>保留空白符序列，但是正常地进行换行。</td>
</tr>
<tr>
<td>pre-line</td>
<td>合并空白符序列，但是保留换行符。</td>
</tr>
<tr>
<td>inherit</td>
<td>规定应该从父元素继承 white-space 属性的值。</td>
</tr>
</tbody>
</table>
</div>
