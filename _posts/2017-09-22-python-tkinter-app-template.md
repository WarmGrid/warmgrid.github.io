---
layout: post
title: 总结 Python Tkinter GUI 常用代码片段
tags:
- Python
- Programming
- Tkinter
- GUI
status: publish
type: post
published: true

summary: '方便快速做出带界面的小程序'

---


平时有做各种桌面小程序的需求, 在 Python 各界面库里折腾一圈, 最后还是回到了 Tkinter, 感觉好处如下:

- Python 原生支持
- 打包之后体积小, Pyinstaller 生成 exe 一般 10m (没太多依赖的话)
- ttk 做出的界面足够精致

当然, Tkinter 不太能够胜任界面元素复杂的应用, 那种情况还是使用重量级的界面库. 

总结一些 Tkinter 常用代码片段如下.


## 兼容支持 Python2 Python3 的模块导入

```python
try:
    # for python2
    import Tkinter as tkinter  
    import Tix as tix
    from tkinter.ScrolledText import ScrolledText
except ImportError:
    # for python3
    import tkinter              
    from tkinter import tix
    from tkinter.scrolledtext import ScrolledText
```


## Thinker 中的 log 面板

```python

import logging
# Logging configuration
logging.basicConfig(filename=__file__.replace('.py', '.log'),
                    level=logging.DEBUG,   # choose debug will show all logs
                    format='%(asctime)s - %(levelname)s - %(message)s')
logger = logging.getLogger()
logger.addHandler(logging.StreamHandler(sys.stdout))


class TextHandler(logging.Handler):
  def __init__(self, widget):
    logging.Handler.__init__(self)
    self.setLevel(logging.DEBUG)
    self.widget = widget
    self.widget.config(state='disabled')
    self.widget.tag_config("INFO", foreground="black")
    self.widget.tag_config("DEBUG", foreground="grey")
    self.widget.tag_config("WARNING", foreground="orange")
    self.widget.tag_config("ERROR", foreground="red")
    self.widget.tag_config("CRITICAL", foreground="red", underline=1)
    self.red = self.widget.tag_configure("red", foreground="red")
  def emit(self, record):
    self.widget.config(state='normal')
    # Append message (record) to the widget
    self.widget.insert(tkinter.END, self.format(record) + '\n', record.levelname)
    self.widget.see(tkinter.END)  # Scroll to the bottom
    self.widget.config(state='disabled')
    self.widget.update() # Refresh the widget


class GUI(tkinter.Frame):
    def __init__(self, master=None):
        tkinter.Frame.__init__(self, master)
        self.root = master
        self.build_logger_panel(row=5)
        logger.info('logger info')
        logger.debug('logger debug')
        logger.error('logger error')

    def build_logger_panel(self, row):
        st = ScrolledText(self.root, state='disabled', width=30, height=20)
        st.configure(font='TkFixedFont', width=70)
        # sticky='we' 吸附到左右两侧
        # columnspan=2 将布局视作 table, 该 widget 跨 2 个列
        st.grid(column=0, row=row, sticky='we', columnspan=2)
        text_handler = TextHandler(st)
        logger.addHandler(text_handler)
```




## Thinker 中的拖放文件支持

需要安装 TkDND (Tcl Plugin) 和 TkinterDnD2 (Python bindings)

- TkDND (Windows 环境需要选 tkdnd2.8-win32-x86_64.tar.gz) [https://sourceforge.net/projects/tkdnd/files/Windows%20Binaries/TkDND%202.8/]()
- TkinterDnD2 [https://sourceforge.net/projects/tkinterdnd/files/TkinterDnD2/]()

```python

try:
    import Tkinter as tkinter   # python2
except ImportError:
    import tkinter              # python3

from TkinterDnD2 import TkinterDnD

class GUI(tkinter.Frame):

    def __init__(self, master=None):
        tkinter.Frame.__init__(self, master)
        self.root = master
        self.build_gui()

    def build_gui(self):
        text, default_value = 'local file path', '...drag file in...'
        label, entry = self.build_label_value_block(text, default_value, position=(0, 0))
        self.add_drop_handle(entry, self.handle_drop_on_local_path_entry) # 给 widget 添加拖放方法

    def build_label_value_block(self, label, default_value, position=(0, 0), entry_size=20):
        label = tkinter.Label(self.root, text=label)
        label.grid(row=position[0], column=position[1], sticky='we', padx=3, pady=1)
        entry = tkinter.Entry(self.root, width=entry_size)
        entry.grid(row=position[0], column=position[1]+1, sticky='we', padx=3, pady=1)
        entry.insert(0, default_value)
        return label, entry

    def add_drop_handle(self, widget, handle):
        widget.drop_target_register('DND_Files')

        def drop_enter(event):
            event.widget.focus_force()
            print('Entering widget: %s' % event.widget)
            return event.action
        def drop_position(event):
            print('Position: x %d, y %d' % (event.x_root, event.y_root))
            return event.action
        def drop_leave(event):
            # leaving 应该清除掉之前 drop_enter 的 focus 状态, 怎么清?
            print('Leaving %s' % event.widget)
            return event.action

        widget.dnd_bind('<<DropEnter>>', drop_enter)
        widget.dnd_bind('<<DropPosition>>', drop_position)
        widget.dnd_bind('<<DropLeave>>', drop_leave)
        widget.dnd_bind('<<Drop>>', handle)

    def handle_drop_on_local_path_entry(self, event):
        if event.data:
            print('Dropped data: %s' % event.data)
            # => ('Dropped data:\n', 'C:/tkDND.htm C:/TkinterDnD.html')
            # event.data is a list of filenames as one string;
            files = event.widget.tk.splitlist(event.data)
            for f in files:
                if os.path.exists(f):
                    print('Dropped file: "%s"' % f)
                    event.widget.delete(0, 'end')
                    event.widget.insert('end', f)
                else:
                    print('Not dropping file "%s": file does not exist.' % f)
        return event.action


if __name__ == '__main__':
    # 注意这里是 TkinterDnD
    root = TkinterDnD.Tk()
    gui = GUI(master=root)
    gui.mainloop()

```



## Pyinstaller 打包为 exe

```python
import os
cmd = 'pyinstaller <main_file.py> -F -w -i icon.ico'
os.system(cmd)
```

`-F` 只生成一个文件, 默认生成文件夹, `-w` 不显示命令行窗口.

在 Pyinstaller 中指定 `-i icon.ico` 可以让打包的 exe 显示出图标, 但是运行后主程序仍然只显示默认图标, 尝试了 `--add-binary icon.ico;icon.ico` 也没用. 可能跟主程序代码有关. 目前只能是在打包生成 exe 后, 同路径下再粘贴一个单独的图标文件.



## 杂项

- 窗口置顶
- 自定义标题
- 显示自定义的图标
- 按 ESC 退出

```python
if __name__ == '__main__':
    root = tkinter.Tk()

    # 窗口置顶
    root.lift()                         
    root.attributes('-topmost', True)

    # 窗口标题
    root.title('TITLE')

    # 在 Windows 程序中显示图标
    if os.path.exists('icon.ico'):
        root.iconbitmap('icon.ico')

    # 绑定 ESC 退出
    root.bind('<Escape>', lambda x: sys.exit())

    gui = GUI(master=root)
    gui.mainloop()
```


## Thinker 各模块的结构

```python
# 如果样式太难看, 把所有形如 tkinter.Radiobutton 改成 ttk.Radiobutton
import ttk
```

todo
















































