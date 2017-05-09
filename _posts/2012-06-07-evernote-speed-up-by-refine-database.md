---
layout: post
title: 优化数据库，解决 Evernote 越用越慢的问题
tags:
- Evernote
- 随感
status: publish
type: post
published: true
summary: 使用 Evernote Debug 模式中的"优化数据库"，别忘提前备份！
---
我的 Evernote 中积攒了 1200+ 的笔记条目，最近发现使用速度越来越慢，没法忍受了。于是 Google 到一个解决方法，<a href="http://www.howtogeek.com/howto/40769/use-evernote%E2%80%99s-secret-debug-menu-to-optimize-and-speed-up-searching/">开启 Evernote Debug 模式优化数据库</a>，介绍如下：

1，首先备份 Evernote 数据库(*.exb文件)，在"工具" -&gt; "选项"中可以看到路径，WinXP 路径默认在

<code>C:\Documents and Settings\Administrator\Local Settings\Application Data\Evernote\Evernote\Databases</code>

并确认 Evernote 已经升级到最新，目前是 v4.5.6。

2，退出 Evernote，从任务管理器清理掉所有与 Evernote 相关的进程。

3，在 Evernote 程序安装路径进入 DOS 命令行，运行 <code>"evernote /debugmenu"</code>，Win7 下也可以开始菜单直接输入这个命令。

4，Evernote 的 Debug 模式会多出来一个菜单，执行"优化数据库"，此后要等上一段时间，优化完后发现在笔记间切换速度，搜索速度的确能够提升不少：）

退出程序，下一次正常启动 Evernote 就好了。

<img src="http://www.howtogeek.com/wp-content/uploads/2011/01/image44.png" alt="" />
