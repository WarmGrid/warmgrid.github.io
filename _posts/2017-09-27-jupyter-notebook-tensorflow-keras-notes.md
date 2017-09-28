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



### 在 Jupyter Notebook 中使用 Sublime Text 风格快捷键

把下面代码放在 `C:\Users\<UserID>\.jupyter\custom\custom.js` 里面.

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





### 在 Jupyter Notebook 中设置自定义快捷键和 cell 背景色

把下面代码放在 `C:\Users\<UserID>\.jupyter\custom\custom.js` 里面.

- 给运行当前 dell 设置了额外的快捷键: F5 / Ctrl+.
- 设置 python indent 为 2 空格
- 依据 ipynb 的文件名给 cell 加上特定背景色

设置背景色是为在打开许多个 notebook 时, 可以简单的区分它们, 另外, 执行中的 cell 会显示为更深的背景色.

```javascript
setTimeout(function() {
    Jupyter.keyboard_manager.command_shortcuts.add_shortcut('f5', {
        help : 'run cell',
        handler : function (event) {
            IPython.notebook.execute_cell();
            return false;}});
    Jupyter.keyboard_manager.command_shortcuts.add_shortcut('ctrl-.', {
        help : 'run cell',
        handler : function (event) {
            IPython.notebook.execute_cell();
            return false;}});
    Jupyter.keyboard_manager.edit_shortcuts.add_shortcut('f5', {
        help : 'run cell',
        handler : function (event) {
            IPython.notebook.execute_cell();
            return false;}});
    Jupyter.keyboard_manager.edit_shortcuts.add_shortcut('ctrl-.', {
        help : 'run cell',
        handler : function (event) {
            IPython.notebook.execute_cell();
            return false;}});
    Jupyter.keyboard_manager.edit_shortcuts.add_shortcut('ctrl-enter', {
        help : 'none',
        // 防止与 Sublime hotkey Ctrl+Enter 冲突
        handler : function (event) {
            return false;}});

    // 设置 python indent 为 2 空格
    var patch = {CodeCell: {cm_config:{indentUnit: 2}}}
    Jupyter.notebook.get_selected_cell().config.update(patch)

    // 依据 ipynb 文件名, 给 cell 加上特定的背景色
    String.prototype.hashCode = function() {
      var hash = 0, i, chr;
      if (this.length === 0) return hash;
      for (i = 0; i < this.length; i++) {
        chr   = this.charCodeAt(i);
        hash  = ((hash << 5) - hash) + chr;
        hash |= 0; // Convert to 32bit integer
      }
      return hash;
    };

    function random_hue_color(label, s, l) {
      // console.log(Math.abs(label.hashCode()))
      var hash_color = (Math.abs(label.hashCode()) % 360) / 360 * 100
      return `hsl(${hash_color}, ${s}%, ${l}%)`
    }

    var notebook_path = IPython.notebook.notebook_path
    var color1 = random_hue_color(notebook_path, 20, 90)
    var color2 = random_hue_color(notebook_path, 40, 80)

    var css = document.createElement("style")
    css.type = "text/css"
    css.innerHTML = `div.cell {background-color: ${color1};}`
    css.innerHTML +=`div.running {background-color: ${color2};}`
    css.innerHTML +=`div.running.selected {background-color: ${color2};}`
    css.innerHTML +='</style>'
    document.body.appendChild(css);
}, 2000)

```

如果只针对一个 notebook 做此设置, 把 setTimeout 里的内容放在 cell 里执行即可, cell 第一行要加上 `%%javascript`.





### 设定 Jupyter 主题

首先安装 `jupyterthemes`:

    pip install --upgrade jupyterthemes

列出预设的主题:

    jt -l

选择主题:

    jt -t grade3 -fs 14 -cellw 1100 -f meslo

目前的主题有:

- chesterish
- grade3
- gruvboxd
- gruvboxl
- monokai
- oceans16
- onedork
- solarizedd
- solarizedl

似乎 solarizedl 不能用.





### Matplotlib 的几种 multiple subplot 写法

```python
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
plt.rcParams['figure.figsize'] = (10.0, 8.0) # set default size of plots
plt.rcParams['image.interpolation'] = 'nearest'
plt.rcParams['image.cmap'] = 'gray'

# multiple plot 方法一 
ax = plt.subplot(2, 1, 2)
plt.plot(hist.history['acc'], '-o')
plt.plot(hist.history['val_acc'], '-o')
plt.legend(['train', 'val'], loc='upper left')
plt.xlabel('epoch')
plt.ylabel('accuracy')
ax.set_ylim([0, 1])  # 这里是 ax, not plt


# multiple plot 方法二 to grid m*n
#　适合输出大量的小图片
f, plots = plt.subplots(21, 10, sharex='all', sharey='all', figsize=(10, 21))
for i in range(203):
    plots[i // 10, i % 10].axis('off')
    plots[i // 10, i % 10].imshow(pat[i], cmap=plt.cm.bone)


# multiple plot 方法三 span plot
# 将框架视为表格, subplot 可以跨行跨列, 并且 index 从 0 开始计数, 更符合直觉...
ax4 = plt.subplot2grid((2, 3), (1, 0), colspan=2)


```





### 查看 Keras 安装信息

```python
import keras
keras.__version__

# >>> '2.0.8'
```





### 设置 Keras 图像通道的顺序

以 `version 2.0.8` 为例, 配置文件在 `C:\Users\<UserID>\.keras\keras.json` 中

```json
{
    "epsilon": 1e-07,
    "backend": "tensorflow",
    "floatx": "float32",
    "image_data_format": "channels_first"   // 另一个值是 "channels_last"
    
    // 对于 2D 图像, 
    // channels_first 对应的 input_shape 是 (3, 128, 128)
    // channels_last 对应的 input_shape 是 (128, 128, 3)
}
```





### Keras 输出模型结构图并显示在 notebook

```python
from keras.utils.visualize_util import plot
plot(model, show_shapes=True, to_file='model.png')

# 在 notebook 中显示 jpg / png
from IPython.display import display, Image
display(Image('model.png', width=500))

# 在 notebook 中显示 svg
from IPython.display import SVG
from keras.utils.visualize_util import model_to_dot
SVG(model_to_dot(model, show_shapes=True).create(prog='dot', format='svg'))

```

















































