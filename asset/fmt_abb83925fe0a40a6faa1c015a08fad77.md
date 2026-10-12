
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
