
```bash
0:000> kb
ChildEBP RetAddr Args to Child
WARNING: Stack unwind information not available. Following frames may be wrong.
00123dd0 30f4cdbd 00123f3c 00000000 ffffffff mso!Ordinal6426+0x64d
00123e00 30f4a597 00123f88 00123f3c 00000000 mso!Ordinal753+0x306e
0012404c 30d4b199 00000000 0012408c 00000000 mso!Ordinal753+0x848
00124074 30d4b148 30d4ae32 014914c8 01491500 mso!Ordinal4196+0x61f
00124078 30d4ae32 014914c8 01491500 014913b0 mso!Ordinal4196+0x5ce
0012407c 014914c8 01491500 014913b0 30dc9d44 mso!Ordinal4196+0x2b8
00124080 01491500 014913b0 30dc9d44 00000000 0x14914c8
00124084 014913b0 30dc9d44 00000000 01491118 0x1491500
00124088 30dc9d44 00000000 01491118 00124e38 0x14913b0
0012408c 00000000 01491118 00124e38 00000000 mso!Ordinal2940+0x158fc
```

由于缺少symbol文件，所以这个栈帧是有可能出错的，不过这里还是可以分析出来的。

第一个是当前函数栈帧，要看下一行`00123e00 30f4a597 00123f88 00123f3c 00000000 mso!Ordinal753+0x306e`，用`ub`指令查看指定地址之前的汇编代码：

```bash
0:000> ub mso!Ordinal753+0x306e
mso!Ordinal753+0x305a:
30f4cda9 23c1 and eax,ecx
30f4cdab 50 push eax
30f4cdac 8d47ff lea eax,[edi-1]
30f4cdaf 50 push eax
30f4cdb0 8b4508 mov eax,dword ptr [ebp+8]
30f4cdb3 6a00 push 0
30f4cdb5 ff750c push dword ptr [ebp+0Ch]
30f4cdb8 e8a0feffff call mso!Ordinal753+0x2f0e (30f4cc5d)
```

漏洞代码出现在函数`0x30F4CC5D`调用之中，所以要再一次载入程序，在`0x30F4CC5D`下断点，然后继续执行

```bash
0:004> bp 0x30F4CC5D
*** ERROR: Symbol file could not be found. Defaulted to export symbols for C:\Program Files\Common Files\Microsoft Shared\office11\mso.dll -
0:004> g
ModLoad: 77b40000 77b62000 C:\WINDOWS\system32\appHelp.dll
ModLoad: 77a20000 77a74000 C:\WINDOWS\System32\cscui.dll
ModLoad: 76600000 7661d000 C:\WINDOWS\System32\CSCDLL.dll
ModLoad: 5b860000 5b8b5000 C:\WINDOWS\system32\netapi32.dll
ModLoad: 76990000 769b5000 C:\WINDOWS\system32\ntshrui.dll
ModLoad: 76b20000 76b31000 C:\WINDOWS\system32\ATL.DLL
ModLoad: 769c0000 76a74000 C:\WINDOWS\system32\USERENV.dll
ModLoad: 75f80000 7607d000 C:\WINDOWS\system32\browseui.dll
ModLoad: 7e290000 7e401000 C:\WINDOWS\system32\shdocvw.dll
ModLoad: 754d0000 75550000 C:\WINDOWS\system32\CRYPTUI.dll
ModLoad: 76c30000 76c5e000 C:\WINDOWS\system32\WINTRUST.dll
ModLoad: 76c90000 76cb8000 C:\WINDOWS\system32\IMAGEHLP.dll
ModLoad: 76f60000 76f8c000 C:\WINDOWS\system32\WLDAP32.dll
ModLoad: 76980000 76988000 C:\WINDOWS\system32\LINKINFO.dll
ModLoad: 36c30000 36c39000 C:\Program Files\Microsoft Office\OFFICE11\msostyle.dll
ModLoad: 39800000 399b3000 C:\Program Files\Microsoft Office\OFFICE11\GdiPlus.DLL
ModLoad: 76f50000 76f58000 C:\WINDOWS\system32\WTSAPI32.DLL
ModLoad: 76360000 76370000 C:\WINDOWS\system32\WINSTA.dll
Breakpoint 0 hit
eax=00123f88 ebx=00000000 ecx=00123dfc edx=00000000 esi=00000000 edi=00000000
eip=30f4cc5d esp=00123dd4 ebp=00123e00 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
mso!Ordinal753+0x2f0e:
30f4cc5d 55 push ebp
```

