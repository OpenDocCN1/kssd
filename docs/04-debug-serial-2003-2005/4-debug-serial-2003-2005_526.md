
# 系网红巾 V1.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6123.htm`](https://www.pediy.com/kssd/pediy06/pediy6123.htm)

- **软件大小**：`333 KB`
- **软件语言**：简体中文
- **软件类别**：国产软件 / 共享版 / 浏览辅助
- **应用平台**：`Win9x/NT/2000/XP`
- **加入时间**：`2003-06-18 11:17:05`
- **联系人**：`liuleilover@163.com`
- **软件下载**：`http://count.skycn.com/softdown.php?id=12504&url=http://js-http.skycn.net/down/redcloth.zip`

## 软件介绍

《系网红巾》专用于限制青少年对不健康网站（色情网站）的浏览。根据青少年接触“色情网站”的特点，提供了覆盖面较广的网址库包含常见的“色情网站”网址，能够有效的消除“色情网站”页面的弹出。并且提供上网时段和上网时间限制，有效控制青少年的上网时间。

## 破解分析

**破解工具**：`TRW 娃娃修改版`

**作者声明**：初学破解，仅作学习交流之用，失误之处敬请大侠赐教

最近碰到几个软件竟然是 `DES` 算法的（感谢 `fly` 兄提醒），头大！找个简单的给自已鼓鼓劲！

无壳，`VC` 编写！用 `bpx getwindowtexta` 下断，你一定会走到这里：

```
0167:00409523  MOV       EAX,[00444520]
0167:00409528  MOV       [ESP+08],EAX
0167:0040952C  LEA       ECX,[ESP+14]
0167:00409530  MOV       DWORD [ESP+34],00
0167:00409538  CALL      00406A60
0167:0040953D  LEA       ECX,[ESP+0C]
0167:00409541  MOV       BYTE [ESP+34],01
0167:00409546  PUSH      ECX
0167:00409547  LEA       ECX,[ESP+18]
0167:0040954B  CALL      00406AF0
0167:00409550  LEA       ECX,[ESP+0C]
                              ; <--[ECX]中为特征码"B694C92F"
0167:00409554  CALL      00422A82
0167:00409559  PUSH      BYTE +71
0167:0040955B  LEA       ECX,[ESP+0C]
0167:0040955F  CALL      00426CB1
0167:00409564  PUSH      ECX
0167:00409565  LEA       EDI,[ESI+0190]
                              ; <--[EDI]中为我们输入的假码"12345678998"
0167:0040956B  MOV       ECX,ESP
0167:0040956D  MOV       [ESP+10],ESP
0167:00409571  PUSH      EDI
0167:00409572  CALL      004227F7
0167:00409577  LEA       ECX,[ESP+18]
0167:0040957B  CALL      00406B70
                              ; <--关键的 Call,跟进!
0167:00409580  TEST      EAX,EAX
                              ; <--测试标志位
0167:00409582  JZ        NEAR 004096B7
                              ; <--跳就玩完了!
0167:00409588  PUSH      BYTE +00
0167:0040958A  PUSH      DWORD 040D
0167:0040958F  MOV       ECX,ESI
................
中间略去很多代码,大概是解除时间限制及使注册按钮变灰吧!
我可不想跟! ;)
0167:00409690  MOV       ECX,[00446788]
0167:00409696  PUSH      EAX
0167:00409697  PUSH      ECX
0167:00409698  MOV       EDX,[ECX]
0167:0040969A  CALL      NEAR [EDX+1C]
0167:0040969D  LEA       ECX,[ESP+1C]
0167:004096A1  MOV       DWORD [0044414C],00
0167:004096AB  MOV       BYTE [ESP+34],01
0167:004096B0  CALL      0040A710
0167:004096B5  JMP       SHORT 004096D2
0167:004096B7  PUSH      BYTE -01
0167:004096B9  PUSH      BYTE +00
0167:004096BB  PUSH      BYTE +75
0167:004096BD  CALL      0042B4E1
                              ; <--注册码错误!
0167:004096C2  MOV       ECX,EDI
0167:004096C4  CALL      00422A0D
0167:004096C9  PUSH      BYTE +00
0167:004096CB  MOV       ECX,ESI
0167:004096CD  CALL      004259AF
0167:004096D2  LEA       ECX,[ESP+14]
0167:004096D6  MOV       BYTE [ESP+34],00
0167:004096DB  CALL      00406AE0
0167:004096E0  LEA       ECX,[ESP+08]
0167:004096E4  MOV       DWORD [ESP+34],FFFFFFFF
0167:004096EC  CALL      00422A82
0167:004096F1  MOV       ECX,[ESP+2C]
0167:004096F5  POP       EDI
0167:004096F6  MOV       [FS:00],ECX
0167:004096FD  POP       ESI
0167:004096FE  ADD       ESP,BYTE +30
0167:00409701  RET
```

