
# 图片磨牙器.exe 菜鸟破解简析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-406.htm`](https://www.pediy.com/kssd/pediy07/pediy7-406.htm)

## 基本信息

- **破解作者**：`pentacle`
- **使用工具**：`OllyDbg1.10`, `PEid0.93`
- **破解平台**：`XP SP2`
- **软件名称**：`图片磨牙器.exe V1.0`
- **软件简介**：一个图片特效工具，有多种特效，还可以把图片转为字符画....... 未注册版有 `50` 次使用限制。
- **软件大小**：`386K`
- **加壳方式**：`UPX`

## 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 破解内容

现在正处在上不上下不下的位置，太难的追不出来，太简单的又浪费时间。刚好想起原先在 XX 杂志光盘上的`图片磨牙器`(刚学会用 `OD` 那儿总喜欢追出注册码,碰到的第一个不是明码比较的,为了不让自己信心受挫，便先掠在一边了),嘿嘿........

用`PEid0.93`查是`UPX`的壳，手动脱壳搞定，都不用修复`IAT`表，脱壳后`1.33M`。

运行后注册，无提示。不管，用`OD`载入。查找`ASCII`字符。(小提示，一般用关键字"注册""Reg"都可以快速找到我们要的)

### 关键代码分析

我们来到这儿：

```assembly
004C7A4B    E8 04C4F3FF     CALL 图片磨牙.00403E54                   ; 判断注册名和注册码的位数
004C7A50    3BD8            CMP EBX,EAX                               ; 注册码长度=注册名长度*2+2
004C7A52    75 75           JNZ SHORT 图片磨牙.004C7AC9                ; 注册名长度和注册码长度不符就跳走
004C7A54    8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
004C7A57    E8 F8C3F3FF     CALL 图片磨牙.00403E54
004C7A5C    8BD0            MOV EDX,EAX
004C7A5E    85D2            TEST EDX,EDX
004C7A60    7E 23           JLE SHORT 图片磨牙.004C7A85
004C7A62    B8 01000000     MOV EAX,1
004C7A67    8B4D F4         MOV ECX,DWORD PTR SS:[EBP-C]
004C7A6A    8A4C01 FF       MOV CL,BYTE PTR DS:[ECX+EAX-1]           ; 依次取注册名字符的 ASCII 值
004C7A6E    8B5D F8         MOV EBX,DWORD PTR SS:[EBP-8]
004C7A71    3A4C03 FF       CMP CL,BYTE PTR DS:[EBX+EAX-1]           ; 注册名与计算后的值对比
004C7A75    74 0A           JE SHORT 图片磨牙.004C7A81                 ; 不相等就 OVER
004C7A77    E8 28BCF3FF     CALL 图片磨牙.004036A4
004C7A7C    E9 85000000     JMP 图片磨牙.004C7B06
004C7A81    40              INC EAX
004C7A82    4A              DEC EDX
004C7A83  ^ 75 E2           JNZ SHORT 图片磨牙.004C7A67                ; 循环校验注册名与计算后的值
004C7A85    B8 6C7B4C00     MOV EAX,图片磨牙.004C7B6C                 ; 谢谢您注册本软件！
```

根据上面的对比，我们肯定是找不着真的注册码了。只好继续找吧

