---
layout: post
title: 用 Markdown 格式做笔记的一些探索
tags:
- PKM
- Programming
status: publish
type: post
published: true

summary: '以 Markdown 剪藏网页, 记录笔记的一些细节心得'

---

# 用 Markdown 格式做笔记的一些探索



### Markdown 的适用场景和不适用场景

Markdown 格式特别适合记录自己创作的笔记, 以及摘抄技术类文章, 好处不用多说了, 但需留意不适合使用 Markdown 的场景, 不能啥都往这上面搬:


- 需要大量的图文混排时, Markdown 不能很方便的编辑图片
- 需要精细排版图片和文字的位置关系, Markdown 只能用 css 实现, 繁琐不直观
- 存在巨大的表格, 存在复杂格式的表格, 这时还是该用专业工具

相信看到这篇文章的朋友已经对笔记管理有很多心得了, 使用 Markdown 做笔记意味着

- 在你的资料库里另立山头, 之前笔记很多的话, 不太可能全都完美的转到 md 格式
- 没法接管 pdf, 传统笔记工具或多或少还是支持 pdf 的, 而 md 对此完全无能为力, 得搭配一个电子书管理工具

现实就是, 你的数据一直散落在各处, 关键不是把它们聚在一起, 关键是想用时能找到

### 重要的工具

对 "存档笔记" 这个需求, 其实是要分为 Markdown "编辑" 和 "管理" 两个部分

对于编辑器, 这里不太推荐那些主打分屏界面的, 因为开双屏模式占空间, 开源代码模式又很难一眼扫到细节

印象笔记的 md 编辑器就是这样, 这类工具都不推荐, 而有些笔记软件自身编辑功能弱, 但可以让用户自定义外部编辑器, 这种就可以用

实践后, 目前主要使用了这几个工具:

- Typora: 编辑器, 美观, 细节贴心, 所见即所得, 方便给人推广
- VSCode + md 系列插件: 用于实现一些高级功能
- VNote: 用于管理, 它的编辑器设计也很有特点



此外 Notion 还有待考察, 功能强大但是很慢



### 从网页剪辑 Markdown 文本

这是笔记的主要来源, 首先默念三遍 `存了不看, 等于没存` 然后这问题会另写文章讨论, 回到剪辑 Markdown 的需求上来: 

- 网页选中文本 / 正文转 Markdown
- 输出到你常用的笔记软件 / 保存到本地 / 剪切板
- 能识别代码块, 不要加很多转义符
- 智能提取作者, 文章主题, 摘要, 标签等, 能提取评论
- 保留代码块的语言标记 如 `js` 等, 对于未指定的能推测是何种语言
- 保留高级格式如脚注等



目前试了很多工具, 发现前三条能做到, 后面的不好办, 以对技术文档页面 https://docs.scrapy.org/en/latest/topics/item-pipeline.html 的处理为例



**bad 转换后的文本存在大量 `\` 转义符**

存在 `_-=` 等字符时, 会被转换成这样:

```
...
drops those items which don’t contain a price:

    vat\_factor \= 1.15

    def process\_item(self, item, spider):
        if item.get('price'):
            if item.get('price\_excludes\_vat'):
                item\['price'\] \= item\['price'\] \* self.vat\_factor
            return item
...
```

- copycat, 但它的其他功能都很好
- 其他大部分同类型扩展, 都不能准确处理转义



**good 不会加入额外的转义符**

会识别为这样

```
drops those items which don’t contain a price:

    vat_factor = 1.15

    def process_item(self, item, spider):
        if item.get('price'):
            if item.get('price_excludes_vat'):
                item['price'] = item['price'] * self.vat_factor
            return item