漏洞代码的地址`x30e9eb88`，之后使用`F10`单步调试，寻找位于该地址附近的函数调用：

一直单步到这里，有一个函数调用，函数地址是`0x30E9EB62`，和漏洞代码地址很接近，应该就是这里了，`F11`步入：

```bash
0:000> p
eax=30d9ed10 ebx=05000000 ecx=00123dc0 edx=00000000 esi=014910f0 edi=00123f88
eip=30f4cc93 esp=00123da4 ebp=00123dd0 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
mso!Ordinal753+0x2f44:
30f4cc93 ff501c call dword ptr [eax+1Ch] ds:0023:30d9ed2c=30e9eb62
```

看一下这里的汇编代码：

```bash
0:000> uf eip
mso!Ordinal6426+0x627:
30e9eb62 57 push edi
30e9eb63 8b7c240c mov edi,dword ptr [esp+0Ch]
30e9eb67 85ff test edi,edi
30e9eb69 7427 je mso!Ordinal6426+0x657 (30e9eb92)
mso!Ordinal6426+0x630:
30e9eb6b 8b442408 mov eax,dword ptr [esp+8]
30e9eb6f 8b4808 mov ecx,dword ptr [eax+8]
30e9eb72 81e1ffff0000 and ecx,0FFFFh
30e9eb78 56 push esi
30e9eb79 8bf1 mov esi,ecx
30e9eb7b 0faf742414 imul esi,dword ptr [esp+14h]
30e9eb80 037010 add esi,dword ptr [eax+10h]
30e9eb83 8bc1 mov eax,ecx
30e9eb85 c1e902 shr ecx,2
30e9eb88 f3a5 rep movs dword ptr es:[edi],dword ptr [esi]
```

需要注意`esi`、`edi`、`ecx`这三个寄存器的值，

```bash
0:000> p
eax=30d9ed10 ebx=05000000 ecx=00123dc0 edx=00000000 esi=014910f0 edi=00123f88
eip=30e9eb63 esp=00123d9c ebp=00123dd0 iopl=0 nv up ei pl zr na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000246
mso!Ordinal6426+0x628:
30e9eb63 8b7c240c mov edi,dword ptr [esp+0Ch] ss:0023:00123da8=00123dc0
0:000> db edi-4
00123dbc a8 5c 59 64 ea 80 fb 3f-00 00 00 05 00 00 00 00 .\Yd...?........
00123dcc 06 40 00 00 00 3e 12 00-bd cd f4 30 3c 3f 12 00 .@...>.....0<?..
00123ddc 00 00 00 00 ff ff ff ff-00 00 00 00 e0 14 49 01 ..............I.
00123dec 20 44 12 00 8c 40 12 00-38 4e 12 00 b0 40 12 00 D...@..8N...@..
00123dfc 00 00 00 00 e4 3f 12 00-97 a5 f4 30 88 3f 12 00 .....?.....0.?..
00123e0c 3c 3f 12 00 00 00 00 00-e0 14 49 01 8c 40 12 00 <?........I..@..
00123e1c 20 44 12 00 00 00 00 00-ff ff ff ff ff ff ff ff D..............
00123e2c ff ff ff ff 00 00 00 00-00 00 00 20 01 01 00 00 ........... ....
```

所以数据要复制到`0x123dc0`。

```bash
0:000> p
eax=014910f0 ebx=05000000 ecx=00123dc0 edx=00000000 esi=014910f0 edi=00123dc0
eip=30e9eb6f esp=00123d9c ebp=00123dd0 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
mso!Ordinal6426+0x634:
30e9eb6f 8b4808 mov ecx,dword ptr [eax+8] ds:0023:014910f8=0004c8ac
0:000> p
eax=014910f0 ebx=05000000 ecx=0004c8ac edx=00000000 esi=014910f0 edi=00123dc0
eip=30e9eb72 esp=00123d9c ebp=00123dd0 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
mso!Ordinal6426+0x637:
30e9eb72 81e1ffff0000 and ecx,0FFFFh
0:000> p
eax=0000c8ac ebx=05000000 ecx=0000c8ac edx=00000000 esi=1104000c edi=00123dc0
eip=30e9eb85 esp=00123d98 ebp=00123dd0 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
mso!Ordinal6426+0x64a:
30e9eb85 c1e902 shr ecx,2
```

