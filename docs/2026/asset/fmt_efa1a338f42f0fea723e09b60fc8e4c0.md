
```
import unicorn



import idc





def func_block_handle(uc, address, size, user_data):

    if address in (0, 0x29BC0):

        uc.emu_stop()





def decode_str():

    idc.auto_wait()



    dir_path = r'/Users/lll19/Downloads/legu/'

    bin_len = idc.prev_addr(idc.BADADDR)

    bin_len = (bin_len / 0x1000 + (1 if bin_len % 0x1000 else 0)) * 0x1000

    bin_path = dir_path + 'elf_bin'

    idc.savefile(bin_path, 0, 0, bin_len)

    f_bin = open(bin_path, 'rb')

    bin_bytes = bytes(f_bin.read())

    f_bin.close()



    stack_size = 0x100000

    stack_top = bin_len

    stack_bottom = stack_top + stack_size



    uc = unicorn.Uc(unicorn.UC_ARCH_ARM, unicorn.UC_MODE_THUMB)

    uc.hook_add(unicorn.UC_HOOK_BLOCK, func_block_handle)

    uc.mem_map(0, bin_len)

    uc.mem_write(0, bin_bytes)

    uc.mem_map(stack_top, stack_size)



    start_addr = idc.get_segm_by_sel(idc.selector_by_name('.init_array'))

    end_addr = idc.get_segm_end(start_addr)

    addr = start_addr

    while addr + 4 <= end_addr:

        func_addr = idc.get_wide_dword(addr)

        addr += 4

        if not func_addr:

            continue

        print hex(func_addr)



        func_end_addr = idc.find_func_end(func_addr)

        uc.reg_write(unicorn.arm_const.UC_ARM_REG_SP, stack_bottom)

        uc.reg_write(unicorn.arm_const.UC_ARM_REG_LR, 0)

        uc.emu_start(func_addr, func_end_addr)



    f_save = open(bin_path, 'wb')

    f_save.write(str(uc.mem_read(0, bin_len)))

    f_save.close()

    idc.loadfile(bin_path, 0, 0, bin_len)



decode_str()
```

然后开始分析`JNI_OnLoad`，这个函数被混淆了，于是继续`unicorn`走起。

根据输出可知，该函数主要执行了以下几个操作，

通过`RegisterNatives`注册了`WrapperProxyApplication.Ooo0ooO0oO()`，

调用`0x1f668`处的函数进行上下文初始化，

调用`0x2B604`处的函数生成解密`key`，

调用`0xcea8`处的函数加载`dex`。

