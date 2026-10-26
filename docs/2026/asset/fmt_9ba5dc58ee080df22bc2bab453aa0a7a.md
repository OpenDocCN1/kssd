
# [原创]MD5算法 - Image To PDF V2.4.0 算法分析

## 文章信息
【文章标题】: MD5算法 - Image To PDF V2.4.0 算法分析

【文章作者】: Suyana

【作者邮箱】: Suyasha@163.com

【作者QQ号】: 517949855(请注明来自看雪论坛)

【软件名称】: Image To PDF V2.4.0

【下载地址】: 自己搜索下载

【加壳方式】: ASPack 2.12

【保护方式】: 注册码

【编写语言】: Borland C++ 1999

【使用工具】: OD, ASPackDie, DeDe

【操作平台】: WinXP

【软件介绍】: 一款不错的PDF制作工具，可以批量地将TIF, JPG, GIF

【作者声明】: 我只是一只小菜鸟，失误之处难免，敬望诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

软件用`ASPack 2.12`加壳，用`AspackDie`脱壳就可以了。用

`Borland C++ 1999`写的，一开始用`DeDe`分析一下：`"TDlgReg" => "Button2Click"` 来到：

```
0040D788   55                     push    ebp
...
0040D7FA   8BC3                   mov     eax, ebx
\* Reference to : TDlgError.\_PROC\_0040DBE0()
|
0040D7FC   E8DF030000             call    0040DBE0    ==>进入
0040D801   837DF800                 cmp     dword ptr [ebp-$08], +$00
```

--------------------------

```
0040DBE0    push    ebx
...
0040DC12    mov     esi, 004DD7F7            ; ASCII "PX4VUTE8Q1YONML6GIHJZFSDCBA9R7K5W32"
...                                          ; 计算注册码要用到的表
0040DC33    pop     ecx
0040DC34    push    eax                      ; /Arg3
0040DC35    push    ebx                      ; |Arg2
0040DC36    lea     edx, [esp+34]        ; |
0040DC3A    push    edx                      ; |
0040DC3B    call    0040EA20               ; \Arg1 ;MD5算法初始化常数
0040DC40    add     esp, 0C
0040DC43    lea     ecx, [esp+2C]
0040DC47    push    ecx                      ; /Arg2
0040DC48    lea     eax, [esp+88]       ; |
0040DC4F    push    eax                      ; |Arg1
0040DC50    call    0040EAB8               ; \Md5算法)，计算出一串值(Md5的值)
0040DC55    add     esp, 8
0040DC58    xor     edi, edi
0040DC5A    mov     esi, ebp
0040DC5C    lea     ebx, [esp+84]
0040DC63     /xor     eax, eax
0040DC65     |mov     al, [ebx]              ; 计算后那串值(Md5的值)
0040DC67     |mov     ecx, 23
0040DC6C     |cdq
0040DC6D     |idiv    ecx                        ; 除以23h
0040DC6F     |xor     eax, eax
0040DC71     |mov     al, [esp+edx+8]        ; 取余数查表，得出注册码
0040DC75     |push    eax                           ; /<%c>
0040DC76     |push    004DD992                ; |Format = "%c"
0040DC7B     |lea     edx, [esp+8]              ; |
0040DC7F     |push    edx                           ; |s
0040DC80     |call    <jmp.&USER32.wsprintfA>; \wsprintfA
0040DC85     |add     esp, 0C
0040DC88     |mov     cl, [esp]
0040DC8B     |mov     [esi], cl
0040DC8D     |inc     edi
0040DC8E     |inc     esi
0040DC8F     |inc     ebx
0040DC90     |cmp     edi, 10                       ;注册码为16位，最后再加上?
0040DC93     \jl      short 0040DC63
0040DC95     mov     byte ptr [ebp+11], 0
0040DC99     add     esp, 98
0040DC9F     pop     ebp
```

现在说一下算法吧，其实很简单，就是把e-mail用`md5`加密,用算出的值依此取ASCII除以`23h`，取余数 查表就可以了，注册码再加上`?`

如：

e-mail:

`suyasha@163.com`

regcode:

`1L5UTAOREQ8E4LGT?`

注册码保存在`setup.ini`文件

一开始并不知道软件用MD5算法，害我跟进MD5算法中，白忙了好久。

上面说到的初始化常数就是填充：

`01 23 45 67 89 AB CD EF FE DC BA 98 76 54 32 10`，因为是高位在后，低位在前，

所以就是`67452301`、`EFCDAB89`、`BADCFEEF`、`10325476`，(有这四个常数基本上就是Md5算法)

--------------------------------------------------------------------------------

## 经验总结

1. 知道了`peid`的一个插件`Krypto ANALYzer`可以查看加密算法。
2. 懂得怎样看MD5算法，主要是看那4个常数，当然这4个常数是可以改的。

--------------------------------------------------------------------------------

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

------------------------------------------------------------------

文章写于2007-08-12

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