这里经过第一步运算后`ecx`的值是`0xC8AC`，然后进行了右移两位（除以4）的操作，因为`rep movs dword`是以四个字节为一个单位进行复制的。

```bash
0:000> p
eax=014910f0 ebx=05000000 ecx=0000c8ac edx=00000000 esi=00000000 edi=00123dc0
eip=30e9eb80 esp=00123d98 ebp=00123dd0 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
mso!Ordinal6426+0x645:
30e9eb80 037010 add esi,dword ptr [eax+10h] ds:0023:01491100=1104000c
0:000> db esi
1104000c 41 61 30 41 61 31 41 61-32 41 61 33 41 61 34 41 Aa0Aa1Aa2Aa3Aa4A
1104001c 61 35 41 61 36 41 61 37-41 61 38 41 61 39 41 62 a5Aa6Aa7Aa8Aa9Ab
1104002c 30 41 62 31 41 62 32 41-62 33 41 62 34 41 62 35 0Ab1Ab2Ab3Ab4Ab5
1104003c 41 62 36 41 62 37 41 62-38 41 62 39 41 63 30 41 Ab6Ab7Ab8Ab9Ac0A
1104004c 63 31 41 63 32 41 63 33-41 63 34 41 63 35 41 63 c1Ac2Ac3Ac4Ac5Ac
1104005c 36 41 63 37 41 63 38 41-63 39 41 64 30 41 64 31 6Ac7Ac8Ac9Ad0Ad1
1104006c 41 64 32 41 64 33 41 64-34 41 64 35 41 64 36 41 Ad2Ad3Ad4Ad5Ad6A
1104007c 64 37 41 64 38 41 64 39-41 65 30 41 65 31 41 65 d7Ad8Ad9Ae0Ae1Ae
```

源数据所在地址为`0x1104000C`。

注意在上面我查看`edi`处的数据时，使用的是`db edi-4`的命令，可以看到`\Yd`已经在栈中了，所以这三个字符和数据的复制操作没什么关系，而`ecx`的值`0xC8AC`正是`msf.rtf`中保存的值，所以这两个字节用来保存`pFragments`的长度。

```bash
0000h: 11 11 11 11 AC C8 41 61 30 41 61 31 41 61 32 41
```

查看一下目前的寄存器情况：

```bash
0:000> r
eax=0000c8ac ebx=05000000 ecx=0000322b edx=00000000 esi=1104000c edi=00123dc0
eip=30e9eb88 esp=00123d98 ebp=00123dd0 iopl=0 nv up ei pl nz na pe nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00000206
mso!Ordinal6426+0x64d:
30e9eb88 f3a5 rep movs dword ptr es:[edi],dword ptr [esi]
```

数据复制的目标地址`edi`是`0x123dc0`，而此时栈帧基址`ebp`是`0x123dd0`，两者仅相差16个字节。所以如果想要覆盖返回地址，一共需要24个字节。

## 3.3 小总结

上面的分析流程和之前分析CVE-2010-2883还是有很大区别的，之前分析的poc文件是一个可以成功进行漏洞利用并攻击的样本，里面包含了shellcode。而这次则是从一个引发异常的样本出发，分析出漏洞位置以及如何进行漏洞利用。

分析到这里，已经可以看出这是一个很典型的栈溢出漏洞，可以把返回地址覆盖成`jmp esp`的地址，然后把shellcode放在`pFragments`中，实现漏洞利用。

# 4. 构造漏洞利用的poc文件

想要构造POC文件，还需要进行一些静态分析，因为在执行完数据的复制之后，还要执行一些代码才能到达覆盖的返回地址处，如果只是简单的把返回地址和shellcode组合在一起，在到达返回地址之前，会首先触发异常。

*注：我之所以能确定不能简单组合是因为自己试了一下，由于栈中原本的数据被覆盖，导致后面的函数执行出现异常，所以需要确定哪部分栈中的数据需要进行特殊处理。*

根据一开始的错误信息：

```bash
mso!Ordinal6426+0x64d:
30e9eb88 f3a5 rep movs dword ptr es:[edi],dword ptr [esi]
```

使用`lm`指令显示模块信息（因为电脑了有两个`mso.dll`，所以这一步还是很有必要的）：
