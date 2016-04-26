---
layout: post
title: "Smali和逆向分析"
date: 2014-09-08 20:37
comments: true
categories: Android
tags: [Smali, 游戏破解, 逆向分析]
---
这篇文章其实2个月前就想写了，但并不好写，就懒得写了，所以拖到现在～其实接触smali这门语法是件蛮偶然的事，接触后发现，次奥，这货在某些领域太有用了，至于为什么我想看完这篇文章应该都明白了。

我自己也只是接触了皮毛，大概用了2个礼拜多一点，也不是很系统的学习，写这篇的目的主要还是想把知道的东西记下来，以后好追溯。

### Smali简介

Smali是Dalvik的寄存器语言，它与Java的关系，简单理解就是汇编之于C。假如你对汇编有足够的驾驭能力，那你可以通过修改汇编代码来改变C/C++代码的走向。当然，学过汇编的都清楚，汇编比[BrainFuck](http://www.muppetlabs.com/~breadbox/bf/)还难学，更不用说去反编译修改了。

但是Smali有一点不一样，就是它很简单，只有一点点的语法，只要你会java，了解Android的相关知识，那你完全可以通过修改Smali代码来反向修改java代码，虽然绕了一点，但是在某些情况下你不得不这么做。还好，Smali很简单。

<!-- more -->

### apktool

说了这么多，还没有说Smali哪来？没错。Smali代码是安卓APK反编译而来的，所以Smali文件和Java文件一一对应。获取Smali文件，我们需要下载一个辅助工具：[ApkTool](https://code.google.com/p/android-apktool/downloads/list)。apktool这个命令行工具如果详细使用功能参数是比较多的，但是这里我们只需要用到2个最基础的功能：

一个是反编译decode：

	apktool d xxx.apk

另一个是打包build：

	apktool b
	
这里要注意的是路径问题，apktool如果没有加入到环境变量中，记得cd到apktool的目录去使用它。另一个是打包，如果只是简单的使用参数b，那要求是要在反编译出来的项目目录下执行，而打包好的文件会保存在这个项目目录下的dist目录。

这是一个HelloWorld的应用程序反编译和打包的目录结构：

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/smali_Screen%20Shot%202014-09-08%20at%209.42.35%20PM.png" alt="" border="0" title="" /><br></br></div>

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/smali_Screen%20Shot%202014-09-08%20at%209.43.37%20PM.png" alt="" border="0" title="" /><br></br></div>

### Smali语法

这里仍然以一个默认的HelloWorld的应用程序进行解释吧。新建一个HelloWorld安卓项目，在MainActivity中只保留onCreate函数。代码如下：

``` java MainActivity.java
	package com.fusijie.helloworld;

	import android.app.Activity;
	import android.os.Bundle;

	public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		}
	}
```

反编译后的Smali文件如下：

``` smali MainActivity.smali
	.class public Lcom/fusijie/helloworld/MainActivity;
	.super Landroid/app/Activity;
	.source "MainActivity.java"

	# direct methods
	.method public constructor <init>()V
    .locals 0

    .prologue
    .line 14
    invoke-direct {p0}, Landroid/app/Activity;-><init>()V

    return-void
	.end method

	# virtual methods
	.method protected onCreate(Landroid/os/Bundle;)V
    .locals 1
    .parameter "savedInstanceState"

    .prologue
    .line 18
    invoke-super {p0, p1}, Landroid/app/Activity;->onCreate(Landroid/os/Bundle;)V

    .line 19
    const/high16 v0, 0x7f03

    invoke-virtual {p0, v0}, Lcom/fusijie/helloworld/MainActivity;->setContentView(I)V

    .line 20
    return-void
	.end method
```

对比一下，可以比较清楚的看出来，smali代码其实就是对java代码一个翻译，只是没有java看起来那么简单，smali把很多应该复杂的东西还原成复杂的状态了。简单解释下这段代码。

* 前三行指明了类名，父类名，和源文件名。
* 类名以“L”开头相信熟悉Jni的童鞋都比较清楚。
* “#”是smali中的注释。
* “.method”和“.end method”类似于Java中的大括号，包含了方法的实现代码段。
* 方法的括号后面指明了返回类型，这同样类似与Jni的调用。
* “.locals”指明了这个方法用到的寄存器数量，当然寄存器可以重复利用，从“V0”起算。
* “.prologue”指定了代码开始处。
* “.line”表明这是在java源码中的第几行，其实这个值无所谓是多少，可以任意修改，主要用于调试。
* “invoke-direct”这是对方法的调用，可以看到这里调用了是Android.app.Activity的init方法，这在java里是隐式调用的。
* “return-void”表明了返回类型，这和java不一样，即使没有返回值，也需要这样写。
* 接下来是onCreate方法，“.parameter”指明了参数名，但是一般没有用，需要注意的是p0代表的是this，p1开始代表函数参数，静态函数没有this，所以从p0开始就代表参数。
* 在实现里先是调用了父类的方法，然后再调用setContentView，注意这里给了一个传参。整形的传参，这个值是先赋给寄存器v0，然后再调用的使用传递进去的。smali中都是这么使用，所有的值必须通过寄存器来中转。这点和汇编很像。

对比了Java代码和Smali代码，可以很清楚的看到，原本只有几行的代码到了Smali，内容被大大扩充了。Smali还原了Java隐藏的东西，同时显式地指定了很多细节。这还只是个最基本的HelloWorld的onCreate函数，如果有内部类，还会分文件显示。

这样看来，其实Smali只能说复杂，不能说难。我这里不打算把Smali的语法再贴一次，这里给出几个链接，算是总结的相对好一点的（其实我都没看到有系统总结的。。。如果你有好的资料，欢迎跟帖分享）

* [Smali语法--数据类型、方法、字段和寄存器](http://liuzhichao.com/p/tag/smali)
* [Android软件安全与逆向分析--Smali文件格式](http://book.2cto.com/201212/12468.html)
* [Smali--Dalvik虚拟机指令语言](http://blog.csdn.net/wdaming1986/article/details/8299996)
* [看雪论坛--关于Smali语法](http://bbs.pediy.com/showthread.php?t=151769)

这里顺便提供2个利器：

* [Smali Sublime Text语法高亮插件](http://liuzhichao.com/p/1476.html)
* [dex2jar，配合Smali事半功倍](https://code.google.com/p/dex2jar/)

### 小试牛刀

了解了Smali的基本语法，那我们要动手试一下，Smali能做什么？仍然以HelloWorld为例，假如我们没有Android项目的源代码，只有一个APK，给他加个新功能吧！

这个功能很简单，只是在HelloWorld中输出一个“Hello, Smali”。

(1)第一步还是先使用apktool来反编译HelloWorld.apk。

(2)打开smali下的com/fusijie/helloworld/MainActivity.smali文件。

(3)原本我们在Java中要写的代码是：

``` java Toast
	Toast.makeText(this, "Hello, Smali", Toast.LENGTH_LONG).show();
```

翻译成Smali就是：

``` smali Toast
	.line xx
    const-string v0, "Hello, Smali"

    const/4 v1, 0x1

    invoke-static {p0, v0, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;

    move-result-object v0

    invoke-virtual {v0}, Landroid/widget/Toast;->show()V
```

(4)最后在插入Smali的时候，我们需要修改2个地方：

* “.locals 1”，因为本来只用到了v0，现在多用了一个v1，所以改为“.locals 2”。
* “.line xx” xx随意改为一个不重复的值即可。

(5)使用apktool打包成apk，因为打包完后原有的密钥会丢失，所以需要重新打上我们自己的密钥，可以参考[很早以前我写的一个帖子](http://www.eoeandroid.com/forum.php?mod=viewthread&tid=300764)。

(6)最后的效果是这样的。

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/smali_Screen%20Shot%202014-09-08%20at%2011.00.38%20PM.png" alt="" border="0" title="" /><br></br></div>

### 干点坏事

从上面一个例子对Smali的用途就很清楚了，没错，Smali注入。现在常见的除了测试以外的用途，Smali注入明显是带有黑客性质的，小的如破解游戏，替换游戏广告，大的甚至利用漏洞去破解密码，偷窃个人资料，财产等等。对Smali，安卓逆向分析，安卓系统安全比较清楚的，这些事其实都不算事。

我这里以一个实际上线的游戏破解为例，看看我们平时在写代码时要注意哪些问题，避免辛辛苦苦写游戏，却在帮人家数钱。这里的破解不是重点，反破解才是重点。

以市场上很火的一款单机游戏《消灭小星星》为例，下载地址是：[http://apk.91.com/Soft/Android/com.brianbaek.popstar-340.html](http://apk.91.com/Soft/Android/com.brianbaek.popstar-340.html)

相同的方法反编译，在/smali/com/zplay/android/sdk/pay/ZplayPay.smali文件的dopay函数开头，注入如下代码：

``` smali popstar_crack
	const/4 v0, 0x1	
    invoke-static {v0}, Lcom/zplay/iap/ZplayJNI;->sendMessage(I)V
    return-void
```
    
原理很简单，这个游戏使用了Zplay的支付系统，在Java层的处理了支付逻辑，如果觉得Smali读起来费劲，那么直接使用dex2jar就能很清楚的看到，支付提醒甚至是中文的。Java层处理完支付逻辑后会给C++层丢个消息，调用C++层的代码去处理游戏逻辑，比如成功支付，那么幸运星就会相应地增加。这里使用native方法进行处理。

所以注入的代码是，一旦进入支付逻辑，直接返回成功，同时强制返回函数，这就实现了支付的破解。当然作为一个有节操，有逼格的游戏从业者，这里就不放出破解版了（不过说得也够明白了）。查找注入点这东西靠的是耐心，细心和运气。

为了方便，一般会先用正常的java写一些调试类，然后反编译出静态的smali放入目标文件夹中以供调试使用。

再放张图，星星用不完了：

<div align="center"><img src="http://www-fusijie-com.qiniudn.com/smali_crack_popstar.jpg" alt="" border="0" title="" /><br></br></div>

### 总结

就像上面说的，Smali能做的不仅仅是这些。有兴趣的可以看看这篇文章[《支付宝钱包手势密码破解实战》](http://blog.csdn.net/hu3167343/article/details/36418063)，我没有去验证过真伪，但是如作者所描述的应该是可行无误。这里用到的一个更高级的功能是将支付宝的加密解密逻辑Smali的jar包导入自己新建的工程，进而直接在自己的程序中集成支付宝的加密解密功能。

在逆向分析游戏的过程中，我也发现了几个重要的点能帮助开发者提高自己程序的安全性。

首先，完全避免破解是不可能的，我们能做的工作就是尽最大可能去妨碍破解者破解游戏，提高破解成本。

* 一定要使用混淆。不单单是第三方SDK，你的代码也是。破解游戏很重要一点就是要抓住游戏的逻辑。代码混淆后，Smali更加晦涩难懂，逻辑也更难掌握。
* 回到开头的话，解读汇编比解读Smali难度大的多得多。所以重要的逻辑可以放到C/C++层去处理就不要放在Java层上去处理。
* 多用连续调用的方式。这样出来的效果是Java只有一行，Smali可能有好几十行，看着都蛋疼。当然这对熟练的破解老手无效～
* 在一些关键的点上，比如支付，多绕一下。而不是像《消灭小星星》这样，直接在Java内用中文显示“支付成功”，同时去调用JNI方法。用dex2jar看一眼就暴露了。

哈哈，总之，小心你的艳照。。。

哇咔咔，竟然写了快4个小时～
