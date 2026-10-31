
# [原创]android调试相关插件

git目录在`472K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3y4Z5j5h3)9^5z5e0l9@1x3U0N6Q4x3V1k6m8L8X3c8J5L8$3W2V1c8r3g2T1N6h3N6Q4x3X3g2Y4K9i4b7`.

包括的项目如下：

## AndroidNdkDebugPlugin

本插件为AndroidStudio插件

背景：ADT已经开始支持jni模块源码调试，然而有一些未实现的部分。AndroidStudio自身支持启动方式调试jni，底层原理还是jdb+lldb方式；然而并未实现附加调试so源码。这样对于动态下发的so的源码级调试无能为力。本插件正是通过逆向并重新实现`android-ndk.jar`来实现调试动态下发so的源码的

原始`android-ndk.jar`启动式调试jni得而实现步骤如下：

`adb push 1.apk /data/local/tmp/1.apk`

`am start -D -n com.example.test/.MainActivity`

等待app启动开启java调试线程

连接jdwp调试模块，下初始java断点

`adb push lldb_server`

`./lldb_server`..............建立lldbserver用于调试jni层

`lldbfrontend.exe`...........建立client连接lldbserver

发送命令使lldb附加进程

设置jni初始断点

恢复java层和jni层执行，等待中断

本插件正是对原始逻辑进行逆向，跳过之前的步骤从而直接连接到进程

使用方式：

1. 编译`android-ndk.jar`，替换android studio自身的插件，重启as
2. 打开任意包含jni源码的app源码，在Run/Debug Configurations选项可以看到远程android进程列表，选择需要attach的进程
3. 运行进程即可，此时插件逻辑从启动进程改为附加进程，最终附加到app实现c++源码调试

## AndBugForWin

AndBug是java层调试工具，封装了jdb利用python实现用户交互。因为jdb本身接口实在难用所以出现该工具。然而它不支持Windows甚至CygWin，因为AndBug中使用了`*nix`系统支持的localfilesystem监听方式而不是用socket端口监听方式，因此稍作修改即可实现win-cygwin上的同等功能

## superddms

本插件用于修复ddms不识别本地android虚拟机的bug，android自带虚拟机的端口`5037`，而不同厂家的虚拟机采用不同端口

## superjdb

关于jdwp的解析如我的这篇帖子：`396K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8r3W2U0K9r3q4G2z5o6V1H3y4o6t1%4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5#2x3e0V1J5y4o6b7#2x3b7`.`.

对于高版本的jdwp服务器端(如android5.0)，支持新的底层命令对字节码进行逐行调试；由于apk中一般去除了行号信息，因此jdb没有字节码的调试功能就像windbg没有反汇编单步步过一样扯淡，本插件实现了java字节码和dalvik字节码逐行调试，使用了java源码自带的jdb-gui

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
