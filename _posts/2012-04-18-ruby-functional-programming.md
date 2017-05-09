---
layout: post
title: Ruby 中的函数式编程
tags:
- 资源
- Programming
- Ruby
status: publish
type: post
published: true
summary: 介绍了map和inject，缓存函数，方法绑定等概念。

---
摘自 &lt;The Ruby Programming Language&gt;，最近学习Ruby，于是结合着中英文版的这书把&lt;函数式编程&gt;章节敲出来了。
<pre style="font-family:'Courier New';white-space:pre-wrap;"><span style="color:#ff8000;">6.8</span><span style="color:#000080;"><strong>.</strong></span> Functional Programming
<span style="color:#ff8000;">6.8</span><span style="color:#000080;"><strong>.</strong></span><span style="color:#ff8000;">1.</span> 对一个Enumerable对象应用一个函数 Applying a Function to an Enumerable

map和inject是Enumerable类定义的两个最重要的迭代器，它们都需要一个代码块。如果用以函数为中心的方式编写程序，我们会喜欢那些可以让函数应用到Enumerable对象上的方法：

<span style="color:#008000;"># This module defines methods and operators for functional programming.</span>
<span style="color:#0000ff;"><strong>module</strong></span> <span style="color:#804000;"><strong>Functional</strong></span>
  <span style="color:#008000;"># Apply this function to each element of the specified Enumerable,</span>
  <span style="color:#008000;"># returning an array of results. This is the reverse of Enumerable.map.</span>
  <span style="color:#008000;"># Use | as an operator alias. Read "|" as "over" or "applied over".</span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># Example:</span>
  <span style="color:#008000;">#   a = [[1,2],[3,4]]</span>
  <span style="color:#008000;">#   sum = lambda {|x,y| x+y}</span>
  <span style="color:#008000;">#   sums = sum|a   # =&gt; [3,7]</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>apply</strong></span><span style="color:#000080;"><strong>(</strong></span>enum<span style="color:#000080;"><strong>)</strong></span>
    enum<span style="color:#000080;"><strong>.</strong></span>map <span style="color:#000080;"><strong>&amp;</strong></span><span style="color:#0000ff;"><strong>self</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>|</strong></span> apply

  <span style="color:#008000;"># Use this function to "reduce" an enumerable to a single quantity.</span>
  <span style="color:#008000;"># This is the inverse of Enumerable.inject.</span>
  <span style="color:#008000;"># Use &lt;= as an operator alias.</span>
  <span style="color:#008000;"># Mnemonic: &lt;= looks like a needle for injections</span>
  <span style="color:#008000;"># Example:</span>
  <span style="color:#008000;">#   data = [1,2,3,4]</span>
  <span style="color:#008000;">#   sum = lambda {|x,y| x+y}</span>
  <span style="color:#008000;">#   total = sum&lt;=data   # =&gt; 10</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>reduce</strong></span><span style="color:#000080;"><strong>(</strong></span>enum<span style="color:#000080;"><strong>)</strong></span>
    enum<span style="color:#000080;"><strong>.</strong></span>inject <span style="color:#000080;"><strong>&amp;</strong></span><span style="color:#0000ff;"><strong>self</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>&lt;=</strong></span> reduce
<span style="color:#0000ff;"><strong>end</strong></span>

<span style="color:#008000;"># Add these functional programming methods to Proc and Method classes.</span>
<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Proc</strong></span><span style="color:#000080;"><strong>;</strong></span> include Functional<span style="color:#000080;"><strong>;</strong></span> <span style="color:#0000ff;"><strong>end</strong></span>
<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Method</strong></span><span style="color:#000080;"><strong>;</strong></span> include Functional<span style="color:#000080;"><strong>;</strong></span> <span style="color:#0000ff;"><strong>end</strong></span>

注意，我们是在Functional模块中定义方法，然后把这个模块包含<span style="color:#000080;"><strong>(</strong></span>include<span style="color:#000080;"><strong>)</strong></span>在Proc和Method类中，这样apply和reduce方法就可以用在proc和lambda对象上。后面的绝大多数方法仍然定义在Functional模块中，因此它们也可以被Proc和Method对象使用。

有了上面定义的apply和reduce方法，我们可以重构下面的统计方法：

