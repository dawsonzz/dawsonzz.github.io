---
title: opencv的示范案例在linux环境下编译
date: 2020-02-01 23:09:58
tags: [linux, opencv, c++]
description: <center>在编译opencv案例遇到的问题以及解决</center>
---

`g++ hello.cpp -std=c++11 `提示找不到对应引用

```ba
/usr/bin/ld: /tmp/cc0YavT8.o: in function `main':
hello.cpp:(.text+0xb4): undefined reference to `cv::imread(cv::String const&, int)'
/usr/bin/ld: hello.cpp:(.text+0x10f): undefined reference to `cv::namedWindow(cv::String const&, int)'
/usr/bin/ld: hello.cpp:(.text+0x166): undefined reference to `cv::imshow(cv::String const&, cv::_InputArray const&)'
/usr/bin/ld: hello.cpp:(.text+0x18e): undefined reference to `cv::waitKey(int)'
//..............................省略
collect2: error: ld returned 1 exit status

```

通过命令`cpp -v`可以看到cpp文件编译时搜索路径是`--libdir=/usr/lib`,而opencv默认安装路径是`/usr/local/lib`

```bas
Using built-in specs.
COLLECT_GCC=cpp
Target: x86_64-pc-linux-gnu
Configured with: /build/gcc/src/gcc/configure --prefix=/usr --libdir=/usr/lib
//..............................省略
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-pc-linux-gnu/9.2.0/include
 /usr/local/include
 /usr/lib/gcc/x86_64-pc-linux-gnu/9.2.0/include-fixed
 /usr/include
End of search list.
```

此时加入`-l`表示链接的库，`-L`表示链接的库搜索路径，编译成功

```bash
g++ hello.cpp -std=c++11  -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc -lopencv_core -L/usr/local/lib/
```

但是运行时提示，还是找不到动态库

```bash
./a.out: error while loading shared libraries: libopencv_highgui.so.3.4: cannot open shared object file: No such file or directory
```

使用命令`ldd a.out`可以查看a.out的链接路径，发现里面并没有我们之前上面写的加入的`/usr/local/lib`

```bash
linux-vdso.so.1 (0x00007ffd769f9000)
libopencv_highgui.so.3.4 => not found
libopencv_imgcodecs.so.3.4 => not found
libopencv_imgproc.so.3.4 => not found
libopencv_core.so.3.4 => not found
libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007fd72fb29000)
libm.so.6 => /usr/lib/libm.so.6 (0x00007fd72f9e3000)
libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007fd72f9c7000)
libc.so.6 => /usr/lib/libc.so.6 (0x00007fd72f800000)
/lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fd72fd50000)
```

因为之前我们输入的只是告诉了编译器，并没有告诉链接器，我们要加入`-Wl,-rpath`

```bash
g++ hello.cpp -std=c++11  -lopencv_highgui -lopencv_imgcodecs -lopencv_imgproc -lopencv_core -L/usr/local/lib -Wl,-rpath=/usr/local/lib/
```

