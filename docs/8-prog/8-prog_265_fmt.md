
# 基于交叉引用的搜索检测 object hook

> 原文：[`www.pediy.com/kssd/pediy10/72608.html`](https://www.pediy.com/kssd/pediy10/72608.html)

<基于交叉引用的搜索检测 object hook>

Author: `sudami` [`sudami@163.com`]

Time: 08/09/12

昨天和 `VXK` 同学探讨了些关于 object hook 检测。基于 `PDB` 文件解析的方法虽简单见效，但不实用，因为写个普通程序就要自带符号的话很臃肿不方便。于是打算从文件搜索，下下策是暴搜（最坏的打算，不稳定通用，而且 obj 函数巨多，肯定搜不过来），恰巧 `dummy` 同学去年时候放了个“通过重定位表找到对某个特定地址的所有引用地址”的代码，在一些方面适用于搜索 Object Original Address。当然，`dummy` 的 `R0` 代码在 Load PE 到内存方面有部分 bug，需要自己修复。

思路如下:

1. 准备表 1,存放系统当前的 Object Function Address.打开"\\\\ObjectTypes",得到 RootDirectory,遍历其中的 HashBuckets,根据不同的对象得到不同的地址,存到表 1 中
2. 准备表 2,存放系统原始的 Object Function Address.

    把系统内核 `ntosxx` 加载一份到 nonpagepool,在这块区域内进行搜索操作;

    ① 对于已导出的 type（eg.`IoDeviceObjectType`、`PsProcessType`),遍历 ntosXX'EAT,得到 RVA -- uAddr,再调用 `LookupImageXRef((ULONG)NtosCopy, uAddr, LookupXRefCallback);`

    在 `LookupXRefCallback` 回调函数中对地址进行搜索,若是我们要找的函数模块,判断时候进行特征匹配,举个例子:

    ```c
    BOOLEAN
    NTAPI
    LookupXRefCallback(
        PULONG RefAddr
    )
    {
        ULONG tmp, address;
        PBYTE lpAddr = (PBYTE)RefAddr;
        ULONG x, i;
        int time = 0;
        // 返回 True 继续寻找，返回 False 停止寻找
        // DbgPrint("%08X --> %08X\n", RefAddr, RefAddr[0]);
        __try
        {
            //
            // 因为调用的是 V 大的 LoadPeFile,所以跟系统加载到内存中的情况是一样的
            // 里面已经进行了对齐.所以 (ULONG)RefAddr - (ULONG)NtosCopy;
            // 就是 RVA,不需要考虑 ImageBase 了.很好很强大; 而 RefAddr 即是我们自己加载
            // 文件到内存,得到的某个函数的地址,比如我要得到内核函数 A 开头的原始 N 字节
            // 直接在 RefAddr 处往后取出便是了.哈哈,很方便~
            // sudami 08/09/11 凌晨
            //
            tmp = (ULONG)RefAddr - (ULONG)NtosCopy; //RVA
            // DbgPrint("0x%08lx\n", (ULONG)ulKernelBase + tmp );
            //
            // 对于每个 RefAddr ,地址+4 开始反汇编,最终找到匹配 XX 函数内部调用的代码.
            // 在它里面就有 IopXX,PspXX,CmpXX...系列的函数原始地址啦.
            // sudami 08/09/11 凌晨
            //
            //
            // coding
            //
            lpAddr+=4;
            switch ( g_nMethod )
            {
            case 1: // IopCreateObjectTypes 函数
                /*++ <IoDeviceObjectType>
                1.    C7 45 D4 B8 00 00 00 mov [ebp+var_2C], 0B8h
                2.    C7 45 E8 4D 57 4A 00 mov [ebp+var_18], offset IopParseDevice
                    C6 45 AF 01 mov byte ptr [ebp+var_54+3], 1
                3.    C7 45 E4 9A 71 4C 00 mov [ebp+var_1C], offset IopDeleteDevice
                4.    C7 45 EC 8A 90 4D 00 mov [ebp+var_14], offset IopGetSetSecurityObject
                    89 5D F0 mov [ebp+var_10], ebx
                    E8 1F 40 F2 FF call ObCreateObjectType
                    --*/
                time = 0;
                for(i=0;i<70;i++)
                {
                    if ( !MmIsAddressValid( &lpAddr[i]) ){
                        continue ;
                    }
                    if(lpAddr[i]==0xC7 && lpAddr[i+1]==0x45)
                    {
                        time+=1;
                        if (time==1)
                        {
                            x = *((DWORD *)((ULONG)RefAddr+i+3+4));
                            if(x!=0xB8)
                                return FALSE;
                        }
                        if(time==2)
                        {
                            x = *((DWORD *)((ULONG)RefAddr+i+3+4));
                            DbgPrint("IopParseDevice - Orig: 0x%08lx\n", \
                                (ULONG)x + (ULONG)ulKernelBase - (ULONG)pNtH->OptionalHeader.ImageBase);
                            continue ;
                        }
                        if(time==3)
                        {
                            x = *((DWORD *)((ULONG)RefAddr+i+3+4));
                            DbgPrint("IopDeleteDevice - Orig: 0x%08lx\n", \
                                (ULONG)x + (ULONG)ulKernelBase - (ULONG)pNtH->OptionalHeader.ImageBase);
                            continue ;
                        }
                        if(time==4)
                        {
                            x = *((DWORD *)((ULONG)RefAddr+i+3+4));
                            DbgPrint("IopGetSetSecurityObject - Orig: 0x%08lx\n", \
                                (ULONG)x + (ULONG)ulKernelBase - (ULONG)pNtH->OptionalHeader.ImageBase);
                            break ;
                        }
                    }
                }
                ...
            }
            }
    }
    ```

    这样,就实现了对部分 Object 函数的原始地址的查找,相比暴力搜索,要快速稳定的多.

    ② 对未导出的 type（eg.`CmpKeyObjectType`、`ObpTypeObjectType`),要想用 `dummy` 的方法,就有些繁琐了.我是这样分析认为的:

    ```asm
    ; 找到指向这些 未导出的全局变量 type 的指针,就好办了.对于已经导出的 type,比如 IoDeviceObjectType, 在 ntoskrnl.exe 的 EAT 取得的地址,就是指向
    ; IoDeviceObjectType 的指针的地址,而不是 IoDeviceObjectType 的地址. 所以,用 MJ 的方法得到 CmpKeyObjectType 的真实地址也没用,要得到的指向
    ; CmpKeyObjectType 这个变量的指针的地址,才能用用 dummy 的方法.见下:
    ;
    ; lkd> dd IoDeviceObjectType
    ; 80558ee4 817a9ca0 817a9900 817a9e70 817f0428
    ; | |
    ; | |-- IoDeviceObjectType 本身的地址
    ; |-- 引用地址, 是指向 IoDeviceObjectType 的指针的地址
    ;
    ; 68 64 8D 48 00 push offset IoDeviceObjectType ; -->在 callback 函数中要找的引用地址是这样的
    ; ........... 即是 80558ee4,而不是 817a9ca0
    ; C7 45 E8 4D 57 4A 00 mov [ebp+ParseProcedure], offset IopParseDevice
    ; 89 5D F0 mov [ebp+QueryNameProcedure], ebx
    ; E8 1F 40 F2 FF call ObCreateObjectType
    ; -- sudami 08/09/12
    ;
    ; 只要这个问题解决,就完全不用暴搜了. 遍历完所有的 type,找到这些指向这些 type 的指针的地址,再用 dummy 的方法找引用,
    ; 再就可以找到真实的 obj 地址了... 但我目前没有找到解决方法. 以下是部分同学的建议:
    ```

    ```text
    /*
    sudami<sudami@163.com> 13:09:11

    得到 CMPxx 系列的只能用暴搜的方法啦.搜 CmpCreateObjectTypes,然后再在里面去地址...

    sudami<sudami@163.com> 13:09:23

    各种方法都试过,只能这样咯~

    神奇的 MJ0011(353729104) 13:09:26

    太挫了

    sudami<sudami@163.com> 13:09:28

    米办法

    神奇的 MJ0011(353729104) 13:09:58

    动态获取就可以了 o(∩_∩)o...

    sudami<sudami@163.com> 13:10:01

    找引用不可能找到,因为形式都是这样的:

    push offset CmpKeyObjectType ; int

    神奇的 MJ0011(353729104) 13:10:26

    谁让你只会找重定位引用的 o(∩_∩)o...

    sudami<sudami@163.com> 13:10:54

    MJ 大姐再深入提示一下啊

    sudami<sudami@163.com> 13:11:02

    免得我做体力活儿啊

    神奇的 MJ0011(353729104) 13:11:05

    动态啊

    sudami<sudami@163.com> 13:11:19

    原始地址,

    sudami<sudami@163.com> 13:11:23

    老 V 说过了

    sudami<sudami@163.com> 13:11:29

    那样获取不行

    sudami<sudami@163.com> 13:11:33

    dokey 的方法吧?

    sudami<sudami@163.com> 13:11:47

    hook 下,然后 setvalue 下

    神奇的 MJ0011(353729104) 13:12:08

    不是

    sudami<sudami@163.com> 13:12:09

    然后取得的 CmpKeyObjectType.还是....

    神奇的 MJ0011(353729104) 13:12:14

    利用 cm callback o(∩_∩)o...

    sudami<sudami@163.com> 13:12:19

    哦

    神奇的 MJ0011(353729104) 13:12:29

    CmpKeyObjectType 直接用我那个 obj hook 里的方法就可以取到啊

    sudami<sudami@163.com> 13:12:38

    CmpKeyObjectType 的地址找到了没用

    神奇的 MJ0011(353729104) 13:12:45

    object header->obj type~

    sudami<sudami@163.com> 13:12:45

    我不需要 CmpKeyObjectType 的原始地址

    sudami<sudami@163.com> 13:13:53

    lkd> dd CmpKeyObjectType
    8068ec68 817b6b40 00000000 00010000 00000000
    8068ec78 02300000 00000004 00000004 02300000

    sudami<sudami@163.com> 13:14:11

    实际要得到 8068ec68 这个地址,而不是 CmpKeyObjectType 的原始地址 817b6b40

    神奇的 MJ0011(353729104) 13:14:24

    反查啊~

    sudami<sudami@163.com> 13:14:23

    哎,自己还是先暴力吧

    sudami<sudami@163.com> 13:14:42

    反查,这么反查啊?

    神奇的 MJ0011(353729104) 13:14:55

    暴搜反插

    神奇的 MJ0011(353729104) 13:14:59

    obrefxxxx

    sudami<sudami@163.com> 15:50:08

    汗,光说不行,V 同学,你自己写点儿 code,测试下就知道了

    主动防御毁灭者(86879759) 15:50:34

    找引用不是找重定位里的么？

    sudami<sudami@163.com> 15:50:49

    是呀

    神奇的 MJ0011(353729104) 15:51:06

    太傻了太傻了 找重定位时取一下值

    神奇的 MJ0011(353729104) 15:51:14

    比较下是不是 CMXXX

    神奇的 MJ0011(353729104) 15:51:17

    就结局了嘛

    主动防御毁灭者(86879759) 15:51:18

    是啊

    sudami<sudami@163.com> 15:51:44

    好,我看看

    主动防御毁灭者(86879759) 15:52:02

    取得是内存里的值不是文件中的

    主动防御毁灭者(86879759) 15:52:11

    否则还是不能找到的...

    主动防御毁灭者(86879759) 15:53:47

    纯文件的搜索也行，马上搞定~~

    主动防御毁灭者(86879759) 15:53:49

    嘿嘿

    sudami<sudami@163.com> 15:53:57

    那样太累了

    sudami<sudami@163.com> 15:53:59

    何必呀

    主动防御毁灭者(86879759) 15:57:11
    */
    ```