sum <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>,</strong></span>y<span style="color:#000080;"><strong>|</strong></span> x<span style="color:#000080;"><strong>+</strong></span>y <span style="color:#000080;"><strong>}</strong></span>        <span style="color:#008000;"># A function to add two numbers</span>
mean <span style="color:#000080;"><strong>=</strong></span> <span style="color:#000080;"><strong>(</strong></span>sum<span style="color:#000080;"><strong>&lt;=</strong></span>a<span style="color:#000080;"><strong>)/</strong></span>a<span style="color:#000080;"><strong>.</strong></span>size           <span style="color:#008000;"># Or sum.reduce(a) or a.inject(&amp;sum)</span>
deviation <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> x<span style="color:#000080;"><strong>-</strong></span>mean <span style="color:#000080;"><strong>}</strong></span> <span style="color:#008000;"># Function to compute difference from mean</span>
square <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> x<span style="color:#000080;"><strong>*</strong></span>x <span style="color:#000080;"><strong>}</strong></span>       <span style="color:#008000;"># Function to square a number</span>
standardDeviation <span style="color:#000080;"><strong>=</strong></span> Math<span style="color:#000080;"><strong>.</strong></span>sqrt<span style="color:#000080;"><strong>((</strong></span>sum<span style="color:#000080;"><strong>&lt;=</strong></span>square<span style="color:#000080;"><strong>|(</strong></span>deviation<span style="color:#000080;"><strong>|</strong></span>a<span style="color:#000080;"><strong>))/(</strong></span>a<span style="color:#000080;"><strong>.</strong></span>size<span style="color:#000080;"><strong>-</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>))</strong></span>

值得注意的是，最后一行尽管很简洁，但是那些非标准的操作符使得它难以阅读。还要注意<span style="color:#000080;"><strong>|</strong></span>符是我们自己定义的，它是左连接的，因此，上面的代码要在一个Enumerable对象上应用多个函数，则须要使用圆括号。
也就是说，必须使用 square<span style="color:#000080;"><strong>|(</strong></span>deviation<span style="color:#000080;"><strong>|</strong></span>a<span style="color:#000080;"><strong>)</strong></span>，而不能使用 square<span style="color:#000080;"><strong>|</strong></span>deviation<span style="color:#000080;"><strong>|</strong></span>a。

<span style="color:#ff8000;">6.8</span><span style="color:#000080;"><strong>.</strong></span><span style="color:#ff8000;">2.</span> 复合函数 Composing Functions

如果有两个函数f、g，有时我们会希望定义一个新函数 h<span style="color:#000080;"><strong>=</strong></span>f<span style="color:#000080;"><strong>(</strong></span>g<span style="color:#000080;"><strong>())</strong></span>，它也可被称为f由g复合。
我们可以用一个方法自动进行函数复合，其代码如下：

<span style="color:#0000ff;"><strong>module</strong></span> <span style="color:#804000;"><strong>Functional</strong></span>
  <span style="color:#008000;"># Return a new lambda that computes self[f[args]].</span>
  <span style="color:#008000;"># Use * as an operator alias for compose.</span>
  <span style="color:#008000;"># Examples, using the * alias for this method.</span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># f = lambda {|x| x*x }</span>
  <span style="color:#008000;"># g = lambda {|x| x+1 }</span>
  <span style="color:#008000;"># (f*g)[2]   # =&gt; 9</span>
  <span style="color:#008000;"># (g*f)[2]   # =&gt; 5</span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># def polar(x,y)</span>
  <span style="color:#008000;">#   [Math.hypot(y,x), Math.atan2(y,x)]</span>
  <span style="color:#008000;"># end</span>
  <span style="color:#008000;"># def cartesian(magnitude, angle)</span>
  <span style="color:#008000;">#   [magnitude*Math.cos(angle), magnitude*Math.sin(angle)]</span>
  <span style="color:#008000;"># end</span>
  <span style="color:#008000;"># p,c = method :polar, method :cartesian</span>
  <span style="color:#008000;"># (c*p)[3,4]  # =&gt; [3,4]</span>
  <span style="color:#008000;">#</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>compose</strong></span><span style="color:#000080;"><strong>(</strong></span>f<span style="color:#000080;"><strong>)</strong></span>
    <span style="color:#0000ff;"><strong>if</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>.</strong></span>respond_to?<span style="color:#000080;"><strong>(</strong></span>:arity<span style="color:#000080;"><strong>)</strong></span> <span style="color:#000080;"><strong>&amp;&amp;</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>.</strong></span>arity <span style="color:#000080;"><strong>==</strong></span> <span style="color:#ff8000;">1</span>
      lambda <span style="color:#000080;"><strong>{|*</strong></span>args<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>[</strong></span>f<span style="color:#000080;"><strong>[*</strong></span>args<span style="color:#000080;"><strong>]]</strong></span> <span style="color:#000080;"><strong>}</strong></span>
    <span style="color:#0000ff;"><strong>else</strong></span>
      lambda <span style="color:#000080;"><strong>{|*</strong></span>args<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>[*</strong></span>f<span style="color:#000080;"><strong>[*</strong></span>args<span style="color:#000080;"><strong>]]</strong></span> <span style="color:#000080;"><strong>}</strong></span>
    <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#008000;"># * is the natural operator for function composition.</span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>*</strong></span> compose
