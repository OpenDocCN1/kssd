
`; 0024a9b0  00000001 03570000 00000400 00000001`

`; 0024a9c0  00000001 03580000 00000800 00000000`

`; 0024a9d0  00000001 035a0000 00001000 00000000`

`; 0024a9e0  00000001 03e50000 00002000 00000000`

..............如果大小不对则

`loc_619391D1` :           ; 实际分配的大小+8

```
lea     eax, [edi+8]
push    eax             ; dwBytes
push    8               ; dwFlags
push    dword ptr [esi+4Ch] ;hHeap
call    ds:HeapAlloc(x,x,x)
```
------------------------------上面的`hHeap`,这个堆句柄,是在类初始化的时候调用下面代码

```
call    ds:GetProcessHeap()
push    10h
mov     ecx, esi
mov     [esi+4Ch], eax        ;得到的!
```

通过上面的分析得出结论:当我们传入的`npoints*16`, `(npoints-1)*8`, `npoints*4`后的大小大于`2000h`最终分配内存执行的代码如下:

`HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, size+8)`

而`npoints*16`的代码实现如下

```
mov     eax, ebx        ;ebx是传入的第一个参数
shl     eax, 4          ;左移4位,*16
push    eax
call    operator new(uint)
```

很明显整数溢出了.

这时设置传入参数为`0x40000801`那么分配的空间大小为

`buf1=0x8018`

`buf2=0x4008`

`KeyFrameArray.field_C=0x200c`

`TimeFrameArray.field_C=0x200c`

如果分配的大小都不为空,则调用

```
// We set an error and go to the cleanup code if the KeyFrameArray array
// is smaller than npoints*2 or TimeFrameArray is smaller than npoints-1
    if ( KeyFrameArrayAccessor.ToDoubleArray(npoints*2, buf_1) < 0 ||
        TimeFrameArrayAccessor.ToDoubleArray(npoints-1, buf_2) < 0)
    {
        err = E_FAIL;
        goto cleanup;
    }
```

又因为我们调用`KeyFrame`时传入的后两个参数很小,所以`npoints*2`是肯定大于数组大小的,所以直接跳到`cleanup`.

`cleanup`中代码会挨个检查2个较小的`buffer`中的每一个`DWORD`，如果不是`NULL`的话，就把它当成对象指针，并调用对象的虚函数。

```
for (i = 0; i < npoints; i++) {
            if (KeyFrameArray.field_C[i] != NULL)
                KeyFrameArray.field_C[i]->func_8();
            ......
}
```

由于这2个`buffer`在分配时是使用了`HEAP_ZERO_MEMORY`参数的，所以`buffer`中是写满了`NULL`的。但问题是检查的范围是0到`npoints`（也就是`0x40000801`），所以这个检查会越界，程序会把`buffer`后面的内存中的数据也当成是`buffer`中的数据进行检查。即他会访问`0x200c`之后的数据。我们前面讨论过了，我们能控制`KeyFrameArray.field_C`这个`buffer`之后的一个`DWORD`。我们把它写成指向`lookaside`表中某一项的一个指针，这样当异常处理代码把这个非`NULL`的`DWORD`当成一个对象的指针，并调用这个对象的`func_8()`虚函数时，我们的`shel​​lcode`就运行了。

