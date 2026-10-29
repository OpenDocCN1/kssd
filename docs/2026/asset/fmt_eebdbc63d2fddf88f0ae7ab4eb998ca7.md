
# [原创]CVE-2010-3962逆向分析过程

![](https://bbs.kanxue.com/view/img/face/28.gif)第一次在看雪发主题贴。

因水平有限，如分析过程有误，欢迎各位大牛拍砖![](https://bbs.kanxue.com/view/img/face/50.gif)

因为没有系统学习过漏洞挖掘，所以过程和方法可能有些问题，希望各位能对此提出宝贵的建议与意见，感激不尽～～

这个漏洞已经发布有一段时间了，现在假期有空，分析一下当作练手，在看雪已经有大牛发过分析贴，但是看到好多人说看不懂，在此将整个分析过程描述一遍，希望能给大家一些帮助。

## 运行环境

Windows XP SP3

IE 6.0.2900.5512

以下将从逆向的角度，通过对汇编代码的分析，找出漏洞产生原因。

首先从网上下载`CVE-2010-3962`利用程序。在此附上一份下载地址：

[4d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0f1@1x3U0q4Q4x3V1j5`.](https://bbs.kanxue.com/elink@1f8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5M7r3I4G2K9i4c8Q4x3X3c8V1j5W2)9J5k6h3y4G2L8g2)9J5c8X3g2^5M7r3I4G2K9i4c8K6i4K6u0r3x3e0f1@1x3U0q4Q4x3V1j5`.)

获得漏洞利用网页之后，做了简单的测试，发现该程序无法正确执行，但可以正常出发漏洞，如果`shellcode`能正确执行的话，只需在`shellcode`开始处加入`0xcc`，略作调整，就能让程序断下来了。

下面开始分析用`WinDbg`加载`IE`，打开利用程序网页，当程序因出错停下来后查看寄存器和调用堆栈。

```
eax=7e2233f1 ebx=0012e1ec ecx=0176ae50 edx=3fffffff esi=00000000 edi=0176ae50

eip=ed7e27c8 esp=0012e1a0 ebp=0012e1b0 iopl=0         nv up ei pl nz na pe nc

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00010206

ed7e27c8 ??              ???



0:000> k

ChildEBP RetAddr  

0012e19c 7e291a6d 0xed7e27c8

0012e1b0 7e2c60e4 mshtml!CLayout::EnsureDispNodeBackground+0x97

0012e274 7e2c5345 mshtml!CTableLayoutBlock::EnsureTableDispNode+0x388
```

可以看到，是在函数`mshtml!CLayout::EnsureDispNodeBackground`中出现了问题。

查看`mshtml!CLayout::EnsureDispNodeBackground`函数的反汇编

```
mshtml!CLayout::EnsureDispNodeBackground:

……

7e291a66 8b07            mov     eax,dword ptr [edi]	//edi为函数第二个参数，可推测出为一类指针，则eax指向类的虚函数表

7e291a68 8bcf            mov     ecx,edi				//__thiscall,ecx传递this指针

7e291a6a ff5030          call    dword ptr [eax+30h]		//根据偏移调用虚函数

7e291a6d 85c0            test    eax,eax

……
```

可以看出是因为虚函数表被修改而造成的错误，重新运行，在`mshtml!CLayout::EnsureDispNodeBackground`处下断点，确定一下，发现，进入`EnsureDispNodeBackground`函数时，虚函数表中内容都是非法的。

下面就要想办法找到在什么地方修改了对象的虚函数表，继续根据调用堆栈向前查找，通过对`mshtml!CTableLayoutBlock::EnsureTableDispNode`的分析可以发现，`mshtml!CLayout::EnsureDispNodeBackground`中第二个参数是在`EnsureTableDispNode`中生成的，这样就节省了很多力气，只要跟踪`EnsureTableDispNode`中出错对象的改变即可。

经过测试，可以发现在第二次调用`mshtml!CDispNode::SetUserClip`之后，`esi`指向类的前四个字节，即虚函数表地址由`0x7e2233f0`变为`0x7e2233f1`，那么问题就出在`mshtml!CDispNode::SetUserClip`里面了。

`mshtml!CDispNode::SetUserClip`的调用过程如下：

```
7e32a11e 33c0            xor     eax,eax

7e32a120 898564ffffff    mov     dword ptr [ebp-9Ch],eax

7e32a126 898560ffffff    mov     dword ptr [ebp-0A0h],eax

7e32a12c 89855cffffff    mov     dword ptr [ebp-0A4h],eax

7e32a132 898558ffffff    mov     dword ptr [ebp-0A8h],eax

7e32a138 8d8558ffffff    lea     eax,[ebp-0A8h]

7e32a13e 50              push    eax

7e32a13f 8bce            mov     ecx,esi

7e32a141 e8d7410e00      call    mshtml!CDispNode::SetUserClip (7e40e31d)
```

我们看看`mshtml!CDispNode::SetUserClip`都做了些什么：

```
mshtml!CDispNode::SetUserClip:

7e40e31d 8bff            mov     edi,edi

7e40e31f 55              push    ebp

7e40e320 8bec            mov     ebp,esp

7e40e322 83ec10          sub     esp,10h

7e40e325 56              push    esi

7e40e326 57              push    edi

7e40e327 8bf9            mov     edi,ecx	//edi指向类对象指针

7e40e329 8b4704          mov     eax,dword ptr [edi+4]	// 将CDispNode类的第一个成员变量值赋给eax

7e40e32c 2500108800      and     eax,881000h

7e40e331 3d00108000      cmp     eax,801000h

7e40e336 6a0f            push    0Fh

7e40e338 5e              pop     esi		//esi = 0x0f

7e40e339 7548            jne     mshtml!CDispNode::SetUserClip+0x66 (7e40e383)	//jne跳转实现

……

mshtml!CDispNode::SetUserClip+0x66:

7e40e383 804f0704        or      byte ptr [edi+7],4	

7e40e387 6800000008      push    8000000h

7e40e38c 8bcf            mov     ecx,edi

7e40e38e e8dae5e7ff      call    mshtml!CDispNode::SetFlagsToRoot (7e28c96d)	//调用成员函数

7e40e393 ff7508          push    dword ptr [ebp+8]	// SetUserClip的参数

7e40e396 8d4df0          lea     ecx,[ebp-10h]

7e40e399 e8c9bce7ff      call    mshtml!CRect::CRect (7e28a067)

7e40e39e 8d4df0          lea     ecx,[ebp-10h]

7e40e3a1 e8a4e8ffff      call    mshtml!CRect::RestrictRange (7e40cc4a)

7e40e3a6 8b4704          mov     eax,dword ptr [edi+4]    //[edi+4]的值赋给eax

7e40e3a9 23c6            and     eax,esi		//esi == 0x0f，经and运算后，eax值为0

7e40e3ab 0fb688101c217e  movzx   ecx,byte ptr mshtml!CDispNode::_extraSizeTable (7e211c10)[eax] ds:0023:7e211c10=00

 //关键！将mshtml!CDispNode::_extraSizeTable值赋给ecx，其值为0

7e40e3b2 8bc7            mov     eax,edi

7e40e3b4 c1e102          shl     ecx,2		//ecx = ecx * 4 = 0

7e40e3b7 2bc1            sub     eax,ecx		//eax = eax – ecx = eax = edi = this指针

7e40e3b9 830801          or      dword ptr [eax],1	//虚函数表被悲剧了～～

7e40e3bc 8b4704          mov     eax,dword ptr [edi+4]

7e40e3bf 23c6            and     eax,esi

7e40e3c1 0fb688101c217e  movzx   ecx,byte ptr mshtml!CDispNode::_extraSizeTable (7e211c10)[eax]

7e40e3c8 8bc7            mov     eax,edi

7e40e3ca c1e102          shl     ecx,2

7e40e3cd 2bc1            sub     eax,ecx

7e40e3cf 8320fd          and     dword ptr [eax],0FFFFFFFDh	//这里本应该也会影响到虚函数表，但因为[eax]第二位恰好为0，故没造成影响

7e40e3d2 8b4704          mov     eax,dword ptr [edi+4]

7e40e3d5 23c6            and     eax,esi

7e40e3d7 0fb680101c217e  movzx   eax,byte ptr mshtml!CDispNode::_extraSizeTable (7e211c10)[eax]

7e40e3de c1e002          shl     eax,2

7e40e3e1 2bf8            sub     edi,eax

7e40e3e3 83c704          add     edi,4

7e40e3e6 8d75f0          lea     esi,[ebp-10h]

7e40e3e9 a5              movs    dword ptr es:[edi],dword ptr [esi]

7e40e3ea a5              movs    dword ptr es:[edi],dword ptr [esi]

7e40e3eb a5              movs    dword ptr es:[edi],dword ptr [esi]

7e40e3ec a5              movs    dword ptr es:[edi],dword ptr [esi]

7e40e3ed 5f              pop     edi

7e40e3ee 5e              pop     esi

7e40e3ef c9              leave

7e40e3f0 c20400          ret     4
```

通过以上的分析，可以发现，造成错误的关键在于`mshtml!CDispNode::_extraSizeTable`值为0，这个值是在对象实例化的时候进行赋值的，我们在看看对象创建时的情况。

`mshtml!CDispContainer::New`的调用：

```
7e2c604a 33c0            xor     eax,eax

……

7e2c607a 50              push    eax

7e2c607b 8d430c          lea     eax,[ebx+0Ch]

7e2c607e 50              push    eax

7e2c607f e8a9fcfeff      call    mshtml!CDispContainer::New (7e2b5d2d)
```

`mshtml!CDispContainer::New`内容：

```
mshtml!CDispContainer::New:

7e2b5d2d 8bff            mov     edi,edi

7e2b5d2f 55              push    ebp

7e2b5d30 8bec            mov     ebp,esp

7e2b5d32 ff750c          push    dword ptr [ebp+0Ch]	//值为0

7e2b5d35 6a00            push    0	

7e2b5d37 6a48            push    48h

7e2b5d39 e8d1bdfdff      call    mshtml!CDispNode::operator new (7e291b0f)		//调用基类的构造函数，在此跟进

7e2b5d3e 85c0            test    eax,eax

7e2b5d40 7416            je      mshtml!CDispContainer::New+0x29 (7e2b5d58)	//创建不成功则跳转

7e2b5d42 8b4d08          mov     ecx,dword ptr [ebp+8]

7e2b5d45 668148068080    or      word ptr [eax+6],8080h	//注意此处，稍后会加以说明
```

`mshtml!CDispNode::operator new`内容：