<span style="color:#0000ff;"><strong>end</strong></span>

示例中的注释演示了如何对Method对象和lambda使用compose方法。我们可以用这个新的<span style="color:#000080;"><strong>*</strong></span>函数复合操作符来简化标准差的计算。仍然使用上面定义的sum、square和deviation，标准差的计算变为：

standardDeviation <span style="color:#000080;"><strong>=</strong></span> Math<span style="color:#000080;"><strong>.</strong></span>sqrt<span style="color:#000080;"><strong>((</strong></span>sum<span style="color:#000080;"><strong>&lt;=</strong></span>square<span style="color:#000080;"><strong>*</strong></span>deviation<span style="color:#000080;"><strong>|</strong></span>a<span style="color:#000080;"><strong>)/(</strong></span>a<span style="color:#000080;"><strong>.</strong></span>size<span style="color:#000080;"><strong>-</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>))</strong></span>

区别在于在square和deviation应用到数组a之前，我们先将它们复合成单个函数。

<span style="color:#ff8000;">6.8</span><span style="color:#000080;"><strong>.</strong></span><span style="color:#ff8000;">3.</span> 局部应用函数 Partially Applying Functions

在函数式编程中，局部应用是指用一个函数和部分参数值产生一个新的函数，这个函数等价于用某些固定参数调用原有的函数。例如：

product <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>,</strong></span> y<span style="color:#000080;"><strong>|</strong></span> x<span style="color:#000080;"><strong>*</strong></span>y <span style="color:#000080;"><strong>}</strong></span>       <span style="color:#008000;"># A function of two arguments</span>
double <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> product<span style="color:#000080;"><strong>(</strong></span><span style="color:#ff8000;">2</span><span style="color:#000080;"><strong>,</strong></span>x<span style="color:#000080;"><strong>)</strong></span> <span style="color:#000080;"><strong>}</strong></span>  <span style="color:#008000;"># Apply one argument</span>

局部应用可以用Functional模块中的方法（和操作符）进行简化:

<span style="color:#0000ff;"><strong>module</strong></span> <span style="color:#804000;"><strong>Functional</strong></span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># Return a lambda equivalent to this one with one or more initial</span>
  <span style="color:#008000;"># arguments applied. When only a single argument</span>
  <span style="color:#008000;"># is being specified, the &gt;&gt; alias may be simpler to use.</span>
  <span style="color:#008000;"># Example:</span>
  <span style="color:#008000;">#   product = lambda {|x,y| x*y}</span>
  <span style="color:#008000;">#   doubler = lambda &gt;&gt; 2</span>
  <span style="color:#008000;">#</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>apply_head</strong></span><span style="color:#000080;"><strong>(*</strong></span>first<span style="color:#000080;"><strong>)</strong></span>
    lambda <span style="color:#000080;"><strong>{|*</strong></span>rest<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>[*</strong></span>first<span style="color:#000080;"><strong>.</strong></span>concat<span style="color:#000080;"><strong>(</strong></span>rest<span style="color:#000080;"><strong>)]}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># Return a lambda equivalent to this one with one or more final arguments</span>
  <span style="color:#008000;"># applied. When only a single argument is being specified,</span>
  <span style="color:#008000;"># the &lt;&lt; alias may be simpler.</span>
  <span style="color:#008000;"># Example:</span>
  <span style="color:#008000;">#  difference = lambda {|x,y| x-y }</span>
  <span style="color:#008000;">#  decrement = difference &lt;&lt; 1</span>
  <span style="color:#008000;">#</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>apply_tail</strong></span><span style="color:#000080;"><strong>(*</strong></span>last<span style="color:#000080;"><strong>)</strong></span>
    lambda <span style="color:#000080;"><strong>{|*</strong></span>rest<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>[*</strong></span>rest<span style="color:#000080;"><strong>.</strong></span>concat<span style="color:#000080;"><strong>(</strong></span>last<span style="color:#000080;"><strong>)]}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#008000;"># Here are operator alternatives for these methods. The angle brackets</span>
  <span style="color:#008000;"># point to the side on which the argument is shifted in.</span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>&gt;&gt;</strong></span> apply_head    <span style="color:#008000;"># g = f &gt;&gt; 2 -- set first arg to 2</span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>&lt;&lt;</strong></span> apply_tail    <span style="color:#008000;"># g = f &lt;&lt; 2 -- set last arg to 2</span>
