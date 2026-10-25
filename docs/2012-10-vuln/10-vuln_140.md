
# CVE-2010-3962 逆向分析过程

> 原文：[`www.pediy.com/kssd/pediy12/128813.html`](https://www.pediy.com/kssd/pediy12/128813.html)

![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")第一次在看雪发主题贴。

因水平有限，如分析过程有误，欢迎各位大牛拍砖![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

因为没有系统学习过漏洞挖掘，所以过程和方法可能有些问题，希望各位能对此提出宝贵的建议与意见，感激不尽～～

这个漏洞已经发布有一段时间了，现在假期有空，分析一下当作练手，在看雪已经有大牛发过分析贴，但是看到好多人说看不懂，在此将整个分析过程描述一遍，希望能给大家一些帮助。

运行环境：

Windows XP SP3

IE 6.0.2900.5512

以下将从逆向的角度，通过对汇编代码的分析，找出漏洞产生原因。

首先从网上下载 CVE-2010-3962 利用程序。在此附上一份下载地址：

[`www.exploit-db.com/exploits/15421/`](http://www.exploit-db.com/exploits/15421/)

获得漏洞利用网页之后，做了简单的测试，发现该程序无法正确执行，但可以正常出发漏洞，如果 shellcode 能正确执行的话，只需在 shellcode 开始处加入 `0xcc`，略作调整，就能让程序断下来了。

下面开始分析用 `WinDbg` 加载 `IE`，打开利用程序网页，当程序因出错停下来后查看寄存器和调用堆栈。

```
eax=7e2233f1 ebx=0012e1ec ecx=0176ae50 edx=3fffffff esi=00000000 edi=0176ae50
eip=ed7e27c8 esp=0012e1a0 ebp=0012e1b0 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00010206
ed7e27c8 ??              ???

0:000> k
ChildEBP RetAddr  
0012e19c 7e291a6d 0xed7e27c8
0012e1b0 7e2c60e4 mshtml!CLayout::EnsureDispNodeBackground+0x97
0012e274 7e2c5345 mshtml!CTableLayoutBlock::EnsureTableDispNode+0x388
```

可以看到，是在函数 `mshtml!CLayout::EnsureDispNodeBackground` 中出现了问题。

查看 `mshtml!CLayout::EnsureDispNodeBackground` 函数的反汇编

```
mshtml!CLayout::EnsureDispNodeBackground:
……
7e291a66 8b07            mov     eax,dword ptr \[edi\]  //edi 为函数第二个参数，可推测出为一类指针，则 eax 指向类的虚函数表
7e291a68 8bcf            mov     ecx,edi        //\_\_thiscall,ecx 传递 this 指针
7e291a6a ff5030          call    dword ptr \[eax+30h\]    //根据偏移调用虚函数
7e291a6d 85c0            test    eax,eax
……
```

可以看出是因为虚函数表被修改而造成的错误，重新运行，在 `mshtml!CLayout::EnsureDispNodeBackground` 处下断点，确定一下，发现，进入 `EnsureDispNodeBackground` 函数时，虚函数表中内容都是非法的。

下面就要想办法找到在什么地方修改了对象的虚函数表，继续根据调用堆栈向前查找，通过对 `mshtml!CTableLayoutBlock::EnsureTableDispNode` 的分析可以发现，`mshtml!CLayout::EnsureDispNodeBackground` 中第二个参数是在 `EnsureTableDispNode` 中生成的，这样就节省了很多力气，只要跟踪 `EnsureTableDispNode` 中出错对象的改变即可。

经过测试，可以发现在第二次调用 `mshtml!CDispNode::SetUserClip` 之后，`esi` 指向类的前四个字节，即虚函数表地址由 `0x7e2233f0` 变为 `0x7e2233f1`，那么问题就出在 `mshtml!CDispNode::SetUserClip` 里面了。

`mshtml!CDispNode::SetUserClip` 的调用过程如下：

```
7e32a11e 33c0            xor     eax,eax
7e32a120 898564ffffff    mov     dword ptr \[ebp-9Ch\],eax
7e32a126 898560ffffff    mov     dword ptr \[ebp-0A0h\],eax
7e32a12c 89855cffffff    mov     dword ptr \[ebp-0A4h\],eax
7e32a132 898558ffffff    mov     dword ptr \[ebp-0A8h\],eax
7e32a138 8d8558ffffff    lea     eax,\[ebp-0A8h\]
7e32a13e 50              push    eax
7e32a13f 8bce            mov     ecx,esi
7e32a141 e8d7410e00      call    mshtml!CDispNode::SetUserClip (7e40e31d)
```

我们看看 `mshtml!CDispNode::SetUserClip` 都做了些什么：

```
mshtml!CDispNode::SetUserClip:
7e40e31d 8bff            mov     edi,edi
7e40e31f 55              push    ebp
7e40e320 8bec            mov     ebp,esp
7e40e322 83ec10          sub     esp,10h
7e40e325 56              push    esi
7e40e326 57              push    edi
7e40e327 8bf9            mov     edi,ecx  //edi 指向类对象指针
7e40e329 8b4704          mov     eax,dword ptr \[edi+4\]  // 将 CDispNode 类的第一个成员变量值赋给 eax
7e40e32c 2500108800      and     eax,881000h
7e40e331 3d00108000      cmp     eax,801000h
7e40e336 6a0f            push    0Fh
7e40e338 5e              pop     esi    //esi = 0x0f
7e40e339 7548            jne     mshtml!CDispNode::SetUserClip+0x66 (7e40e383)  //jne 跳转实现
……
mshtml!CDispNode::SetUserClip+0x66:
7e40e383 804f0704        or      byte ptr \[edi+7\],4  
7e40e387 6800000008      push    8000000h
7e40e38c 8bcf            mov     ecx,edi
7e40e38e e8dae5e7ff      call    mshtml!CDispNode::SetFlagsToRoot (7e28c96d)  //调用成员函数
7e40e393 ff7508          push    dword ptr \[ebp+8\]  // SetUserClip 的参数
7e40e396 8d4df0          lea     ecx,\[ebp-10h\]
7e40e399 e8c9bce7ff      call    mshtml!CRect::CRect (7e28a067)
7e40e39e 8d4df0          lea     ecx,\[ebp-10h\]
7e40e3a1 e8a4e8ffff      call    mshtml!CRect::RestrictRange (7e40cc4a)
7e40e3a6 8b4704          mov     eax,dword ptr \[edi+4\]    //\[edi+4\]的值赋给 eax
7e40e3a9 23c6            and     eax,esi    //esi == 0x0f，经 and 运算后，eax 值为 0
7e40e3ab 0fb688101c217e  movzx   ecx,byte ptr mshtml!CDispNode::\_extraSizeTable (7e211c10)\[eax\] ds:0023:7e211c10=00
 //关键！将 mshtml!CDispNode::\_extraSizeTable 值赋给 ecx，其值为 0
7e40e3b2 8bc7            mov     eax,edi
7e40e3b4 c1e102          shl     ecx,2    //ecx = ecx \* 4 = 0
7e40e3b7 2bc1            sub     eax,ecx    //eax = eax  ecx = eax = edi = this 指针
7e40e3b9 830801          or      dword ptr \[eax\],1  //虚函数表被悲剧了～～
7e40e3bc 8b4704          mov     eax,dword ptr \[edi+4\]
7e40e3bf 23c6            and     eax,esi
7e40e3c1 0fb688101c217e  movzx   ecx,byte ptr mshtml!CDispNode::\_extraSizeTable (7e211c10)\[eax\]
7e40e3c8 8bc7            mov     eax,edi
7e40e3ca c1e102          shl     ecx,2
7e40e3cd 2bc1            sub     eax,ecx
7e40e3cf 8320fd          and     dword ptr \[eax\],0FFFFFFFDh  //这里本应该也会影响到虚函数表，但因为\[eax\]第二位恰好为 0，故没造成影响
7e40e3d2 8b4704          mov     eax,dword ptr \[edi+4\]
7e40e3d5 23c6            and     eax,esi
7e40e3d7 0fb680101c217e  movzx   eax,byte ptr mshtml!CDispNode::\_extraSizeTable (7e211c10)\[eax\]
7e40e3de c1e002          shl     eax,2
7e40e3e1 2bf8            sub     edi,eax
7e40e3e3 83c704          add     edi,4
7e40e3e6 8d75f0          lea     esi,\[ebp-10h\]
7e40e3e9 a5              movs    dword ptr es:\[edi\],dword ptr \[esi\]
7e40e3ea a5              movs    dword ptr es:\[edi\],dword ptr \[esi\]
7e40e3eb a5              movs    dword ptr es:\[edi\],dword ptr \[esi\]
7e40e3ec a5              movs    dword ptr es:\[edi\],dword ptr \[esi\]
7e40e3ed 5f              pop     edi
7e40e3ee 5e              pop     esi
7e40e3ef c9              leave
7e40e3f0 c20400          ret     4
```

通过以上的分析，可以发现，造成错误的关键在于 `mshtml!CDispNode::\_extraSizeTable` 值为 `0`，这个值是在对象实例化的时候进行赋值的，我们在看看对象创建时的情况。

`mshtml!CDispContainer::New` 的调用：

```
7e2c604a 33c0            xor     eax,eax
……
7e2c607a 50              push    eax
7e2c607b 8d430c          lea     eax,\[ebx+0Ch\]
7e2c607e 50              push    eax
7e2c607f e8a9fcfeff      call    mshtml!CDispContainer::New (7e2b5d2d)
```

`mshtml!CDispContainer::New` 内容：

```
mshtml!CDispContainer::New:
7e2b5d2d 8bff            mov     edi,edi
7e2b5d2f 55              push    ebp
7e2b5d30 8bec            mov     ebp,esp
7e2b5d32 ff750c          push    dword ptr \[ebp+0Ch\]  //值为 0
7e2b5d35 6a00            push    0  
7e2b5d37 6a48            push    48h
7e2b5d39 e8d1bdfdff      call    mshtml!CDispNode::operator new (7e291b0f)    //调用基类的构造函数，在此跟进
7e2b5d3e 85c0            test    eax,eax
7e2b5d40 7416            je      mshtml!CDispContainer::New+0x29 (7e2b5d58)  //创建不成功则跳转
7e2b5d42 8b4d08          mov     ecx,dword ptr \[ebp+8\]
7e2b5d45 668148068080    or      word ptr \[eax+6\],8080h  //注意此处，稍后会加以说明
```

`mshtml!CDispNode::operator new` 内容：



```
mshtml!CDispNode::operator new:  //三个参数的值依次为 0x48 , 0x00 , 0x00
7e291b0f 8bff            mov     edi,edi
7e291b11 55              push    ebp
7e291b12 8bec            mov     ebp,esp
7e291b14 51              push    ecx
7e291b15 51              push    ecx
7e291b16 8b4508          mov     eax,dword ptr [ebp+8]    //eax 为 0x48
7e291b19 53              push    ebx
7e291b1a 8b5d10          mov     ebx,dword ptr [ebp+10h]  //ebx 为 0x00
7e291b1d 56              push    esi
7e291b1e 0fb6b3101c217e  movzx   esi,byte ptr mshtml!CDispNode::_extraSizeTable (7e211c10)[ebx]  //关键！esi 为 0
7e291b25 c1e602          shl     esi,2    //esi = esi * 4 = 0
7e291b28 57              push    edi
7e291b29 03c6            add     eax,esi  //eax = eax + esi = eax = 0x48
7e291b2b 50              push    eax
7e291b2c e87563feff      call    mshtml!_MemAllocClear (7e277ea6)    //分配内存
7e291b31 8bf8            mov     edi,eax  //分配内存首地址
7e291b33 85ff            test    edi,edi
7e291b35 740e            je      mshtml!CDispNode::operator new+0x47 (7e291b45)  //分配不成功则跳转

mshtml!CDispNode::operator new+0x28:
7e291b37 03fe            add     edi,esi  //edi = edi + esi = edi
7e291b39 f6c340          test    bl,40h
7e291b3c 895f04          mov     dword ptr [edi+4],ebx  //将_extraSizeTable 值赋给[edi+4]
7e291b3f 0f85391b0a00    jne     mshtml!CDispNode::operator new+0x32 (7e33367e)
……
```

由以上代码可以看到，在构造函数中，默认地使用了 `0x00` 作为 `_extraSizeTable` 的大小，并将其值放入 `[edi+4]` 处，及类首地址便宜为 `0x04` 处，此时再看

代码:

```
7e2b5d45 668148068080    or      word ptr [eax+6],8080h  //注意此处，稍后会加以说明
```

`[eax+4]` 处为 `__extraSizeTable` 的大小，而 `[eax+6]` 处为 `0x8080`，可以猜测 `_extraSizeTable` 的数据大小应该为 2 字节，但事实是否如此，在最后还有一个说法。

通过以上的分析，可以看到，在创建 `CdispContainer` 的时候，因为使用默认的 0 作为 `_extraSizeTable` 的大小，从而未分配内存，在其后的 `mshtml!CDispNode::SetUserClip` 函数中，需要对 `_extraSizeTable` 进行修改，而没有进行严格的检验，从而造成了漏洞的产生。

以上就是从逆向的角度分析的该漏洞的成因，至于原理方面，在看雪已经有大牛发过该漏洞的分析贴，在此附上链接，不再赘述：

[`bbs.pediy.com/showthread.php?t=125122`](http://bbs.pediy.com/showthread.php?t=125122)

可能因为环境测试环境不同，造成分析结果会有一些差异，根据个人的调试过程，在此对 llydd 大大的分析提出一点质疑：

代码:

```
.text:7E36B4C4　CDispNode::SetUserClip
...............................
.text:7E36B54D                 mov     eax, [edi+4]　<-------当这里为 0 时,虚函数表就被悲剧了.
.text:7E36B550                 and     eax, esi
.text:7E36B552       movzx   ecx, ds:uchar const * const CDispNode::_extraSizeTable[eax]
.text:7E36B559                 mov     eax, edi
.text:7E36B55B                 shl     ecx, 2
.text:7E36B55E                 sub     eax, ecx
.text:7E36B560                 or      dword ptr [eax], 1   <-----------------------虚函数表被改了
```

由前面的分析可以发现 `[edi+4]` 本身不为 0，其值为 `0x8c800000`，`esi` 为 `0x0000000f`，经过

代码:

```
7e40e3a9 23c6            and     eax,esi
```

运算之后，`eax` 值才为 0，事实上，由 `esi` 的值应该可以确定，`_extraSizeTable` 的数据大小只有四位。
