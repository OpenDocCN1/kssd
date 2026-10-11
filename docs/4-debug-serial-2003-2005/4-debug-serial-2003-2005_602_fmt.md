
# B-Jigsaw 7.7 破解分析

> 原文：[`www.pediy.com/kssd/pediy06/pediy6171.htm`](https://www.pediy.com/kssd/pediy06/pediy6171.htm)

**ADC 游戏完全破解之二——破解 `B-Jigsaw`**

这个游戏和前两天破解的 `B-Puzzle` 很相似（同一家公司的产品）。参考我的上篇文章 [`tongtian.net/cgi-bin/topic.cgi?forum=19&topic=4835&show=0`](http://tongtian.net/cgi-bin/topic.cgi?forum=19&topic=4835&show=0)。

前半部分算出 8 位码的过程完全一样：先得到 `NAME`，如果小于 8 个字符就用空格来补，然后取第 1,3,5,7 个字符与 `"awgsJiBtAANrPNYOntA"` 中第 2,4,6,8 个字符进行异或运算。将每次得到的值变为十进制后作为两位 `CODE`（如果是 2 位十进制数就把它翻转；如果是 1 位十进制数 `X` 就写成 `'1X'`；如果是 3 位 `XYZ` 就写成 `XY`），这样最后的 `CODE` 就是 8 位。（这一部分的分析省略）

这个软件好像反 `Win32DASM`，一用就“停止响应”，所以下面的代码是用 `SoftICE` 抓下来的，与 `Win32Dasm` 的不太一样，各位多多包涵 :)

得到那个八位码后，跳转到以下位置：

```
017F:00411AE7  MOV       EDI,EBX        <--循环开始
017F:00411AE9  PUSH      EDI
017F:00411AEA  LEA       EAX,[EBP-08]
017F:00411AED  PUSH      EAX
017F:00411AEE  CALL      004D0D30
017F:00411AF3  ADD       ESP,08
017F:00411AF6  LEA       EAX,[EBP-08]
017F:00411AF9  CALL      004D11BC
017F:00411AFE  MOV       EDX,[EBP-08]
017F:00411B01  ADD       EDI,EDX
017F:00411B03  DEC       EDI
017F:00411B04  MOVSX     ECX,BYTE PTR [EDI]      <--[edi]处是 NAME 中依次取得的字符
                                         <--我们把这个字符设为 R
017F:00411B07  MOV       EAX,ECX       <--eax=R
017F:00411B09  MOV       [EBP-70],EBX
017F:00411B0C  SHL       EAX,03        <--eax 左移 3 位，即 eax=eax*8
017F:00411B0F  MOV       EDX,[EBP-70]
017F:00411B12  SUB       EAX,ECX       <--eax=eax-R
017F:00411B14  LEA       ECX,[EBP-08]
017F:00411B17  PUSH      EDX
017F:00411B18  PUSH      ECX
017F:00411B19  ADD       ESI,EAX       <--esi 是上面求得 8 位码的数值形式
                                         <--esi=esi+eax
017F:00411B1B  CALL      004D0D30
017F:00411B20  ADD       ESP,08
017F:00411B23  LEA       EAX,[EBP-08]
017F:00411B26  CALL      004D11BC
017F:00411B2B  MOV       EDX,[EBP-70]
017F:00411B2E  MOV       ECX,[EBP-08]
017F:00411B31  ADD       EDX,ECX
017F:00411B33  DEC       EDX
017F:00411B34  MOVSX     EAX,BYTE PTR [EDX]      <--这里仍然是 R
017F:00411B37  MOV       EDX,EAX        <--edx=R
017F:00411B39  MOV       [EBP-74],EBX
017F:00411B3C  SHL       EDX,04        <--左移 4 位，即 edx=edx*16
017F:00411B3F  MOV       ECX,[EBP-74]
017F:00411B42  SUB       EDX,EAX       <--edx=edx-R
017F:00411B44  PUSH      ECX
017F:00411B45  LEA       EDX,[EDX*4+EAX]         <--edx=edx*4+R
017F:00411B48  LEA       EAX,[EBP-08]
017F:00411B4B  PUSH      EAX
017F:00411B4C  ADD       ESI,EDX       <--esi=esi+edx
017F:00411B4E  CALL      004D0D30
017F:00411B53  ADD       ESP,08
017F:00411B56  LEA       EAX,[EBP-08]
017F:00411B59  CALL      004D11BC
017F:00411B5E  MOV       EDX,[EBP-74]
017F:00411B61  MOV       ECX,[EBP-08]
017F:00411B64  ADD       EDX,ECX
017F:00411B66  DEC       EDX
017F:00411B67  MOVSX     EAX,BYTE PTR [EDX]      <--仍然是 R
017F:00411B6A  LEA       EDX,[EAX*2+EAX]         <--edx=R*2+R
017F:00411B6D  SHL       EDX,05                  <--edx 左移 5 位，即 edx=edx*32
017F:00411B70  SUB       EDX,EAX                 <--edx=edx-R
017F:00411B72  SHL       EDX,04                  <--edx 左移 4 位，即 edx=edx*16
017F:00411B75  ADD       EDX,EAX                 <--edx=edx+R
017F:00411B77  ADD       ESI,EDX                 <--esi=esi+edx
017F:00411B79  INC       EBX
017F:00411B7A  LEA       EAX,[EBP-08]
017F:00411B7D  CALL      004D1028
017F:00411B82  CMP       EBX,EAX
017F:00411B84  JLE       00411AE7                <--循环结束，取得下一个字符
                                         <--循环次数为 NAME 长度(包括空格)
017F:00411B8A  CALL      KERNEL32!GetTickCount

<--下面的操作是把 esi 变为字符等等，esi 即为真码

017F:00411B8F  MOV       EDX,[EBP-6C]
017F:00411B92  SUB       EAX,EDX
017F:00411B94  CMP       EAX,000003E8
017F:00411B99  JBE       00411BA8
017F:00411B9B  MOV       ECX,[004DDFFC]
017F:00411BA1  MOV       EAX,[ECX]
017F:00411BA3  CALL      004837C4
017F:00411BA8  MOV       WORD PTR [EBP-58],0080
017F:00411BAE  MOV       EDX,004D5A4A
017F:00411BB3  LEA       EAX,[EBP-10]
017F:00411BB6  CALL      004D0DB4
017F:00411BBB  INC       DWORD PTR [EBP-4C]
017F:00411BBE  XOR       EDX,EDX
017F:00411BC0  MOV       WORD PTR [EBP-58],0008
017F:00411BC6  MOV       WORD PTR [EBP-58],008C
017F:00411BCC  MOV       [EBP-40],EDX
017F:00411BCF  LEA       EDX,[EBP-40]
017F:00411BD2  INC       DWORD PTR [EBP-4C]
017F:00411BD5  MOV       EAX,ESI
017F:00411BD7  CALL      004BAD80
017F:00411BDC  LEA       EDX,[EBP-40]
017F:00411BDF  LEA       EAX,[EBP-10]
017F:00411BE2  CALL      004D0EFC
017F:00411BE7  DEC       DWORD PTR [EBP-4C]
017F:00411BEA  LEA       EAX,[EBP-40]
017F:00411BED  MOV       EDX,00000002
017F:00411BF2  CALL      004D0EB8
017F:00411BF7  MOV       WORD PTR [EBP-58],0098
017F:00411BFD  LEA       EDX,[EBP-10]
017F:00411C00  MOV       EAX,[EBP-44]
017F:00411C03  CALL      004D0EE8
017F:00411C08  MOV       EAX,[EBP-44]
017F:00411C0B  MOV       EDX,00000002
017F:00411C10  MOV       WORD PTR [EBP-58],00A4
017F:00411C16  PUSH      EAX
017F:00411C17  LEA       EAX,[EBP-10]
017F:00411C1A  DEC       DWORD PTR [EBP-4C]
017F:00411C1D  CALL      004D0EB8
017F:00411C22  DEC       DWORD PTR [EBP-4C]
017F:00411C25  LEA       EAX,[EBP-0C]
017F:00411C28  MOV       EDX,00000002
017F:00411C2D  CALL      004D0EB8
017F:00411C32  DEC       DWORD PTR [EBP-4C]
017F:00411C35  LEA       EAX,[EBP-08]
017F:00411C38  MOV       EDX,00000002
017F:00411C3D  CALL      004D0EB8
017F:00411C42  POP       EAX
017F:00411C43  MOV       WORD PTR [EBP-58],0098
017F:00411C49  INC       DWORD PTR [EBP-4C]
017F:00411C4C  MOV       EDX,[EBP-68]
017F:00411C4F  MOV       FS:[00000000],EDX
017F:00411C56  POP       EDI
017F:00411C57  POP       ESI
017F:00411C58  POP       EBX
017F:00411C59  MOV       ESP,EBP
017F:00411C5B  POP       EBP
017F:00411C5C  RET
```

整理一下：`ESI = ESI + (R*8 - R) + ((R*16 - R)*4 + R) + ((R*3*32 - R)*16 + R)`

比上次那个 `B-Puzzle` 更复杂一些，我写的注册机 (Borland Pascal 7.0)：

```pascal
Program CrackBJigsaw;

var st, name: string;
    s: array[1..8] of longint;
    i, p: longint;
    code: longint;

begin
    st := 'awgsJiBtAANrPNYOntA';
    write('Please input your name:');
    readln(name);

    if length(name) <= 8 then name := name + '        '; {8 blanks}

    p := 1;
    repeat
        i := (ord(st[p+1]) xor ord(name[p]));
        if i >= 100 then i := i div 10;
        if i < 10 then begin
            s[p] := 1;
            s[p+1] := i;
        end
        else begin
            s[p] := i mod 10;
            s[p+1] := i div 10;
        end;
        p := p + 2;
    until p > 8;

    code := s[1]*10000000 + s[2]*1000000 + s[3]*100000
            + s[4]*10000 + s[5]*1000 + s[6]*100 + s[7]*10 + s[8];

    for p := 1 to length(name) do
        code := code + ord(name[p]) * 2 * p * (p - 1);

    for p := 1 to length(name) do
    begin
        i := ord(name[p]);
        code := code + i*7 + (i*15*4 + i) + (i*3*32 - i)*16 + i;
    end;

    writeln('CODE:BJ', code);
    writeln('Crack By RoBa');
end.
```

一个可用的 `NAME`: `RoBa`
`CODE`: `BJ74955108`

这个软件仍然有 Bug：`name` 为空，用我的注册机可以算出 `code` 为 `78801284`。在 `NAME` 处什么也不填，在 `CODE` 处填 `BJ78801284`，呵呵~~~~，注册成功了。

`4dd466d0e1e0cc123afbe504da2462ca`