<span style="color:#0000ff;"><strong>end</strong></span>

使用这些方法和操作符，可以简单地用 product<span style="color:#000080;"><strong>&gt;&gt;</strong></span><span style="color:#ff8000;">2</span> 来定义我们的double函数。
使用局部应用，我们使标准差计算变得更加抽象，这可以通过一个更通用的difference函数来实现：

difference <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>,</strong></span>y<span style="color:#000080;"><strong>|</strong></span> x<span style="color:#000080;"><strong>-</strong></span>y <span style="color:#000080;"><strong>}</strong></span>  <span style="color:#008000;"># Compute difference of two numbers</span>
deviation <span style="color:#000080;"><strong>=</strong></span> difference<span style="color:#000080;"><strong>&lt;&lt;</strong></span>mean      <span style="color:#008000;"># Apply second argument</span>

<span style="color:#ff8000;">6.8</span><span style="color:#000080;"><strong>.</strong></span><span style="color:#ff8000;">4.</span> 缓存函数 Memoizing Functions

Memoization是函数式编程的一个术语，表示缓存函数调用的结果。如果一个函数对同样的参数输入总是返回相同的结果，另外出于某种需要我们认为这些参数会不断使用，而且执行这个函数比较消耗资源，那么memoization可能是一个有用的优化。我们可以通过下面的方法使Proc和Method对象的memoization自动化：

<span style="color:#0000ff;"><strong>module</strong></span> <span style="color:#804000;"><strong>Functional</strong></span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># Return a new lambda that caches the results of this function and</span>
  <span style="color:#008000;"># only calls the function when new arguments are supplied.</span>
  <span style="color:#008000;">#</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>memoize</strong></span>
    cache <span style="color:#000080;"><strong>=</strong></span> <span style="color:#000080;"><strong>{}</strong></span>  <span style="color:#008000;"># An empty cache. The lambda captures this in its closure.</span>
    lambda <span style="color:#000080;"><strong>{|*</strong></span>args<span style="color:#000080;"><strong>|</strong></span>
      <span style="color:#008000;"># notice that the hash key is the entire array of arguments!</span>
      <span style="color:#0000ff;"><strong>unless</strong></span> cache<span style="color:#000080;"><strong>.</strong></span>has_key?<span style="color:#000080;"><strong>(</strong></span>args<span style="color:#000080;"><strong>)</strong></span>  <span style="color:#008000;"># If no cached result for these args</span>
        cache<span style="color:#000080;"><strong>[</strong></span>args<span style="color:#000080;"><strong>]</strong></span> <span style="color:#000080;"><strong>=</strong></span> <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>[*</strong></span>args<span style="color:#000080;"><strong>]</strong></span>  <span style="color:#008000;"># Compute and cache the result</span>
      <span style="color:#0000ff;"><strong>end</strong></span>
      cache<span style="color:#000080;"><strong>[</strong></span>args<span style="color:#000080;"><strong>]</strong></span>                  <span style="color:#008000;"># Return result from cache</span>
    <span style="color:#000080;"><strong>}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#008000;"># A (probably unnecessary) unary + operator for memoization</span>
  <span style="color:#008000;"># Mnemonic: the + operator means "improved"</span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>+</strong></span>@ memoize        <span style="color:#008000;"># cached_f = +f</span>
