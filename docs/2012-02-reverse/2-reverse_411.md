
# 浅谈在分析壳时 IDC 的使用

> 原文：[`www.pediy.com/kssd/pediy06/pediy6389.htm`](https://www.pediy.com/kssd/pediy06/pediy6389.htm)

高手莫笑

似乎很少 `ida` 的帖子,呵呵

抛个砖头引些玉米吧

以下为 `linson` 兄弟的壳开头的一段代码,花指令不算多,我就不整理了

```
XJ:004040BC                 mov     ecx, 0A88h

XJ:004040C1                 lea     edi, byte_401E5A[ebp]

XJ:004040C7                 mov     esi, edi

XJ:004040C9

XJ:004040C9 loc_4040C9:                     ; CODE XREF: XJ:004040FBj

XJ:004040C9                 lodsb

XJ:004040CA                 stc

XJ:004040CB                 rol     al, 42h

XJ:004040CE                 rol     al, 42h

XJ:004040D1                 jmp     short loc_4040D4

XJ:004040D1 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040D3                 db 0E8h

XJ:004040D4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040D4

XJ:004040D4 loc_4040D4:                     ; CODE XREF: XJ:004040D1j

XJ:004040D4                 ror     al, 0E0h

XJ:004040D7                 stc

XJ:004040D8                 add     al, 9Dh

XJ:004040DA                 add     al, 5Dh

XJ:004040DC                 add     al, cl

XJ:004040DE                 clc

XJ:004040DF                 jmp     short loc_4040E2

XJ:004040DF ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040E1                 db 0E9h

XJ:004040E2 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040E2

XJ:004040E2 loc_4040E2:                     ; CODE XREF: XJ:004040DFj

XJ:004040E2                 add     al, cl

XJ:004040E4                 rol     al, 4Fh

XJ:004040E7                 add     al, cl

XJ:004040E9                 xor     al, 82h

XJ:004040EB                 sub     al, 4Fh

XJ:004040ED                 clc

XJ:004040EE                 jmp     short loc_4040F1

XJ:004040EE ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040F0                 db 0E8h

XJ:004040F1 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040F1

XJ:004040F1 loc_4040F1:                     ; CODE XREF: XJ:004040EEj

XJ:004040F1                 jmp     short loc_4040F4

XJ:004040F1 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040F3                 db 0C2h

XJ:004040F4 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040F4

XJ:004040F4 loc_4040F4:                     ; CODE XREF: XJ:004040F1j

XJ:004040F4                 add     al, 0E2h

XJ:004040F6                 jmp     short loc_4040F9

XJ:004040F6 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040F8                 db 0C2h

XJ:004040F9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

XJ:004040F9

XJ:004040F9 loc_4040F9:                     ; CODE XREF: XJ:004040F6j

XJ:004040F9                 clc

XJ:004040FA                 stosb

XJ:004040FB                 loop    loc_4040C9

XJ:004040FD                 mov     ebp, 55115115h

XJ:00404102                 pushf
```

以上循环的作用是解码 `offset` = `4040FD` , `size` = `A88`

`ida` 中 `file` - `idc` `file...` 打开 `des.idc`

`shift` + `F2`

输入命令 : `des(0x4040FD,0xA88);`

```
------des.idc------

static des(from,size){

  auto i,x,y,z;

  for (i=size;i>0;i=i-1){

    x=Byte(from);

    x=rol(x,0x42);

    x=rol(x,0x42);

    x=ror(x,0xE0);

    x=x+0x9D;

    x=x+0x5D;

    x=x+i;

    x=x+i;

    x=rol(x,0x4F);

    x=x+i;

    x=x^0x82;

    x=x-0x4F;

    x=x+0xE2;

    PatchByte(from,x);

    from = from +1;

  }

}

static rol(source,time){

  auto target,y,z;

  target=source&0x00FF;

  y=target>>(8 - time%8);

  z=target<<(time%8);

  target=y|z;

  return target;

}

static ror(source,time){

  auto target,y,z;

  target=source&0x00FF;

  y=target<<(8 - time%8);

  z=target>>(time%8);

  target=y|z;

  return target;

}

---------------------
```

执行后看到

```
XJ:004040FD                 mov     eax, [esp+20h]

XJ:00404102                 js      short near ptr dword_40410C+2

XJ:00404103                 or      al, bh

XJ:00404105                 test    ecx, ebp
```

跟 `od` 里看到的有点不同  不知道什么原因啦(`ida` `4.5` 下),问 `ida` `pro` 的作者吧

在 `XJ:004040FD` 一行按 `'u'` --> `undefine` 然后 `'c'` --> `code`

好了,一样了

```
XJ:004040FD                 mov     eax, [esp+20h]

XJ:00404101                 inc     eax

XJ:00404102                 js      short near ptr dword_40410C+2
```

`4dd466d0e1e0cc123afbe504da2462ca`
