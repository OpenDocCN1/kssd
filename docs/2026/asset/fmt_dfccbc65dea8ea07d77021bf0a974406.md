
`mshtml!CDispNode::operator new`：

```
// 三个参数的值依次为0x48 , 0x00 , 0x00
7e291b0f 8bff            mov     edi,edi
7e291b11 55              push    ebp
7e291b12 8bec            mov     ebp,esp
7e291b14 51              push    ecx
7e291b15 51              push    ecx
7e291b16 8b4508          mov     eax,dword ptr [ebp+8]		//eax为0x48
7e291b19 53              push    ebx
7e291b1a 8b5d10          mov     ebx,dword ptr [ebp+10h]	//ebx为0x00
7e291b1d 56              push    esi
7e291b1e 0fb6b3101c217e  movzx   esi,byte ptr mshtml!CDispNode::_extraSizeTable (7e211c10)[ebx]	//关键！esi为0
7e291b25 c1e602          shl     esi,2		//esi = esi * 4 = 0
7e291b28 57              push    edi
7e291b29 03c6            add     eax,esi	//eax = eax + esi = eax = 0x48
7e291b2b 50              push    eax
7e291b2c e87563feff      call    mshtml!_MemAllocClear (7e277ea6)		//分配内存
7e291b31 8bf8            mov     edi,eax	//分配内存首地址
7e291b33 85ff            test    edi,edi
7e291b35 740e            je      mshtml!CDispNode::operator new+0x47 (7e291b45)	//分配不成功则跳转
```

```
mshtml!CDispNode::operator new+0x28:
7e291b37 03fe            add     edi,esi	//edi = edi + esi = edi
7e291b39 f6c340          test    bl,40h
7e291b3c 895f04          mov     dword ptr [edi+4],ebx	//将_extraSizeTable值赋给[edi+4]
7e291b3f 0f85391b0a00    jne     mshtml!CDispNode::operator new+0x32 (7e33367e)
……
```

由以上代码可以看到，在构造函数中，默认地使用了`0x00`作为`_extraSizeTable`的大小，并将其值放入`[edi+4]`处，及类首地址便宜为`0x04`处，此时再看

```
7e2b5d45 668148068080    or      word ptr [eax+6],8080h	//注意此处，稍后会加以说明
```

`[eax+4]`处为`__extraSizeTable`的大小，而`[eax+6]`处为`0x8080`，可以猜测`_extraSizeTable`的数据大小应该为2字节，但事实是否如此，在最后还有一个说法。

通过以上的分析，可以看到，在创建`CdispContainer`的时候，因为使用默认的`0`作为`_extraSizeTable`的大小，从而未分配内存，在其后的`mshtml!CDispNode::SetUserClip`函数中，需要对`_extraSizeTable`进行修改，而没有进行严格的检验，从而造成了漏洞的产生。

以上就是从逆向的角度分析的该漏洞的成因，至于原理方面，在看雪已经有大牛发过该漏洞的分析贴，在此附上链接，不再赘述：

<http://bbs.pediy.com/showthread.php?t=125122>

可能因为环境测试环境不同，造成分析结果会有一些差异，根据个人的调试过程，在此对`llydd`大大的分析提出一点质疑：

```
.text:7E36B4C4　CDispNode::SetUserClip
...............................

.text:7E36B54D                 mov     eax, [edi+4]　<-------当这里为0时,虚函数表就被悲剧了.
.text:7E36B550                 and     eax, esi
.text:7E36B552       movzx   ecx, ds:uchar const * const CDispNode::_extraSizeTable[eax]
.text:7E36B559                 mov     eax, edi
.text:7E36B55B                 shl     ecx, 2
.text:7E36B55E                 sub     eax, ecx
.text:7E36B560                 or      dword ptr [eax], 1   <-----------------------虚函数表被改了
```

由前面的分析可以发现`[edi+4]`本身不为0，其值为`0x8c800000`，`esi`为`0x0000000f`，经过

```
7e40e3a9 23c6            and     eax,esi
```

运算之后，`eax`值才为0，事实上，由`esi`的值应该可以确定，`_extraSizeTable`的数据大小只有四位。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
