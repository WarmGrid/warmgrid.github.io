---
layout: post
title: 使用应答文件实现 Windows 无人值守安装
tags:
- Windows
- System
status: publish
type: post
published: true

summary: '终于学会怎么做那种带一堆预装软件的盗版盘了, 这是第一部分'

---


### 应答文件 (Answer File) 简介

Windows 应答文件是一个名为 `Autounattend.xml` 的配置, 描述在系统安装过程中必须指定的内容. 这样在安装 Windows 系统中, 就不必手工选择怎么"格式化分区", 勾选"接受许可条款"...等等项目.

只要 Windows 安装启动时, 能在光驱, 软盘, USB 设备的根目录中, 找到任何一个名为 `Autounattend.xml` 的文件, 安装程序就会尝试解析该文件, 实现无人值守安装. 

如果是在虚拟机中安装系统, 可以给虚拟机添加另一个光驱, 把应答文件做成 iso 镜像放在这个虚拟光驱里. 这样更方便测试. 测试正常后, 再把应答文件编辑到 Windows 安装镜像 iso 的根目录, 这就和正常的安装镜像没什么区别了.


### 创建应答文件


应答文件就是个普通的 xml, 文件长这样子:

```xml
<settings pass="windowsPE">
    <component name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <SetupUILanguage>
            <UILanguage>zh-CN</UILanguage>
        </SetupUILanguage>
        <InputLocale>zh-CN</InputLocale>
        <SystemLocale>zh-CN</SystemLocale>
        <UILanguage>zh-CN</UILanguage>
        <UILanguageFallback>en-US</UILanguageFallback>
        <UserLocale>zh-CN</UserLocale>
    </component>
    <component 2>
    .......
    </component 2>
    <component 3>
    .......
    </component 3>
    .......
    .......
```

可以由 Windows AIK 的 "系统映像管理器" 来辅助生成, Windows AIK 是微软官方提供的工具包, 约1.7G, [下载地址](https://www.microsoft.com/en-us/download/details.aspx?id=5753)

步骤如下, 以 Windows Server 2008 为例:

1. 打开Microsoft Windows AIK -> Windows 系统映像管理器
2. 从 Windows Server 2008 安装镜像 iso 文件里取出 `Install.wim`
3. 在 Windows 系统映像管理器里, "选择 Windows 映像", 载入之前的 `Install.wim`
4. 创建编录文件 (*.clg), 这个东西对应具体的一个 Windows 系统, 一个 `Install.wim` 里可以有很多系统, 比如 "Windows Server 2008 R2 企业版" / "Windows Server 2008 R2 数据中心版" 等等
5. 新建应答文件, 然后依次添加应答文件的各种选项
6. 保存时, 工具会检查你的填写是否有错误, 不报警就 OK 了

制作应答文件是个细心的活儿, 最好手边就有个虚拟机方便你测试. 显然要填全所有项目, 如果漏填了某些项, 那么安装过程到时候还是会停下来等待用户输入, 这就没有意义了. 


### 应答文件中有用的项目

- 设置语言 (分为安装过程的语言和 Windows 操作系统的语言)
- 磁盘格式化, 分区
- 指定安装哪个操作系统
- 设置管理员密码
- 自动同意使用者授权协议
- 设置启动系统时要自动执行的命令

指定安装哪个操作系统的代码大致是这样, 

```xml
<ImageInstall>
    <OSImage>
        <InstallFrom>
            <MetaData wcm:action="add">
                <Key>/IMAGE/NAME</Key>
                <Value>Windows Server 2008 R2 SERVERENTERPRISE</Value>
            </MetaData>
        </InstallFrom>
        <InstallTo>
            <DiskID>0</DiskID>
            <PartitionID>1</PartitionID>
        </InstallTo>
    </OSImage>
</ImageInstall>
```

一个 wim 里可能对应许多版本的安装程序, 查看具体的安装程序名字,  可以用 Windows AIK 里的 `imagex` 工具:

    imagex /info C:\install.wim 


具体不多写了, 请参考

- [https://msdn.microsoft.com/zh-cn/library/hh825212.aspx#AnswerFile]()
- [http://www.cnblogs.com/dreamer-fish/p/3476921.html]()
- [http://www.cnblogs.com/dreamer-fish/p/3468388.html]()


### 在应答文件中设置安装后自动执行命令

    在 amd64_Microsoft-Windows-Shell-Setup_neutral/FirstLogonCommands 下
    新建 AsynchronousCommand
    FirstLogonCommands 只在第一次登录时运行
    
    在 amd64_Microsoft-Windows-Shell-Setup_neutral/LogonCommands 下
    新建 AsynchronousCommand
    LogonCommands 在每次登录时都运行一次

需要管理员权限, 另外这个 "执行命令" 是要填写在 "开始菜单->运行" 里运行命令, 不是在 cmd 里的执行命令. 

比如, 填写 `notepad` 是有效的, 填写 `echo test_line > C:\test.txt` 无效, 因为 `echo` 是在 `cmd.exe` 里的命令.

如果需要 `cmd` 里执行命令, 这里要写成 `cmd /s echo test_line &gt; C:\test.txt`


### 使用应答文件

只要应答文件名叫 `Autounattend.xml` (不区分大小写), 且位于光驱, 软盘, USB 设备的根目录中, 就能在安装镜像启动时被识别出来, 于是就不用手工指定安装信息啦. 

想在光驱中使用应答文件, 那就要把应答文件做成 iso 镜像, 可以使用 Windows AIK 里的 `oscdimg` 工具做 iso 镜像, 这个在 `开始 -> Microsoft Windows AIK -> 部署工具命令提示` 中, 写法是:

    > oscdimg -n <folder contain Autounattend.xml> <iso_filename.iso>

注意是把整个文件夹做成 iso 文件! 比如:

    > oscdimg -n unattend_folder unattend.iso

要注意启动顺序, Windows 安装镜像必须放在第一个加载的设备, 应答文件放第二个. 当你有两个(虚拟)光驱时别弄反了. 测试成功后, 可以把 `Autounattend.xml` 编辑到 iso 镜像的根目录里, 这样就只用一个光驱, 更方便使用.





