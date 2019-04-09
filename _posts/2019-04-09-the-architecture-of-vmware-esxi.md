---
layout: post
title: 翻译 - VMware ESXi 架构
tags:
- vSphere
status: publish
type: post
published: true

summary: '介绍 VMware ESXi 的组件, VMkernel, 系统映像, DCUI, 使用的端口, VI API 等'

---



## VMware ESXi的体系结构

### 介绍

VMware®ESXi是下一代虚拟机管理程序，为虚拟基础架构提供了新的基础。这种创新的架构独立于任何通用操作系统运行，提供更高的安全性，更高的可靠性和简化的管理。紧凑型架构旨在直接集成到虚拟化优化的服务器硬件中，实现快速安装，配置和部署。

从功能上讲，ESXi相当于ESX 3，可提供相同级别的性能和可扩展性。但是，已删除基于Linux的服务控制台，将占用空间缩减到小于32MB的内存。服务控制台的功能由新的远程命令行界面替代，并符合系统管理标准。由于ESXi在功能上与ESX相当，因此它支持整个VMware Infrastructure 3产品套件，包括VMware虚拟机文件系统，Virtual SMP，VirtualCenter，VMotion，VMware Distributed Resource Scheduler，VMware High Availability，VMware Update Manager和VMware Consolidated Backup。




### ESXi组件

VMware ESXi体系结构包括底层操作系统（称为VMkernel）以及在其上运行的进程。VMkernel提供了运行系统上所有进程的方式，包括管理应用程序和代理以及虚拟机。它可以控制服务器上的所有硬件设备，并管理应用程序的资源。在VMkernel上运行的主要进程是：

- Direct Console User Interface (DCUI) - 可通过服务器控制台访问的低层配置和管理界面，主要用于初始基本配置
- 虚拟机监视器，这是个为虚拟机提供执行环境的进程，以及一个称为VMX的帮助进程。每个正在运行的虚拟机都有自己的VMM和VMX进程
- 用于从远程应用程序启用高层VMware Infrastructure管理的各种代理
- 公共信息模型 (Common Information Model, CIM) 系统：CIM是通过一组标准API实现远程应用程序的硬件级管理的接口