<span style="color:#0000ff;"><strong>end</strong></span>

下面是如何使用memoize方法或一元<span style="color:#000080;"><strong>+</strong></span>操作符的例子：

<span style="color:#008000;"># A memoized recursive factorial function</span>
factorial <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>return</strong></span> <span style="color:#ff8000;">1</span> if x<span style="color:#000080;"><strong>==</strong></span><span style="color:#ff8000;">0</span><span style="color:#000080;"><strong>;</strong></span> x<span style="color:#000080;"><strong>*</strong></span>factorial<span style="color:#000080;"><strong>[</strong></span>x<span style="color:#000080;"><strong>-</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>];</strong></span> <span style="color:#000080;"><strong>}.</strong></span>memoize
<span style="color:#008000;"># Or, using the unary operator syntax</span>
factorial <span style="color:#000080;"><strong>=</strong></span> <span style="color:#000080;"><strong>+</strong></span>lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>return</strong></span> <span style="color:#ff8000;">1</span> if x<span style="color:#000080;"><strong>==</strong></span><span style="color:#ff8000;">0</span><span style="color:#000080;"><strong>;</strong></span> x<span style="color:#000080;"><strong>*</strong></span>factorial<span style="color:#000080;"><strong>[</strong></span>x<span style="color:#000080;"><strong>-</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>];</strong></span> <span style="color:#000080;"><strong>}</strong></span>

注意这里的factorial方法是一个递归函数，它自己也会调用缓存版本的自身函数，得到最佳的缓存效果。否则，如果定义一个非缓存版本的递归函数，然后根据它定义一个缓存版本方法，优化效果就没有那么突出了：

factorial <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> <span style="color:#0000ff;"><strong>return</strong></span> <span style="color:#ff8000;">1</span> if x<span style="color:#000080;"><strong>==</strong></span><span style="color:#ff8000;">0</span><span style="color:#000080;"><strong>;</strong></span> x<span style="color:#000080;"><strong>*</strong></span>factorial<span style="color:#000080;"><strong>[</strong></span>x<span style="color:#000080;"><strong>-</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>];</strong></span> <span style="color:#000080;"><strong>}</strong></span>
cached_factorial <span style="color:#000080;"><strong>=</strong></span> <span style="color:#000080;"><strong>+</strong></span>factorial <span style="color:#008000;"># Recursive calls aren't cached!</span>

<span style="color:#ff8000;">6.8</span><span style="color:#000080;"><strong>.</strong></span><span style="color:#ff8000;">5.</span> 符号、方法和Proc Symbols<span style="color:#000080;"><strong>,</strong></span> Methods<span style="color:#000080;"><strong>,</strong></span> <span style="color:#0000ff;"><strong>and</strong></span> Procs

Symbol、Method和Proc类有亲密的关系。我们已经看到method方法可以用一个Symbol对象作为参数，然后返回一个Method方法。
Ruby1<span style="color:#000080;"><strong>.</strong></span>9为Symbol类增加了一个有用的to_proc方法，这个方法允许用<span style="color:#000080;"><strong>&amp;</strong></span>打头的符号作为代码块被传入到一个迭代器中。在这里，这个符号被假定为一个方法名。在用to_proc方法创建的Proc对象被调用时，它会调用第一个参数所表示的名字的方法，剩下的参数则作为参数传递给这个方法。示例如下：

<span style="color:#008000;"># Increment an array of integers with the Fixnum.succ method</span>
<span style="color:#000080;"><strong>[</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>,</strong></span><span style="color:#ff8000;">2</span><span style="color:#000080;"><strong>,</strong></span><span style="color:#ff8000;">3</span><span style="color:#000080;"><strong>].</strong></span>map<span style="color:#000080;"><strong>(&amp;</strong></span>:succ<span style="color:#000080;"><strong>)</strong></span>  <span style="color:#008000;"># =&gt; [2,3,4]</span>

