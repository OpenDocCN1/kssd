
# [原创]乐固libshella 2.10.1分析笔记

这篇文章是记录本人在学习Legu脱壳的心得，分析的样本是Legu `libshella-2.10.so`的版本。

本文分为几个部分：

- 修复So文件
- 第一次解密
- 第二次解密
- 解密Dex
- Dalvik下加载Dex原理分析
- Art下加载Dex原理分析
- Dalvik下脱壳机编写
- Art下脱壳机编写

## 修复So文件

Legu的核心加固代码都是在`libshella-x.so`里面，

我们用IDA打开`libshella-2.10.1.so`，意料之中的是，IDA打开什么也看不到。

![](https://bbs.kanxue.com/upload/attach/201706/23C7WFB7RJ2NFHG.jpg)

在这里我用ThomasKing的ELF修复工具试着修复一下，修复之后已经可以看到很多函数了

但是修复后的So文件在IDA还是看不到`init_array`段，`JNI_OnLoad`函数也是加密状态。[![](https://bbs.kanxue.com/upload/attach/201706/629060_f6g0tv4mteys0jo.png)](https://bbs.kanxue.com/upload/attach/201706/629060_f6g0tv4mteys0jo.png)

由于So加载完之后会调用`init_array`函数，我们从Android源码入手来获取`init_array`的地址，在Android源码 linker.cpp的代码中有这么一段代码就是来调用init和`init_aray`函数的

```
CallFunction("DT_INIT", init_func);
CallArray("DT_INIT_ARRAY", init_array, init_array_count, false);
```

将手机中的linker拖入ida，找到`DT_INIT_ARRAY`字符串，可以获取调用`CallArray`的地址，在我的linker中，`0x295E`处就是调用init段的地址

![](https://bbs.kanxue.com/upload/attach/201706/629060_aolb74w2tk0aezm.png)

我用修复后的So文件替换掉原始的`Libshela-2.10.1.so`文件后，App还可以成功运行，在这里就直接用修复后的So来动态调试

## 第一次解密

在`init_array`的函数中，首先会解密出`JNI_OnLoad`和反调试代码等等

解密是一个while循环，会从`libshella.so` `0x2000`处开始解密，到`0x3954`处结束解密，解密完成之后，就只是创建了一个反调试线程，检测到反调试就执行`raise(9)`,最简单的方法可以将call raise函数的代码nop掉

![](https://bbs.kanxue.com/upload/attach/201706/629060_wxv9u9uqrim6211.png)

我没有关注`init_array`的解密算法是什么样的，当它解密完成之后，我将`0x2000-0x3954`偏移处的代码dump出来了，然后替换掉上面修复后的`libshella_fix.so`对应的字节，这样生成的So就包含解密后的`Jni_OnLoad`函数了![]()

## 第二次解密

![](https://bbs.kanxue.com/upload/attach/201706/629060_ixfia7e5wu0f5yp.png)

![]()

咋一看，很奇怪的是在`JNI_OnLoad`函数中，还会自身再调用`JNI_OnLoad`函数，关键地方在于`sub_1968()`函数，

这个函数比较庞大，我分析了半天，没分析处到底是怎么解密的，但是这里并不影响后面的分析，

当这个函数执行完之后，通过`dlsym` 获取`JNI_OnLoad`函数的地址已经不是开始的`JNI_OnLoad`函数地址，我们这里成为`new_JNI_Onload`，而是处于动态分配的debug内存区域，其实这些debug内存区域的代码就是前面`sub_1968()`函数解密出来的

[![](https://bbs.kanxue.com/upload/attach/201706/629060_qzox1plty1n264g.png)](https://bbs.kanxue.com/upload/attach/201706/629060_qzox1plty1n264g.png)

为了方便分析，我将`libshella.so` debug区域以及`libc.so` `libdvm.so`进行了内存快照拍摄![]()

[![](https://bbs.kanxue.com/upload/attach/201706/629060_zgpmtz5pkzkfftj.png)](https://bbs.kanxue.com/upload/attach/201706/629060_zgpmtz5pkzkfftj.png)

我们用上面内存快照拍摄的生成的idb来分析`new_JNI_OnLoad`函数，在`new_OnLoad`函数中，看到了久违的`registerNative`注册操作，注册了Java层的`load`，`runCreate`等native函数

![](https://bbs.kanxue.com/upload/attach/201706/629060_xq9b09t83sr2n0m.png)

## 解密Dex

![](https://bbs.kanxue.com/upload/attach/201706/629060_oeo40tifcfw4cet.png)

![]()

在Java层的`attachBaseContext`函数中，首先执行的是native层的`load`函数，

在`load`函数中，首先会判断当前是dalvik还是art虚拟机，然后执行对应的加载Dex方法

![](https://bbs.kanxue.com/upload/attach/201706/629060_l7sxucdskl5u27z.png)

![]()

在解密真正的dex之前，首先是获取解密前dex的存储位置，很简单的是，`解密前的Dex存储地址=内存odex地址+DexHeade->dataOff+DexHeade->dataSize`

[![](https://bbs.kanxue.com/upload/attach/201706/ZGBQMS7ADRGA3W7.jpg)](https://bbs.kanxue.com/upload/attach/201706/ZGBQMS7ADRGA3W7.jpg)

获取到解密前dex的内存地址，就会执行`decrypt`函数对dex进行解密，手动脱壳的话，在这里就可以dump处真正的dex了![]()

## Dalvik下加载Dex

在我的那篇阿里早期加固代码还原的帖子中，在Dalvik下的Dex加载方式是通过`Dalvik_dalvik_system_DexFile_openDexFile_bytearray`这个方式进行内存加载的，但是Legu在这里使用了一种更加高级的方法

首先Legu会通过`loadDex`函数加载`mix.dex`，从而得到一个表示`mix.dex`的`mCookie`对象

![](https://bbs.kanxue.com/upload/attach/201706/629060_oqhuby4i1uarua0.png)

![]()

然后Legu会构造一个`0x34`字节的结构体`DexHeaderBak`，用来保存真正Dex的`DexHeader`信息

![](https://bbs.kanxue.com/upload/attach/201706/629060_aej1yjgzk52u9kz.png)

![]()

然后用`mmap`分配一段内存`mmap_buffer`，在其中填充一些真正Dex的信息，我画了一张图描述`mmap_bufffer`的内存结构

[![](https://bbs.kanxue.com/upload/attach/201706/629060_xaa956o5k1a8lcg.png)](https://bbs.kanxue.com/upload/attach/201706/629060_xaa956o5k1a8lcg.png)

在Android4.4中，`mCookie`对象是指向`DexOrJar`结构的指针，![]()

```
struct DexOrJar {
    char*       fileName;
    bool        isDex;
    bool        okayToFree;
    RawDexFile* pRawDexFile;
    JarFile*    pJarFile;
    u1*         pDexMemory; // malloc()ed memory, if any
};

struct RawDexFile {
    char*       cacheFileName;
    DvmDex*     pDvmDex;
};
```

Legu会通过`mixdex_cookie`获取`pRawDexFile`指针，再来获取`pDvmDex`指针，

![](https://bbs.kanxue.com/upload/attach/201706/629060_fhbjh44jobe9o8m.png)

![]()

最后将`pDvmDex`指向的内容全部替换为`mmap_buffer`结构体中的内容，这样`mix.dex`的`mCookie`对象已经表示为真正的Dex，而不是原本的`mix.dex`了，关于Legu为什么知道这么做，可能要分析`Dalvik_dalvik_system_DexFile_openDexFile_bytearray`这个函数的原理了。

![](https://bbs.kanxue.com/upload/attach/201706/629060_77yd9svh7vmsydt.png)![]()

Dalvik下Dex的加载大部分都完成了，后面Legu加载的方法其实也就是`MultiDex`的多Dex加载方法，这里不做分析。

## Art（Android 6.0）下加载Dex分析

Android4.4下有`Dalvik_dalvik_system_DexFile_openDexFile_bytearray`这个函数加载Dex，但是Android N以上就没有了这个函数，

Legu在Art下会`hook`几个系统函数，并且获取`libart.so`中的`art::dexFile::OpenFile`函数的地址

[![](https://bbs.kanxue.com/upload/attach/201706/629060_kuq2y6j8ekc4giz.png)](https://bbs.kanxue.com/upload/attach/201706/629060_kuq2y6j8ekc4giz.png)

![]()

让我开始很疑惑的是，Legu用`OpenFile`函数打开内存中的oat文件，并没有任何加载dex的操作

我分析了下Android 6.0下的`OpenFile`函数

![](https://bbs.kanxue.com/upload/attach/201706/629060_yfirol7gjll6duv.png)

![]()

`OpenFile`首先会call `fstat`函数获取`location`的大小，但是此时执行的却是hook后的`fstat`，Legu会替换真正的`location`的大小，而是返回真正的Dex大小，

然后根据前面的大小调用`MapFile`函数，`MapFile`最终调用了`mmap`函数，此时执行的还是hook后的`mmap`，

`fake_mmap`代码如下，首先会解密处真正的dex内容，然后用真正的Dex地址替换`mmap`返回值

[![](https://bbs.kanxue.com/upload/attach/201706/629060_i674u8y7o2ejb5m.png)](https://bbs.kanxue.com/upload/attach/201706/629060_i674u8y7o2ejb5m.png)

![]()

`OpenFile`进行了`mmap`操作后，会进行`OpenMemory`加载Dex，经过上面的hook，表面上是打开`base.odex` oat文件，实际上`OpenMemory`真正的Dex文件，因此我们可以Hook `OpenMemory`达到dump dex的目的，所以可以看出Android Art下使用`OpenMemory`函数来加载dex文件。

## Dalvik下脱壳机编写

根据上面的分析，在Dalivk下可以很容易地获取到真正Dex的内存位置：`真正Dex存储地址=内存odex地址+DexHeade->dataOff+DexHeade->dataSize`

App通过运行之后，注入后虽然可以Dump Dex，但是dump 的Dex跟原始的字节有几个字节不同，导致重打包运行出错，Legu在加载的时候是真正的Dex文件，但是脱离Legu代码运行起来后发现有几个字节变了，刚开始老以为是Legu对方法字节码做了处理，后来调试发现是Dalik自己改变的，字节码的变化不知道是不是Dalvik对字节码进行优化导致的。

Art下Dump 得到的Dex是完成正确的，推荐大家在Art下进行Dump Dex

```
void DumpDex_kitkat(char* pkgName)
{
    int pid=getpid();
    printf("pid:%d\n",pid);
    char filename[100]={0};
    char dumpfilepath[256]={0};
    char* s;
    unsigned int startAddr=NULL;
    unsigned int endAddr=NULL;
    unsigned int mainDexAddr;
    char* oatPath[256]={0};
    errno=0;
    sprintf(dumpfilepath,"/data/data/%s/dump.dex",pkgName);
    sprintf(filename,"/proc/%d/maps",pid);
    FILE *fp;
    fp = fopen(filename, "r");
    if(fp!=NULL)
    {
        char line [2048];
        while (fgets(line, sizeof(line), fp ) != NULL ) /* read a line */
        {
            if (strstr(line, pkgName) != NULL)
            {
                if (strstr(line, "classes.dex") != NULL)
                {
                    LOGI("dvm-found odex address");
                    s = strchr(line, '-');
                    if (s == NULL)
                        LOGI(" Error: string NULL");
                    *s++ = '\0';
                    //strtoul:将字符串转化成无符号整型
                    startAddr = (void *)strtoul(line, NULL, 16);
                    endAddr = (void *)strtoul(s, NULL, 16);
                    LOGI(" dvm classes.odex addr %x-%x", startAddr,endAddr);
                    break;
                }
            }
        }
        fclose ( fp);
    }
    else
    {
        LOGI("fopen maps failed");
        return;
    }
    if(startAddr==NULL || endAddr==NULL)
    {
        LOGI("found odex or oat file failed");
        return;
    }
    mainDexAddr=startAddr+0x28;
    LOGI("dexAddr:%s",(unsigned char*)mainDexAddr);
    int magic=*(unsigned int*)mainDexAddr;
    if(magic!=0x0A786564)
    {
        LOGI("not find main Dex");
        return ;
    }
    unsigned int OrgDexOffset=getOrgDexOffset(mainDexAddr);
    LOGI("OrgDexOffset:%d",OrgDexOffset);
    unsigned int realDexAddr=mainDexAddr+OrgDexOffset;
    magic=*(unsigned int*)realDexAddr;
    if(magic!=0x0A786564)
    {
        LOGI("not find real Dex");
        return ;
    }
    unsigned int dexSize=*(unsigned int*)(realDexAddr+0x20);
    LOGI("dexSize:%d",dexSize);
    void* buffer=malloc(dexSize);
    if(buffer==0)
    {
        LOGI("malloc dexsize buffer failed");
        return;
    }
    memcpy(buffer,(void*)realDexAddr,dexSize);
    FILE* fd_dump=fopen(dumpfilepath,"wb+");
    if(fd_dump==NULL)
    {
        LOGI("fopen dumpfile error:%s",strerror(errno));
        return;
    }
    fwrite(buffer,dexSize,1,fd_dump);
    free(buffer);
    fflush(fd_dump);
    fclsoe(fd_dump);
}
```

## Art下脱壳机编写

我这里是针对Android 6.0下hook `OpenMemory`函数，在5.1下`OpenMemory`函数可能参数有所改变要另外做处理

注入zygote Hook `OpenMemory`函数dump dex

```
uint32_t new_art_dexFile_openMemory(void* DexFile_thiz,char* base,int size,void* location,
    void* location_checksum,void* mem_map,void* oat_dex_file,void* error_meessage )
{
```
