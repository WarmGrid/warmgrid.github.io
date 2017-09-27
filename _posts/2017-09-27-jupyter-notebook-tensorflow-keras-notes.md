---
layout: post
title: Jupyter Notebook, TensorFlow, Keras 笔记
tags:
- Deep Learning
- Python
- Data Mining
- Programming Tools
status: publish
type: post
published: true

summary: '随时总结机器学习工具链的笔记'

---



[TOC]



## Jupyter Notebook

### 在 Jupyter Notebook 中使用 Sublime Text 风格快捷键

把下面代码放在 `C:\Users\<UserID>\.jupyter\custom\custom.js` 里面

```javascript
require(["codemirror/keymap/sublime", "notebook/js/cell", "base/js/namespace"],
    function(sublime_keymap, cell, IPython) {
        // setTimeout(function(){ // uncomment line to fake race-condition
        cell.Cell.options_default.cm_config.keyMap = 'sublime';
        var cells = IPython.notebook.get_cells();
        for(var cl=0; cl< cells.length ; cl++){
            cells[cl].code_mirror.setOption('keyMap', 'sublime');
        }

        // }, 1000)// uncomment  line to fake race condition
    }
);
```















## TensorFlow










## Keras

### 查看 Keras 安装信息

```python
import keras
keras.__version__
```
























































