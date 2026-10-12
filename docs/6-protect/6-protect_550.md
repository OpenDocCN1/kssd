
# 驱动壳编写总结

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-611.htm`](https://www.pediy.com/kssd/pediy08/pediy8-611.htm)

Author: `PolyMeta`  
Email: `PolyMeta@whitecell.org`  
Homepage: `http://www.whitecell.org`  
Date: `2006-05-27`

壳作为一种主要的软件保护手段大概可以分为压缩壳和加密壳两类。而现在流行的加壳程序无论是压缩的还是加密的几乎都是针对应用层程序的，对于驱动程序的保护壳则几乎是空白。笔者只在一些国外加密狗的驱动中见过类似应用层的保护壳。本篇文章主要介绍驱动加壳程序与应用层加壳程序在编写上的区别以及一些注意事项。

## 1. 校验和的计算

驱动程序被加壳后必须重新进行校验和的计算，否则加壳后的驱动加载会失败。

```asm
********************计算 pe 文件校验和********************
CalcPECheckSum PROC lpBaseAddr:DWORD,dwFileSize:DWORD
  LOCAL  CheckSum:DWORD
  pushad
  mov    ecx,dwFileSize
        inc    ecx
        shr    ecx,1
        xor    eax,eax
        clc
        mov    esi,lpBaseAddr
    cal_checksum:
        adc    ax,word ptr [esi]
  inc    esi
  inc    esi
        loop   cal_checksum
        mov    ebx,dwFileSize
        add    eax,ebx
        mov    CheckSum,eax
        popad
        mov    eax,CheckSum 
  ret
CalcPECheckSum endp
********************************************************************
```

## 2. 原始 IAT 的处理

由于原驱动程序被加上了我们的外壳，所以原驱动程序的 IAT 表的填写工作要由我们的外壳程序来完成。应用层壳一般通过 `GetModuleHandle` 和 `GetProcAddress` 两个 API 来完成这个工作，或者自己实现这两个 API 的功能。而驱动壳是要随驱动程序一起被加载到内核当中去的，但内核里没有这两个函数，需要我们自己对这两个函数做内核的实现。当然也可以用 `MmGetSystemRoutineAddress` 函数，不过它只能得到 `ntoskrnl.exe` 和 `hal.dll` 两个模块的函数，对于其它模块则无能为力了，影响壳的通用性。

壳的 `GetModuleHandle` 函数可以通过遍历 `PsLoadedModuleList` 链表来实现，关于遍历这个链表的方法可以参照 Futo 的代码，通过 `DRIVEROBJECT` 的 `DriverSection` 成员来完成，而驱动对象可以从堆栈当中找到。

壳的 `GetProcAddress` 函数的实现就很简单了，内核模块本身也是 PE 文件，直接遍历一下 PE 的导出表就 ok 了。

还有一点需要注意的就是 `UNICODE` 的转换，PE 文件里面的字符串是以 `ASCII` 方式存储的，而内核里的字符串多半是用 `UNICODE` 方式存放的，这点需要注意。

## 3. 节表的处理

在给程序加壳的时候一般都要添加新节，用于存放壳的代码，应用层程序的节表的最后一项和第一个节之间一般是有一个很大的空间可以用来添加新的节表项的，但一般情况下驱动程序节表的最后一项后面紧接着就是第一个节，根本没有足够的 `0x28` 大小的空间存放新的节表项。解决的方法有两种，第一种将所有的节向后移动，而第二种方法则是将 PE 头整体向前移动覆盖掉部分无用的 dos 头，留出足够的空间存放新的节表项。

另外一点需要注意的是，加壳后的驱动程序的每一个节表项必须满足如下两个公式，才能被系统正常加载：

1. `VirtualAddress == PointerToRawData`
2. `SizeOfRawData >= VirtualSize`

至于为什么，笔者也没搞清楚，这只是笔者通过分析驱动加载代码及实验的结果，哪位仁兄知道还请赐教:)

## 4. 重定位表的处理

由于驱动程序是要被加载到内核空间中，所以外壳必须实现原来由系统完成的原驱动程序的重定位工作。原驱动程序的重定位表的处理方法跟应用层 DLL 文件的处理方法完全一样，代码如下：

```asm
  mov  eax,dword ptr [ebp+OriginalRelocateAddr]
  add  eax,dword ptr [ebp+ModuleHandle]
  mov  ecx,dword ptr [ebp+OriginalRelocateSize]
  mov  ebx,eax
  mov  esi,dword ptr [ebp+ModuleHandle]
  sub  esi,dword ptr [ebp+OriginalBaseAddr] ;esi=diff
  NextRelocateBlock:
  .if ecx == 0
    jmp FixAllRelocate
  .endif
  assume ebx : ptr IMAGE_BASE_RELOCATION
  push ecx
  mov  ecx,dword ptr [ebx].SizeOfBlock
  sub  ecx,sizeof IMAGE_BASE_RELOCATION
  shr  ecx,1
  mov  eax,ebx
  add  eax,sizeof IMAGE_BASE_RELOCATION
    NextRelocateEntry:
    xor edi,edi
    mov di,word ptr [eax]
    shr edi,12
    .if edi == IMAGE_REL_BASED_HIGHLOW
      movzx edi,word ptr [eax]
      and edi,0fffh
      add edi,dword ptr [ebx].VirtualAddress
      add edi,dword ptr [ebp+ModuleHandle]
      add dword ptr [edi],esi
    .endif
    add eax,2
    loop NextRelocateEntry
  pop  ecx
  sub  ecx,dword ptr [ebx].SizeOfBlock
  add  ebx,dword ptr [ebx].SizeOfBlock
  jmp NextRelocateBlock
  FixAllRelocate:
```

需要注意的一点：驱动程序被加壳后必须要有重定位表，否则驱动加载会失败，解决的方法需要自己构造一个假的重定位表来替换原始的重定位表。

另外，由于驱动壳的特殊性，`shell` 的编写和驱动程序的编写没什么区别，稍有错误就会蓝屏。

写这篇文章的主要目的是对笔者在编写驱动加壳程序的过程中所遇到的一些问题及其解决方法的总结，避免以后忘了，同时也给想写驱动壳的兄弟们一点我的心得，少走一些弯路。

`WSS(Whitecell Security Systems)`，一个非营利性民间技术组织，致力于各种系统安全技术的研究。坚持传统的 hacker 精神，追求技术的精纯。

`WSS` 主页：`http://www.whitecell.org/`  
`WSS` 论坛：`http://www.whitecell.org/forums/`

`4dd466d0e1e0cc123afbe504da2462ca`