如果不用Symbol<span style="color:#000080;"><strong>.</strong></span>to_proc方法，我们会不得不更啰嗦一些：
<span style="color:#000080;"><strong>[</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>,</strong></span><span style="color:#ff8000;">2</span><span style="color:#000080;"><strong>,</strong></span><span style="color:#ff8000;">3</span><span style="color:#000080;"><strong>].</strong></span>map <span style="color:#000080;"><strong>{|</strong></span>n<span style="color:#000080;"><strong>|</strong></span> n<span style="color:#000080;"><strong>.</strong></span>succ <span style="color:#000080;"><strong>}</strong></span>

Symbol<span style="color:#000080;"><strong>.</strong></span>to_proc 最初是作为Ruby1<span style="color:#000080;"><strong>.</strong></span>8的扩展被引入的，它通常使用如下方式实现：

<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Symbol</strong></span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>to_proc</strong></span>
    lambda <span style="color:#000080;"><strong>{|</strong></span>receiver<span style="color:#000080;"><strong>,</strong></span> <span style="color:#000080;"><strong>*</strong></span>args<span style="color:#000080;"><strong>|</strong></span> receiver<span style="color:#000080;"><strong>.</strong></span>send<span style="color:#000080;"><strong>(</strong></span><span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>,</strong></span> <span style="color:#000080;"><strong>*</strong></span>args<span style="color:#000080;"><strong>)}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
<span style="color:#0000ff;"><strong>end</strong></span>

这个实现使用send方法（参见第8<span style="color:#000080;"><strong>.</strong></span>4.3节）来调用一个符号命名的方法。不过，也可以像下面这样做：

<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Symbol</strong></span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#8080ff;"><strong>to_proc</strong></span>
    lambda <span style="color:#000080;"><strong>{|</strong></span>receiver<span style="color:#000080;"><strong>,</strong></span> <span style="color:#000080;"><strong>*</strong></span>args<span style="color:#000080;"><strong>|</strong></span> receiver<span style="color:#000080;"><strong>.</strong></span>method<span style="color:#000080;"><strong>(</strong></span><span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>)[*</strong></span>args<span style="color:#000080;"><strong>]}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
<span style="color:#0000ff;"><strong>end</strong></span>

除了to_proc，还能定义一些相关而且有用的工具方法，首先从Module类开始：

<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Module</strong></span>
  <span style="color:#008000;"># Access instance methods with array notation. Returns UnboundMethod,</span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>[]</strong></span> instance_method
<span style="color:#0000ff;"><strong>end</strong></span>

这里我们为Module类的instance_method定义了一个快捷方式。注意，这个方法会返回一个UnboundMethod对象，它在绑定到一个对象前不能被调用，下面是一个使用这种新记号的例子（注意，我们可以用名字像索引一样获得一个方法<span style="color:#000080;"><strong>!</strong></span>）：

String<span style="color:#000080;"><strong>[</strong></span>:reverse<span style="color:#000080;"><strong>].</strong></span>bind<span style="color:#000080;"><strong>(</strong></span><span style="color:#808080;">"hello"</span><span style="color:#000080;"><strong>).</strong></span>call   <span style="color:#008000;"># =&gt; "olleh"</span>

用同样的语法糖，绑定一个无绑定的方法也可以变得简单：

<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>UnboundMethod</strong></span>
  <span style="color:#008000;"># Allow [] as an alternative to bind. </span>
  <span style="color:#0000ff;"><strong>alias</strong></span> <span style="color:#000080;"><strong>[]</strong></span> bind
<span style="color:#0000ff;"><strong>end</strong></span>

使用这里定义的别名，以及已有的用来调用方法的<span style="color:#000080;"><strong>[]</strong></span>别名，上面的代码会变成：

String<span style="color:#000080;"><strong>[</strong></span>:reverse<span style="color:#000080;"><strong>][</strong></span><span style="color:#808080;">"hello"</span><span style="color:#000080;"><strong>][]</strong></span>   <span style="color:#008000;"># =&gt; "olleh"</span>

第一个方括号索引一个方法，第二个方括号将它绑定，而第三个方括号则调用它。

接下来，既然我们用<span style="color:#000080;"><strong>[]</strong></span>操作符来索引一个类的方法，那么就用<span style="color:#000080;"><strong>[]=</strong></span>来定义一个实例方法：

