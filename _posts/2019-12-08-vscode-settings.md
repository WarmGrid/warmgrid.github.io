---
layout: post
title: 自用 VSCode 常用配置记录
tags:
- IDE
- Programming
status: publish
type: post
published: true

summary: ''
---







### Python Test Explorer

比自带单元测试方便, 显示更清楚

最好禁用掉 VSCode 自带测试模块, 否则 "Run Test" 等会显示两次, 相互干扰

```
"python.testing.pytestEnabled": false
```



### Bracket Pair Colorizer 2

```json
    "bracketPairColorizer.consecutivePairColors": [
        "()",
        "[]",
        "{}",
        [
            "YellowGreen",
            "Orchid",
            "SkyBlue",
            "Tan",
        ],
        "Red"
    ],
```



### Markdown Preview Enhanced



参考 [在 VSCode 下用 Markdown Preview Enhanced 愉快地写文档 - 知乎](https://zhuanlan.zhihu.com/p/56699805)

启用 "执行代码块" 功能

```
"markdown-preview-enhanced.enableScriptExecution": true
```

使用样例

```
语法为
​```python{cmd="path/to/python"}
# 要执行的代码
# 要执行的代码
# 要执行的代码
​```

以下隐藏代码块, 且输出部分用 markdown 渲染
​```python{cmd="path/to/python" hide output="markdwon"}
# 要执行的代码
# 要执行的代码
# 要执行的代码
​```
```




































