---
layout: post
category: programming
title: 为 Sublime Text 3 的 build 输出结果增加代码高亮
date: 2014-06-14
summary: 方法是在 build 配置里加 "syntax" 属性.

---


其实 Sublime Text 3 的 build 输出结果也可以有代码高亮:

Sublime Text 3 的 build 配置有个可选参数 "syntax" (见 [Build Systems](http://sublime-text-unofficial-documentation.readthedocs.org/en/latest/reference/build_systems.html)), 所以从安装路径里借用一个语法高亮配置, 将 "syntax" 的值设为这个文件的路径就行了.

首先需要有个 build 配置, 这个基本大家都配置过了, 以 Windows 上运行 Python 代码为例, `Tools` -> `Build System` -> `New Build System...` 新建一个配置, 这个文件通常要存在用户目录下:

    C:\Users\{用户名}\AppData\Roaming\Sublime Text 3\Packages\User\Python.sublime-build

build 配置的内容大致像这样, 注意需要增加 "syntax" 这个属性, 默认提供的模板是不带 "syntax" 这一行:

    {
      "cmd": ["python", "-u", "$file"],
      "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
      "selector": "source.python",
      // 增加下面这一行
      "syntax": "Packages/User/Python.tmLanguage",
      "encoding": "cp936"
    }

> Windows 上需要 `"encoding": "cp936"`, 否则中文字符不能正确显示, 这在网上是早就解决过的问题.

"syntax" 的值是后缀 `*.tmLanguage` 的语法高亮文件, 路径应该从 `Packages` 开始. 有两个方式可以取得这种语法高亮文件:

其一, Sublime Text 3 安装路径里就有一大堆这种东西, 默认是放在打包的 `*.sublime-package` 里(其实就是个 zip 包, 可以改后缀预览一下). 比如程序安装在 C 盘, 那么以下的包里就有一个语法高亮配置:

    C:\Program Files\Sublime Text 3\Packages\Python.sublime-package

把 "syntax" 设为 `"Packages/Python/Python.tmLanguage"` 就可以了, 这样就是直接引用安装路径的包里的高亮配置.

其二, 接上文, 这个包改成 zip 后缀就能把 `Python.tmLanguage` 复制出来, 于是也可以提取一个自己折腾... 自定义的配置还是统一放这里:

    C:\Users\{用户名}\AppData\Roaming\Sublime Text 3\Packages\User

那么 build 配置就应该像这样:

    "syntax": "Packages/User/MyBuildOutput.tmLanguage",




在 Mac 上的配置也相同:

    {
      "cmd": ["/Library/Frameworks/Python.framework/Versions/3.4/bin/python3", "-u", "$file"],
      "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
      "selector": "source.python",
      // 增加下面这一行
      "syntax": "Packages/User/Python.tmLanguage",
      "env": {"LANG": "en_US.UTF-8"}
    }


