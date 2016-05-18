---
title: ejoy2d 环境搭建小记
tags: [ejoy2d, 游戏引擎, 环境]
date: 2016-05-18 22:50:53
categories: [ejoy2d]
---

之前在优化 Cocos2d-x 的粒子系统时，参考了云风的 ejoy2d，这两天心血来潮突然想完整看下源码，之前只是看了 `sprite`，`particle system`，`renderer` 这部分。

克隆了源码之后发现编译过不了，上次也遇到了。毕竟时隔快一年了，当时的情况也忘了，不知道是引擎的问题还是说开发环境变了，现在又要重新找一遍问题，也是蛋疼。所以这次就干脆记下来方便查阅。

程序员比起写代码，应该更害怕编译链接这种问题，因为这些问题很大程度不是由自身引起的或者说使用不当，所以把控感不强。

<!-- more -->

### 编译 glfw3

地址：[https://github.com/glfw/glfw](https://github.com/glfw/glfw)

编译手册：[http://www.glfw.org/docs/latest/compile.html](http://www.glfw.org/docs/latest/compile.html)

我这里是 mac，所以直接

```
cd <glfw-root-dir>
mkdir build
cd build
cmake ..
make
sudo make install
make clean
```
这时候可以看到 glfw 被安装到 `/usr/local` 目录中：

```
Install the project...
-- Install configuration: ""
-- Up-to-date: /usr/local/include/GLFW
-- Up-to-date: /usr/local/include/GLFW/glfw3.h
-- Up-to-date: /usr/local/include/GLFW/glfw3native.h
-- Up-to-date: /usr/local/lib/cmake/glfw3/glfw3Config.cmake
-- Up-to-date: /usr/local/lib/cmake/glfw3/glfw3ConfigVersion.cmake
-- Up-to-date: /usr/local/lib/cmake/glfw3/glfw3Targets.cmake
-- Installing: /usr/local/lib/cmake/glfw3/glfw3Targets-noconfig.cmake
-- Up-to-date: /usr/local/lib/pkgconfig/glfw3.pc
-- Installing: /usr/local/lib/libglfw3.a
```

### 编译 freetype2

freetype2 地址：[https://www.freetype.org/index.html](https://www.freetype.org/index.html)

编译：

```
make
sudo make install
```

同样可以看到 freetype2 被安装到 `/usr/local` 目录中。

如果编译 freetype2 出现如下问题：

```
install: ./builds/unix/freetype-config: No such file or directory
make: *** [install] Error 71
```

可以参考：[https://lists.nongnu.org/archive/html/freetype/2014-01/msg00000.html](https://lists.nongnu.org/archive/html/freetype/2014-01/msg00000.html)

当然如果你安装过 X11，X11 里已经带了 freetype2，不过位置就不同了。

关于 X11 和 OSX：[https://support.apple.com/en-us/HT201341](https://support.apple.com/en-us/HT201341)


### 编译 ejoy2d

地址：[https://github.com/ejoy/ejoy2d.git](https://github.com/ejoy/ejoy2d.git)

我当前的节点是：[5c7ae1db5dd28c6fdbcb1db2a2b6183e54a6c1e1](https://github.com/ejoy/ejoy2d/tree/5c7ae1db5dd28c6fdbcb1db2a2b6183e54a6c1e1)

ejoy 已经自带 Makefile 了。稍微修改一下头文件和静态库的搜索目录，添加链接的 framework 即可：

``` git diff
-macosx : CFLAGS += -I/usr/include $(shell freetype-config --cflags) -D __MACOSX
-macosx : LDFLAGS += -lglfw3  -framework OpenGL -lfreetype -lm -ldl
+macosx : CFLAGS += -I/usr/local/include $(shell freetype-config --cflags) -D __MACOSX
+macosx : LDFLAGS += -L/usr/local/lib -lglfw3 -lfreetype -lm -ldl -framework OpenGL -framework Cocoa -framework IOKit -framework CoreVideo -framework Carbon

```

然后执行：

```
make
```

可以看到生成了 `ej2d` 文件。

测试例在 `examples/` 目录下，执行

```
./ej2d examples/flappybird.lua
```

即可浏览测试例。

