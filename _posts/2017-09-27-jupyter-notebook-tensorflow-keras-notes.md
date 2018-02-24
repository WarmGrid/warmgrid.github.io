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





### 运行 Jupyter Notebook Cell 时显示执行状态

主要依靠 `sys.stdout.flush()` 和 `print('\r...')`.

`\r` 表示从行首打印.

```python
import time
for i in range(10):
    time.sleep(0.3)
    print('\r任务进度: {0}%'.format(i*10+10), end='')
    sys.stdout.flush()



# >>> 任务进度: 70%  (会不断输出更新到这一行)
```





### 模块变更后让 Jupyter Notebook 能自动载入


```python
%load_ext autoreload
%autoreload 2
```





### 用指定的浏览器启动 Jupyter Notebook

在 `C:\Users\<USER>\.jupyter` 路径下应该有一个 `jupyter_notebook_config.py` 配置文件. 如果没有, 运行 `jupyter notebook --generate-config` 生成它.

里面所有内容都是被注释掉的, 找到 browser 的部分, 添加类似下面的内容:

```python
import webbrowser
webbrowser.register('firefox', None, webbrowser.GenericBrowser('C:/Program Files/Mozilla Firefox/firefox.exe'))
c.NotebookApp.browser = 'firefox'
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
    css.innerHTML +=`div.CodeMirror {font-family: "Yahei Mono"; font-size: 20px;}`
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


### 调整 pandas, plt 的预设

包括 pandas 打印时的每行最大字符数, 和 matplotlib 的中文显示, 字体大小, 数字负号显示等等

```python

import numpy as np
import pandas as pd

pd.set_option('display.width', 200)   # 每行最大字符
pd.set_option('precision', 3)         # 显示数字精度


import matplotlib as mpl
import matplotlib.pyplot as plt
%matplotlib inline

plt.rcParams['axes.unicode_minus'] = False     # 显示数字负号
plt.rcParams['font.sans-serif'] = ['SimHei']   # 显示中文字体

mpl.rcParams['figure.figsize'] = 10, 6
mpl.rcParams['figure.dpi'] = 80
mpl.rcParams['savefig.dpi'] = 100
mpl.rcParams['font.size'] = 12
mpl.rcParams['legend.fontsize'] = 'large'
mpl.rcParams['figure.titlesize'] = 'medium'

plt.style.use('seaborn-whitegrid')

import seaborn as sns
sns.set(style="white")
```





### 指定 plt 尺寸

除了 `mpl.rcParams['figure.figsize'] = 10, 6` 办法之外, 还可以

    fig = plt.figure(figsize=(8, 8))
    ax = fig.add_subplot(111)
    # 或者
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 6))





### 在subplot 时准确指定位置, 支持跨行列

造一个函数用来返回 subplot 时的 ax, 接收形如 '2x3,1x0,1x2' 的参数 position

- 第一组表示图表的整体长宽, 如 2x3 表示 2 行, 3 列
- 第二组表示该区域的左上角座标, 从 0 开始计数
- 第三组表示是否跨行列, 如 1x2 表示跨 1 行, 跨 2 列

```python
def get_subplot_ax(position):
    ''' 注意 grid, index, span 都是先 行 后 列'''
    position = position.split(',')
    if len(position) == 3:
        grid, position, span = position
    elif len(position) == 2:
        grid, position = position
        span = '1x1'
    elif len(position) == 1:
        grid = position[0]
        position = span = '1x1'
    to_tuple = lambda s: tuple(int(n) for n in s.split('x'))
    rowspan, colspan = to_tuple(span)
    ax = plt.subplot2grid(to_tuple(grid), to_tuple(position), colspan=colspan, rowspan=rowspan)
    return ax
```





### 在 subplot 的多个图中保持座标轴 range 一致

需要用到 sharex sharey 属性, 每个属性都有 col row 两个值, 这四种组合都有用

```python
f, (ax1, ax2) = plt.subplots(2, 1, sharey='col')
# sharey='col' 当 subplots 处在同一列时, 共享 y 轴
# sharey='row' 当 subplots 处在同一行时, 共享 y 轴
```





### Matplotlib 直接引用 seaborn 绘图样式

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl
plt.style.use('ggplot')

data = np.random.randn(50)
```


全部样式如下:

```python
print(plt.style.available)

# => ['seaborn-paper', 'seaborn-poster', 'seaborn-darkgrid', 'fast', 'ggplot', 'seaborn-pastel', 'bmh', 'seaborn-deep', 'fivethirtyeight', 'grayscale', 'seaborn', 'seaborn-talk', 'seaborn-bright', 'seaborn-white', 'seaborn-ticks', 'seaborn-dark-palette', 'seaborn-dark', '_classic_test', 'seaborn-colorblind', 'dark_background', 'seaborn-whitegrid', 'seaborn-notebook', 'Solarize_Light2', 'seaborn-muted', 'classic']
```

这些样式都很好:

- seaborn-darkgrid
- seaborn-pastel
- bmh
- ggplot


### Matplotlib 使用 seaborn 调色板中的单个颜色

预览 seaborn 色板

```python
sns.palplot(sns.color_palette("Set2", 9))
sns.palplot(sns.cubehelix_palette(8))
sns.palplot(sns.color_palette("coolwarm", 12))
sns.palplot(sns.color_palette("hls", 6))
sns.palplot(sns.color_palette("husl", 6))
```

使用色板中的颜色

```python
import seaborn as sns
import itertools

palette = sns.color_palette(sns.color_palette("Set2", 12))
palette = itertools.cycle(palette)

# 之后在需要使用颜色的地方:
ax.plot(seq, '.-', color=next(palette))

```

注意必须用 `itertools.cycle()`, 如果只是 `palette = iter(palette)` 转迭代器, 那么 `next(...)` 达到次数后会遇到 `StopIteration`. 而 `itertools.cycle()` 是无限的, 可以一直循环这些颜色.




### Matplotlib 的命名颜色

以下代码可以打印所有的命名颜色

```python
import matplotlib
for name, hex in matplotlib.colors.cnames.items():
    print(name, hex)

pylab.rcParams['figure.figsize'] = 8, 24

import matplotlib.pyplot as plt
import matplotlib.patches as patches
import matplotlib.colors as colors
import math

fig = plt.figure()
ax = fig.add_subplot(111)

ratio = 1.0 / 3.0
count = math.ceil(math.sqrt(len(colors.cnames)))
x_count = count * ratio
y_count = count / ratio
x = 0
y = 0
w = 1 / x_count
h = 1 / y_count

for c in colors.cnames:
    pos = (x / x_count, y / y_count)
    ax.add_patch(patches.Rectangle(pos, w, h, color=c))
    ax.annotate(c, xy=pos)
    if y >= y_count-1:
        x += 1
        y = 0
    else:
        y += 1
plt.show()
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

















































