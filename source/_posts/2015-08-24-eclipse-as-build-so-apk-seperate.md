---
title: Eclipse/AndroidStudio + NDK 单独编译 so 和 apk
date: 2015-08-24 20:36:47
comments: true
categories: cocos2d-x
tags: [单独编译, so, apk]
---

### 概况

* 环境：Mac OSX/Cocos2d-x v3.7/NDK r10c/Ant v1.9.5/ADT v23.0.6/AndroidStudio v1.3.1
* 注意：此教程与Cocos2d-x版本关系极大，不保证其他版本的Cocos2d-x可以正常应用此教程，但是原理都是一样的。

从Cocos2d-x v3.4开始，考虑到维护成本，build_native.py脚本不再直接调用ndk-build，而是调用Cocos Console. 具体PR可见：[Github](https://github.com/cocos2d/cocos2d-x/commit/a1e8dec3840b3042054f04b3197da81412b29cae)。

<!-- more -->

### 在Eclipse中编译so及apk

#### 使用build_native.py脚本编译

这一种方式是会直接调用Cocos Console的，所以会直接打包成apk。

##### 导入Cocos2d-x项目和libCocos2dx

##### 开启C++编译

右键项目属性->Builders->勾选CDT Builder。

##### C/C++ Build->Build command

默认是

	python ${ProjDirPath}/build_native.py -b release

修改为

	python ${ProjDirPath}/build_native.py -b debug

原因是采用release模式要求输入keystore，如果已经设置了keystore，这一步可以跳过，否则在最终打包apk会提示错误，因为这个模式下（非终端）无法输入keytore，注意这里的模式不是-m(mode)，是-b(build)。（如何设置keystore？往下看）

##### 设置环境变量

因为Cocos console需要一些环境变量，而在Eclipse环境下无法读取到zsh或者bash的配置，所以需要在Eclipse的环境里设置这些变量：C/C++->Environment

	PATH -> ${PATH}:/Users/Jacky/Cocos2d-x/v3/tools/cocos2d-console/bin/
	ANT_ROOT -> /usr/local/Cellar/ant/1.9.5/libexec/bin
	ANDROID_SDK_ROOT -> /Users/Jacky/AndroidDev/sdk
	NDK_ROOT -> /Users/Jacky/AndroidDev/android-ndk-r10c（如果你设置了全局的NDK_ROOT，你会发现这个变量已经存在）

这些实际上也是第一次配置Cocos Console需要设置的值。

##### 如何设置keystore

在proj.android/ant.properties文件末尾，添加keystore信息

	key.alias.password=xxx
	key.store.password=yyy
	key.store=/Users/Jacky/AndroidDev/cocos.keystore
	key.alias=zzz

#### 使用ndk-build编译

##### 导入Cocos2d-x项目和libCocos2dx

##### 开启C++编译

右键项目属性->Builders->勾选CDT Builder。

##### C/C++ Build->Build command

默认是

	python ${ProjDirPath}/build_native.py -b release

修改为

	/Users/Jacky/AndroidDev/android-ndk-r10c/ndk-build -j4

-j4表示编译使用的CPU核数，我这里是4核，Cocos console会自动获取CPU核数并且全开，这里需要你手动指定。

##### 指定NDK Tool Chain版本

设置NDK Tool Chain版本，proj.android/jni/Application.mk文件末尾，指定NDK Tool Chain版本。因为一些关键字如override是C++11新增的特性，GCC从4.7版本开始支持此特性。

	NDK_TOOLCHAIN_VERSION = 4.9

##### 拷贝资源

手动拷贝Resources目录下资源到proj.android/assets/目录下。

#### 总结

这里介绍了2种在Eclipse中交叉编译的方式，第一种方式调用的是Cocos Console，它主要完成三件事：交叉编译，拷贝资源和打包apk。第二种方式调用的是ndk-build，它只做了交叉编译这件事，所以需要自己手动拷贝资源，打包apk。当然第一种方式交叉编译最终调用的也是ndk-build，因此以上两种方式也可以在终端运行。

### 在Android Studio中编译so及apk

#### 导入Cocos2d-x项目

#### 修改proj.android-studio/app/build.gradle

在文件开头导入

	import org.apache.tools.ant.taskdefs.condition.Os

在文件末尾加入

```
android{
   //ndk-build
   task ndkBuild(type: Exec,  dependsOn: 'copyRes') {
       workingDir file('jni')
       commandLine getNdkBuildCmd(), '-j4'
   }
   tasks.withType(JavaCompile) {
       compileTask -> compileTask.dependsOn ndkBuild
   }
   //ndk-clean
   task ndkClean(type: Exec) {
       workingDir file('jni')
       commandLine getNdkBuildCmd(), 'clean'
   }
   clean.dependsOn ndkClean
   //copy resource
   task createDir{
       doLast{
           delete 'assets'
           mkdir('assets')
       }
   }
   task copyRes(type: Copy, dependsOn: 'createDir'){
       from '../../Resources'
       into 'assets'
   }
}
def getNdkDir() {
    if (System.env.ANDROID_NDK_ROOT != null)
        return System.env.ANDROID_NDK_ROOT
    Properties properties = new Properties()
    properties.load(project.rootProject.file('local.properties').newDataInputStream())
    def ndkdir = properties.getProperty('ndk.dir', null)
    if (ndkdir == null)
        throw new GradleException("NDK location not found. Define location with ndk.dir in the local.properties file or with an ANDROID_NDK_ROOT environment variable.")
    return ndkdir
}
def getNdkBuildCmd() {
    def ndkbuild = getNdkDir() + "/ndk-build"
    if (Os.isFamily(Os.FAMILY_WINDOWS))
        ndkbuild += ".cmd"
    return ndkbuild
}
```

#### 添加NDK路径

在proj.android-studio/local.properties末尾加入ndk路径。

	ndk.dir=/Users/Jacky/AndroidDev/android-ndk-r10c

#### 指定NDK ToolChain版本

在proj.android-studio/app/jni/Application.mk末尾指定NDK ToolChain版本。

	NDK_TOOLCHAIN_VERSION = 4.9

### 使用Cocos Console在编译过程中插入自定义步骤

如果希望在编译过程中插入一些自定义的脚本，比如拷贝一些第三方库的依赖，那可以采用如下方式进行设置。

[http://www.cocos2d-x.org/wiki/Cocos2d-console#Add-custom-steps-during-compiling](http://www.cocos2d-x.org/wiki/Cocos2d-console#Add-custom-steps-during-compiling)

不想看英文版，也可以看中文的[http://www.cocoachina.com/bbs/read.php?tid-319299.html](http://www.cocoachina.com/bbs/read.php?tid-319299.html)

### 常见问题

Q.Eclipse出现An internal error occurred during: "C/C++ Indexer". java.lang.NullPointerException或者Serializing CDT project settings错误？

A.先关闭Eclipse，删除proj.android/.cproject文件中的类似如下内容后重启Eclipse。

```
        <cconfiguration id="0.1230402123.1377291156">
            <storageModule buildSystemId="org.eclipse.cdt.managedbuilder.core.configurationDataProvider" id="0.1230402123.1377291156" moduleId="org.eclipse.cdt.core.settings" name="Debug">
                <externalSettings/>
                <extensions>
                    <extension id="org.eclipse.cdt.core.VCErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                    <extension id="org.eclipse.cdt.core.GmakeErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                    <extension id="org.eclipse.cdt.core.CWDLocator" point="org.eclipse.cdt.core.ErrorParser"/>
                    <extension id="org.eclipse.cdt.core.GCCErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                    <extension id="org.eclipse.cdt.core.GASErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                    <extension id="org.eclipse.cdt.core.GLDErrorParser" point="org.eclipse.cdt.core.ErrorParser"/>
                </extensions>
            </storageModule>
            <storageModule moduleId="org.eclipse.cdt.core.externalSettings"/>
        </cconfiguration>
```

如果还不行，建个新项目，把proj.android/.cproject和proj.android/.project替换到项目中，版本控制回退这两个文件也行，再次进入需要重新配置。

参考[StackOverflow](http://http//stackoverflow.com/questions/25384264/after-import-an-cocos2d-project-eclipse-raise-an-two-weird-errors-android-lib)。

Q.即使没有更改C++源码，ndk-build每一次都重新进行编译？

A.这个问题应该是存在于旧版本（r8c就有这个bug），新的ndk貌似还没遇到这样的问题，本教程应该不会涉及这个问题。

参考[StackOverflow](http://http//stackoverflow.com/questions/13885400/every-ndk-build-is-a-full-rebuild)。

Q.要进行ndk-gbd调试发现没有生成gdbserver及gdb.setup怎么办？

A.在使用ndk-buld的时候加入 NDK_DEBUG=1，即

	ndk-build -j4 NDK_DEBUG=1