是枚举所有的重定位，每个地址向下搜索

`C7 45 B4 3F 00 0F 00`

`sudami<sudami@163.com>` 15:57:47

你太疯狂了

`主动防御毁灭者(86879759)` 15:59:24

纯 `ring3`, 纯 `ring3`

`sudami<sudami@163.com>` 16:00:37

我暂时不钻牛角尖了

`主动防御毁灭者(86879759)` 16:00:54

看了 `xp`/`2003` `vista` 都是这个特征串

\*/

③ 找不到方便的解决方案,只能用下下策啦.于是我做了点笨活儿,暴搜文件,关键代码如下:

```c
// 经过一系列的读 PE 后，得到一些偏移值。计算 PE 在内存中需要的空间。
// 为其分配一个非分页内存
// 将文件的内容都读取到这里
// 获取文件的大小,申请一块内存来存放它
//DbgPrint("获取文件的大小,申请一块内存来存放它\n");

ZwQueryInformationFile (ntFileHandle, &ioStatus, &fsi,
        sizeof(FILE_STANDARD_INFORMATION), FileStandardInformation);

FileContent =  ExAllocatePool (NonPagedPool, fsi.EndOfFile.LowPart);

if (FileContent == NULL)
{
    ntStatus = STATUS_UNSUCCESSFUL;
    ZwClose(ntFileHandle);
    DbgPrint("ExAllocatePool Error\n");
    goto End;
}

byteOffset.LowPart = 0;
byteOffset.HighPart = 0;
ntStatus = ZwReadFile(ntFileHandle,
    NULL,
    NULL,
    NULL,
    &ioStatus,
    FileContent,
    fsi.EndOfFile.LowPart,
    &byteOffset,
    NULL);

if (!NT_SUCCESS(ntStatus))
{
    ZwClose(ntFileHandle);
    ExFreePool(FileContent);
    DbgPrint("ZwReadFile 将要读的内容，读到一片非分页内存失败 Error\n");
    goto End;
}

if (fsi.EndOfFile.LowPart <= 0)
{
    ntStatus = STATUS_NOT_FOUND;
    ZwClose(ntFileHandle);
    ExFreePool(FileContent);
    DbgPrint("NeedSize <= 0 Error\n");
    goto End;
}

GetHeaders (FileContent, &pfh, &poh, &psh);
//DbgPrint("psh: %08lx\n", (PVOID)psh);
//DbgPrint("start search....\n");
// g_CmpCloseKeyObject_addr
for (i = 0; i < fsi.EndOfFile.LowPart; i++)           
{
    if ( (FileContent[i] == 0x8B) && (FileContent[i+1] == 0xFF) && (FileContent[i+2] == 0x55) && (FileContent[i+3] == 0x8B) &&
        (FileContent[i+4] == 0xEC) && (FileContent[i+5] == 0x83) && (FileContent[i+6] == 0x7D) && (FileContent[i+7] == 0x18) &&
        (FileContent[i+8] == 0x01) && (FileContent[i+9] == 0x77) && (FileContent[i+10] == 0x24) && (FileContent[i+11] == 0x56) 
        )
    {
        //DbgPrint("文件偏移 i: %08lx\n", (PVOID)i);
        sudami_1 = Offset2RVA( i, psh, pfh->NumberOfSections );
        if (sudami_1  == 0) {
            DbgPrint("sudami_1 == 0 Error\n");
            goto NotFound;
        }
        if (sudami_1  > SizeOfImage) {
            DbgPrint("sudami_1 > SizeOfImage Error\n");
            goto NotFound;
        }
        sudami_1 += ModuleBase;
        if (!MmIsAddressValid((PVOID)sudami_1 )) {
            DbgPrint("!MmIsAddressValid((PVOID)sudami_1 ) Error\n");
            goto NotFound;
        }
        g_CmpCloseKeyObject_addr = (DWORD)sudami_1;
        DbgPrint( "CmpCloseKeyObject - Orig:\t0x%08x\n", (ULONG)g_CmpCloseKeyObject_addr );
        break;
    }     
}
```

