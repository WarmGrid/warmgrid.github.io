---
layout: post
title: Jupyter Notebook 的几个使用技巧
tags:
- Python
- Programming Tools
status: publish
type: post
published: true

summary: '尽可能武装成全能笔记本'

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
    
    图片


```python
plt.plot([1,3,4,2]);  # 加分号, 不再打印这行文字
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
