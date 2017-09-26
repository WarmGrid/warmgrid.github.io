---
layout: post
title: 使用应答文件和映像捕获制作自定义 Windows 安装系统
tags:
- Windows
- System
status: publish
type: post
published: true

summary: '终于学会怎么做那种带一堆预装软件的盗版盘了, 这是第二部分'

---

本文介绍怎么制作包含了用户软件的, 并且可无人值守安装的 Windows 镜像. 整个流程大致是:

1. 制作 Windows 安装应答文件
2. 正常安装一个 Windows 操作系统, 完后装好常用的软件
3. 清理这个新安装的系统, 去掉安装中指定的用户信息, 仅留下预装软件
4. 做出带 imagex 映像捕获功能的 Windows PE 启动盘
5. 用 Windows PE 启动, 捕获 2 中的操作系统, 生成 *.wim 映像
6. 把 *.wim 和安装应答文件编辑到普通的 Windows iso 镜像中

其中的两个技术, `安装应答文件` 和 `映像捕获` 是独立的, 前者负责无人值守安装, 后者负责在安装包里添加用户指定的软件. 可以根据需求只关注其中的一项. 

步骤 1 请参考之前的 [文章](/2017/09/18/install-windows-with-answer-file.html), 本篇介绍后面的步骤 2-6.



### 安装 Windows 系统和常用软件

这一步没什么好说的, 就是最普通最正常的安装一遍 Windows 系统. 不必太认真设定用户帐号, 密码, 驱动等, 因为之后的步骤里这类信息会被抹掉. 完后可以给系统装些自己习惯用的软件, 例如 Chrome, Everything 等. 以下步骤均以安装 Windows Server 2008 为例.



### 清理系统, 构建 Windows 参考安装

这一步是抹去用户帐号等信息, 留下干净的 Windows 系统, 即 "参考安装". 使用的是 Windows 自带的 `sysprep` 工具. 在命令行执行:

    C:\windows\system32\sysprep\sysprep.exe /oobe /generalize /shutdown

其中, `/generalize` 将删除硬件特定的信息, `/oobe` 将设置下次重启时进到"欢迎使用 Windows"界面. 关机后, 就不要再启动这个系统了.



### 创建 Windows PE 启动盘

这一步要找一台安装了 [Windows AIK 工具包](https://www.microsoft.com/en-us/download/details.aspx?id=5753) 的电脑. 运行 `开始` -> `Microsoft Windows AIK` -> `部署工具命令提示` (需要管理员权限), 在开启的窗口中执行:

    copype.cmd x86 C:\winpe_x86

64 位系统要把所有的 x86 改成 x64.

这时 `C:\winpe_x86` 下应该生成了一整套文件夹, 可以检查一下. 

检查无误后, 要把 `imagex.exe` 复制进去, 执行以下命令或手工复制均可. 注意 64 位机器的 `imagex.exe` 是在 `Windows AIK\Tools\amd64` 里!

    copy "C:\Program Files\Windows AIK\Tools\x86\imagex.exe" c:\winpe_x86\iso\

再把 `C:\winpe_x86\winpe.wim` 复制到 `C:\winpe_x86\ISO\sources\boot.wim`.

这时可以制作 Win PE 镜像了, 还是在 `开始` -> `Microsoft Windows AIK` -> `部署工具命令提示`, 执行:

    oscdimg -n -bC:\winpe_x86\etfsboot.com C:\winpe_x86\ISO C:\winpe_x86\winpe_x86.iso

这一步是把 ISO 文件夹的内容做成 `*.iso` 镜像. 一般的 cmd 命令里, 参数标志和参数值之间, 加不加空格均可, 但这个 `oscdimg` 的 `-b` 后面必须不能加空格. 做出来的镜像应该在 200m 左右.



### 捕获参考安装至 *.wim 映像

使用 "上一步" 的 Win PE 镜像启动 "上上一步" 抹掉用户信息的 Windows 参考安装. 然后执行捕获, 最后会得到一个 `*.wim` 格式的文件, 这就是 Windows 安装镜像的核心文件.

    E:\imagex.exe /capture D: D:\myimage.wim "Custom Server 2008 Install" /compress fast /verify

这个步骤有点儿像做 ghost 备份的感觉. 不同的地方是, `.wim` 的来源和存放位置可以是同样一个分区, 很先进. 执行命令前, 可能需要通过 dir 了解一下每个盘的盘符. 比如我自己执行时, `E:` 是 Win PE 的文件内容, `D:` 是 Windows 参考安装的系统盘. 此外, 要留意整个参考安装系统的大小, 如果预装大量软件, 那么 `/compress fast` 也是有可能超过 4G 的, 在 FAT32 系统里会失败. 最后, 要记住自己设的 label "Custom Server 2008 Install", 之后在 `Autounattend.xml` 里有用.

这一步时间比较长, 完后无论使用什么手段, 要把生成的 wim 文件弄出来. 如果是宿主机上运行虚拟机里的 Win PE, 可以考虑使用网络共享:

    # (宿主机事先建好一个共享文件夹)
    # 在 Win PE 启动的系统中
    > net use Y: \\<主机计算机名>\<主机共享文件夹名> 
    # (输入宿主机用户名, 需要带着机器名) 如 \\<主机计算机名>\<主机用户名>   
    # (输入宿主机用户密码)
    > copy D:\myimage.wim Y:



### 把 wim 编辑到 iso 安装镜像中

有了 wim 文件后, 可以把它用于网络部署, 也可以把它添加在 iso 中, 这样就是一个预装了自己软件的安装镜像.

使用 Windows AIK 的映像管理器对 wim 文件生产编录, 如 "install_winserver2008 custom.clg", 印象中这个 label 是对应此前 `imagex.exe /capture` 时自己设的 label.

找对应的 Windows 安装镜像, (比如我们准备的是 Windows Server 2008，则这时也用 2008 的 iso)，使用 UltraISO 等工具, 删掉里面的 install.wim 和所有的 *.clg 文件，把我们自己的 wim 和 clg 放进去, 重新保存为 iso.

完成! 我们终于有了一个预装好自定义软件的 Windows 安装镜像. 可以结合 `安装应答文件` 的部分, 把安装时的初始设定也做成无人值守的, 就更加完美了~






























































