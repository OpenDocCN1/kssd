
# [虚伪]slv unpackme 脱壳 & 去校验

其实壳本身不要紧，问题是vm里面有个校验。

sm同学手下留情，我勉强能搞一个运行正常的，没精力还原vm了。

在`virutalfree`的`retn`上`f4`，直到`[esp]`是一个`exe image`内的地址`f7`返回:

```
0040FA91    B8 BE180000     mov     eax, 18BE
0040FA96    BA 00004000     mov     edx, slv_unpa.00400000
0040FA9B    03C2            add     eax, edx
0040FA9D  - FFE0            jmp     eax
```

在`jmp eax`上`f4 f7`到`oep`:

```
004018BE    68 EA1AF500     push    0F51AEA
004018C3  - E9 EF01B500     jmp     00F51AB7
004018C8    CC              int3
004018C9    CC              int3
```

`jmp`到`vm`了，不过这段内存不可`dump`，`ctrl f2`再来看哪里分配的:

`bp virtualalloc`，不停的`ctrl f9`看到`eax=00F50000`停下:

```
0040FBB6    6A 40           push    40
0040FBB8    68 00100000     push    1000
0040FBBD    50              push    eax
0040FBBE    6A 00           push    0
0040FBC0    FF13            call    [ebx]                            ; kernel32.VirtualAlloc
0040FBC2    8BD0            mov     edx, eax
0040FBC4    8BFA            mov     edi, edx
0040FBC6    8B4E FC         mov     ecx, [esi-4]
0040FBC9    F3:A4           rep     movsb
```

往下走`ecx=000151E9`，复制一个`unpackme.exe`到`2.exe`，增加一个`section header`:

```
vaddr=0x0022000
vsize=0x20000
```

加载`2.exe`在`0040FBC2`上`f4`把`eax`改成`422000`.

然后按上面到方法走到`oep`，用`lordpe` `dump`一份(`ollydump`我从来不成功) `dumped.exe`

接着用`imprec` `fixdump` 生成 `3.exe`

一运行非法了，点调试挂上`od`:

```
00422CCA    8910            mov     [eax], edx//eax=00140688
00422CCC    E9 0D0E0000     jmp     00423ADE
```

向上看全是屁股...啊不对，全是花指令，不过只有 `EB`，`nop`掉:

```
00422CA4    33C0            xor     eax, eax
00422CAC    AC              lodsb
00422CB1    8B1487          mov     edx, [edi+eax*4]
00422CB7    33C0            xor     eax, eax
00422CBD    AC              lodsb
00422CC1    8B0487          mov     eax, [edi+eax*4]
00422CCA    8910            mov     [eax], edx
00422CCC    E9 0D0E0000     jmp     00423ADE
```

看样子只是一个虚拟机指令，好像丢掉东西了.

校验是哪里来的?

想法一：`GetFileSize`，估计sm同学不屑使用，怀着侥幸试一下果然没有用.

想法二：在某处设置了标记，前面`virtualalloc`都被释放掉了，`vm`段我们也`dump`了，能在哪呢?

联想到sm同学一贯喜欢在`pe header`里面插东西，就看看`pe header`

加载`2.exe`在`oep`的时候按`alt m`在`pe header`上`f2`果然中断了访问`[400110]==1000`

加载`3.exe`也中断了，`[400110]==18BE`

原来校验了`entrypoint`，估计后面当作常数运算了，因为所有`kbys`都会把`1000`作为入口.

解决方案有两种，一种写一段入口代码把入口改为`1000`，另一种是挪动，因为写保护不方便，我选择挪动.

`401000` `ctrl r`找到两处参考，都改`401005`

```
004011DA    E8 21FEFFFF     call    3.00401000
004011EC    E8 0FFEFFFF     call    3.00401000
```

修改这里的代码:

```
00401000 > /E9 B9080000     jmp     4.004018BE
00401005   |68 EA9C4200     push    4.00429CEA
0040100A  -|E9 A82A0200     jmp     4.00423AB7
```

保存到文件，最后`lordpe`修改`1000`为入口，保存`4.exe`，嗯出图片了.

不过点了出来`she`之后不会关掉自己反而弹出自己，估计是虚拟机里有调用`1000`退出，

结果跳到`18be`又`DialogParam`了

那么只好选择写保护了，用`lordpe`添加一个输入函数`VirutalProtect`

在`oep`下面找片空地写代码，最后跳回`18be`

```
004018C8 >  B8 00004000     mov     eax, 00400000
004018CD    0340 3C         add     eax, [eax+3C]
004018D0    8D78 28         lea     edi, [eax+28]
004018D3    50              push    eax
004018D4    54              push    esp
004018D5    6A 04           push    4
004018D7    6A 04           push    4
004018D9    57              push    edi
004018DA    FF15 1E304400   call    [44301E]                         ; kernel32.VirtualProtect
004018E0    58              pop     eax
004018E1    B8 00100000     mov     eax, 1000
004018E6    AB              stosd
004018E7  ^ EB D5           jmp     004018BE
```

把入口改成`18c8`，搞定收工.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [SLV_UNPACKME.rar](javascript:void(0);)

  （227.19kb，59次下载）