<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Module</strong></span>
  <span style="color:#008000;"># Define a instance method with name sym and body f.</span>
  <span style="color:#008000;"># Example: String[:backwards] = lambda { reverse }</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#000080;"><strong>[]=(</strong></span><span style="color:#8080ff;"><strong>sym</strong></span><span style="color:#000080;"><strong>,</strong></span> f<span style="color:#000080;"><strong>)</strong></span>
    <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>.</strong></span>instance_eval <span style="color:#000080;"><strong>{</strong></span> define_method<span style="color:#000080;"><strong>(</strong></span>sym<span style="color:#000080;"><strong>,</strong></span> f<span style="color:#000080;"><strong>)</strong></span> <span style="color:#000080;"><strong>}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
<span style="color:#0000ff;"><strong>end</strong></span>

<span style="color:#000080;"><strong>[]=</strong></span>操作符的定义有点让人费解——这是Ruby的髙级内容。
define_method是Module的私有方法，我们用instance_eval方法（Object的一个公开方法）运行一个代码块（包括一个私有方法的调用），就像它位于方法定义的模块中一样。
在第8章中我们将再次看到instance_eval和define_method方法。

让我们用新的<span style="color:#000080;"><strong>[]=</strong></span>操作符定义一个新的Enumerable<span style="color:#000080;"><strong>.</strong></span>average方法：

Enumerable<span style="color:#000080;"><strong>[</strong></span>:average<span style="color:#000080;"><strong>]</strong></span> <span style="color:#000080;"><strong>=</strong></span> lambda <span style="color:#0000ff;"><strong>do</strong></span>
  sum<span style="color:#000080;"><strong>,</strong></span> n <span style="color:#000080;"><strong>=</strong></span> <span style="color:#ff8000;">0.0</span><span style="color:#000080;"><strong>,</strong></span> <span style="color:#ff8000;">0</span>
  <span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>.</strong></span>each <span style="color:#000080;"><strong>{|</strong></span>x<span style="color:#000080;"><strong>|</strong></span> sum <span style="color:#000080;"><strong>+=</strong></span> x<span style="color:#000080;"><strong>;</strong></span> n <span style="color:#000080;"><strong>+=</strong></span> <span style="color:#ff8000;">1</span> <span style="color:#000080;"><strong>}</strong></span>
  <span style="color:#0000ff;"><strong>if</strong></span> n <span style="color:#000080;"><strong>==</strong></span> <span style="color:#ff8000;">0</span>
    <span style="color:#0000ff;"><strong>nil</strong></span>
  <span style="color:#0000ff;"><strong>else</strong></span>
    sum<span style="color:#000080;"><strong>/</strong></span>n
  <span style="color:#0000ff;"><strong>end</strong></span>
<span style="color:#0000ff;"><strong>end</strong></span>

现在我们有了<span style="color:#000080;"><strong>[]</strong></span>和<span style="color:#000080;"><strong>[]=</strong></span>操作符，它们可以用于获得和设置一个类或模块的成员方法。我们也可以对类的单键方法做相似的事情（也包括类或模块的类方法<span style="color:#000080;"><strong>)</strong></span>。任何对象都可以有单键方法，不过给Object类定义一个<span style="color:#000080;"><strong>[]</strong></span>操作符有点说不通，因为已经有太多的子类已经定义了这个操作符，因此我们可以用另外一种方式，给Symbol类定义这样的操作符：

