
# x64 gs寄存器的一点资料

编写shellcode很重要的一步就是搜索`kernel32`的基址, 毕竟我们要获取API的地址。

x86下众所周知的`fs`寄存器搜索`kernel32`基址的办法, 在x64下已经失效了。x64下`fs`的角色已经换成了`gs`。

暂时发现存储的一些感兴趣的东西, 如果有其他的, 希望能告知我。![](https://bbs.kanxue.com/view/img/face/1.gif)

- `gs:[0x30]` — `TEB`
- `gs:[0x40]` — Pid
- `gs:[0x48]` — Tid
- `gs:[0x60]` — `PEB`
- `gs:[0x68]` — `LastError`

值得一提的是, 虽然`gs:[0x60]`直接存放的是`PEB`, 但是由于vista/7后的地址随机化机制, 还是从`TEB`获取比较靠谱。

```
0:009> dt 000007fffff98000 \_TEB
ntdll!\_TEB
+0×000 NtTib            : \_NT\_TIB
+0×038 EnvironmentPointer : (null)
+0×040 ClientId         : \_CLIENT\_ID
+0×050 ActiveRpcHandle  : (null)
+0×058 ThreadLocalStoragePointer : (null)
+0×060 ProcessEnvironmentBlock : 0x000007ff`fffd5000 \_PEB    //这里即是PEB
```

```c
//用c语言描述就是
#define x64\_GetPeb()                    ( (LONG64\*)(\*((LONG64\*)((BYTE\*)x64\_GetTeb()+0×060))) )
```

得到了`PEB`, 剩下的就和x86下一样了, 只是偏移不一样了。

```
0:009> dt 0x000007ff`fffd5000 \_PEB
ntdll!\_PEB
+0×000 InheritedAddressSpace : 0 ”
+0×001 ReadImageFileExecOptions : 0 ”
+0×002 BeingDebugged    : 0×1 ”
+0×003 BitField         : 0×8 ”
+0×003 ImageUsesLargePages : 0y0
+0×003 IsProtectedProcess : 0y0
+0×003 IsLegacyProcess  : 0y0
+0×003 IsImageDynamicallyRelocated : 0y1
+0×003 SkipPatchingUser32Forwarders : 0y0
+0×003 SpareBits        : 0y000
+0×008 Mutant           : 0xffffffff`ffffffff Void
+0×010 ImageBaseAddress : 0×00000000`ff310000 Void
+0×018 Ldr              : 0×00000000`77222640 \_PEB\_LDR\_DATA
```

```
0:009> dt 0×00000000`77222640 \_PEB\_LDR\_DATA
ntdll!\_PEB\_LDR\_DATA
+0×000 Length           : 0×58
+0×004 Initialized      : 0×1 ”
+0×008 SsHandle         : (null)
+0×010 InLoadOrderModuleList : \_LIST\_ENTRY [ 0x00000000`00202780 - 0x2421b0 ]
+0×020 InMemoryOrderModuleList : \_LIST\_ENTRY [ 0x00000000`00202790 - 0x2421c0 ]
+0×030 InInitializationOrderModuleList : \_LIST\_ENTRY [ 0x00000000`00202890 - 0x2421d0 ]
```

同样的, 搜索`InInitializationOrderModuleList`即可得到`Kernel32`的基址。

由于x64下vs2005没法直接使用内联汇编, 所以只把必须使用汇编来做的事情写成单独的`asm`。

x64下, 指针的长度已经是8个字节, 所以偏移不一样了。

```
0:009> dt \_LDR\_DATA\_TABLE\_ENTRY
ntdll!\_LDR\_DATA\_TABLE\_ENTRY
+0×000 InLoadOrderLinks : \_LIST\_ENTRY
+0×010 InMemoryOrderLinks : \_LIST\_ENTRY
+0×020 InInitializationOrderLinks : \_LIST\_ENTRY
+0×030 DllBase          : Ptr64 Void
+0×038 EntryPoint       : Ptr64 Void
+0×040 SizeOfImage      : Uint4B
+0×048 FullDllName      : \_UNICODE\_STRING
```

C语言表述x64下这个搜索的过程

![](https://bbs.kanxue.com/upload/attach/201201/258689_s89r8smk9cgshlo.jpg)

win7 x64下测试通过.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Snap2.jpg](javascript:void(0);)
  （37.42kb，383次下载）