总之,做 `object` `hook` 的检测,我没有找到一个相对轻松的方法.做了一部分,还有大部分函数没去找,那也只是体力活.

觉得这个思路可行,所以拿出来说说,你可以参考创新...

`ps:` 期间 `VXK` 大牛的思路一直很新颖很神奇,帮了不少忙,`thanks a lot`!他 `BLOG` 里面的科普文章很值得大家借鉴:

`http://hi.baidu.com/killvxk`

附件是凌乱的代码,和一个简单的 `sys` 代码写的不规范,我没有认真的去整理,稍稍改动,防止被坏人 `copy` 做坏事.

`--------------------------------------------------`

参考文章:

1. [`hi.baidu.com/killvxk/blog/item/e43a9598ff40db0e6e068c03.html`](http://hi.baidu.com/killvxk/blog/item/e43a9598ff40db0e6e068c03.html)
   基于交叉引用的搜索未导出的某些东西~
2. [`hi.baidu.com/killvxk/blog/item/ced0eefcfb3b0743d6887d39.html`](http://hi.baidu.com/killvxk/blog/item/ced0eefcfb3b0743d6887d39.html)
3. [`hi.baidu.com/killvxk/blog/item/e6e6374ddb694affd72afce8.html`](http://hi.baidu.com/killvxk/blog/item/e6e6374ddb694affd72afce8.html)
   关于重定向的 `hook` 到底是哪个？
4. [`hi.baidu.com/sudami/blog/item/3b164cf76c041124720eecf7.html`](http://hi.baidu.com/sudami/blog/item/3b164cf76c041124720eecf7.html)
   保护干涉文件的玩意
5. [`hi.baidu.com/sudami/blog/item/ec16d2dd81bd39325882dd51.html`](http://hi.baidu.com/sudami/blog/item/ec16d2dd81bd39325882dd51.html)
   `VXK` 同学的文件重定向
