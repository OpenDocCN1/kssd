
# pi 过 win7 实现方法

> 原文：[`www.pediy.com/kssd/pediy12/135497.html`](https://www.pediy.com/kssd/pediy12/135497.html)

pi 的 shellcode 在 win7 下无法正常运行，正好前段时间在网上看到一篇关于 win7 下 shellcode 编写的文章，提到了 `kernel32.dll` 地址的查找问题，所以就实践了一下，全当加深理解，大牛飘过![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

思路很简单因为以前查找 `kernel32.dll` 的地址方法在 win7 下失效，所以用新的代码替换之。

简单介绍一下两种查找 `kernel32.dll` 的方法。

## 老方法

`find kernel32.dll`

```
find_kernel32:
push esi
xor eax, eax
mov eax, [fs:eax+0x30]
test eax, eax
js find_kernel32_9x ;win9x or nt
find_kernel32_nt:
mov eax, [eax + 0x0c]
mov esi, [eax + 0x1c]
lodsd
mov eax, [eax + 0x8]
jmp find_kernel32_finished
find_kernel32_9x:
mov eax, [eax + 0x34]
lea eax, [eax + 0x7c]
mov eax, [eax + 0x3c]
find_kernel32_finished:
pop esi
ret
```

具体什么原理就不多言了，详细请看《Win 7 下定位 kernel32.dll 基址及 shellcode 编写》http://bbs.pediy.com/showthread.php?t=122260，这个方法在 win7 上无效

## 新的方法

```
00401000 51 push ecx
00401001 56 push esi
00401002 57 push edi
00401003 33C9 xor ecx,ecx
00401005 64:8B71 30 mov esi,dword ptr fs:[ecx+30]
00401009 8B76 0C mov esi,dword ptr ds:[esi+C]
0040100C 8B76 1C mov esi,dword ptr ds:[esi+1C]
0040100F 8B46 08 mov eax,dword ptr ds:[esi+8]
00401012 8B7E 20 mov edi,dword ptr ds:[esi+20]
00401015 8B36 mov esi,dword ptr ds:[esi]
00401017 66:394F 18 cmp word ptr ds:[edi+18],cx
0040101B ^ 75 F2 jnz short KrnlBase.0040100F
0040101D 5F pop edi
0040101E 5E pop esi
0040101F 59 pop ecx
00401020 C3 retn
```

这个方法是通用版本![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")，呵呵，所以 xp，win7 通吃。

下面就是查找了：

有两个思路

第一就是直接在 shellcode 里面搜索代码

`0xF6, 0x5F, 0x64, 0xA1, 0x30, 0x00, 0x00, 0x00, 0x8B, 0x40, 0x0C, 0x8B, 0x70, 0x1C, 0xAD, 0xFF, 0x70, 0x08`

然后修改之。

第二呢就是 od 跟踪，两个方法都可以，殊途同归啊，不过思路还是很重要的，多多益善。

顺利查找到代码后就是改代码了，用新的代码替换老的。

`0xe8, 0xdd, 0x04, 0x00, 0x00, 0x50, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90` 红色部分需要修改因为 shellcode 的长度不同，会影响它的值，这个值大概就是与新代码的距离。

新的 shellcode `0x51, 0x56, 0x57, 0x33, 0xc9,0x64, 0x8B,0x71, 0x30, 0x8B, 0x76,0x0C, 0x8B, 0x76, 0x1C, 0x8B, 0x46, 0x08, 0x8B, 0x7E, 0x20, 0x8B, 0x36, 0x66, 0x39,0x4F, 0x18, 0x75, 0xF2, 0x5f,0x5e, 0x59, 0xc3`

修改好后，运行程序，成功！

楼主的 shellcode 不通用,通过比较字符串感觉不太好,在 2K3 下直接就死....

```
0:000> !peb
PEB at 7ffde000
    InheritedAddressSpace:    No
    ReadImageFileExecOptions: No
    BeingDebugged:            Yes
    ImageBaseAddress:         01000000
    Ldr                       00191ea0
    Ldr.Initialized:          Yes
    Ldr.InInitializationOrderModuleList: 00191f58 . 001921a0
    Ldr.InLoadOrderModuleList:           00191ee0 . 00192190
    Ldr.InMemoryOrderModuleList:         00191ee8 . 00192198
            Base  TimeStamp                     Module
         1000000 49a5f699 Feb 26 09:55:37 2009 E:\\Program Files\\Debugging Tools for Windows (x86)\\pdbcopy.exe
        7c920000 4d00f280 Dec 09 23:15:12 2010 C:\\WINDOWS\\system32\\ntdll.dll
        7c800000 49c4f481 Mar 21 22:06:57 2009 C:\\WINDOWS\\system32\\kernel32.dll
        77be0000 4802be3f Apr 14 10:15:27 2008 C:\\WINDOWS\\system32\\msvcrt.dll
        77bd0000 4802bdbf Apr 14 10:13:19 2008 C:\\WINDOWS\\system32\\VERSION.dll
```

下面的代码能在 XPSP3/WIN7 下取到,2K3 下没环境，未测试，有条件的朋友可以测试下.

code：

```
xor eax,eax
mov eax,fs:[eax+30h]
mov eax,[eax+0ch]
mov esi,[eax+0ch]
lodsd
mov eax, [eax]
mov eax,[eax+18h]    ;Kernel.dll 的基地址
```