跟进那个关键的 `Call`，来到如下代码：

```
0167:00406B70  PUSH      BYTE -01
0167:00406B72  PUSH      DWORD 004349D8
0167:00406B77  MOV       EAX,[FS:00]
0167:00406B7D  PUSH      EAX
0167:00406B7E  MOV       [FS:00],ESP
0167:00406B85  SUB       ESP,BYTE +0C
0167:00406B88  PUSH      EBX
0167:00406B89  PUSH      EBP
0167:00406B8A  PUSH      ESI
0167:00406B8B  PUSH      EDI
0167:00406B8C  MOV       ESI,ECX
0167:00406B8E  MOV       EAX,[ESI+04]
                              ; <--[ESI+04]中为特征码
0167:00406B91  LEA       EDI,[ESI+04]
0167:00406B94  MOV       DWORD [ESP+24],00
0167:00406B9C  MOV       ECX,[EAX-08]
                              ; <--ECX=B,为输入假码的位数
0167:00406B9F  TEST      ECX,ECX
                              ; <--判断有无输入注册码
0167:00406BA1  JNZ       00406BB8
0167:00406BA3  LEA       ECX,[ESP+10]
0167:00406BA7  PUSH      ECX
0167:00406BA8  MOV       ECX,ESI
0167:00406BAA  CALL      00406AF0
0167:00406BAF  LEA       ECX,[ESP+10]
0167:00406BB3  CALL      00422A82
0167:00406BB8  PUSH      BYTE +2D
0167:00406BBA  LEA       ECX,[ESP+30]
0167:00406BBE  CALL      00420B62
0167:00406BC3  PUSH      ECX
                              ; <--ECX 中为特征码
0167:00406BC4  MOV       ECX,ESP
0167:00406BC6  MOV       [ESP+1C],ESP
0167:00406BCA  PUSH      EDI
                              ; <--[EDI]中为输入的假码
0167:00406BCB  CALL      004227F7
0167:00406BD0  MOV       ECX,ESI
0167:00406BD2  CALL      00406C90
                              ; <--此 Call 将特征码由字串形式转换为 Hex
                              ; 形式,如我的机子上返回在 EAX 中的值为
                              ; B694C92F,我的特征码为"B694C92F"
0167:00406BD7  PUSH      ECX
0167:00406BD8  LEA       EDX,[ESP+30]
                              ; <--[ESP+30]中为输入的码
0167:00406BDC  MOV       EBX,EAX
0167:00406BDE  MOV       ECX,ESP
0167:00406BE0  MOV       [ESP+18],ESP
0167:00406BE4  PUSH      EDX
0167:00406BE5  MOV       [ESP+20],EBX
0167:00406BE9  CALL      004227F7
0167:00406BEE  MOV       ECX,ESI
0167:00406BF0  CALL      00406D00
                              ; <--此 Call 的作用对输入的注册码进行处理
                              ; (只处理输入注册码的前 16 位,不足 16 位的在
                              ; 注册码前面部分以 0 补足),返回值在 EAX 和 EDX
                              ; 中,如我们输入 12345678998,则返回在 EAX 中
                              ; 的为 34578998,EDX 中为 00000012,如果输入的
                              ; 是字符,则只处理[a,f]与[A,F]之间的字符,其
                              ; 余以 0 代替,如我们输入 abcdefghijk,则 Call 后
                              ; EAX 中为 DEF00000,EDX 中为 00000ABC,代码就不
                              ; 列出了,有兴趣的跟一下.
0167:00406BF5  MOV       ESI,EAX
                              ; <--EDX 中为处理后的注册码的第二部分
0167:00406BF7  MOV       EAX,EDX
                              ; <--EAX 中为处理后的注册码的第一部分
0167:00406BF9  SHR       EAX,10
                              ; <--取 EAX 的高位部分
0167:00406BFC  MOV       DI,AX
                              ; <--高位部分存入 DI 中
0167:00406BFF  MOV       EAX,EDX
0167:00406C01  MOV       [ESP+14],EAX
0167:00406C05  MOV       BP,DX
0167:00406C08  SAR       EAX,1F
                              ; <--1F 为 31 位,相当于 EAX 清 0 吧?!
0167:00406C0B  MOV       ECX,10
                              ; <--ECX=10
0167:00406C10  MOV       EAX,ESI
0167:00406C12  SHR       EBX,10
                              ; <--EBX 中为特征码,取其高位部分
0167:00406C15  XOR       EDI,8182
                              ; <--DI=DI^8182
0167:00406C1B  XOR       EBP,8384
                              ; <--EBP=EBP^8384
0167:00406C21  CALL      00414340
                              ; <--取处理后注册码的第一部分的低位部分
                              ; 返回值在 EAX 中
0167:00406C26  XOR       EAX,8586
                              ; <--AX=AX^8586
0167:00406C2B  XOR       ESI,8788
                              ; <--ESI=ESI^8788
0167:00406C31  CMP       BX,DI
                              ; <--BX 中为转换后特征码的高 4 位
0167:00406C34  JNZ       00406C61
                              ; <--第 1 个关键跳转
0167:00406C36  CMP       BX,BP
0167:00406C39  JNZ       00406C61
                              ; <--第 2 个关键跳转
0167:00406C3B  MOV       ECX,[ESP+18]
                              ; <--ECX=B694C92F,应该记得怎么来的吧!
0167:00406C3F  CMP       CX,AX
                              ; <--CX 为 ECX 的低 4 位
0167:00406C42  JNZ       00406C61
                              ; <--第 3 个关键跳转
0167:00406C44  CMP       CX,SI
0167:00406C47  JNZ       00406C61
                              ; <--第 4 个关键跳转
0167:00406C49  LEA       ECX,[ESP+2C]
0167:00406C4D  MOV       DWORD [ESP+24],FFFFFFFF
0167:00406C55  CALL      00422A82
0167:00406C5A  MOV       EAX,01
                              ; <--EAX=01,成功的标志!
0167:00406C5F  JMP       SHORT 00406C74
0167:00406C61  LEA       ECX,[ESP+2C]
0167:00406C65  MOV       DWORD [ESP+24],FFFFFFFF
0167:00406C6D  CALL      00422A82
0167:00406C72  XOR       EAX,EAX
```



```
      <--EAX=0,失败!

0167:00406C74  MOV      ECX,[ESP+1C]

0167:00406C78  POP      EDI

0167:00406C79  POP      ESI

0167:00406C7A  POP      EBP

0167:00406C7B  MOV      [FS:00],ECX

0167:00406C82  POP      EBX

0167:00406C83  ADD      ESP,BYTE +18

0167:00406C86  RET      04
```

由此我们可以逆推出我的机器上的注册码了:

注册码第 1~4 位为 `B694⁸¹⁸²=3716`

注册码第 5~8 位为 `B694⁸³⁸⁴=3510`

注册码第 9~12 位为 `C92F⁸⁵⁸⁶=4CA9`

注册码第 13~16 位为 `C92F⁸⁷⁸⁸=4EA7`

由此得到一组可用的注册码:

特征码:`B694C92F`

注册码:`371635104CA94EA7`

软件注册成功后将注册码保存在注册表的：

`"HKEY_LOCAL_MACHINE\Software\RedCloth\Page5\regkey"`中

Crack By ShenGe[BCG] 03.06.18

`4dd466d0e1e0cc123afbe504da2462ca`
