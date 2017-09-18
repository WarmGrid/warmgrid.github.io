---
layout: post
title: 总结 Windows 中的一些实用工具
tags:
- Windows
- Software
- Hotkey
status: publish
type: post
published: true

summary: '每天都在用的'

---


### Cmder

[http://cmder.net/]()

命令行工具, 自带 git, 可注册到右键菜单, 自定义 alias, Ctr+V直接粘贴, 选中文字直接复制, 等等. 介绍见:

[https://jeffjade.com/2016/01/13/2016-01-13-windows-software-cmder/]()




### Wox 

[http://www.getwox.com/]()

Launchy 类的快速启动程序. 输入拼音(字头)可以直接对应到中文. 计算器, 搜索, 替代命令行等功能都有. 似乎比同类软件慢, 但插件很强大.

介绍文章 [http://www.iplaysoft.com/wox.html]()

Dash Doc 插件 [http://www.getwox.com/plugin/9]()

可以把各类文档集成在 Wox 里面查询, 支持所有的 Dash Doc. 用法:

1. 安装插件 `wpm install Dash.Doc`
2. 下载 Doc 资源, 解压放在 `Plugins\{docplugin-directory}\Docset\{docname}`
3. 注意命名时不能有空格, 应以下划线代替, 类似 `Python_3.docset` 这样
4. 重启 Wox

- Dash 的官方文档 [https://github.com/Kapeli/feeds]()
- Dash 的用户添加文档 [https://github.com/Kapeli/Dash-User-Contributions/tree/master/docsets]()




### OneQuick

[https://github.com/XUJINKAI/OneQuick]()

屏幕边缘操作以及剪贴板增强, 给屏幕四周设定了不同的操作, 如下, 可以自己定制, 语法同 AutoHotkey 语法.

    左上角 · LT
    滚轮/单击滚轮：音量大小/静音
    shift + 滚轮：屏幕亮度
    右键：窗口操作菜单

    右上角 · RT
    滚轮/单击滚轮：歌曲上一首/下一首/播放或暂停
    右键：用户自定义菜单

    左边框 & 右边框 · L-R
    滚轮：翻页
    shift + 滚轮：翻页x5
    ctrl + shift + 滚轮：翻到第一页/最后一页

    上边框 · T
    滚轮：标签页切换

    下边框 · B
    滚轮/单击滚轮：(win10) 虚拟桌面切换/查看(win+tab键)

自己的设置


```yaml
hotkey:
  switch: 1
  buildin:
    win_z: xmenu_show_great_menu
    ctrl_alt_r: OneQuick.Command_run
    win_rclick: WinMenu.Show
    shift_wheeldown: "send {PgDn}"
    shift_wheelup: "send {PgUp}"
    win_t: sys.win.topmost
    # 当前窗口置顶
    win_n: notepad
    win_shift_l: Sys.Power.LockAndMonitoroff
    ctrl_shift_alt_r: OneQuick.Reload

screen-border:
  switch: 1
  action:
    LT:
      rclick: winmenu.show
      shift_wheeldown: Sys.Screen.BrightnessDown
      shift_wheelup: Sys.Screen.BrightnessUp
      wheelclick: "send {volume_mute}"
      wheeldown: "Send {volume_down}"
      wheelup: "Send {volume_up}"
    RT:
      rclick: xmenu_show_screen_rt_menu
      wheelclick: "send {media_play_pause}"
      wheeldown: "send {media_next}"
      wheelup: "send {media_prev}"
    T:
      wheelup: "send +#{left}"
      wheeldown: "send +#{right}"
      # 把窗口发到其它的屏幕
    B:
      wheelclick: "send !{tab}"
      # rclick: "send !{F4}"
      wheelup: Sys.Win.GotoPreTab
      wheeldown: Sys.Win.GotoNextTab
      win_wheeldown: "send ^#{right}"
      win_wheelup: "send ^#{left}"
      # 切换虚拟桌面, Windows 10 自带这功能, Windows 7 用同类软件
    L:
      lclick: "Send {enter}"
      wheelclick: "Send ^p"
      rclick: "Send ^s"
      wheelup: "send +^1"
      wheeldown: "send +^2"
    R:
      lclick: "Send ^x"
      wheelclick: "Send ^c"
      rclick: "Send ^v"
      wheelup: "send {backspace}"
      wheeldown: "send {delete}"



```




特别有名的就不说了, 包括但不限于:

- Everything: [https://www.voidtools.com/]()
- AutoHotkey: [https://autohotkey.com/download/]()
- MacType (已能很好的支持 Win10): [https://github.com/snowie2000/mactype/releases]()








































































