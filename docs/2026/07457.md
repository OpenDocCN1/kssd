
# [原创]重载内核之二接管流程

重载内核的相关文章实在是太多了，鉴于还是有很多初学者研究这一块，本文仅作为一个引导作用，文笔不好，见谅。

我的博客：`aafK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7$3W2V1P5h3S2W2`

开发环境：`VS2010` + `WinDDK`

测试环境：`VirtualDDK` + `VMware` + `Win7 sp1 x86`

第一部分链接：`http://bbs.pediy.com/showthread.php?t=187863`

## 第二部分：接管流程

在第一部分完成之后，已经重载了一份新的`NT`内核，接下来就是接管执行流程。

自`Windows XP`以后从`RING3`进`RING0`统一调用`sysenter`汇编指令，但仍保留了`INT 2E`中断，在`sysenter`调用之后会执行内核空间中的`KiFastCallEntry`函数，这个函数负责所有的应用层请求，即使是`INT 2E`也会走这个函数，而`KiFastCallEntry`在经过简单的处理后会通过`KeServiceDescriptorTable`或者`KeServiceDescriptorTableShadow`来获取内核函数指针并调用，重载内核的目的就是为了在执行内核函数的时候转移到新的模块上，从而不被其他`HOOK`所影响。那么需要做的就是在调用内核函数的时候转移指令，但并不是所有内核函数都由`ntoskrnl`实现，内核中`GUI`函数却是由`win32k.sys`实现，由于目前只重载了`ntoskrnl`，所以只能做到转移`SSDT`中的函数。额外说明一下，即使`NT`内核有不同的版本，但所有`NT`内核的导出模块名称都是`ntoskrnl.exe`（详见输出表），所以我所说的`ntoskrnl`泛指`NT`内核，而不是具体的文件名。

在`KiFastCallEntry`中调用内核函数的地方大家很容易找到资料，最先是谁提出的我记不清了，反正现在是被某出名软件所使用，下面`HOOK`的地方与其他软件可能会冲突，所以在测试的时候需要保证没有安装任何安全软件。

用户态程序调用内核函数的流程简述为：用户态->`ntdll.ZwApi`->`sysenter`->内核态->`KiFastCallEntry`->`ServiceTable`->`ServiceRoutine`。`ServiceTable`分为两种，上面已经提到了，接下来要做的就是在调用`SSDT`中函数的时候转移到新模块上，在`WRK`中可以找到这样的代码：

```
        mov     esi, edx                ; (esi)->User arguments
        mov     ebx, [edi]+SdNumber     ; get argument table address
        xor     ecx, ecx
        mov     cl, byte ptr [ebx+eax]  ; (ecx) = argument size
        mov     edi, [edi]+SdBase       ; get service table address
        mov     ebx, [edi+eax*4]        ; (ebx)-> service routine
        sub     esp, ecx                ; allocate space for arguments
        shr     ecx, 2                  ; (ecx) = number of argument DWORDs
        mov     edi, esp                ; (edi)->location to receive 1st arg
        //省略部分代码
        call    ebx                     ; call system service
```

显然，最后一句的`call`指令就是调用了内核函数，而需要`hook`的地方就是`sub esp, ecx`这一句，此时，`edi`指向`ServiceTable`，`eax`为函数索引，`ebx`为函数地址，而且这一句连同下面的两句指令共`7`字节，可以容纳一个`JMP`/`CALL`指令，绝佳的`HOOK`点。至于这个地址怎么定位，就要用字节码搜索了，实在没有什么好办法，但总比直接写地址或定偏移的那种硬编码好。

```
PVOID __declspec(naked) _GetKiFastCallEntryAddress()
{
  __asm
  {
    MOV  ECX, 0x00000176;
    RDMSR;
    RETN;
  }
}
PVOID FindHookKiFastCallEntryAddress(PVOID lpKiFastCallEntry)
{
  /*
  sub    esp, ecx
  shr    ecx, 2
  mov    edi, esp
  */
  UCHAR HookBytes[] = {0x2B, 0xE1, 0xC1, 0xE9, 0x02, 0x8B, 0xFC};

  return RtlFindMemory(lpKiFastCallEntry, 0x300, HookBytes, sizeof(HookBytes));
}
```

