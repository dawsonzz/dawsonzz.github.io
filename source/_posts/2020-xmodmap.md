---
title: linux下全局改按键
date: 2020-02-05 20:53:49
tags: [linux, vim]
description: <center>在使用vim的过程中，因为手指不够长，按esc键比较费劲，网上有很多人把capslk大写锁定键改为了esc键，还有的人把答谢锁定键改为了ctrl键，使用组合键ctrl+[进行退出，这里我采用第二种方案。</center>
---

## 下载工具

```bash
sudo pacman -S xorg 
```

## 配置

查看当前键位设置

```bash
xmodmap -pke
```

将键位设置保存到`～/.xmodmap`

```bash
xmodmap -pke > ~/.xmodmap  
```

接下来我们`ctrl`键和大写锁定键的内容替换，我要知道他们对应的keycode是多少，通过命令xev，按下按键可以看到现实keycode

```bash
xev
```

通过这个方式知道了`ctrl`和大写锁定键的keycode分别是37和66

```bash
//ctrl键
KeyRelease event, serial 37, synthetic NO, window 0x5800001,
root 0x115, subw 0x0, time 2057253, (160,-14), root:(2157,360),
state 0x14, keycode 37 (keysym 0xffe3, Control_L), same_screen YES,
XLookupString gives 0 bytes: 
XFilterEvent returns: False
//大写锁定键
KeyRelease event, serial 37, synthetic NO, window 0x5800001,
root 0x115, subw 0x0, time 2059683, (160,-14), root:(2157,360),
state 0x12, keycode 66 (keysym 0xffe5, Caps_Lock), same_screen YES,
XLookupString gives 0 bytes: 
XFilterEvent returns: False

```

接下来编辑`～/.xmodmap`的内容，把keycode 37（`ctrl`）的内容复制到keycod 66（大写锁定）后面

```bash
//原文件
keycode  37 = Control_L NoSymbol Control_L   
...
keycode  66 = Caps_Lock NoSymbol Caps_Lock
//修改后
keycode  37 = Control_L NoSymbol Control_L  
...
keycode  66 = Control_L NoSymbol Control_L  
```

在文件头部添加，清除大写锁定键和`ctrl`键

```bash
clear lock                                                                  
clear control  
```

在文件尾部添加，用于更新，因为这里我已经没有使用大写锁定键了，所以就没有添加`add lock = Caps_Lock`语句

```bash
add control = Control_L Control_R
```

退出之后使用xmodmap更新,修改成功

```bash
xmodmap ～/.xmodmap
```

