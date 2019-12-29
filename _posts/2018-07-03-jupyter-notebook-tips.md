---
layout: post
title: Jupyter Notebook 的几个使用技巧
tags:
- Python
- Programming Tools
status: publish
type: post
published: true

summary: '扩展 Jupyter Notebook 用途, 尽可能打造成全能的代码笔记本'

---



### 在 cell 里显示图片

发现了三种方法:

1\. 对于 markdown cell, 使用标准的外链图片语法 `![title](url)` 

2\. 对于 markdown cell, 复制图片后直接粘贴到 cell, 似乎每单元格只能粘贴一张图, 图片将以 base64 编码随 notebook 保存下来

3\. 对于 code cell, 使用 `IPython.display`, 把图片显示到 output 区域, 基本用法是:

```python
from IPython.display import Image
display(Image(filename=path))         # 显示本地图片
display(Image(url=path))              # 显示 web 图片
display(Image(url=path, width=width)) # 附加属性
```



### 直接执行 shell 命令

行首加 `!` 可以直接执行 shell 命令, 且执行结果可以赋值给变量

结合 `%pwd` 显示当前工作目录, 结合 `%cd` 改变当前工作目录

此外, Windows 安装了 git 之后, 会附送一些 Linux 风格的命令, 因此这时可以直接执行类似 `!ls` 这种命令

```python
# Windows 系统, 执行 cmd 默认具备的命令
files = !dir /B | grep .ipynb$
files
```

    ['jupyter notebook tips.ipynb', 'pandas 常用命令总结.ipynb']


```python
# Windows 系统, 安装 git 之后
files = !ls -alh | grep .ipynb$
files
```

    ['-rw-r--r-- 1 Probe 197121  50K Jun 27 10:14 jupyter notebook tips.ipynb',
     '-rw-r--r-- 1 Probe 197121 6.0K Jun 26 09:44 pandas 常用命令总结.ipynb']



### 不输出最后一行表达式的值

在 code cell 执行代码时, 默认会输出最后一行表达式的值, 在代码末尾加分号 `;` 可以避免输出 (末尾加 `;` 的效果大概等同于执行了一个空语句)

比如使用 matplotlib 做图, `plt.plot()` 将返回一个以字符串表达的图形实例, 但是我们通常只看到图就行了, 不需要把这个字符串显示出来

```python
%matplotlib inline
import matplotlib.pyplot as plt
plt.plot([1,3,4,2])   # 这里输出图片前会打印一行文字 [<matplotlib.lines.Line2D at ...>]
```

    [<matplotlib.lines.Line2D at 0xf3588d0>]
    
    (显示图片)


```python
plt.plot([1,3,4,2]);  # 加分号, 不再打印这行文字 [<matplotlib.lines.Line2D at ...>]
```



### 导出 ipynb 文件到 html 和 md

`jupyter nbconvert` 命令可以把 notebook 导出为 html (markdown / py / ...), 比如:

    jupyter nbconvert --to=html --no-prompt --template=nbextensions "filename.ipynb"

- 有时在工作界面中需要隐藏部分 cell (一般用 nbextensions 里面的 Hide Input 插件), 但是导出时会把所有 cell 都给显示出来, 这时使用 option `--template=nbextensions` 可使导出结果也隐藏掉这些 cell
- 使用 `--to=html_embed` 可以把外链图片也嵌到文档中, 加大文件体积但是不再依赖网络


`--to` 总共支持以下 options:

    asciidoc, custom, html, html_ch, html_embed, html_toc, html_with_lenvs, html_with_toclenvs, latex, latex_with_lenvs, markdown, notebook, pdf, python, rst, script, selectLanguage, slides


