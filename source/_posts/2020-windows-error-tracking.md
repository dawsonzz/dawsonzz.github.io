---
title: windows应用程序报错原因追溯
date: 2020-03-21 21:30:02
tags: [windows]
description: <center></center>
---

## 前言

编写程序时，提示应用程序无法正常启动，然而什么提示都没有，应该怎么去查原因,**但还是没能解决问题**

## 步骤

右键“我的电脑”，然后点击“管理”→“事件查看器”→“Windows 日志”→“应用程序”，查看错误信息

```
“E:\xxxxx\boost_python-vc120-mt-gd-1_57.dll”的激活上下文生成失败。 找不到从属程序集 Microsoft.VC90.DebugCRT,processorArchitecture="amd64",publicKeyToken="1fc8b3b9a1e18e3b",type="win32",version="9.0.21022.8"。 请使用 sxstrace.exe 进行详细诊断。
```

大概意思就是我缺少了`boost_python-vc120-mt-gd-1_57.dll`相关的依赖

使用VS2013下的dumpbin程序查看依赖，在命令行中`D:\Microsoft Visual Studio 12.0\VC\bin> .\dumpbin.exe /dependents xxxxxx`

```
Image has the following dependencies:

    python26.dll
    MSVCP90D.dll
    MSVCR90D.dll
    KERNEL32.dll
```

对比Windows日志的内容，我们缺的就是MSVCR90D.dll这个文件了，这是要安装VS2008才有的带有调试信息的动态库。

## 图形端的查看工具

https://github.com/lucasg/Dependencies

## 安装VS2008报错

错误提示安装`运行时系统必备`失败，进入到VS2008安装包下的WCU目录，手动把里面的安装包点一遍，再重新运行安装程序

## dll文件复制

将```Microsoft Visual Studio 9.0\VC\redist\Debug_NonRedist\x86\Microsoft.VC90.DebugCRT``下的文件复制到exe所在的文件夹。再次运行提示:

```
在指令清单中找到的组件标识与请求组件的标识不匹配。 参考是 Microsoft.VC90.DebugCRT,processorArchitecture="amd64",publicKeyToken="1fc8b3b9a1e18e3b",type="win32",version="9.0.21022.8"。 定义是 Microsoft.VC90.DebugCRT,processorArchitecture="x86",publicKeyToken="1fc8b3b9a1e18e3b",type="win32",version="9.0.21022.8"
```

把`Microsoft.VC90.DebugCRT.manifest`文件对应项修改`processorArchitecture="amd64"`

## 日志记录错误

```
svchost (8544,R,98) TILEREPOSITORYS-1-5-18: 打开日志文件 C:\WINDOWS\system32\config\systemprofile\AppData\Local\TileDataLayer\Database\EDB.log 时出现错误 -1023 (0xfffffc01)。
```

然后在C:\Windows\System32\config\systemprofile\AppData\Local\创建TileDataLayer文件夹，再到TileDataLayer文件夹中创建Database文件夹。我的文件夹中并没有自动出现文件，于是重启后进行以下步骤，就看到出现了文件。

1. 点击“开始”->在搜索栏内输入“cmd”，右键点击cmd.exe，选择以管理员身份运行，跳出提示框时选择继续。

2. 键入sfc /scannow ，然后按 Enter。系统开始扫描，请您耐心等待。

结果还是无法运行

```
taskhostw (6948,D,0) WebCacheLocal: 向文件 "C:\Users\dawson\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat" 中偏移量 14745600 (0x0000000000e10000) 写入 32768 (0x00008000) 字节的请求成功，但是花费了 OS 异常的长时间(21 秒)。此问题可能是硬件故障造成的。请与您的硬件供应商联系获得进一步协助诊断此问题。
```

## 最终解决方案

在其他地方下载了`boost_python-vc120-mt-gd-1_57.dll`解决了问题