<span style="color:#008000;">#</span>
<span style="color:#008000;"># Add [] and []= operators to the Symbol class for accessing and setting</span>
<span style="color:#008000;"># singleton methods of objects. Read : as "method" and [] as "of".</span>
<span style="color:#008000;"># So :m[o] reads "method m of o".</span>
<span style="color:#008000;">#</span>
<span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#0080c0;"><strong>Symbol</strong></span>
  <span style="color:#008000;"># Return the Method of obj named by this symbol. This may be a singleton</span>
  <span style="color:#008000;"># method of obj (such as a class method) or an instance method defined</span>
  <span style="color:#008000;"># by obj.class or inherited from a superclass.</span>
  <span style="color:#008000;"># Examples:</span>
  <span style="color:#008000;">#   creator = :new[Object]  # Class method Object.new</span>
  <span style="color:#008000;">#   doubler = :*[2]         # * method of Fixnum 2</span>
  <span style="color:#008000;">#</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#000080;"><strong>[](</strong></span><span style="color:#8080ff;"><strong>obj</strong></span><span style="color:#000080;"><strong>)</strong></span>
    obj<span style="color:#000080;"><strong>.</strong></span>method<span style="color:#000080;"><strong>(</strong></span><span style="color:#0000ff;"><strong>self</strong></span><span style="color:#000080;"><strong>)</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
  <span style="color:#008000;"># Define a singleton method on object o, using Proc or Method f as its body.</span>
  <span style="color:#008000;"># This symbol is used as the name of the method.</span>
  <span style="color:#008000;"># Examples:</span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;">#  :singleton[o] = lambda { puts "this is a singleton method of o" }</span>
  <span style="color:#008000;">#  :class_method[String] = lambda { puts "this is a class method" }</span>
  <span style="color:#008000;">#</span>
  <span style="color:#008000;"># Note that you can't create instance methods this way. See Module.[]=</span>
  <span style="color:#008000;">#</span>
  <span style="color:#0000ff;"><strong>def</strong></span> <span style="color:#000080;"><strong>[]=(</strong></span><span style="color:#8080ff;"><strong>o</strong></span><span style="color:#000080;"><strong>,</strong></span>f<span style="color:#000080;"><strong>)</strong></span>
    <span style="color:#008000;"># We can't use self in the block below, as it is evaluated in the</span>
    <span style="color:#008000;"># context of a different object. So we have to assign self to a variable.</span>
    sym <span style="color:#000080;"><strong>=</strong></span> <span style="color:#0000ff;"><strong>self</strong></span>
    <span style="color:#008000;"># This is the object we define singleton methods on.</span>
    eigenclass <span style="color:#000080;"><strong>=</strong></span> <span style="color:#000080;"><strong>(</strong></span><span style="color:#0000ff;"><strong>class</strong></span> <span style="color:#000080;"><strong>&lt;&lt;</strong></span> <span style="color:#0080c0;"><strong>o</strong></span><span style="color:#000080;"><strong>;</strong></span> <span style="color:#0000ff;"><strong>self</strong></span> <span style="color:#0000ff;"><strong>end</strong></span><span style="color:#000080;"><strong>)</strong></span>
    <span style="color:#008000;"># define_method is private, so we have to use instance_eval to execute it.</span>
    eigenclass<span style="color:#000080;"><strong>.</strong></span>instance_eval <span style="color:#000080;"><strong>{</strong></span> define_method<span style="color:#000080;"><strong>(</strong></span>sym<span style="color:#000080;"><strong>,</strong></span> f<span style="color:#000080;"><strong>)</strong></span> <span style="color:#000080;"><strong>}</strong></span>
  <span style="color:#0000ff;"><strong>end</strong></span>
<span style="color:#0000ff;"><strong>end</strong></span>

通过这里定义的Symbol<span style="color:#000080;"><strong>.[]</strong></span>方法，以及前面描述的Functional模块，我们可以写出如下精巧的（也是难读的）代码：

dashes <span style="color:#000080;"><strong>=</strong></span> :*<span style="color:#000080;"><strong>[</strong></span><span style="color:#808000;">'-'</span><span style="color:#000080;"><strong>]</strong></span>       <span style="color:#008000;"># Method * of '-'</span>
puts dashes<span style="color:#000080;"><strong>[</strong></span><span style="color:#ff8000;">10</span><span style="color:#000080;"><strong>]</strong></span>        <span style="color:#008000;"># Prints "----------"</span>
y <span style="color:#000080;"><strong>=</strong></span> <span style="color:#000080;"><strong>(</strong></span>:+<span style="color:#000080;"><strong>[</strong></span><span style="color:#ff8000;">1</span><span style="color:#000080;"><strong>]*</strong></span>:*<span style="color:#000080;"><strong>[</strong></span><span style="color:#ff8000;">2</span><span style="color:#000080;"><strong>])[</strong></span>x<span style="color:#000080;"><strong>]</strong></span>   <span style="color:#008000;"># Another way to write y = 2*x + 1</span>

Symbol类定义的<span style="color:#000080;"><strong>[]=</strong></span>与Module类的<span style="color:#000080;"><strong>[]=</strong></span>相似，都使用instance_eval调用define_method方法。不同点在于单键方法并不像实例方法一样在类中定义，而是在对象的中定义，在第7章中，我们还将见到eigenclass。</pre>