![Figure01](https://raw.githubusercontent.com/WarmGrid/warmgrid.github.io/master/_posts/the-architecture-of-vmware-esxi/01.png)

图1显示了整个ESXi体系结构的图表。以下部分对每个组件进行了详细的研究。

#### VMkernel

VMkernel是由VMware开发的类似POSIX的操作系统，提供与其他操作系统类似的某些功能，例如进程创建和控制，信号，文件系统和进程线程。它专门用于支持运行多个虚拟机，并提供以下核心功能：

- 资源调度
- I/O堆栈
- 设备驱动程序

VMkernel的一些更相关的方面将在以下部分中介绍。

#### 文件系统

VMkernel使用简单的内存文件系统来保存ESXi配置文件，日志文件和暂存的补丁。为了便于熟悉，文件系统的结构设计与ESX服务控制台中的结构相同。例如，ESXi配置文件位于 `/etc/vmware` 中，日志文件位于 `/var/log/vmware` 中。暂存补丁上传到 `/tmp`。

此文件系统独立于用于存储虚拟机的VMware VMFS文件系统。与ESX一样，可以在主机系统中的本地磁盘或共享存储上创建VMware VMFS数据存储。如果主机使用的唯一VMFS数据存储位于外部共享存储上，则ESXi系统实际上不需要本地硬盘驱动器。通过运行无盘设置，您可以通过避免硬盘故障并降低功耗和冷却消耗来提高可靠性。

远程命令行界面为内存文件系统和VMware VMFS数据存储区提供文件管理功能。通过HTTPS `get` 和 `put` 实现对文件系统的访问，本地配置的用户和组可对访问进行身份验证，并用本地权限加以控制。

由于内存文件系统在电源关闭时不会保留，因此日志文件无法在重新启动后继续存在。ESXi能够配置远程syslog服务器，使您能够将所有日志信息保存在外部系统上。

> [在 ESXi 上配置 syslog (2003322) | VMware KB](https://kb.vmware.com/articleview?docid=2003322&lang=zh_CN)



#### 用户和组

可以在ESXi系统上定义本地用户和组。它们提供了一种区分通过Virtual Infrastructure Client，远程命令行界面或VIM API访问系统的用户的方法。

与其他操作系统一样，组可用于组合多个用户。例如，可以使用组一次为许多用户设置权限。有一些系统用户和组是预定义的，用于识别在VMkernel中运行的某些进程。

可为每个用户或组单独设置管理权限。用户和组定义存储在文件系统中的文件 `/etc/passwd`, `/etc/shadow` 和 `/etc/group` 中，与其他操作系统一样，密码是使用标准 `crypt` 函数生成的。



#### 用户环境

术语“用户环境 (User World)”指的是在VMkernel操作系统中运行的进程。与在诸如Linux的通用PO​​SIX兼容操作系统中发现的环境相比，用户环境运行的环境是受限的。例如：

- 只能使用有限的信号集
- 可用API是POSIX的子集
- `/proc` 文件系统非常有限
- 单个交换文件可用于所有用户环境进程。如果存在本地磁盘，则会在小型VFAT分区中自动创建交换文件。否则，用户可以在其中一个连接的VMFS数据存储上自由设置交换文件。

简而言之，用户环境不是用作运行任意应用程序的通用机制，而是仅为需要在管理程序环境中运行的进程提供够用的框架。

在用户环境中运行了几个重要的进程。这些可以被认为是本机VMkernel应用程序，并在以下部分中进行了描述。


#### Direct Console User Interface

Direct Console User Interface（DCUI）是仅在ESXi系统的控制台上显示的本地用户界面。它提供类似BIOS的菜单驱动界面，用于与系统交互。其主要目的是初始配置和故障排除。VMkernel中定义的系统用户之一是 `dcui`，当与系统中的其他组件通信时，DCUI进程使用该用户来识别自身。

DCUI配置任务包括：

- 设置管理密码
- 配置网络，如果没有以DHCP方式自动完成的话

故障排除任务包括

- 执行简单的网络测试
- 查看日志
- 重启代理
- 恢复默认值

意图是让用户拿DCUI进行最低配置，然后用远程管理工具（如VI Client，VirtualCenter或远程命令行界面）执行所有其他配置和持续管理任务。

使用DCUI的任何人都必须输入管理级密码，例如root密码。最初root密码为空。VMware强烈建议您在将服务器连接到任何不受信任的网络之前设置此密码。例如，在没有连接任何网络电缆的情况下打开服务器，设置密码，将服务器连接到网络，然后选择通过DHCP获取IP信息的选项。或者，如果服务器位于受信任的网络上，则可以使用VI Client设置管理员密码。您可以通过使其成为“localadmin”组的一部分，为其他本地用户提供访问DCUI的能力。此方法提供了一种在不分发root密码的情况下授予对DCUI的访问权限的方法，但显然您只能将此权限授予受信任的帐户。




#### 其他用户环境进程

VMware用于实现某些管理功能的代理已从在服务控制台中运行移植到在用户环境中运行。

- `hostd` 进程为VMkernel提供编程接口，由VI Client直接连接和VI API使用。它是对用户进行身份验证并跟踪哪些用户和组具有哪些权限的进程。它还允许您创建和管理本地用户。
- `vpxa` 进程是用于连接VirtualCenter的代理。它作为一个名为vpxuser的特殊系统用户运行。它充当hostd代理和VirtualCenter之间的中介。
- 用于提供VMware HA功能的代理也已从在服务控制台中运行移植到在其自身的用户环境中运行。
- `syslog` 守护进程也作为用户环境运行。如果启用远程日志记录，则该守护程序会将所有日志转发到远程目标，并将其放入本地文件中。
- 处理iSCSI目标初始发现的进程，此后所有iSCSI流量都由VMkernel处理，就像处理任何其他设备驱动程序一样。请注意，iSCSI网络接口与主VMkernel网络接口相同。

> 所以连接 iSCSI target 需要配一块 vmkernel 网卡

此外，ESXi还有NTP时间同步和SNMP监视进程。

> 另一篇 "VMware Infrastructure 架构概述" 的这个图很清晰
>
> ![Figure15](https://raw.githubusercontent.com/WarmGrid/warmgrid.github.io/master/_posts/vmware-infrastructure-architecture-overview/15.png)



#### 开启的网络端口

ESXi打开了有限几个的网络端口。最重要的端口和服务如下：

- 80 - 此端口提供反向代理，该代理仅在显示浏览到服务器时看到的静态网页时打开。否则，此端口会将所有流量重定向到端口443，以便为ESXi主机提供SSL加密通信。
- 443（反向代理） - 此端口还充当许多服务的反向代理，以便为这些服务提供SSL加密通信。这些服务包括VMware Virtual Infrastructure API（VI API），可提供对RCLI，VI Client，VirtualCenter Server和SDK的访问。
- 427（服务位置协议） - 此端口提供对服务位置协议的访问，服务位置协议是搜索VI API的通用协议。
- 5989 - 此端口对CIM服务器是开放的，CIM服务器是第三方管理工具的接口。
- 902 - 此端口已打开以支持较旧的VIM API，特别是旧版本的VI Client和VirtualCenter。


有关打开端口的完整列表，请参阅“ESX Server 3i配置指南”。




### 系统映像设计

ESXi旨在以各种格式进行分发，包括直接嵌入到服务器的固件中或作为要安装在服务器引导磁盘上的软件。图2显示了ESXi系统映像的内容图。无论映像存在于闪存上还是计算机的硬盘驱动器上，都存在相同的组件：

- 4MB引导加载程序分区，在系统启动时运行。
- 48MB引导库，包含32MB核心管理程序代码，以及相同大小的第二个备用引导库。下面解释两个引导组的原因。
- 540MB存储分区，包含各种实用程序，例如VI Client和VMware Tools映像。
- 110MB核心转储分区，通常为空，但在系统出现问题时可以保存诊断信息。

![Figure02](https://raw.githubusercontent.com/WarmGrid/warmgrid.github.io/master/_posts/the-architecture-of-vmware-esxi/02.png)

ESXi系统具有两个独立的内存库 (banks of memory)，每个内存库都存储一个完整的系统映像，作为应用更新的故障保护。升级系统时，新版本将加载到不活动的内存库中，系统将设置为在重新启动时使用更新的库。如果在引导过程中检测到任何问题，系统将自动从先前使用的内存块引导。您还可以在引导时手动干预以选择要用于该引导的映像，因此您可以在必要时退出更新。

在任何给定时间，存储分区中通常有两个版本的VI Client和两个版本的VMware Tools，对应于两个引导库中的管理程序版本。要使用的特定版本取决于当前活动的引导库。

核心管理程序代码还可以包含由服务器供应商（OEM）提供的自定义代码，这些代码提供其他功能，例如硬件监视和支持信息。例如，如果已从服务器制造商以嵌入形式获取ESXi，或者在硬盘驱动器上安装了自定义版本的ESXi，则会出现这些自定义项。对现有ESXi安装的任何更新都会自动包含对此自定义代码的正确更新。





### 启动和操作

当系统首次引导时，VMkernel会发现设备并为其选择适当的驱动程序。它还会发现本地磁盘驱动器，如果磁盘为空，则对它们进行格式化，以便它们可用于存储虚拟机。

在此初始引导期间，VMkernel使用合理的默认值自动创建配置文件（例如，使用DHCP获取网络标识信息）。用户可以使用Direct Console User Interface或标准VMware管理工具调整默认值：VMware VirtualCenter和VI Client。在嵌入式ESXi版本中，配置存储在内存模块的可读写的特定部分中。在后续重新引导时，系统从此持久性内存中读取配置。在其余的引导过程中，系统已初始化，驻留文件系统内置于内存中。加载硬件驱动程序，启动各种代理程序，最后启动DCUI进程。

系统启动并运行后，所有进一步的例行操作都与ESX 3中的操作大致相同。由于ESXi不再包含服务控制台，因此不再需要在ESX平台上执行的许多管理活动。他们只需要配置和管理服务控制台本身。之前在服务控制台中执行的其他管理任务现在可以通过以下方式之一执行：

- 使用VI Client，它提供基于Windows的图形用户界面，用于平台的交互式配置。VI Client已得到增强，可提供以前仅在服务控制台中可用的功能。
- 使用远程命令行界面，通过加密和经过身份验证的通信通道，从Linux或基于Windows的服务器启用平台的脚本和基于命令行的配置的新接口。
- 使用良好定义的API的外部代理，例如VI API和CIM管理标准。

此外，您可以使用VirtualCenter管理ESXi，就像使用任何ESX 3系统一样。您可以拥有ESX 3和ESXi系统的混合环境。VirtualCenter以完全相同的方式在VI Client用户界面中显示两种类型的系统; ESXi管理特有的某些功能适用于配备该版本的主机。





### ESXi的管理模型

ESXi的体系结构带来了一种新的管理模型。该模型的核心原则是：基于无状态，可互换设备的计算基础设施; 集中式的策略管理; 使用定义明确且标准化的API与系统进行通信，而不是难以锁定和审计的非结构化交互式会话。以下部分更详细地描述了此管理模型的某些方面。


#### 状态信息

一些配置文件完整描述了ESXi系统的状态。这些文件控制诸如虚拟网络和存储的配置，SSL密钥，服务器网络设置和本地用户信息之类的功能尽管这些配置文件都在内存文件系统中找到，但它们也会定期复制到持久存储。例如，在ESXi Embedded中，有一小部分服务器固件被指定为可读写。如果突然断电，您可以重新启动服务器并将其恢复到最后一个副本的确切配置。维护状态不需要任何其他内容，因此甚至可以从服务器中删除内部硬盘。

您还可以下载包含所有状态信息的备份文件。这允许您将ESXi系统的状态复制到另一个类似的系统上。您可以创建服务器配置的备份，如果服务器发生灾难性故障，您可以轻松地用相同的单元替换它，然后通过恢复备份文件将该新设备置于相同的状态。

#### 公共信息模型

公共信息模型 (Common Information Model, CIM) 是一种开放标准，定义了如何表示和管理计算资源。它为ESXi的硬件资源实现了无代理，基于标准的监控框架。该框架由CIM对象管理器（通常称为CIM代理）和一组CIM提供程序组成。

CIM提供程序用作提供对设备驱动程序和底层硬件的管理访问的机制。硬件供应商（包括服务器制造商和特定硬件设备供应商）可以编写提供程序以提供对其特定设备的监视和管理。VMware还编写了实施服务器硬件，ESX / ESXi存储基础架构和虚拟化特定资源监控的提供商。这些提供程序在ESXi系统内运行，因此设计为非常轻量级，专注于特定的管理任务。ESXi中的CIM对象管理器实现了标准CMPI接口，开发人员可以使用该接口插入新的提供程序。但是，提供程序必须与系统映像打包在一起，并且无法在运行时安装。

CIM代理从所有CIM提供商处获取信息，并通过标准API（包括WS-MAN）将其呈现给外部世界。图3显示了CIM管理模型的图表。

![Figure03](https://raw.githubusercontent.com/WarmGrid/warmgrid.github.io/master/_posts/the-architecture-of-vmware-esxi/03.png)


#### VI API

VMware Virtual Infrastructure API为开发应用程序以与VMware Infrastructure集成提供了强大的界面。VI API使您的程序或框架能够调用VirtualCenter上的VirtualCenter Web Service接口功能来管理和控制ESX / ESXi。VI SDK为开发人员提供了一个完整的环境，用于创建以各种编程语言与ESXi交互的应用程序。

VI API实际上是VMware提供的管理客户端使用的，例如VI Client和远程命令行界面。此外，此API适用于VirtualCenter以及ESX / ESXi。唯一的区别是影响多个主机的某些功能（如VMotion）仅在VirtualCenter中实现。图4描述了VI API如何与VMware Infrastructure一起使用。

![Figure04](https://raw.githubusercontent.com/WarmGrid/warmgrid.github.io/master/_posts/the-architecture-of-vmware-esxi/04.png)

VI API和CIM标准一起提供了从远程或中心位置管理ESXi系统的综合方法。这种模式的优点在于，不是依赖于本地安装的代理，而是必须在底层平台更新时进行调整，并在更新时重新安装和管理，所有与系统监视和管理相关的软件都可以存在于外部和集中式系统。与管理多个分布式代理相比，维护此软件变得更加容易。这种管理方法还使ESXi主机成为无状态实体，因为在主机上无需本地安装。消除代理在本地运行也意味着所有计算资源都可用于运行虚拟机。




### 总结

与其他虚拟化平台相比，ESXi体系结构具有多种优势，包括：

- 少量状态信息 - 作为无状态计算节点，ESXi系统可以作为实际用途进行处理，所有状态信息都可以从保存的配置文件轻松上载。
- 更高的安全性 - ESXi系统占用空间小，界面最小，整体攻击面较低。
- 类似硬件的可靠性 - 当它集成到固件中时，软件比存储在磁盘上的可能性要小得多。消除本地磁盘驱动器的选项可以提供更高的系统可靠性。
- 表1总结了ESX 3和ESXi之间的体系结构差异


--------- | VMware ESXi | VMware ESX 3
磁盘占用空间 | 32MB | 2GB
Bootstrap | 直接从引导加载程序 | 服务控制台驱动
直接管理互动 | DCUI | 服务控制台shell会话
硬件监控代理 | CIM插件模块 | 服务控制台中的完整应用
其他代理 | 仅通过VI SDK实现 | 服务控制台中的完整应用
脚本，自动化和故障排除 | DCUI，远程命令行界面和VI SDK | 服务控制台shell和VI SDK
其他软件 | 移动到外部环境 | 驻留在服务控制台中

表1：ESXi和ESX 3之间的差异




### 关于作者

Charu Chaubal是VMware的技术营销经理，专注于企业数据中心管理，专注于安全性。在此之前，他曾在Sun Microsystems工作，在那里他有超过7年的设计和开发分布式资源管理和网格基础设施软件解决方案的经验。Charu获得宾夕法尼亚大学工程学士学位和博士学位。来自加州大学圣巴巴拉分校，他在那里研究了复杂流体的数值模拟。他是众多出版物的作者，并在数据中心自动化和数字价格优化领域拥有多项专利。

####致谢

作者要感谢Olivier Cremel和John Gilmartin在撰写本文档时提供的宝贵帮助。


