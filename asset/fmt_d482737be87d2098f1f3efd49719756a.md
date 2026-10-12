
# 【加密与解密 第三版】---- 调试篇 --- TraceMe 的学习

> 原文：[`www.pediy.com/kssd/pediy11/117965.html`](https://www.pediy.com/kssd/pediy11/117965.html)

大家自行利用【函数断点】或【堆栈查看分析法】 或者其他的手段,定位到程序的关键位置.

代码:

```
004011CA   .  8A4424 4C     mov     al, byte ptr [esp+4C]                    ;  将用户名的第一个字节复制给 al

004011CE   .  84C0         test    al, al                                   ;  检测是否输入了用户名

004011D0   .  74 76        je      short 00401248

004011D2   .  83FB 05      cmp     ebx, 5                                     ;  将侧用户名的长度是否小于 5

004011D5   .  7C 71        jl      short 00401248

004011D7   .  8D5424 4C    lea     edx, dword ptr [esp+4C]                    ;  提取输入的注册名【edx】

004011DB   .  53          push    ebx                                        ;  用户名长度入栈

004011DC   .  8D8424 A00000>lea     eax, dword ptr [esp+A0]                  ;  提取输入的序列号【eax】

004011E3   .  52          push    edx                                        ;  用户名

004011E4   .  50          push    eax                                        ;  序列号

004011E5   .  E8 56010000  call    00401340                                   ;  用户名与注册码的检测过程【算法过程】

004011EA   .  8B3D BC404000 mov     edi, dword ptr [<&USER32.GetDlgItem>]     ;  USER32.GetDlgItem

004011F0   .  83C4 0C      add     esp, 0C                                    ;  平衡堆栈

004011F3   .  85C0        test    eax, eax

004011F5   .  74 37        je      short 0040122E                             ;  关键跳---【决定注册是否成功】
```

`004011E5   .  E8 56010000   call    00401340                     ;  用户名与注册码的检测过程【算法过程】`

这里的 `call` 就是注册码的计算过程,我们写注册机就是利用这个算法过程.

代码:

```
\=================================================

用户名与序列号的算法过程如下：

00401340  /$  55          push    ebp                                      ;  【算法过程】用户名与注册码的检测过程

00401341  |.  8B6C24 0C   mov     ebp, dword ptr [esp+C]

00401345  |.  56          push    esi

00401346  |.  57          push    edi

00401347  |.  8B7C24 18   mov     edi, dword ptr [esp+18]                  ;  【for(i=3,j=0,i<len,i++,j++)】

0040134B  |.  B9 03000000  mov     ecx, 3                                    ;  i=3,ecx 作为变量 i 使用

00401350  |.  33F6        xor     esi, esi                                   ;  code=0

00401352  |.  33C0        xor     eax, eax                                   ;  j=0,eax 作为 j 使用

00401354  |.  3BF9        cmp     edi, ecx                                     ;  i<len 吗

00401356  |.  7E 21       jle     short 00401379

00401358  |.  53          push    ebx                                      ;  注意这一句与 401378 呼应

00401359  |>  83F8 07      /cmp      eax, 7                                   ;  j>7,j=0

0040135C  |.  7E 02        |jle      short 00401360

0040135E  |.  33C0        |xor      eax, eax                                 ;  防止处理中文时符号扩展

00401360  |>  33D2        |xor      edx, edx                                 ;  清零

00401362  |.  33DB        |xor      ebx, ebx

00401364  |.  8A1429      |mov      dl, byte ptr [ecx+ebp]                    ;  name\[i\]

00401367  |.  8A98 30504000 |mov     bl, byte ptr [eax+405030]                ;  Table\[j\],00405030 地址处放的是数据表

0040136D  |.  0FAFD3      |imul     edx, ebx                                   ;  edx=name\[i\]\*Table\[j\]

00401370  |.  03F2        |add      esi, edx                                   ;  code +=edx

00401372  |.  41          |inc      ecx                                       ;  i++

00401373  |.  40          |inc      eax                                       ;  j++

00401374  |.  3BCF        |cmp      ecx, edi                                   ;  i<len

00401376  |.^ 7C E1       \\\jl       short 00401359

00401378  |.  5B          pop     ebx

00401379  |>  56          push    esi                                      ;  /code

0040137A  |.  68 78504000  push    00405078                                  ;  |Format = "%ld"

0040137F  |.  55          push    ebp                                      ;  |name

00401380  |.  FF15 9C404000 call    dword ptr [<&USER32.wsprintfA>]         ;  \\wsprintfA

00401386  |.  8B4424 1C   mov     eax, dword ptr [esp+1C]                  ;  将参数从堆中传给 eax【序列号 Code 的指针】

0040138A  |.  83C4 0C      add     esp, 0C

0040138D  |.  55          push    ebp                                      ;  /ebp 指向真正的序列号

0040138E  |.  50          push    eax                                      ;  |eax 指向输入的序列号

0040138F  |.  FF15 04404000 call    dword ptr [<&KERNEL32.lstrcmpA>]       ;  \\lstrcmpA

00401395  |.  F7D8        neg     eax

00401397  |.  1BC0        sbb     eax, eax

00401399  |.  5F          pop     edi

0040139A  |.  5E          pop     esi

0040139B  |.  40          inc     eax

0040139C  |.  5D          pop     ebp

0040139D  \\.  C3          retn
```

在这里利用 `KeyMake` 写一个注册机,下面是具体的代码，大家只需将代码保存到扩展名为 `rek` 的文本文件中,然后用 `keyMake` 打开，编译就可以了.

代码:

```
.const



.data

szHomePage db "http://www.pediy.com",0

szEmail    db "mailto:Gall@163.com",0

szErrMess  db "输入的序列号不正确！",0

dataTemp   db 0CH,0AH,13H,09H,0CH,0BH,0AH,08H

szBuffer   db 50 dup(0)

szFmt      db "%ld",0





.code

mov ebp,eax          ; eax 指向的是第一个文本框

invoke lstrlen,ebp   ; 获取输入的字符的长度

mov edi,eax          ; lstrlen 返回的是输入的注册名的长度

mov ecx,3H           ; 注意这里的数值后面需要加 H 标记

xor esi,esi

xor eax,eax

x2:

mov ebx,edi          ; ebx 中存放的是注册名的长度

cmp eax,7H

jle x1

xor eax,eax          ; 上面有一句 push ebx

x1:

xor edx,edx

xor ebx,ebx

mov dl,byte ptr [ecx+ebp]

mov bl,byte ptr [eax+dataTemp]

imul edx,ebx

add esi,edx

inc ecx

inc eax

cmp ecx,edi

jl x2

invoke wsprintf,addr szBuffer,addr szFmt,esi

lea eax,szBuffer
```

注册机见下面的附件。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `KenGen.rar`  

 |