![](https://bbs.kanxue.com/upload/attach/202202/745332_89P4A2KB2ZG9QY3.jpg)

先分析下`sub_1f668`处的初始化过程，东西比较多就不贴图了

首先是一些常规操作，获取`vm`类型、`PackageInfo`、`ActivityThread`、`ClassLoader`等。

然后是通过`GetMethodID`获取之前注册的`WrapperProxyApplication.Ooo0ooO0oO()`的`MethodID`，然后在`MethodID`指向的内存（通过Android源码可知，`MethodID`实际是`ArtMethod`对象的指针）查找之前注册的函数地址，找到后保存该偏移值，后面会通过这个偏移值对系统`native`方法进行`hook`。

再然后就是加载`0OO00l111l1l`，解析该文件，把该文件的各种数据指针缓存起来，用于后面数据解密

再然后判断系统是否升级，读取`prodexdir/`的消息`.updateIV.dat`中缓存的数据与`libart.so`和`dex2oat`的大小进行比较，如果不相等则将`.odex`和`.vdex`文件删除并更新`.updateIV.dat`

`0OO00l111l1l`数据结构如下，前4字节为`dex`的数量，后面分别为三种数据，通过后面分析可知

第一部分数据为压缩的`dex`，其中的指令被抽取了

第二部分为压缩且加密的索引数据

第三部分为压缩且加密的指令数据

![](https://bbs.kanxue.com/upload/attach/202202/745332_DSGTDFJ6WCZ6RK8.jpg)

看下`sub_2B604`的代码，比较简单，读取文件内容与`byte_31391`处的字符处理后存放在`byte_36A8C`

![](https://bbs.kanxue.com/upload/attach/202202/745332_JYP82AN8NCKTNZ8.jpg)

大概分析下`sub_cea8`过程，

调用`sub_1CD90`，对`DexFile.defineClassNative()`进行`hook`，原理是先获取`defineClassNative`的`MethodID`，然后通过先前获取到的偏移值，将该函数的本地函数指针保存起来，再通过`RegisterNatives`重新为该函数注册一个本地函数。

调用`java`方法`com.wrapper.proxyapplication.MultiDex.preparetoinstallDexes()`，获取`dexElements`

`hook`几个系统函数`mmap`、`execve`、`execv`，`hook`的方式是通过遍历重定位项实现的。这个几个`hook`在我分析的过程中没用上

多线程调用函数`sub_CE14`解密加载`dex`并`opt`

等待所有线程结束后，获取`mCookie`缓存起来

取消之前`hook`的几个系统函数，并设置几个环境变量

构造当前应用的原始`application`，并调用其`attach()`方法

剩下的流程都不重要了，

现在只关心解密函数`sub_CE14`，

首先调用`sub_CC2C`进行`dex`解压缩并写入`/data/data/com.baidu.homework/files/prodexdir`下的`dex`文件，再调用`java`方法`com.wrapper.proxyapplication.MultiDex.installDexes()`进行`dex`替换

解压缩函数为`sub_2B2AC`，看看反编译代码，全是各种字符操作，具体算法就不看了，待会儿直接上`unicorn`

`dex`加载成功后调用`sub_10EB4`进行指令解密

首先对第二部分数据进行解密，解密函数为`sub_2315C`，解密完成后调用`sub_2B2AC`解压缩，再调用`sub_115F4`生成每个类结构的索引

然后对第三部分数据进行解密，解密函数为`sub_2315C`，解密完成后调用`sub_2B2AC`解压缩，得到所有抽取的指令

现在只剩最后一步了，那就是在`class`加载的时候，对抽取的指令进行填充，这里需要分析的是前面`hook`的`DexFile.defineClassNative()`，对应的函数为`sub_1C900`，其主要内容如下：

首先查找`class`所在的`dex`和对应的索引（`sub_22B90`）

然后进行指令填充（`sub_101B8`）

最后调用原来的函数地址

![](https://bbs.kanxue.com/upload/attach/202202/745332_W8TYYEHKDBBVQ6G.jpg)

`sub_22B90`的代码和对应的数据如下，通过分析可知，

从偏移`8`开始是一个`hash`表，每个表项`3`个字段，分别为`hash`值、类名偏移、类结构索引偏移

偏移为`4`的字段为`hash`表的大小

![](https://bbs.kanxue.com/upload/attach/202202/745332_88EJ9QD9GQZSWBQ.jpg) ![](https://bbs.kanxue.com/upload/attach/202202/745332_ZUXP3CNZ9T8XTHF.jpg)

函数`sub_101B8`对`class`指令进行填充，函数被混淆了，既然这样，也就不看了，直接`unicorn`

![](https://bbs.kanxue.com/upload/attach/202202/745332_WYWA4NBW7NST6K3.jpg)

到此为止，所有的流程都分析完了。

开始准备脱壳脚本，需要模拟执行的函数流程分为以下几部分：

1、执行`0x2B604`处的代码。读取`tosversion`文件的内容，处理后作为解密的`key`（该处只读取了`16`个字节，但是解密的时候，复制了`32`字节的`key`，但是不影响，实际执行解密的时候只用了前`16`字节）

2、循环执行`0x2B2AC`处的代码。解压出所有被抽取指令的`dex`

3、循环执行`0x2315C`、`0x2B2AC`、`0x115F4`这三处代码，分别对应`class`信息的解密、解压、建索引

4、循环执行`0x2315C`、`0x2B2AC`这两处代码，分别对应指令的解密、解压

5、遍历所有`dex`的`hash`表，循环执行`0x101b8`处的代码进行指令修复

#
# 完整的脱壳脚本就不贴了，提示超出字数限制了，我把它放在附件里面了。。。。

# 样本打包后附件大小也超限制了。。。。

# 样本我放网盘了，附件只保留了脚本文件。。。。

# 。

# 。

# 。

# 。

# 。

# 。

贴个脱壳修复后的图

![](https://bbs.kanxue.com/upload/attach/202202/745332_949325S2QY9GEBV.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

[#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)

上传的附件：

- [legu_0.zip](javascript:void(0);)

  （4.10kb，469次下载）