```
.text:100071DA loc_100071DA:                           ; CODE XREF: CPathCtl::KeyFrame(uint,tagVARIANT,tagVARIANT)+2D4j
.text:100071DA                 mov     eax, [ebp+arg_4]
.text:100071DA
.text:100071DD
.text:100071DD loc_100071DD:                           ; CODE XREF: CPathCtl::KeyFrame(uint,tagVARIANT,tagVARIANT)+2AFj
.text:100071DD                 add     eax, esi
.text:100071DF                 cmp     dword ptr [eax], 0  ;判断是否为NULL
.text:100071E2                 jz      short loc_100071EC
.text:100071E2
.text:100071E4                 mov     eax, [eax]          ;不为0,把这个DWORD当成对象指针
.text:100071E6                 mov     ecx, [eax]
.text:100071E8                 push    eax
.text:100071E9                 call    dword ptr [ecx+8]   ;执行对象的第三个虚函数
.text:100071E9
.text:100071EC
.text:100071EC loc_100071EC:                           ; CODE XREF: CPathCtl::KeyFrame(uint,tagVARIANT,tagVARIANT)+2B9j
.text:100071EC                 cmp     dword ptr [esi], 0
.text:100071EF                 jz      short loc_100071F9
.text:100071EF
.text:100071F1                 mov     eax, [esi]
.text:100071F3                 mov     ecx, [eax]
.text:100071F5                 push    eax
.text:100071F6                 call    dword ptr [ecx+8]
.text:100071F6
.text:100071F9
.text:100071F9 loc_100071F9:                           ; CODE XREF: CPathCtl::KeyFrame(uint,tagVARIANT,tagVARIANT)+2C6j
.text:100071F9                 add     esi, 4
.text:100071FC                 dec     ebx
.text:100071FD                 jnz     short loc_100071DA
```

### 如何利用漏洞:

`048a0040`

`048a2058`

`2018h`,包括`ENTRY`头实际数据`2010H`,我们就是在`200c`后的4字节要设置为`00151e58`

### 7.3 XPSP2+IE7下MS06067漏洞的利用

环境搭建:

找了台`XPSP2+IE7`,查看`06067`的公告发现`IE7`下无洞,通过去掉注册表中的`KILLBIT`和重新注册控件触发成功.

`danim.dll(6.3.1.146)`

`daxctle.ocx(6.3.1.146)`,注意`ms`的公告中有时把这个控件的名字写成了`daxctrle.ocx`.

`lmrt.dll(6.3.1.146)`

`mmutilse.dll(6.3.1.146)`

    未完....侍续....

附:

for LFH on win7 to find the size there's a complex xor:

`eax = address of headers (i.e., app addr - 8)`

if top bit of `eax+0x7` is set then does the +0x50 cookie xor

else `(*(((eax>>3) ^ (1st-header-dword) ^ *ntdll!RtlpLFHKey ^ heapbase) + 0x10) << 3)`

      - `*(eax+0x7) & 0x3f`

`ntdll!RtlSizeHeap+0xa4`:

```
770a46e9 8a4807           mov     cl,[eax+0x7]
770a46ec 80f904           cmp     cl,0x4
770a46ef 0f8481530500     je      ntdll!RtlSizeHeap+0xac (770f9a76)
```
  I haven't explored this side but apparently some other types of blocks
  as well:

  `ntdll!RtlSizeHeap+0xac`:

```
770f9a76 50               push    eax
770f9a77 56               push    esi
770f9a78 e881ba0200       call    ntdll!RtlpGetSizeOfBigBlock (771254fe)
770f9a7d e9c1acfaff       jmp     ntdll!RtlSizeHeap+0x167 (770a4743)
```
`ntdll!RtlSizeHeap+0xb8`:

```
770a46f5 57               push    edi
770a46f6 84c9             test    cl,cl
770a46f8 794f             jns     ntdll!RtlSizeHeap+0xd4 (770a4749)
```
`ntdll!RtlSizeHeap+0xbd`:

```
770a46fa 8b10             mov     edx,[eax]
770a46fc 8bc8             mov     ecx,eax
770a46fe c1e903           shr     ecx,0x3
770a4701 33ca             xor     ecx,edx
770a4703 330da4001777     xor     ecx,[ntdll!RtlpLFHKey (771700a4)]
770a4709 33ce             xor     ecx,esi
770a470b 668b4910         mov     cx,[ecx+0x10]
770a470f 0fb7f9           movzx   edi,cx
```
`ntdll!RtlSizeHeap+0x15f`:

```
770a473b 8bc7             mov     eax,edi
770a473d c1e003           shl     eax,0x3
770a4740 2bc1             sub     eax,ecx
770a4742 5f               pop     edi
```
for now bailing and having extra padding size be assumed to be just request

size aligned forward to chunk alignment

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
