
# 猛男测试，挑战 fsg 2.0 脱壳

> 原文：[`www.pediy.com/kssd/pediy11/116704.html`](https://www.pediy.com/kssd/pediy11/116704.html)

目标：脱掉外层的 `fsg` 其实不顶用，有几个函数被混淆了，找出来他们还原

比如其中某一个函数

代码：

```
0040332D   $  55            push    ebp
0040332E   .  8BEC         mov     ebp, esp
00403330   .  56            push    esi
00403331   .  57            push    edi
00403332   .  53            push    ebx
00403333   .  EB 39         jmp     short unpacked.0040336E
00403335       B8            db      B8
00403336   .^ 78 ED         js      short unpacked.00403325
00403338   >  E8 01000000   call    unpacked.0040333E
0040333D       3B            db      3B                            ;  CHAR ';'
0040333E   .  8D6424 04     lea     esp, \[esp+4\]
00403342   .  97            xchg    eax, edi
00403343   .  F2:          prefix repne:                       ;  Superfluous prefix
00403344   .  EB 02         jmp     short unpacked.00403348
00403346       A6            db      A6
00403347       83            db      83
00403348   >  E9 B6170000   jmp     unpacked.00404B03
0040334D       BD            db      BD
0040334E       16            db      16
0040334F       BB            db      BB
00403350   >  F3:          prefix rep:                          ;  Superfluous prefix
00403351   .  E8 01000000   call    unpacked.00403357
00403356   .  A7            cmps
00403357   $  8D6424 04     lea     esp, \[esp+4\]
```

要还原成可读的代码，例如：

代码：

```
0040332D   $  55            push    ebp
0040332E   .  8BEC         mov     ebp, esp
00403330   .  56            push    esi
00403331   .  57            push    edi
00403332   .  53            push    ebx
00403333   .  68 00001000   push    100000
00403338   .  68 21AF4000   push    0040AF21
0040333D   .  E8 E0F7FFFF   call    00402B22
00403342   .  2BC0          sub     eax, eax
00403344   .  8905 5DAF4000 mov     \[40AF5D\], eax
0040334A   .  8905 71AF4000 mov     \[40AF71\], eax
00403350   .  C7C6 A1AE4000 mov     esi, 0040AEA1
00403356   .  FFB5 08000000 push    dword ptr \[ebp+8\]
0040335C   .  E8 1BF7FFFF   call    00402A7C
00403361   .  40            inc     eax
00403362   .  0F84 C10B0000 je      00403F29
```

![](img/962324edb48d57effae0c0571850a66b.jpg "酷")

不要贴中间结果，不是重点，要可以运行的 `bin`。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `fsg2.0.rar`

`JCC` 不知道有什么好办法处理 不跳的话 下面碰到往上的 `JMP` 就完了

只会人肉记住标签 ![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")

批量处理 只有模拟分支吗 ? ![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

代码：

```
<0040332D>

@L00000001:
  push ebp
  mov ebp,esp
  push esi
  push edi
  push ebx
  push 100000
  push 40AF21
  call 00402B22
  xor eax,eax
  mov dword ptr \[40AF5D\],eax
  mov dword ptr \[40AF71\],eax
  mov esi,40AEA1

@L00000002:
  push dword ptr \[ebp+8\]
  call 00402A7C
  inc eax
  je @L00000003
  test ecx,ecx
  je @L00000004
  dec eax
  mov dword ptr \[esi+8\],eax
  mov dword ptr \[esi+C\],ecx
  push eax
  push ecx
  call 00402AB1
  test eax,eax
  je @l1
  test ecx,ecx
  je @l1
  mov dword ptr \[esi\],eax
  jmp short @L00000002

@L00000003:
  push 40A21D
  jmp @L00000007

@L00000004:
  push 40A235
  jmp @L00000005
@l1:
  push 40A252
  jmp @L00000005
@L00000005:
  mov ebx,40AEA1
  push dword ptr \[ebx\]
  push dword ptr \[ebx+4\]
  call 00402AD3
  xor eax,eax

@L00000006:
  push eax
  push eax
  push dword ptr \[ebx+C\]
  push dword ptr \[ebx+8\]
  call 00407284
  push dword ptr \[ebx+8\]
  call 00407278
  push dword ptr \[ebx+8\]
  call 00407230

@L00000007:
  mov ebx,402B6B
  push 40AF49
  call ebx
  push 40AF35
  call ebx

@L00000008:
  push 40AF5D
  call ebx
  push 40AF71
  call ebx
  push 40AF21
  call ebx
  pop eax
  pop ebx
  pop edi
  pop esi
  leave
  retn 0C
```

引用: 最初由 `kunkun` 发布 ![查看帖子](img/showthread.php?p=837442#post837442) ![](img/962324edb48d57effae0c0571850a66b.jpg "酷")

`JCC` 不知道有什么好办法处理 不跳的话 下面碰到往上的 `JMP` 就完了

只会人肉记住标签 ![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")

批量处理 只有模拟分支吗 ? ![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

`<0040332D>`

`@L00000001:`

  `push ebp`

  `mov ebp,esp`

  `push esi`

  `push...`

不够长啊。不过这题我想考的是拓扑排序。

代码：