另外, 如果要在存储 notebook 时自动导出 html, 参考
[Version Control for Jupyter Notebook – Towards Data Science](https://towardsdatascience.com/version-control-for-jupyter-notebook-3e6cef13392d)



### 链接到另一个 notebook

跟 markdown 的链接语法一样, 基本形式是:

    [Another Notebook](./Another Notebook.ipynb)
    
可以链接到指定的标题位置, 假设这个标题叫 `header-text`

    [Another Notebook 的标题: header-text](./Another Notebook.ipynb#header-text)
    
链接到文档自身的某个标题

    [文档自身的标题: header-text](#header-text)



### 查看两个 notebook 的 diff

参考文档 [nbdime](http://nbdime.readthedocs.io/en/stable/) 首先安装 nbdime 工具

    pip install nbdime

安装后就可以用两个命令:

- `nbdiff` 将显示命令行 diff
- `nbdiff-web` 将开启一个 web 界面显示富文本 diff

基本使用方法, 传入两个 notebook 路径:

    nbdiff notebook_1.ipynb notebook_2.ipynb
    nbdiff-web notebook_1.ipynb notebook_2.ipynb

结合 git 的使用方法, 传入 commit id:

    nbdime config-git --enable --global  # 启用对 git diff 的支持

    nbdiff [<commit> [<commit>]] [<path>]
    nbdiff-web [<commit> [<commit>]] [<path>] 



### 对 notebook 进行版本控制

用 git 即可, 需要查看 diff 参考上一节



### 几个有用的快捷键

在编辑模式中:

`Ctrl + Shift + -` 拆分 cell

`Shift + M` 合并 cell

在命令行模式中:

`O` 循环切换 output 显示



### 在 notebook 以及导出 html 里嵌入任意文件

有时需要把小文件如 csv, pdf 等直接嵌入 notebook 中, 这样当 notebook 导出为一个 html 文件时, 顺便也就共享了这些小文件, 免除了外链文件的麻烦

实现思路是以 base64 编码这些小文件, 然后以 Data URI 的格式写到 `<a>` 的 `href` 属性里, Data URI 语法形如:

    data:[<mime type>][;charset=<charset>][;base64],<encoded data>

方法是在 cell 中粘贴运行以下代码:

```python
import base64, os
from IPython.display import HTML

def embed_file(fname):
    # 在 notebook 中嵌入文件
    # output 区域将输出一个 <a> 链接, 文件内容以 base64 编码在 uri 中, 可以右键另存
    # 只能嵌入小文件, 500KB 以内比较合理, 文件太大会卡的要死
    # application/octet-stream 是默认的 MIME types
    with open(fname, "rb") as f:
        file_encoded = base64.b64encode(f.read()).decode()  # decode 转字符串, 否则还是 bytes
    basename = os.path.basename(fname)
    tag = '<a download="{}" href="data:application/octet-stream;base64,{}" >(使用右键另存) {}</a>'.format(
              basename, file_encoded, basename)
    display(HTML(data=tag));
```
之后, 在另一个 cell 里执行:

```python
embed_file(r'C:\Users\Probe\Downloads\test.epub')
```

就可以把这文件嵌入 notebook 了, 导出 html 时, 文件将随之保存, 用户可以从这个 html 里下载它, 如同从联网的页面下载一样

经测试, 嵌入的文件在 500KB 以内是没什么问题的, 如果文件太大, 还是应该老老实实用外链


**附, 对 Data URI 的介绍**

Data URI 一般用在 img, 或 css 需定义图片资源时, 例如:

    <img width="16" height="16" alt="star" src="data:image/gif;base64,R0lGODlhEAAQA......B0UjIQA7" />

其中, 所有的 MIME types 可以参考 [Incomplete list of MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)

`application/octet-stream` 是除了文本以外的默认 MIME type, 虽然可以根据文件名推测出更具体的 MIME type, 但是测试了没什么用, 还是都写这个 `application/octet-stream` 吧

> 长度限制: 虽然 Firefox 支持无限长度的 data URLs，但是标准中并没有规定浏览器必须支持任意长度的 data URIs。比如，Opera 11浏览器限制 URLs 最长为 65535 个字符，这意味着 data URLs 最长为 65529 个字符（如果你使用纯文本 data:, 而不是指定一个 MIME 类型的话，那么 65529 字符长度是编码后的长度，而不是源文件）。  
> [Data URLs](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/data_URIs)



### 在 markdown cell 里 escape `$`

有些编程语言如 CSharp 等会大量使用 `$` 符号, 在写笔记时 markdown cell 会把成对 `$` 中间的内容识别为 latex, 需要禁用这个功能

Jupyter Notebook 的 markdown 是 gfm, 所以又分为两种情况: 

1. 禁止 markdown cell 中的普通文本 (例如 `$xxx$`) 被识别为 latex
2. 禁止 markdown cell 中的 codeblock (例如 ```c#\n\n $xxx$ \n```) 被识别为 latex

对于第一种情况, 在 custom.js 里添加如下内容就可以了, 改掉 latex 的定界符 ['$','$']

    MathJax.Hub.Config({
        tex2jax: {
            inlineMath: [['==','=='], ['\\(','\\)']],  // 这里默认设置是 [['$','$'], ['\\(','\\)']],
            processEscapes: true
        }
    });

注意这个更改必须是在 custom.js 里, 完后需要重启 jupyter (如果已经打开了 notebook, 只是添加一个 %%html 开头的 cell 里去设置, 没用)

对于第二种情况, 没找到好办法, 一旦进入 codeblock, 转换规则就不受 MathJax.Hub.Config 控制了

目前唯一找到的办法是换行, 例如 `$xxxxx \n\n xxxx$` 这之间有连续两个换行, 就不会被识别为 latex



### 使用其他编程语言执行 cell code

Jupyter Notebook 是有许多种编程语言 kernel 的, 但有时候在 python notebook 中, 要临时对单独的 cell 换个语言解释执行, 或者在一个 notebook 里同时需要很多种语言解释执行, 可以自定义 cellmagic 来实现这个功能

在 Jupyter Notebook 里, cellmagic 是指以两个 `%` 开头的魔法命令(linemagic 是一个 `%`)

这里自定义一个 `%%csharp`, 用于执行 C# 代码, 首先找个普通的 cell 粘贴运行以下代码:


```python
import subprocess
import os
from IPython.core.magic import register_cell_magic

def run_command(cmd):
  print('> running: ', cmd)
  try:
    output = subprocess.check_output(
        cmd, stderr=subprocess.STDOUT, shell=True, timeout=3,
        universal_newlines=True)
  except subprocess.CalledProcessError as exc:
    print("status: FAIL", exc.returncode, exc.output)
    return exc.returncode
  else:
    print("output: \n{}\n".format(output))
    return 0

def compile_and_run(code, filename):
  with open(filename + '.cs', 'w') as f:
    f.write(code)
  returncode = run_command('mcs {}.cs'.format(filename))
  if returncode == 0:
    run_command('{}.exe'.format(filename))

@register_cell_magic
def csharp(line, cell):
    "csharp build cell magic"
    compile_and_run(cell, filename=os.getcwd().replace('\\', '/')+'/temp')
```

上面的代码中, 最重要的是 `def csharp(line, cell):` 这里, 决定了自定义的 cellmagic 怎么触发, 其余两个函数 `compile_and_run` `run_command` 是为了方便调用的, 未来可以改造成执行 C 代码, 等等

运行之后, 新建个 cell, 在最开始指定 `%%csharp`, 就可以使用 CSharp 执行了 (这里我的环境是 Mono, 所以是 `mcs file.cs`)


到这一步仍然有问题, 还需要对 cell 指定对应语言的语法高亮, 否则会沿用 python 的语法高亮

方法是在 custom.js 里添加一段:


    require(['notebook/js/codecell'], function(codecell) {
        codecell.CodeCell.options_default.highlight_modes['magic_text/x-csharp'] = {'reg':[/^%%csharp/]};
        Jupyter.notebook.events.one('kernel_ready.Kernel', function(){
            Jupyter.notebook.get_cells().map(function(cell){
                if (cell.cell_type == 'code'){ cell.auto_highlight(); } 
            }) ;
        });
    });

就可以了, 这样就可以对 `%%csharp` 开头的 cell 显示为 CSharp 的语法高亮

注意: `'magic_text/x-csharp'` 是各种语言的标识, 通常都可以直接猜出来, 但 C 语言是 `x-csrc` 而不是 `x-c`, 所以对于 C 语言, 这里的写法应该是:

    codecell.CodeCell.options_default.highlight_modes['magic_text/x-csrc'] = {'reg':[/^%%gcc/]};