```

- 简阅, 只能转换整个网页, 对代码块使用四空格缩进, 会删掉正文出现的所有 `<h1>`

- markdownizr, 会在代码块周围遗留 `<pre>`

- [Web Clipper](https://github.com/webclipper/web-clipper) 在一些网站能用, 一些网站失效

  

**better 能自动识别语言种类**

希望最终得到这样的效果, 没找到这种工具

    drops those items which don’t contain a price:
        
        ```python
            vat_factor = 1.15
        
            def process_item(self, item, spider):
                if item.get('price'):
                    if item.get('price_excludes_vat'):
                        item['price'] = item['price'] * self.vat_factor
                    return item
        ```



同样, 能顺带提取评论的插件也没找到, 需要自己定制

保留脚注也别想了, 例如 [MarginNote & LiquidText | 文献阅读工具能怎么用？](https://sspai.com/post/54112) 里的脚注, 这个不重要

结论: 简单情况用 copycat, 搞不定就看 Web Clipper 和 简阅, 选个排版好的




### 对 Markdown 的一部分文字标记高亮

Markdown 基本语法不提了, 下面说说用得上的 Markdown 扩展语法

怎么对 Markdown 的一部分文字标记高亮颜色, 目前主要看到了这两个办法



**方案1 用等号标记高亮**

写法是这样 `周围的文字==高亮文字==周围的文字`

Typora 自带支持, 类比于其他软件的明黄色高亮, 可用 css 调整样式, 不太破坏原文

缺点: 就一种颜色, 没法跨段落, 没法对代码块的一小部分内容做标注



**方案2 内嵌 HTML 元素 span, 自己定义格式细节**

其实就是在 Markdown 里混合写 HTML, "遇到复杂格式用 HTML 来表示" 就是 md 的初衷

- 使用 style `<span style='color:red;background:#def;font-size:1.5rem;font-family:宋体;'>文字</span>`
- 类似的方法 `<span class="text-warning">`, 自定义`<myspan>xxx</myspan>` 等

灵活, 可以有多个颜色和样式

缺点: 文本的语义不清楚, 标记太多了扰乱文档

想兼容更多软件, 需要用 `style` 的办法, 在多种软件里都能保留样式 

而 `class` 的办法在 Typora 中会过滤掉这些标记 (在 VSCode 中可以保留)



### 在 Markdown 文档里添加注记

就是类似 pdf 软件的注记功能

**方案1 使用脚注**

简单好用, 跟周围文本风格统一

注意区分

```markdown
1 普通链接 [an example](http://example.com/ "Title")

2 参考链接 (链接引用) [an example][id] 这个都是方括号

[id]: http://example.com/  "Optional Title Here"

3 脚注 [^footnote]

[^footnote]: 脚注内容可以使用**富文本**
```

缺点: 从语义上是给原作者用的, 不像是做笔记时的附注



**方案2 使用 md 扩展语法 Admonition**

参考 [VSCode Markdown Extended](https://marketplace.visualstudio.com/items?itemName=jebbs.markdown-extended#extended-syntaxes) 写法是

```markdown
!!! note

    This is the **note** admonition body

    !!! danger Danger Title
        This is the **danger** admonition body   

!!! quote 自定义标题

    在 Admonition 中使用自定义标题

!!! snippet ""

    在 Admonition 中隐藏标题
```

需要 VSCode + 扩展 `Markdown Extended`, 启动 md 预览就能看到了

在 Typora 里虽然不识别, 但也保留了相对易读的样式, 类似一个四空格缩进的代码块, 推荐

另附所有支持的标记:

```
note | summary, abstract, tldr | info, todo | tip, hint | success, check, done | question, help, faq | warning, attention, caution | failure, fail, missing | danger, error, bug | example, snippet | quote, cite
```



**方案3 使用 md 扩展语法 markdown-it-container**

这个可以渲染嵌套的 div, 表达更复杂的结构

来源用法同上, 需要配套 bootstrap

```
::: alert alert-danger
这是一个警告
:::

::::: container
:::: row
::: col-xs-6 alert alert-success
success text
:::
::: col-xs-6 alert alert-info
warning text
:::
::::
:::::
```

过于繁琐, 有点像是一个 HTML 方言了, 用到的机会不多



恰当的使用颜色高亮 + 注记, 有一些笔记的样子了, 如果做到 LiquidText 那样就更好了, 还需研究





### 在 Markdown 中引用本地文件

**方案1 `file://` 协议, 使用绝对路径**

即指向本地文档的一条超链接

```markdown
[test](file://D:/Todo/Evernote/test.md)
```
这样写常见软件一般都能识别

存在的问题: 

- 首先 `file://` 协议不能用相对路径
- 在 Typora 里点击这种链接, 无法直接打开 md 后缀, 打开其他格式和文件夹都 ok
- 如果路径里面有中文或空格等, 部分软件无法匹配到这个路径
- VSCode 无法打开文件夹, 打开各种文件都可以识别



**方案2 VSCode `@import`**

需要 VSCode + [插件 Markdown Preview Enhanced](https://shd101wyy.github.io/markdown-preview-enhanced/#/zh-cn/file-imports) 写法:

```
@import "你的文件"
或
<!-- @import "your_file" -->
```

会把指定文件导入到当前文档

支持常见文件格式, 比如导入 css 会追加样式, 导入 csv 会渲染为表格, 等等





### 指向另一个 Markdown 文本的指定位置

比如 OneNote 和 Notion 有这个功能, 为每个段落都生成自己的链接

在编辑 Markdown 时, 没找到这个的解决办法

但是 Markdown 生成 HTML 之后, 倒是很好解决, 参考 [Markdown 拓展 Header Anchors | VuePress](https://v1.vuepress.vuejs.org/zh/guide/markdown.html#header-anchors)





### 检索多个 Markdown 文件

Typora 里有同目录的文件内容搜索

各大笔记软件, VSCode, VNote 当然更不用说了, 都提供了更多的搜索



问题在于:

1. 搜索文字可以模糊匹配, 有权重, 类似网络搜索引擎
2. 需要关联笔记推荐
3. 搜索网络时, 把本地的搜索结果添加到侧栏里 (参考印象笔记 - 剪藏)
4. 汇总 / 提取 / 搜索自己的高亮和注记
5. 搜索 Markdown 图片上的文字 (参考印象笔记)



以上全没找到答案



### 在 Markdown 中修复中英文空格

这个不能用 pangu, 文中有 `行内代码` 时会给修复乱掉, 可以使用 [louisun/HeySpace: 中英文混排自动加空格](https://github.com/louisun/HeySpace) 效果挺好

笔记来源太多, 别太在意这些细节























