通过`MSR`寄存器获取`KiFastCallEntry`的函数地址，有兴趣的朋友可以翻阅`WRK`中系统初始化部分的代码。`HOOK`函数这里采用了普通的`MDL`方式来写入只读内存，可能会有人问了，在写入地址的一瞬间别的`CPU`执行到了这里怎么办，我个人认为是杞人忧天，现在的`CPU`都有缓存机制，也就意味着内存和`CPU`缓存未必同步，在你写入的一瞬间即使别的`CPU`执行到了这里，那也是缓存中的代码，和内存中的并不一致，有兴趣的朋友可以查找`CPU`的`TLB`、`TIB`的资料。当然这只是降低了写入与执行冲突的几率，并不代表完全没有可能蓝屏，但话说回来，你要修改一个可执行指令，不可避免的有几率冲突，这是完全无法避免的，所以尽可能是降低几率就可以了。

```
VOID __declspec(naked) _MyKiFastCallEntryFrame()
{
  __asm
  {
    pop  edx;  //save ret address
    sub  esp, ecx;
    push edx;  //restore ret address
    mov  edx, dword ptr [edi + eax * 0x04];

    shr  ecx, 2;
    mov  edi, esp;
    add  edi, 0x04;
    retn;
  }
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT lpDriverObject, IN PUNICODE_STRING lpRegPath)
{
  PVOID lpHookKiFastCallEntryAddress;
  UCHAR HookCode[7];

  DbgPrint("Driver Load.\n");
  InitializePsLoadedModuleList(lpDriverObject);
  g_lpNewNtoskrnlAddress = ReloadNtModule(PsLoadedModuleList);
  lpHookKiFastCallEntryAddress = FindHookKiFastCallEntryAddress(GetKiFastCallEntryAddress());
  HookCode[0] = 0xE8;
  *(ULONG*)&HookCode[1] = (ULONG_PTR)_MyKiFastCallEntryFrame - (ULONG_PTR)lpHookKiFastCallEntryAddress - 5;
  HookCode[5] = 0x90;
  HookCode[6] = 0x90;
  RtlCopyMemoryEx(lpHookKiFastCallEntryAddress, HookCode, sizeof(HookCode));
  return STATUS_SUCCESS;
}
```

编译并测试，通过`ARK`查看效果。注意！`_MyKiFastCallEntryFrame`是在`Windows 7`中测试的，`XP`下并不通用，因为执行上下文并不一样，`XP`中是`mov ebx, dword ptr [edi + eax * 0x04]`而`Windows 7`是`mov edx, dword ptr [edi + eax * 0x04]`，这里我就不做兼容性写法了，手里也没有`XP`的虚拟机。`XP`已经停止服务了，相信没有多少人打算再使用了，如果非要兼容`XP`，自己做下简单的修改即可，即把`edx`换成`ebx`。

![](https://bbs.kanxue.com/upload/attach/201405/625093_5zutplyffe9d5od.jpg)

测试一段时间后并没有蓝屏现象，说明`HOOK`成功，但是目前并没有写任何过滤内容，下面就来丰富一下过滤函数，即所有的`SSDT`都走新内核。并继续考虑兼容`XP`。

```
PVOID ServiceCallFilter(PVOID *lppServiceTableBase, ULONG_PTR ServiceIndex)
{
  if (lppServiceTableBase == KeServiceDescriptorTable->ServiceTableBase)
  {
    if (ServiceIndex == 190)
    {
      DbgPrint("Call NtOpenProcess\n");
    }
    return (ULONG_PTR)lppServiceTableBase[ServiceIndex] - (ULONG_PTR)g_lpNtoskrnlAddress + (ULONG_PTR)g_lpNewNtoskrnlAddress;
  }
  return lppServiceTableBase[ServiceIndex];
}

VOID __declspec(naked) _MyKiFastCallEntryFrame()
{
  __asm
  {
    pop  ebx;  //save ret address
    sub  esp, ecx;
    push ebx;  //restore ret address

    push eax;
    push ecx;
    push edx;

    push eax;
    push edi;
    call ServiceCallFilter;
    mov  ebx, eax;

    pop  edx;
    pop  ecx;
    pop  eax;
    mov  edx, ebx;

    shr  ecx, 2;
    mov  edi, esp;
    add  edi, 0x04;
    retn;
  }
}
```

由于仅测试效果，我就暂时写`190`，这是`NtOpenProcess`的函数索引。

![](https://bbs.kanxue.com/upload/attach/201405/625093_phv1ycu2omd1j0c.jpg)

至此所有的`SSDT`中的函数已全部转移到新模块中。但这还只是一个半成品，仅做测试使用，因为后面还需要改动很多。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [001.jpg](javascript:void(0);)

  （38.84kb，5次下载）

- [002.jpg](javascript:void(0);)

  （43.15kb，4次下载）

- [2.重载内核之接管流程.doc](javascript:void(0);)

  （69.00kb，234次下载）
