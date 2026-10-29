
### 3.插桩脱壳法

插桩脱壳法，就是在Android源码里面定位到相应的脱壳点，然后插入相应的代码，重新编译源码生成系统镜像，最后就可以使用定制的系统进行脱壳。

我们在[源码编译（1）——Android6.0源码编译详解](https://bbs.pediy.com/thread-269575.htm)中已经讲述了如何编译源码，接下来我们进行插桩脱壳。

同理，还是定位脱壳点，我们还是随便定位一个脱壳点`LoadMethod`，然后进行插桩。

![image-20220613220518548](https://bbs.kanxue.com/upload/attach/202206/905443_QM6K888MKVR74HU.png)

```
//add
char dexfilepath[100]=0;
memset(dexfilepath,0,100);
sprintf(dexfilepath,"%d_%zu_LoadMethod.dex",getpid(),dex_file.Size());
int dexfd = open(dexfilepathm,O_CREAT|O_RDWR,666);
if(dexfd>0){
    int result = write(dexfd,dex_file.Begin(),dex_file.Size());
    if(result>0){
        close(dexfd);
        LOG(WARNING)<<"LoadMethod"<<dexfilepath;
    }
}
//add
```

同理我们在`execute`同样插桩此段代码，最后进行编译，编译成功。

![image-20220613172222202](https://bbs.kanxue.com/upload/attach/202206/905443_H3FU3M4GFAVDSDR.png)

然后给程序授权`sdcard`权限，再次启动应用，就可以看见脱取的dex文件就保存在`sdcard`目录下。

![image-20220613215956911](https://bbs.kanxue.com/upload/attach/202206/905443_Z4NKWZWCYS43AU5.png)

再次将`sdcard`下dex文件打开，这里我们已经看见了`8732435`这个文件，再次打开脱取成功。

![image-20220613172222202](https://bbs.kanxue.com/upload/attach/202206/905443_EWQ8RH85DD8YYN2.png)

### 4.反射脱壳法

反射脱壳法的核心思想就是利用前面我们提到的`mCooike`值。

```
核心思路：反射 + mCookie
步骤：
1、找到加固apk的任一class，一般选择主Application或Activity
2、通过该类找到对应的Classloader
3、通过该Classloader找到BaseDexClassLoader
4、通过BaseDexClassLoader找到其字段DexPathList
5、通过DexPathList找到其变量Element数组dexElements
6、迭代该数组，该数组内部包含DexFile结构
7、通过DexFile获取其变量mCookie和mFileName

至此我们已经获取了mCookie

对该mCookie的解释:
#1、4.4以下好像，mCookie对应的是一个int值，该值是指向native层内存中的dexfile的指针
#2、5.0是一个long值，该值指向native层std::vector<const DexFile*>* 指针，注意这里有多个dex，你需要找到你要的
#3、8.0，该值也是一个long型的值，指向底层vector，但是vector下标0是oat文件，从1开始是dex文件
// 至于你手机是那个版本，如果没有落入我上面描述的，你需要自己看看代码

8、根据mCookie对应的值做转换，最终你能找到dexfile内存指针
9、把该指针转换为dexfile结构，通过findClassDef来匹配你所寻找的dex是你要的dex
10、dump写文件
```

综述`mCookie`是在native层就是`dexfile`的指针，我们利用反射原理来获取`mCookie`，从而就可以进行脱壳了，这里我们同样使用`frida`演示：

编写hook代码。

![image-20220613190231102](https://bbs.kanxue.com/upload/attach/202206/905443_DNMYNN4NDYZAZ2F.png)

![image-20220613190303491](https://bbs.kanxue.com/upload/attach/202206/905443_NQW7Y6EVNHQD8QP.png)

我们看见了和上面同样大小的`8841876_mCookie.dex`。

![image-20220613190401266](https://bbs.kanxue.com/upload/attach/202206/905443_SMJCHGF7F6XC6BF.png)

使用工具打开，发现同样脱壳成功。

![image-20220613190438224](https://bbs.kanxue.com/upload/attach/202206/905443_VRHWF36TFJ8Z36Y.png)

### 5.动态调试脱壳法

所谓动态调试法，核心原理和上面一样，就是我们在动态调试的过程中找到`DexFile`的起始地址和大小，然后执行脚本进行dump。

首先选取脱壳点，我们还是选择`DexFile::DexFile`。

![image-20220613210130186](https://bbs.kanxue.com/upload/attach/202206/905443_HYNNHUZSYQZWPMZ.png)

动态调试的步骤我在前面的文章中已经做了详细的讲解，不会的朋友去看前面的文章。

首先我们启动`android_server`。

![image-20220613194111711](https://bbs.kanxue.com/upload/attach/202206/905443_YD2S78M7HQ9MBEZ.png)

然后我们附加上进程。

![image-20220613194326114](https://bbs.kanxue.com/upload/attach/202206/905443_2BGHYQAVMXN38VV.png)

![image-20220613195349008](https://bbs.kanxue.com/upload/attach/202206/905443_3DXB4XUZ5TN7UAY.png)

![image-20220613203024990](https://bbs.kanxue.com/upload/attach/202206/905443_RWUW7P8RZKB5M3Z.png)

然后我们打开`libart.so`，并定位到`DexFile::DexFile`。

![image-20220613210859021](https://bbs.kanxue.com/upload/attach/202206/905443_EY8SAHZUDMGFG4G.png)

然后在该函数下断点，然后F9过来。

![image-20220613211145356](https://bbs.kanxue.com/upload/attach/202206/905443_HKQM6BW44V7E2HB.png)

此处我们就可以很明显看到`X1`就是我们的起始地址，`X4`是我们的偏移值。

编写脚本进行hook：

```
static main(void){
    auto fp, begin, end, dexbyte;
    fp = fopen("d:\\dump.dex", "wb+");
    begin = 0x76FCD93020;
    end = begin + 0x7EEC5600;
    for ( dexbyte = begin; dexbyte<end;dexbyte++ )
    {
        fputc(Byte(dexbyte), fp);
    }
}
```

![image-20220613214647627](https://bbs.kanxue.com/upload/attach/202206/905443_A3M7EMAKQFHG9EX.png)

直接运行`run`。

然后我们查看`dump.dex`文件。

![image-20220613223330340](https://bbs.kanxue.com/upload/attach/202206/905443_N6GJV42VY3V9RZ4.png)

![image-20220613215148251](https://bbs.kanxue.com/upload/attach/202206/905443_7UEEANG6STMKHQB.png)

我们可以发现这里是代理类，还没有到我们想要的dex，我们再次F9，再次到这里，地址再次改变，再次结合长度来计算，我们每次计算可以取小点值，先试一下。

![image-20220613215343895](https://bbs.kanxue.com/upload/attach/202206/905443_S43C2KD539HDHA9.png)

发现还是不是，我们需要不停测试直到dump出dex为此。

这里大家可以下去按照此方法尝试，或者换一个脱壳点来尝试。

### 6.特殊API脱壳法

所谓特殊的API脱壳法就是通过Android自身提供的API来获得Dex，这主要是参考`Fdex2`，前面我们讲了`Fdex2`主要是利用Android7.0及以下提供了`getDex()`和`getBytes()`两个API，我们可以直接获得`class`对象，然后直接调用这两个API。

![image-20220613191047342](https://bbs.kanxue.com/upload/attach/202206/905443_JCF94H3VCDB2AWK.png)

![image-20220613191158377](https://bbs.kanxue.com/upload/attach/202206/905443_QZMY24QDWR2D2D8.png)

编写hook代码：

![image-20220613192251923](https://bbs.kanxue.com/upload/attach/202206/905443_7ZEBNGDDBYRZX2M.png)

```
1.使用frida枚举所有Classloader
2.确定正确的ClassLoader并获取目标类的Class对象
3.通过Class对象获取得到dex对象
4.通过dex对象获取内存字节流并保存
```

然后我们查看程序的类对象，随便dump一个类对象。

![image-20220613191722932](https://bbs.kanxue.com/upload/attach/202206/905443_V6XG2HJXB3945JV.png)

![image-20220613192447143](https://bbs.kanxue.com/upload/attach/202206/905443_SPQ89H72VHE2C4J.png)

然后我们再次用工具打开。

![image-20220613192548342](https://bbs.kanxue.com/upload/attach/202206/905443_MM9V7UDZ3N46PRD.png)

![image-20220613192659791](https://bbs.kanxue.com/upload/attach/202206/905443_NAS4Y7JS2VPTUM5.png)

发现就可以成功的dump。

通过这种方式，我们发现神奇的事我们还可以抽取壳的情况，比如我们之前为空类。

![image-20220613192926853](https://bbs.kanxue.com/upload/attach/202206/905443_C9HF2JX3AMDQP9K.png)

我们明显可以发现这里是采用了函数抽取的技术，一般的一代壳dump方案是无法解决抽取壳的，我们使用特殊API方法。

![image-20220613193028308](https://bbs.kanxue.com/upload/attach/202206/905443_24VE45Q6UFGVX2U.png)

再次打开，成功dump。

![image-20220613193102209](https://bbs.kanxue.com/upload/attach/202206/905443_75FSRB4YMDZN793.png)

这其实主要是抽取壳的一个回填时机的问题，这个详细放在以后抽取壳中讲解。

## 六、实验总结

本文总结了当下dex整体加壳的基本原理，和常用的一些脱壳方案，并一一进行复现，还有一些文件监控法等，由于我平时用的很少就没列举了，复现实验过程中由于涉及到不同的实验，所以我用了Android 6.0 Android 7.0 Android 8.0三台机器进行实验，所以大家可以注意下对应的方法和其Android版本，这里彻底解决了整体加壳的脱壳方案，到这里可以掌握脱壳、抓包、Hook、反Hook、反调、反签等基本手段，这样在进行Android App漏洞挖掘过程中将事半功倍。后面我将继续讲解Android App漏洞中的XSS漏洞、Sql注入漏洞、文件上传漏洞、端口扫描漏洞、WebView漏洞等。

脱壳脚本相关样本会放在github，所有的脱壳脚本和工具和上传知识星球。

github：[github](https://bbs.kanxue.com/elink@629K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2)

## 七、参考文献

1. `https://bbs.pediy.com/thread-252630.htm#msg_header_h2_4`
2. `https://bbs.pediy.com/thread-254555.htm#msg_header_h2_4`
3. `https://cb1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4F1M7i4g2S2L8X3E0W2i4K6u0W2j5$3!0E0/post/id/221905?display=mobile`
4. `https://9e0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4q4B7x3K6l9I4i4K6u0W2j5$3!0E0/news/317.html`

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-6-16 10:50 被随风而行aa编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)
[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
[#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)
[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)
[#程序开发](https://bbs.kanxue.com/forum-161-1-124.htm)
[#源码框架](https://bbs.kanxue.com/forum-161-1-127.htm)
