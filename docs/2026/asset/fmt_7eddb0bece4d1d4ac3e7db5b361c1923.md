
# [讨论]安卓CTF-爬楼梯-动态调试-反调试分析

## 【1】关于逆向破解简述

程序界面，点击按钮"爬一层楼"则已爬的楼层+1

按钮"爬到了，看FLAG"无法点击

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_37W8JCNCM7HTJPJ.png)

1. 分析清单文件，获取包名，以及入口类

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_85Z8FC9EUFTX429.png)

2. 分析入口类

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_HGNRAYCFAD3P2C8.png)

3. `AndroidKill` 修改并重新编译

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_KVM9MXDHYEQFEN3.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_733M8VBZMRMAP7Y.png)

此时，重新安卓编译后的 APK，点击一次已爬楼层按钮后，则能够点击出现 FLAG 了

虽然此时已经告破，但是想研究一下 so 层的反调试和 so 层的方法具体做了什么

## 【2】静态分析 so 文件

1. 发现不是静态加载方法，那么应该是 `JNI_onLoad` 动态注册；发现在 `JNI_onLoad` 前还有 `_init` 函数，所以分析顺序：`_init` -> 初始化数组 -> `JNI_onLoad` -> 注册方法

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_VEBFAPN8328TAS5.png)

2. 分析 `_init`

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_BP9HE7N3UP4HWRW.png)

2.1 但是奇怪了，动态调试了好几次，都在 `fork()` 后就出错了，提示：`got SIGCHLD signal (Child status has changed)`

### 2.2 只能将 `fork()` `nop` 掉，才能跑起来（这里是为什么呢？求指教）

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_NUZNUGAV7PCUUNN.png)

3. 接下来就能够断在 `JNI_onLoad` 函数

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_FGJZP9K3TEX33UW.png)

3.1 通过动态调试 `JNI_onLoad` 的注册方法的参数，获得注册方法位置是 `0xF90`

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_MZ2KK5UHKHF4JDP.png)

分析传入参数，通过算法便能计算除 FLAG

## 【3】求助疑问

3.1 但是很奇怪，找到注册方法后，并且下了断点，放开程序运行，结果加载失败，程序并不会运行起来，而是就卡在那，但是 IDA 动态调试页面却又是运行状态

3.2 是否是在 `_init` 的时候 `nop` 那句 `fork()` 出现了问题？导致无法加载成功？

3.3 分析 `_init` 具体是如何反调试的呢？

![图片描述](https://bbs.kanxue.com/upload/attach/202206/941979_BP9HE7N3UP4HWRW.png)

上传示例 apk，以上几个问题还望有大神能够解惑！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#NDK分析](https://bbs.kanxue.com/forum-161-1-119.htm)

上传的附件：

- [`CFF_100.apk`](javascript:void(0);)（1.18MB，57次下载）
