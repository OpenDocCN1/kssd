
然后我们直接进行修改

![image-20221110190749792](upload/attach/202211/905443_T3MW8UF5A2GAHGX.png)

（7）`ro.build.description`

![image-20221110191315720](upload/attach/202211/905443_CBXBAPV76VKYYJU.png)

![image-20221110191035722](upload/attach/202211/905443_BQAVKXCQDA7MSCB.png)

![image-20221110191132102](upload/attach/202211/905443_Y5JCS52HZTF29FS.png)

![image-20221110191528854](upload/attach/202211/905443_W현MGK2KYYKTQXVF.png)

应修改为：

```
aosp_sailfish-user 8.1.0 OPM1.171019.011 eng.Windaa.20221109.050000 release-keys
```

![image-20221110191702936](upload/attach/202211/905443_98V6JYRR7Q6YG8K.png)![image-20221110191739877](upload/attach/202211/905443_Q9WWSZSPBR2E3QK.png)

（8）`ro.build.fingerprint`

![image-20221110191832725](upload/attach/202211/905443_C69MZ9BQUAS5RKK.png)

一样，我们进行定位

![image-20221110191909861](upload/attach/202211/905443_RXGF5GRK2K96PT6.png)

![image-20221110213839174](upload/attach/202211/905443_JT5F45V662AJRCY.png)

然后我们进行修改

```
Android/aosp_sailfish/sailfish:8.1.0/OPM1.171019.011/Windaa11090500:user/release-keys
```

![image-20221110213959934](upload/attach/202211/905443_Y9VP64CUNFJ57CM.png)

（9）`ro.product.model`、`ro.product.brand`、`ro.product.name`

一些朋友想要设备向google的原设备一样，可以修改手机的代号和产品名称，这里就不修改了，感兴趣朋友可以修改

### 3.4 root定制

root定制这里参考通用的方法：xu为自定义的名称

```
AOSP/system/extras/su下修改Android.mk中su修改为xu
AOSP/system/core/libcutils/fs_config.cpp中/system/xbin/su修改为/system/xbin/xu
AOSP/system/sepolicy/private/file_contexts中/system/xbin/su修改为/system/xbin/xu
```

第一处

![image-20221110193527770](upload/attach/202211/905443_VFKAMJ5UY5A6NV9.png)

第二处

![](upload/attach/202211/905443_CPPUHM8K3KMG7RA.png)

第三处

![image-20221110193712743](upload/attach/202211/905443_2UDS86D86984ZFV.png)

### 3.5 编译

然后再次编译，选择user-debug版

```
lunch：24
make -j4
```

![image-20221109214415835](upload/attach/202211/905443_X8PTGU63TMYRFMZ.png)

### 3.6 效果演示

我们重新刷机，刷机完成后，就可以发现定制的镜像和原理看起来一致

定制后：

![image-20221115142107682](upload/attach/202211/905443_XCAP2PMDBR54QBX.png)

定制前（user版）：

![image-20221110152811681](upload/attach/202211/905443_KJ3M5GFSPQP8QM4.png)

我们可以试下，看能不能正常root

![image-20221115142222210](upload/attach/202211/905443_P2DZX7YDGSP7YFU.png)

可以发现是没有问题的

## 4. 总结

本文编译了user版和user-debug版，并通过指纹定制将user-debug魔改为user版，为后续试验提供了更多的方便，相关文件后续上传github

## 5. 参考文献

```
https://bbs.pediy.com/thread-273485.htm#msg_header_h2_0
http://lzonel.cn/3136.html
https://blog.csdn.net/u010142437/article/details/78030485
https://www.jianshu.com/p/dd6cca4ec27d
```

# 源码编译（4）——root指纹定制与抹除

目录

- [源码编译（4）——root指纹定制与抹除](https://bbs.kanxue.com/#源码编译（4）——root指纹定制与抹除)
- [1.前言](https://bbs.kanxue.com/#1.前言)
- [2.基础知识](https://bbs.kanxue.com/#2.基础知识)
  - 2.1 指纹解析
  - 2.2 root检测方式
- [3.实验](https://bbs.kanxue.com/#3.实验)
  - 3.1 user版本的编译
  - 3.2 userdebug版本编译
  - 3.3 root指纹抹除
  - 3.4 root定制
  - [3.5编译](https://bbs.kanxue.com/#3.5编译)
  - [3.6效果演示](https://bbs.kanxue.com/#3.6效果演示)
- [4.总结](https://bbs.kanxue.com/#4.总结)
- [5.参考文献](https://bbs.kanxue.com/#5.参考文献)

## 1. 前言

本文为源码定制学习的root指纹抹除篇，通过本文的学习，读者可以掌握Android指纹的基本定制能力和root定制能力，本文参考了看雪大佬[应用root检测通杀篇](https://bbs.pediy.com/thread-273485.htm)，为实验记录笔记。有问题，可以一一指出：

本文第二节主要讲述基本原理

本文第三节主要讲述实验

本文第四节主要为总结

## 2. 基础知识

### 2.1 指纹解析

需要对Android指纹进行修改，最重要是明白`build.prop`的参数含义，这里截取网上文章中的一部分，方便大家理解：

|  |  |

| --- | --- |