```assembly
004C6DDC    E8 E71BF4FF     CALL 图片磨牙.004089C8                   ; 取注册码第 1、2 位初始 EDI
004C6DE1    8BF8            MOV EDI,EAX                               ; 以后依次取出 2 位注册码作为 EDI
004C6DE3    C745 EC 0300000>MOV DWORD PTR SS:[EBP-14],3
004C6DEA    8D45 C8         LEA EAX,DWORD PTR SS:[EBP-38]
004C6DED    50              PUSH EAX
004C6DEE    B9 02000000     MOV ECX,2
004C6DF3    8B55 EC         MOV EDX,DWORD PTR SS:[EBP-14]
004C6DF6    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004C6DF9    E8 5ED2F3FF     CALL 图片磨牙.0040405C
004C6DFE    8B4D C8         MOV ECX,DWORD PTR SS:[EBP-38]            ; 取注册码第 3、4 位为 EAX
004C6E01    8D45 CC         LEA EAX,DWORD PTR SS:[EBP-34]
004C6E04    BA EC6E4C00     MOV EDX,图片磨牙.004C6EEC
004C6E09    E8 92D0F3FF     CALL 图片磨牙.00403EA0
004C6E0E    8B45 CC         MOV EAX,DWORD PTR SS:[EBP-34]
004C6E11    E8 B21BF4FF     CALL 图片磨牙.004089C8
004C6E16    8BD8            MOV EBX,EAX
004C6E18    3B75 F4         CMP ESI,DWORD PTR SS:[EBP-C]
004C6E1B    7D 03           JGE SHORT 图片磨牙.004C6E20
004C6E1D    46              INC ESI
004C6E1E    EB 05           JMP SHORT 图片磨牙.004C6E25
004C6E20    BE 01000000     MOV ESI,1
004C6E25    8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
004C6E28    0FB64430 FF     MOVZX EAX,BYTE PTR DS:[EAX+ESI-1]        ; 每个循环依次取机器的 ASCII 值
004C6E2D    33C3            XOR EAX,EBX                              ; EAX=EAX XOR EBX
004C6E2F    8945 E8         MOV DWORD PTR SS:[EBP-18],EAX
004C6E32    3B7D E8         CMP EDI,DWORD PTR SS:[EBP-18]            ; 异或后的 EAX 值与 EDI(也就是上一轮最终 EAX 值)比较
004C6E35    7C 0F           JL SHORT 图片磨牙.004C6E46
004C6E37    8B45 E8         MOV EAX,DWORD PTR SS:[EBP-18]           ; EDI>=EAX
004C6E3A    05 FF000000     ADD EAX,0FF                              ; EAX=EAX+0FF
004C6E3F    2BC7            SUB EAX,EDI                              ; EAX=EAX-EDI
004C6E41    8945 E8         MOV DWORD PTR SS:[EBP-18],EAX
004C6E44    EB 03           JMP SHORT 图片磨牙.004C6E49
004C6E46    297D E8         SUB DWORD PTR SS:[EBP-18],EDI            ; EDI < EAX。EAX=EAX-EDI
004C6E49    8D45 C4         LEA EAX,DWORD PTR SS:[EBP-3C]
004C6E4C    8B55 E8         MOV EDX,DWORD PTR SS:[EBP-18]
004C6E4F    E8 28CFF3FF     CALL 图片磨牙.00403D7C
004C6E54    8B55 C4         MOV EDX,DWORD PTR SS:[EBP-3C]
004C6E57    8D45 F0         LEA EAX,DWORD PTR SS:[EBP-10]
004C6E5A    E8 FDCFF3FF     CALL 图片磨牙.00403E5C
004C6E5F    8BFB            MOV EDI,EBX
004C6E61    8345 EC 02      ADD DWORD PTR SS:[EBP-14],2
004C6E65    8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
004C6E68    E8 E7CFF3FF     CALL 图片磨牙.00403E54
004C6E6D    3B45 EC         CMP EAX,DWORD PTR SS:[EBP-14]
004C6E70  ^ 0F8F 74FFFFFF   JG 图片磨牙.004C6DEA                      ; 循环计算
```

上面搞明白后注册机就出来了。还有这个是根据硬件代码来计算机器代码的。不过这块我看的有点头晕。就打算功力加深后再回头看看。初步看应该是`C`盘序列号(去除`-`)同常数`1234567`来计算的。具体的你们可以看一下，不过别忘了给我发一份啊。

计数值保存在`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\Main`下面的`Number`中，大家破解也可以通过`50`次用完等来进行破解。

### 计算注册码示例

附上计算注册码的流程：

- **机器码**：`2828362830362`
- **注册名**：`penta`
- **注册码**：`13B12FAF1C4E`

```text
            EAX=EAX XOR EDI       EAX=EAX + FF   EAX=EAX-EDI

①p"70"     83  =B1  XOR 32 机①              70 =83 -13
            ||  ~~3、4 位~~~~~~~              ~~  ~~1、2 位注册码
         13 < 83  注册码

②e"65"     17 =2F  XOR 38 机②   116=17  +FF   65 =116-B1
            ||  ~~5、6 位~~~~~~~   ~~~             ~~~ ~~3、4 位注册码
         B1 > 17  注册码

③n"6E"     9D =AF  XOR 32 机③             6E =9D -2F
            ||  ~~7、8 位~~~~~~~              ~~  ~~5、6 位注册码
         2F < 9D  注册码

④t"74"     24 =1C  XOR 38 机④    23 =24  +FF   74 =23 -AF
            ||  ~~9,10 位~~~~~~     ~~~          ~~  ~~7、8 位注册码
         AF > 24  注册码

⑤a"61"     7D =4C  XOR 33 机⑤             61 =7D -1C
            ||  ~~11,12 ~~~~~~~              ~~  ~~9、10 位注册码
         1C < 9D  位注册码
```

--------------------------------------------------------------------------------

## 破解总结

原先因为汇编语言不熟练，加之不是明码比较，所以浪费不少时间了。建议：初学破解的先找前几年的软件破解，这个就可以从报刊杂志的合订本上找啦。比如我就用`2001`和`2002`电脑报合订本光盘上的软件练手的。基本上很难的软件在精华中都可以找到破解例程。但还是建议自己动手(写总结再看破文)，实在不行就先看看破解文章。

其实我在破解这个软件是根据重启验证时找到上面的注册核心，因为在注册表中输入注册名和注册码方便。

--------------------------------------------------------------------------------

## 版权声明

本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
