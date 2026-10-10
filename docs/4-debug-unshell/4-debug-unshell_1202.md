

# ZProtect 脱壳无 KEY 解码总结

> 原文：[`www.pediy.com/kssd/pediy11/112126.html`](https://www.pediy.com/kssd/pediy11/112126.html)

【文章标题】: ZProtect 脱壳无 KEY 解码总结  
【文章作者】: A.D.  
【软件名称】: 某 ZP 加壳的程序  
【使用工具】: OllyDBG 1.0 原版 + StrongOD + ODbgScript  
【操作平台】: WinXP SP3  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
【首发论坛】: 一蓑烟雨 www.unpack.cn 转帖请注明  
【文章目的】: ZProtect 旧版加壳的程序，如果使用了注册框和试用功能，可以通过 y3y3y3 大大的 LPK.DLL 来绕过。但是如果是 ZP 1.49 或者企业版，又或者没有使用试用功能，程序本身虽然依然可以由 LPK.DLL 绕过，可绕过后的程序并不能正常运行。这是因为用 ZP 加壳时，程序的部分代码用密钥进行了加密，在绕过注册框后，必须正确地使这些代码解密，程序才可以正常运行。而本文的目的就是探讨如何在没有注册码的情况下使之正确解码，使程序可以正常运行，进而可以脱壳。   
【致谢人员】: hyperchem、ximo、Hmily、ffzy  
   

引用: ZPROTECT 加壳的程序 有些是需要用 key 来解码的。  
这个用 key 解码的过程是通过从 key 中计算一个出一个 dword，然后对它进行 md5 运算，得到的这个值用做解码的密钥（16 个字节）。  
解码的方法为：在全部代码解码完成以后，进入 key 解码阶段，  
用这个密钥异或特定的区域（一般是程序原始区段的前 0x400 个字节），每 0x10 一个单位（以为这是密钥的长度。）  
至于如何猜密钥就自己想吧。说了对你不好。  
1.44 以后的版本有两次异或，一次异或是一个常数的 md5 值，一次是从 key 中的到 dword 的 md5。  
注意区分。  
hyperchem 发表于 2010-4-14 02:02  

  
  
【无 KEY 解码原理】  
在正确理解上面 hyperchem 大大给出的提示的情况下，对 ZP 加壳的程序进行研究。  
了解到 ZProtect 加壳程序的运行流程如下：  
代码:

```
OD 初始断点处=>待输入注册码的注册框=>程序代码整体解码=>特定区域 KEY 解码=>到达真正 OEP

```

而特定区域 KEY 解码的流程，通过我之前的疑问贴 ZProtect 1.4.9 脱壳练习遇到的疑问二 10 楼的帖子已经解释过了，如下：  
代码:

```
第一次解码=>第二次解码=>第三次解码

```

事实上，我并不很确切地知道三次解码分别进行的是什么操作，是用的 KEY 的 MD5，还是其他什么，我只是通过自己加壳的计算器的两种状态下的运行，知道了前两次解码的结果不管是输入正确注册码还是绕过注册框其结果都是一样的，而第三次解码的结果则不同。  
之后我尝试在第三次解码前，把解码的对象 XOR 数值改成正确的数值，发现解码成功，到达真正 OEP 时程序可以运行。  
所以要想实现无 KEY 解码，其原理就是在第三次解码前把用以解码的 XOR 数值改成正确的数值，而问题也转化为如何在没有注册码的情况下猜解出正确的解码 XOR 数值。  
  
【无 KEY 解码流程】  
\============================  
1、准备工作:  
在整个猜解的过程当中，我们需要记录下一些信息，首先建立一个 TXT 文档，内容如下：  
**1、真正 OEP：**  
**2、IATSTART：**  
**3、密钥位置：**  
**数值：**  
**4、解码代码段：**  
**5、正确解码：**  
**6、IATEND：**  
而整个猜解的流程，我通过自己编写的 ODbgScript 脚本来实现，首先建立一个脚本模板：  
代码:

```
var oep        //ESP 定律 用到的断点，并不是真正的 OEP，只是用以判断已经到真正的 OEP 了
 
//清空断点
bphwcall
bpmc
 
//获取 OEP
FoundOEP:
        mov oep,eip
        cmp \[oep\],60,1
        je GetOEP
        sti
jmp FoundOEP
        GetOEP:
        sti
        mov oep,esp
 
//绕过注册框
BypassRegist:
        bphws 77D56D7D        //注册对话框断点
        run
        mov eax,232c
        mov eip,77D56D99
        bphwcall
 
//到达真正 OEP
GoOEP:
        bphws oep,"r"
        run
        bphwcall
        sti        //这里有多少个 sti 在于 ESP 定律断点断下来后 F7 几次到达真正 OEP
        sti
 
//退出函数
Exit:
        ret

```

以上准备工作做好后，就可以正式开始无 KEY 解码。  
\============================  
2、载入程序，运行上面的原始模板脚本。运行完后，程序停在真正 OEP 处。记录下来：  
**1、真正 OEP：esp 定律=>12FFA0=>46F610 //其中 12FFA0 就是脚本中的 oep**  
\============================  
3、在真正 OEP 上下文中的 CALL 里找 IAT 表的位置。  
代码:

```
0046F610   55               PUSH EBP
0046F611   8BEC             MOV EBP,ESP
0046F613   83C4 F0          ADD ESP,-10
0046F616   B8 B8F34600      MOV EAX,程序.0046F3B8
0046F61B   E8 D068F9FF      CALL 程序.00405EF0  ☆跟进

```

  
代码:

```
00405EF0   53               PUSH EBX
00405EF1   8BD8             MOV EBX,EAX
00405EF3   33C0             XOR EAX,EAX
00405EF5   A3 A4004700      MOV DWORD PTR DS:\[4700A4\],EAX
00405EFA   6A 00            PUSH 0
00405EFC   E8 2BFFFFFF      CALL 程序.00405E2C  ☆跟进

```

  
代码:

```
00405E2C  -FF25 F8314700    JMP DWORD PTR DS:\[4731F8\]                //这里的 4731F8 应该就是 IAT 表当中的一项了
00405E32   8BC0             MOV EAX,EAX
00405E34  -E9 8B470A00      JMP 程序.004AA5C4
00405E39   90               NOP
00405E3A   8BC0             MOV EAX,EAX
00405E3C  -E9 FF430A00      JMP 程序.004AA240
00405E41   90               NOP

```

  
然后就在数据窗口中去到 4731F8，转成长型-地址看看，上下文如下：  
代码:

```
00472FF0  00000000
00472FF4  00000000
00472FF8  00000000
00472FFC  00000000
00473000  640D72D8
00473004  2852F1E9
00473008  685733E4
0047300C  343B2735
...
004731D8  685733E4
004731DC  00D000EE
004731E0  6447DD5C
004731E4  00D000FC
004731E8  685733E4
004731EC  3476B475
004731F0  00D0010A
004731F4  2818542D
004731F8  00A73E10

```

  
通过这里可以判断出 IATSTART 的开始位置可能为 00473000 ，记录如下：  
**2、IATSTART：473000=> //因为可能所以先空着**  
\============================  
4、接下来，通过找到的 IAT 表，反过来找解码的时候。  
代码:

```
在脚本模板中相应位置添加如下代码，使得在绕过注册框后，停在解码 IAT 表的时候。
//根据找到的 IAT 表首地址+1F，反找解码 XOR 数值位置
FoundXor:
        bphws 47301F,"w"
        run
        jmp Exit

```

  
重载程序，运行脚本，脚本结束时，程序停在 473000 第一次解码的时候。  
通过内存窗口找到 473000 所在区段 401000，右键数据，HEX-16 位，转到 473000 的地址。  
这时，从内存窗口中，我们可以看到 IAT 表第一次解码的情况，按一下 F9，到达第二次解码的时候，再按一下 F9，到达第三次解码的时候，这时，按着 F7 不放，我们可以看到内存中 IAT 表正在一位一位地解码，而解码的汇编代码段如下：  
代码:

```
00A745C3   8BD0             MOV EDX,EAX
00A745C5   83E2 0F          AND EDX,0F
00A745C8   8A0C14           MOV CL,BYTE PTR SS:\[ESP+EDX\]                //XOR 对象的位置
00A745CB   300C38           XOR BYTE PTR DS:\[EAX+EDI\],CL
00A745CE   83C0 01          ADD EAX,1
00A745D1   3BC3             CMP EAX,EBX
00A745D3  ⁷C EE            JL SHORT 00A745C3

```

  
通过反复观察，可以知道 XOR 对象的位置和 XOR 对象的数值，填表如下：  
**3、密钥位置：12FEE0~12FEEF**  
**数值：0012FEE0 F1 D3 FF 84 43 29 77 32 86 2D F2 1D C4 E5 72 62**  
\============================  
5、在知道了解码时 XOR 对象的位置，和错误的解码数值是多少后，我们就可以反过来找有多少个区段被解码，以方便后文无 KEY 解码脚本的书写。  
解码的时候，这个位置的数值必然是错误的解码 XOR 数值，所以我们只要在这个位置写入这个错误数值的时候把程序断下来，然后看看程序解码了哪里，就可以知道确切的区段数和每个区段的起始位置了。  
在脚本模板里，把 FoundXor 函数段去掉，替换为：  
代码:

```
FoundDecodeSection:
        bphws 12FEEF,"w"                //填写密钥终止位置
        bphws oep,"r"
FoundDecodeSection2:
        run
        cmp eip,oep
        je FoundDecodeSection3
        find 12FEE0,#F1D3FF8443297732862DF21DC4E57262#        //填写密钥起始位置和错误密钥数值
        cmp $RESULT,0
        je FoundDecodeSection2
        msg "请对内存中的程序代码段按 F2 断点，F9 一次后找到的操作的地址即为解码段起始处，再回到脚本按空格继续"
        pause
        jmp FoundDecodeSection2
FoundDecodeSection3:
        jmp GoOEP

```

重载程序，运行脚本。  
每当脚本弹出对话框的时候，按确定，然后在内存区段窗口，对加壳程序的代码段按 F2，下内存访问断点，F9 继续。  
程序断在如下代码：  
代码:

```
00A745CB   300C38           XOR BYTE PTR DS:\[EAX+EDI\],CL
00A745CE   83C0 01          ADD EAX,1
00A745D1   3BC3             CMP EAX,EBX
00A745D3  ⁷C EE            JL SHORT 00A745C3
00A745D5   83C4 14          ADD ESP,14

```

  
这时，可以看到  
代码:

```
CL=F1
DS:\[00401000\]=2D ('-')

```

  
现在我们可以知道程序解码的第一个区段的起始处是 00401000 了，记录下来。重新回到脚本窗口，空格让脚本继续运行。  
反复进行以上步骤，直到脚本运行结束。把刚才所有的结果记录下来：  
**4、解码代码段：401000、470000、473000、477000、47F000**  
\============================  
6、在知道了解码段后，我们要做的就是把所有解码段在运行到第三次解码前的内容记录下来。每一个解码段的内容，大概只需要记录一个屏幕就够用了。我记录的长度是 37F。如果解码段的长度不够长，就把这个长度改短点。记录这些内容的目的是用来猜解正确的解码 XOR 数值。  
重新编辑模板脚本，把刚才的 FoundDecodeSection 3 个函数去掉，按解码段数量，添加下面代码，每个解码段一一对应：  
代码:

```
//根据找到的解码地址+37F，反找二次解码后的结果，推最后结果
GuessXor1:
        bphws 40137F,"w"        //解码代码段 401000 后 37F 长
        run
        run
        msg "401000 解码结果"
        bphwcall
GuessXor2:
        bphws 47037F,"w"        //解码代码段 470000 后 37F 长
        run
        run
        msg "470000 解码结果"
        bphwcall
GuessXor3:
        bphws 47337F,"w"        //解码代码段 473000 后 37F 长
        run
        run
        msg "473000 解码结果"
        bphwcall
GuessXor4:                        //因为+37F 断不下来，观察发现这个解码段很短，只能+1FF
        bphws 4771FF,"w"        //解码代码段 477000 后 1FF 长
        run
        run
        msg "477000 解码结果"
        bphwcall
GuessXor5:
        bphws 47F37F,"w"        //解码代码段 47F000 后 37F 长
        run
        run
        msg "47F000 解码结果"
        bphwcall

```

重载程序，运行脚本，每当程序运行到弹出对话框时，在内存窗口中去到相应的位置，把第二次解码后的结果记录下来，长度就是你选择的长度，然后点击确定继续运行脚本，直到脚本运行完毕。  
分别以 5 个文件保存记录下来的信息，  
文件一 00401000.txt：  
代码:

```
00401000  2D B1 B2 E0 A9 DF 67 75 0D 72 C0 14 F3 F4 4E 56  -辈喋 u.r?篝 NV
00401010  29 A1 F3 E0 AA D8 25 0A 22 1E A0 33 FC 99 3D 33  )◇嗒?."?=3
00401020  2D F5 80 95 CF 55 65 1A 4E 0E E5 75 9F F1 0D 3E  -UeN.>
00401030  48 D3 F3 E0 AA D8 25 E5 62 1E A5 E5 DD E5 0E 56  H 芋嗒?ュ蒎 V
00401040  28 A6 BB 8E DE BD 42 7F 10 1A A5 75 9D 75 B1 A9  (暴
00401050  D6 DE 79 20 F2 C8 65 1A 63 1A E7 0C E9 90 4F 56  洲 y 蛉 ec?OV
00401060  29 A1 F2 1F AA D8 25 8A 0E 0E E5 75 9C F1 19 39  )◎%?9
00401070  5B C5 F1 E0 AA D8 25 E5 9D 1E A5 E5 1D E5 0E 56  \[篷嗒?ュ?V
00401080  28 A9 B1 81 D8 BC 4C 74 03 72 A0 75 9D F5 4E A9  (┍trN
00401090  D6 5E 0D 70 32 C8 65 1A 68 18 F6 01 EF 9C 20 31  .p2h? 1
004010A0  C5 B1 B2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  疟侧%bNV
004010B0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004010C0  C5 B1 B2 E0 AE D8 25 1A 62 1E A5 75 59 C0 0E 56  疟侧%bY?V
004010D0  F9 94 B2 E0 7E ED 65 1A BA 2B E5 75 51 C0 0E 56  侧~?Q?V
004010E0  3D 92 B2 E0 9A EB 65 1A 0E 2D E5 75 9A A1 01 34  =-4
004010F0  43 C4 91 94 52 C8 65 1A 65 19 F1 3A FF 9F 2B 35  Ce?
00401100  5D 4D E2 A0 AA D8 25 1A 62 1E A5 73 CE 8C 3D 22  \]M%b="
00401110  4C CC F2 E0 B2 C9 65 1A 6D 14 EC 3C F3 81 2B 24  L 舔嗖 m?+$
00401120  4F C0 91 85 AA D8 25 1A 63 1E A5 75 9D F5 4E 56  O?cNV
00401130  29 61 F2 E0 AA D8 25 1A 24 18 F6 0C EE 81 2B 3B  )a 蜞%$?+;
00401140  2A A1 0D 1F 66 5B 61 3E 66 E6 4C A4 DB F5 4E D5  \*?f\[a>fほ
00401150  6D 85 F6 18 43 37 63 1A 62 9D E1 51 99 0D A7 AF  mC7cbQ?Н
00401160  6F A1 F2 2C 66 9D 34 5A 62 51 B4 35 9D AC 5F 16  o◎,f?ZbQ?\_
00401170  29 A0 F2 E0 AA D8 25 1A 62 1E A5 75 9D 35 4E 56  )嗒?b?NV
00401180  29 A1 F2 E0 EC BD 34 5A 62 16 A5 75 9D F5 4E 56  )◎囔?ZbNV
00401190  29 2C B2 E0 4A C9 65 1A 13 0F E5 75 9D F5 4E 56  ),侧 JNV
004011A0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004011B0  29 A1 F2 E0 4A C9 65 1A 6E 1E A5 75 3D E5 0E 56  )◎n=?V
004011C0  ED 94 B2 E0 5E 8F 65 1A 62 46 E5 75 45 C0 0E 56  侧^bFE?V
004011D0  E5 94 B2 E0 BA 80 65 1A 52 2D E5 75 F1 C6 0E 56  侧 eR-衿 V
004011E0  38 F5 BB 8E DE BD 57 7C 03 7D C0 11 D2 97 24 33  8 趸|}?$3
004011F0  4A D5 79 20 43 DB B0 10 62 8E 2E B5 74 76 C0 5C  J C 郯 b?v
00401200  29 31 79 20 43 23 B7 10 62 8E 2E B5 74 56 D9 5C  )1y C#?b?V
00401210  29 31 79 20 43 BF B7 10 62 8E 2E B5 74 6E C1 5C  )1y C 糠 b?n
00401220  29 31 79 20 43 BF B4 10 62 8E 2E B5 74 5E D1 5C  )1y C 看 b?^
00401230  29 31 79 20 43 EB B2 10 62 8E 2E B5 74 16 D2 5C  )1y C 氩 b?
00401240  29 31 79 20 43 97 BD 10 62 8E 2E B5 74 76 DC 5C  )1y Cb?v
00401250  29 31 79 20 43 77 B1 10 62 8E 2E B5 74 A2 DD 5C  )1y Cw?b?⑤\\
00401260  29 31 79 20 55 FD A5 2B 25 1E 2E B5 74 DA D2 5C  )1y U+%.谝\\
00401270  29 31 79 20 43 3B BF 10 62 8E 2E B5 74 EE DF 5C  )1y C;?b?钸\\
00401280  29 31 79 20 43 AF A8 10 62 8E 2E B5 74 BE DE 5C  )1y Cb?巨\\
00401290  29 31 79 20 43 EF BA 10 62 8E 2E B5 74 EA DF 5C  )1y C 锖 b?赀\\
004012A0  29 31 79 20 43 5F B3 10 62 8E 2E B5 74 5E C0 5C  )1y C\_?b?^
004012B0  29 31 79 20 43 0B B2 10 62 8E 2E B5 74 E2 D4 5C  )1y C ?b?庠\\
004012C0  29 31 79 20 43 4F B5 10 62 8E 2E B5 74 2A DF 5C  )1y CO?b?\*
004012D0  29 31 79 20 43 53 BB 10 62 8E 2E B5 74 56 D2 5C  )1y CS?b?V
004012E0  29 31 79 20 43 1B B0 10 62 8E 2E B5 74 5E D7 5C  )1y C?b?^
004012F0  29 31 79 20 43 17 B8 10 62 8E 2E B5 74 5A C0 5C  )1y C?b?Z
00401300  29 31 79 20 43 3B AB 10 62 8E 2E B5 CE 76 8A EA  )1y C;?b?滴 v
00401310  92 AB F2 E0 AA 8C CD 43 9D E1 5A 83 D9 D1 62 57  蜞 ZW
00401320  5D A4 FD 57 F6 FC 15 91 A1 9D 61 31 C6 36 C5 96  \]W 鳇 1?
00401330  C0 22 6A EA AA 48 AE DA 8B 69 3C 7F 9D 65 C5 96  ?j 戟 H<
00401340  C0 42 6F EA AA 48 AE DA 8B 65 3F 7F 9D 65 C5 96  o 戟 H?
00401350  C0 76 6E EA AA 48 AE DA 8B 35 37 7F 9D 65 C5 96  n 戟 H?7
00401360  C0 AE 68 EA AA 48 AE DA 8B 6D 31 7F 9D 65 C5 96  喇 h 戟 H1
00401370  7A F7 4C 04 8F 9F 25 99 5C 1E D0 4F F5 B1 48 56  z%醣 HV

```

文件二 00470000.txt：  
代码:

```
00470000  29 A1 F2 E0 AA D8 25 1A 60 93 E5 75 9D F5 4E 56  )◎嗒?\`uNV
00470010  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00470020  1B B2 79 20 A8 D8 AE DA 62 93 E5 75 9D 78 0E 56   ㄘbuV
00470030  29 2C B2 E0 AA D8 25 1A 62 1E A5 75 75 D5 0E 56  ),侧%bu?V
00470040  51 83 B2 E0 52 FD 65 1A 62 D5 69 BD 54 22 81 9E  Qb"
00470050  E4 6F 29 38 70 01 EF C6 BF C0 7A 95 7C 16 4E B2  )8p 锲坷 zN
00470060  CC 2C B2 E0 AA D8 25 1A 62 1E 81 35 D8 87 3C 39  ?侧%b?<9
00470070  5B A1 79 20 F8 AD 4B 6E 0B 73 C0 55 F8 87 3C 39  \[ Kn s<9
00470080  5B 81 D2 C0 8A F8 44 6E 42 2E 95 45 AD C5 7E 66  \[nB.~f
00470090  19 A1 79 20 9A E9 17 29 56 2B 93 42 A5 CC 0F 14   )V+ヌ
004700A0  6A E5 B7 A6 55 27 DA E5 62 1E A5 75 9D F5 4E 56  j 宸'阱 bNV
004700B0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004700C0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004700D0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004700E0  1B A1 79 20 B5 D8 39 1A 7D 1E BB 75 82 F5 50 56   地 9}PV
004700F0  36 A1 ED E0 B4 D8 3A 1A 7C 1E BA 75 82 F5 53 56  6№啻?|SV
00470100  36 A1 EC E0 B5 D8 3B 1A 7D 1E BA 75 83 F5 51 56  6§嗟?}QV
00470110  37 A1 ED E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  7№嗒?bNV
00470120  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00470130  29 A1 F2 E0 0E A2 65 1A 62 1E A5 75 9D F5 4E 56  )◎?bNV
00470140  79 24 B2 E0 F6 5D 65 1A 62 1E A5 F5 9D F5 4E 16  y$侧 ebヵN
00470150  29 A1 F2 20 AA D8 25 1A 62 1E A5 75 9C F5 4E 56  )◎ %bNV
00470160  2B A1 F2 E0 A9 D8 25 1A AA 73 E5 75 4D 98 0E 56  +◎喋?M?V
00470170  29 A1 B2 96 90 B3 2E C4 58 5E 2E B5 97 F5 4E 56  )〔?^.V
00470180  16 B2 79 20 2A 13 81 56 62 42 83 70 B8 DB 64 32   \*bB 港 d2
00470190  89 CF B2 E0 02 B6 65 1A D2 70 E5 75 25 9B 0E 56  侧%?V
004701A0  E9 CF B2 E0 62 B6 65 1A B2 70 E5 75 45 9B 0E 56  橄侧 bE?V
004701B0  C9 CF B2 E0 42 B6 65 1A 92 70 E5 75 65 9B 0E 56  上侧 Be?V
004701C0  29 CE B2 E0 A2 B7 65 1A 72 71 E5 75 85 9A 0E 56  )尾啖 rqV
004701D0  09 CE B2 E0 82 B7 65 1A 52 71 E5 75 A5 9A 0E 56  .尾 RqV
004701E0  69 CE B2 E0 E2 B7 65 1A 32 71 E5 75 C5 9A 0E 56  i 尾噔 2qV
004701F0  49 CE B2 E0 C2 B7 65 1A 12 71 E5 75 E5 9A 0E 56  I 尾嗦 qV
00470200  A9 CE B2 E0 22 B7 65 1A F2 71 E5 75 05 9A 0E 56  ┪侧"?V
00470210  89 CE B2 E0 02 B7 65 1A D2 71 E5 75 25 9A 0E 56  侧%?V
00470220  E9 CE B2 E0 62 B7 65 1A 60 1E A5 75 9D F5 4E 56  槲侧 b\`NV
00470230  2A A1 F2 E0 AA D8 25 1A 66 1E A5 75 9D F5 4E 56  \*◎嗒?fNV
00470240  2C A1 F2 E0 AA D8 25 1A 06 1E A5 75 9D F5 4E 56  ,◎嗒?NV
00470250  4C A1 F2 E0 AA D8 25 1A 08 1E A5 75 9D F5 4E 56  L◎嗒?NV
00470260  19 D2 B2 E0 AA D8 25 1A EA 6D E5 75 9D F5 4E 56  也嗒?NV
00470270  C9 D2 B2 E0 AA D8 25 1A F6 6A E5 75 9D F5 4E 56  梢侧%NV
00470280  C5 D5 B2 E0 AA D8 25 1A 26 6B E5 75 9D F5 4E 56  耪侧%&kNV
00470290  B5 D4 B2 E0 AA D8 25 1A 32 68 E5 75 9D F5 4E 56  翟侧%2hNV
004702A0  21 D6 B2 E0 AA D8 25 1A 0A 69 E5 75 9D F5 4E 56  !植嗒?.iNV
004702B0  35 D9 B2 E0 AA D8 25 1A A2 69 E5 75 9D F5 4E 56  5 俨嗒?NV
004702C0  5D D9 B2 E0 AA D8 25 1A 16 66 E5 75 9D F5 4E 56  \]俨嗒?fNV
004702D0  5D D9 B2 E0 AA D8 25 1A 16 66 E5 75 9D F5 4E 56  \]俨嗒?fNV
004702E0  5D D9 B2 E0 AA D8 25 1A 16 66 E5 75 9D F5 4E 56  \]俨嗒?fNV
004702F0  F9 D9 B2 E0 AA D8 25 1A 1A 6C E5 75 9D F5 4E 56  侧%lNV
00470300  A5 D8 B2 E0 AA D8 25 1A 22 64 E5 75 9D F5 4E 56  ヘ侧%"dNV
00470310  E8 BF AB C0 AA D8 25 1A 35 1E A2 F5 9D F5 4E 56  杩%5ⅤNV
00470320  28 2C B2 E0 AA D8 25 1A 63 93 E5 75 9D 78 0E 56  (,侧%cuV
00470330  D6 5E 0D 1F A9 D8 2F 1A 60 1E B5 75 97 F5 5F 56  .┴/\`\_V
00470340  3B A1 F8 E0 B9 D8 AE DA 8E 1C E4 75 61 F7 0F 56  ;▲喙禺 a?V
00470350  25 A2 B3 E0 8A DB 64 1A 52 1D E4 75 DD F6 0F 56  %⒊R 蓥 V
00470360  79 A2 B3 E0 CE DB 64 1A 16 1D E4 75 19 F6 0F 56  y⒊辔?V
00470370  B1 A2 B3 E0 02 DB 64 1A DA 1D E4 75 55 F6 0F 56  雹赤?U?V

```

文件三 00473000.txt：  
代码:

```
00473000  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 F1 C2 49 56  )◎嗒?b 衤 IV
00473010  05 90 F5 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  嗒?bNV
00473020  CD 98 F5 E0 12 E9 22 1A 62 1E A5 75 9D F5 4E 56  踵?bNV
00473030  29 A1 F2 E0 80 E2 22 1A AE 2F A2 75 9D F5 4E 56  )◎??NV
00473040  29 A1 F2 E0 AA D8 25 1A 08 24 A2 75 41 C4 49 56  )◎嗒?$AV
00473050  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 2F CF 49 56  )◎嗒?b/V
00473060  C5 90 F5 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  踵%bNV
00473070  D7 9B F5 E0 AA EA 22 1A 62 1E A5 75 9D F5 4E 56  踵"bNV
00473080  29 A1 F2 E0 94 E3 22 1A 72 2C A2 75 9D F5 4E 56  )◎?r,NV
00473090  29 A1 F2 E0 AA D8 25 1A B4 21 A2 75 B9 C6 49 56  )◎嗒??蛊 IV
004730A0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 B9 B5 49 56  )◎嗒?b 沟 IV
004730B0  1D 92 F5 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  嗒?bNV
004730C0  85 E5 F5 E0 EA EC 22 1A 62 1E A5 75 9D F5 4E 56  踵觎"bNV
004730D0  29 A1 F2 E0 B2 97 22 1A B2 28 A2 75 9D F5 4E 56  )◎嗖??NV
004730E0  29 A1 F2 E0 AA D8 25 1A 4C 51 A2 75 45 C3 49 56  )◎嗒?LQEV
004730F0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 4D BA 49 56  )◎嗒?bMV
00473100  D5 97 F5 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  踵%bNV
00473110  E1 F0 F5 E0 F2 EF 22 1A 62 1E A5 75 9D F5 4E 56  狃踵蝻"bNV
00473120  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 7D 52 04 56  )◎嗒?b}RV
00473130  5D 0C B8 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  \].膏%bNV
00473140  01 10 B8 E0 AA D8 25 1A 62 1E A5 75 71 54 04 56  膏%bqTV
00473150  99 00 B8 E0 62 68 6F 1A FE B2 EF 75 9D F5 4E 56  ?膏 bhoNV
00473160  49 02 B8 E0 9A 71 6F 1A A2 BD EF 75 9D F5 4E 56  I 膏 o⒔NV
00473170  29 A1 F2 E0 36 7B 6F 1A 62 1E A5 75 3D 5B 04 56  )◎?{ob=\[V
00473180  29 A1 F2 E0 AA D8 25 1A 6A B9 EF 75 9D F5 4E 56  )◎嗒?j 癸 uNV
00473190  B1 0B B8 E0 8E 77 6F 1A 0E B7 EF 75 0D 56 04 56  ?膏 o 凤 u.VV
004731A0  29 A1 F2 E0 AA D8 25 1A 66 BB EF 75 19 55 04 56  )◎嗒?f 伙 uUV
004731B0  29 A1 F2 E0 AA D8 25 1A FA AE EF 75 9D F5 4E 56  )◎嗒?NV
004731C0  F5 10 B8 E0 16 79 6F 1A 62 1E A5 75 9D F5 4E 56  ?膏 yobNV
004731D0  29 A1 F2 E0 F6 79 6F 1A 62 1E A5 75 9D F5 4E 56  )◎圉 yobNV
004731E0  AD 0E B8 E0 AA D8 25 1A 62 1E A5 75 B1 51 04 56  ?膏%bV
004731F0  29 A1 F2 E0 6E 7D 6F 1A 62 1E A5 75 9D F5 4E 56  )◎}obNV
00473200  29 A1 F2 E0 5E 73 6F 1A 32 B9 EF 75 9D F5 4E 56  )◎so2 癸 uNV
00473210  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00473220  29 A1 F2 E0 9E 70 6F 1A 42 AE EF 75 9D F5 4E 56  )◎poBuNV
00473230  29 A1 F2 E0 3E 7A 6F 1A 62 1E A5 75 9D F5 4E 56  )◎?zobNV
00473240  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 79 56 04 56  )◎嗒?byVV
00473250  39 04 B8 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  9 膏%bNV
00473260  29 A1 F2 E0 DA 7D 6F 1A 62 1E A5 75 FD 59 04 56  )◎嘹}obV
00473270  35 09 B8 E0 E2 78 6F 1A 66 AF EF 75 9D F5 4E 56  5.膏 ofuNV
00473280  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 85 56 04 56  )◎嗒?bV
00473290  61 07 B8 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  a 膏%bNV
004732A0  29 A1 F2 E0 D6 7A 6F 1A 8E BA EF 75 75 5D 04 56  )◎嘀 zou\]V
004732B0  F9 0A B8 E0 AA D8 25 1A 96 AF EF 75 39 45 04 56  ?膏%9EV
004732C0  B1 00 B8 E0 D2 71 6F 1A 26 B9 EF 75 D5 56 04 56  ?膏 o&癸 uV
004732D0  A5 05 B8 E0 AA 77 6F 1A 62 1E A5 75 9D F5 4E 56  ?膏 obNV
004732E0  29 A1 F2 E0 D6 76 6F 1A 62 1E A5 75 9D F5 4E 56  )◎嘀 vobNV
004732F0  29 A1 F2 E0 AA D8 25 1A 8E B3 EF 75 D9 51 04 56  )◎嗒?V
00473300  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D 5C 04 56  )◎嗒?bV
00473310  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 15 5E 04 56  )◎嗒?b^V
00473320  29 A1 F2 E0 22 7A 6F 1A 62 1E A5 75 65 51 04 56  )◎?zobeQV
00473330  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 B1 52 04 56  )◎嗒?bV
00473340  29 A1 F2 E0 06 7A 6F 1A 62 1E A5 75 9D F5 4E 56  )◎?zobNV
00473350  29 A1 F2 E0 AA D8 25 1A 7E AF EF 75 9D F5 4E 56  )◎嗒?~uNV
00473360  29 A1 F2 E0 12 76 6F 1A F2 BE EF 75 51 53 04 56  )◎?vo 蚓 QSV
00473370  C1 0A B8 E0 92 72 6F 1A 62 1E A5 75 39 5F 04 56  ?膏 ob9\_V

```

文件四 00477000.txt：  
代码:

```
00477000  29 C1 B5 E0 BA B8 62 1A C6 1E E2 75 8D 85 09 56  )恋嗪?.V
00477010  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477020  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477030  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477040  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477050  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477060  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477070  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477080  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477090  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004770A0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004770B0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004770C0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004770D0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004770E0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004770F0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477100  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477110  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477120  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477130  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477140  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477150  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477160  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477170  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477180  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
00477190  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004771A0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004771B0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004771C0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004771D0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004771E0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV
004771F0  29 A1 F2 E0 AA D8 25 1A 62 1E A5 75 9D F5 4E 56  )◎嗒?bNV

```

文件五 0047F000.txt：  
代码:

```
0047F000  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 46 56  )◎啖 IG!bFV
0047F010  28 A1 F2 E0 FA D8 25 9A 60 1E A5 75 05 F5 4E D6  (◎帔?
0047F020  2A A1 F2 E0 AA D9 25 9A 67 1E A5 75 85 F4 4E D6  \*◎嗒?N
0047F030  2F A1 F2 E0 9A D9 25 9A 68 1E A5 75 4D F4 4E D6  /◎?M
0047F040  25 A1 F2 E0 52 D9 25 9A 6C 1E A5 75 DD F7 4E D6  %◎?蓣 N
0047F050  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 49 56  )◎啖 IG!bIV
0047F060  28 A1 F2 E0 F2 DA 25 9A 60 1E A5 75 ED F7 4E D6  (◎囹?眵 N
0047F070  2A A1 F2 E0 22 DA 25 9A 66 1E A5 75 3D F7 4E D6  \*◎??=
0047F080  2C A1 F2 E0 12 DA 25 9A 64 1E A5 75 4D F7 4E D6  ,◎??M
0047F090  2E A1 F2 E0 42 DA 25 9A 62 1E A5 75 95 64 2C 6D  .◎?,m
0047F0A0  29 A1 F2 E0 A1 D8 25 1A 62 14 A5 F5 9D F6 4E D6  )◎唷?bヵN
0047F0B0  39 AB F2 60 B2 DB 25 9A 7E 14 A5 F5 AD F6 4E D6  9\`槽%ヵN
0047F0C0  07 AB F2 60 E2 DB 25 9A 5C 14 A5 F5 FD F6 4E D6  \`廑%ヵN
0047F0D0  65 AB F2 60 D2 DB 25 9A 3C 14 A5 F5 0D F6 4E D6  e\`役%?ヵ.
0047F0E0  41 AB F2 60 02 DB 25 9A 10 14 A5 F5 5D F6 4E D6  A\`??ヵ\]
0047F0F0  AB AB F2 60 72 DB 25 9A EC 14 A5 F5 6D F6 4E D6  r?ヵm
0047F100  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F110  28 A1 F2 E0 A2 DC 25 9A 62 1E A5 75 95 64 2C 6D  (◎啖?,m
0047F120  29 A1 F2 E0 AB D8 25 1A CA 14 A5 F5 BD F1 4E D6  )◎喃??ヵ今 N
0047F130  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 5C 56  )◎啖 IG!b\\V
0047F140  C6 AE F2 E0 92 DC 25 9A 92 11 A5 75 CD F1 4E D6  飘蜞%婉 N
0047F150  D8 AE F2 E0 C2 DC 25 9A 90 11 A5 75 1D F1 4E D6  禺蜞萝%
0047F160  DA AE F2 E0 32 DC 25 9A 96 11 A5 75 2D F1 4E D6  诋蜞 2?-
0047F170  DC AE F2 E0 62 DC 25 9A 94 11 A5 75 7D F1 4E D6  墚蜞 b?}
0047F180  DE AE F2 E0 52 DC 25 9A 9A 11 A5 75 8D F0 4E D6  蕻蜞 R?N
0047F190  D0 AE F2 E0 82 DD 25 9A 98 11 A5 75 DD F0 4E D6  挟蜞%蒺 N
0047F1A0  D2 AE F2 E0 F2 DD 25 9A 9E 11 A5 75 ED F0 4E D6  耶蜞蜉%眇 N
0047F1B0  D4 AE F2 E0 22 DD 25 9A 9C 11 A5 75 3D F0 4E D6  援蜞"?=
0047F1C0  D6 AE F2 E0 12 DD 25 9A 62 0E A5 75 4D F0 4E D6  之蜞?M
0047F1D0  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9E F5 4E 56  )◎啖 IG!bNV
0047F1E0  E9 AB F2 60 42 DD 25 9A AC 14 A5 F5 9D F3 4E D6  楂 B?ヵN
0047F1F0  CF AB F2 60 B2 DE 25 9A 62 1E A5 75 95 64 2C 6D  汐厕%,m
0047F200  29 A1 F2 E0 AA D8 22 1A 9B 61 A5 75 AD F3 4E D6  )◎嗒?N
0047F210  D3 DE F2 E0 E2 DE 25 9A 99 61 A5 75 FD F3 4E D6  愚蜞廪%aN
0047F220  D5 DE F2 E0 D2 DE 25 9A 9F 61 A5 75 0D F3 4E D6  辙蜞肄%a.
0047F230  D7 DE F2 E0 02 DE 25 9A 9D 61 A5 75 5D F3 4E D6  邹蜞?a\]
0047F240  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9C F5 4E 56  )◎啖 IG!bNV
0047F250  DD AB F2 60 72 DE 25 9A 62 1E A5 75 95 64 2C 6D  莴 r?,m
0047F260  29 A1 F2 E0 AA D8 24 1A 62 1E A5 75 6D F3 4E 56  )◎嗒?bmV
0047F270  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F280  29 A1 F2 E0 AA DF 25 1A 62 1E A5 75 95 64 2C 6D  )◎嗒?b,m
0047F290  29 A1 F2 E0 AA D8 24 1A 62 1E A5 75 8D F2 4E 56  )◎嗒?bNV
0047F2A0  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F2B0  29 A1 F2 E0 8A DF 25 1A 62 1E A5 75 95 64 2C 6D  )◎?b,m
0047F2C0  29 A1 F2 E0 AA D8 24 1A 62 1E A5 75 AD F2 4E 56  )◎嗒?bNV
0047F2D0  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F2E0  29 A1 F2 E0 EA DF 25 1A 62 1E A5 75 95 64 2C 6D  )◎嚓?b,m
0047F2F0  29 A1 F2 E0 AA D8 24 1A 62 1E A5 75 CD F2 4E 56  )◎嗒?b 万 NV
0047F300  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F310  29 A1 F2 E0 CA DF 25 1A 62 1E A5 75 95 64 2C 6D  )◎嗍?b,m
0047F320  29 A1 F2 E0 AA D8 24 1A 62 1E A5 75 ED F2 4E 56  )◎嗒?b 眚 NV
0047F330  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F340  29 A1 F2 E0 2A DF 25 1A 62 1E A5 75 95 64 2C 6D  )◎??b,m
0047F350  29 A1 F2 E0 AA D8 24 1A 62 1E A5 75 0D F2 4E 56  )◎嗒?b.V
0047F360  29 A1 F2 E0 A2 49 47 21 62 1E A5 75 9D F5 4F 56  )◎啖 IG!bOV
0047F370  29 A1 F2 E0 0A DF 25 1A 62 1E A5 75 95 64 2C 6D  )◎??b,m

```

\============================  
7、有了上面的每个解码区段第二次解码后的结果，我们就可以进行正确解码 XOR 数值的猜测了，猜测的方法在第 8 步细说。  
在猜出了解码后，不管解码是否正确，我们都应该尝试解码，然后根据结果加以判断猜出来的是否正确，下面是解码脚本函数。  
去掉上面的 GuessXor 各个函数，替换为如下代码：  
代码:

```
//还原解码
RestoreDecode:
        var XorAddress
        mov XorAddress,12FEE0                //解码 XOR 数值起始位置
        var XorAddressEnd
        mov XorAddressEnd,XorAddress
        add XorAddressEnd,F
        var XorNum
        mov XorNum,5                        //解码次数-即解码区段数
        bphws XorAddressEnd,"w"
RestoreDecodeLoop:
        run
        find XorAddress,#F1D3FF8443297732862DF21DC4E57262#                //错误解码 XOR 数值
        cmp $RESULT,XorAddress
        jne RestoreDecodeLoop
        mov XorValueFoundAddress,$RESULT
        repl XorValueFoundAddress,#F1D3FF8443297732862DF21DC4E57262#,#29A1F2E0AAD8251A621EA5759DF54E56#,10        //填入错误与正确的解码 XOR 数值
        mov XorValueFoundAddress,0
        dec XorNum
        cmp XorNum,0
        jne RestoreDecodeLoop
        bphwcall

```

然后，重载程序，运行脚本，脚本完成后，如果按 F9 运行程序成功，则解码成功，如果不成功，则重新进行解码 XOR 数值的猜解。  
\============================  
8、在完成第 6 步后，我们有了上面的每个解码区段第二次解码后的结果，我们就可以进行正确解码 XOR 数值的猜测了。  
猜测过程省略，猜测方法有以下几个准则，请自行灵活应用：  
a.解码 XOR 数值长度为 0x10，然后循环往复地进行解码。所以每一行解码结果就是一个解码 XOR 数值调用的循环。如果是通过 IAT 表进行解码 XOR 数值的猜解，可以把 0x10 长度的解码 XOR 数值平均分成 4 段来猜解，每段 4 个十六进制数。  
b.不管变量 A 等于多少，xor A,A，结果 A 都是变为 0 的，这就是 XOR 的其中一种作用，用于清空某个数值。  
**已知: A xor B = 0**  
**\=>**  
**结论: A=B。**  
所以如果第三次解码后结果该是 00 的地方，那么第二次解码时这个位置的数值是多少，其对应的解码 XOR 数值就该是多少。这是猜解码的核心原理。而在 IAT 所在区段后的几个区段中通常会有一个区段，一眼看过去，每一个列都有大量重复的数值，而这些数值就有可能正是正确的解码 XOR 数值。  
同样的，如果知道了 XOR 结果应该为多少，而第二次解码后数值为多少，那么也是可以推断出解码 XOR 数值的。  
**已知: A xor B = C，A 和 C 已知**  
**\=>**  
**结论: B=A xor C**  
c.IAT 表的正确解码的结果应该是存在规律的，例如我这个程序的正确解码结果：  
代码:

```
0047312C  004AA7E0  程序.004AA7E0
00473130  004AAD74  程序.004AAD74
00473134  00D00000
00473138  00D0000E
0047313C  00D0001C
00473140  004AB128  程序.004AB128
00473144  00D0002A
00473148  00D00038
0047314C  004AA1EC  程序.004AA1EC
00473150  004AA1B0  程序.004AA1B0
00473154  004AB0C8  程序.004AB0C8
00473158  004AAC9C  程序.004AAC9C
0047315C  00D00046
00473160  004AA360  程序.004AA360
00473164  004AA930  程序.004AA930
00473168  004AA3C0  程序.004AA3C0
0047316C  00D00054
00473170  00D00062
00473174  004AA39C  程序.004AA39C
00473178  00D00070
0047317C  004AAEA0  程序.004AAEA0
00473180  00A73E10
00473184  00D0007E
00473188  004AA708  程序.004AA708
0047318C  00D0008C
00473190  004AAA98  程序.004AAA98
00473194  004AAF24  程序.004AAF24
00473198  004AA96C  程序.004AA96C
0047319C  004AA390  程序.004AA390
004731A0  00D0009A
004731A4  00D000A8
004731A8  004AA504  程序.004AA504
004731AC  004AA084  程序.004AA084
004731B0  00D000B6
004731B4  00000000                                        //不同 DLL 间的间隔
004731B8  004AB098  程序.004AB098
004731BC  00D000C4

```

我们可以从中看出，主要的结构有两种，004AXXXX 和 00D0XXXX，并且不同 DLL 间的间隔符号绝对是 00000000。  
所以如果我们看到 IAT 表中一行结果有很多个 0，那么我可以猜测这个数值应该是间隔符号，应该全部为 0，而不为 0 的位则代表了猜测的那位 XOR 数值是错的，需要更改。  
而另一种很容易通过 IAT 表判断的情况是这样：  
代码:

```
00465FFC  00000000
00466000  0050B468  1\_0zc.0050B468
00466004  7EFCF878
00466008  0050BA80  1\_0zc.0050BA80
0046600C  0050BBAC  1\_0zc.0050BBAC
00466010  0050BDC8  1\_0zc.0050BDC8
00466014  7EAC4350
00466018  0050B3C0  1\_0zc.0050B3C0
0046601C  0050B984  1\_0zc.0050B984
00466020  00000000
00466024  7EFCF2BC
00466028  0050B2E8  1\_0zc.0050B2E8
0046602C  0050BDF8  1\_0zc.0050BDF8
00466030  0050B330  1\_0zc.0050B330
00466034  7EFCF424
00466038  0050B8D0  1\_0zc.0050B8D0
0046603C  0050B4C8  1\_0zc.0050B4C8
00466040  0050B264  1\_0zc.0050B264
00466044  7EFCF7B0
00466048  0050B930  1\_0zc.0050B930
0046604C  0050BD2C  1\_0zc.0050BD2C
00466050  0050C218  1\_0zc.0050C218
00466054  7EFCFF5C

```

解码后，我们发现 IAT 表中，每隔 4 行的数值都跟其他行的结构很不一致，那么证明猜测出来的解码 XOR 数值的那 4 位是有问题的，而其他 12 位至少开头两位是没问题的，因为他们的结构一致。  
d.在除了 IAT 所在解码区段外的其他几个区段，一般有一些区段里在正确解码后是含有**有意义的英文单词**的。如下：  
  
代码:

```
00470070  72 00 8B C0 52 75 6E 74 69 6D 65 20 65 72 72 6F  r.Runtime erro
00470080  72 20 20 20 20 20 61 74 20 30 30 30 30 30 30 30  r     at 0000000
00470090  30 00 8B C0 30 31 32 33 34 35 36 37 38 39 41 42  0.0123456789AB

```

通过这些单词的正确与否，我们可以确认解码 XOR 数值的哪些位可能是正确的，哪些位是不正确的。上面的"Runtime error"看起来是正确的，那么他们对应的第 0、4~16 位的解码 XOR 数值都是对的了。  
e.判断解码是否正确的终极标准，就是程序可以正常运行。\============================  
9、在猜解完成后，我们要填写好我们的破解记录文件 cracking.txt，这样才算是真正完成了破解。  
**1、真正 OEP：esp 定律=>12FFA0=>46F610**  
**2、IATSTART：473000=>47312C**  
**3、密钥位置：12FEE0~12FEEF**  
**数值：0012FEE0 F1 D3 FF 84 43 29 77 32 86 2D F2 1D C4 E5 72 62**  
**4、解码代码段：401000、470000、473000、477000、47F000**  
**5、正确解码：29A1F2E0AAD8251A621EA5759DF54E56**  
**6、IATEND：473768**  
**PS：一些注意事项**  
a.在成功解码后，要进行脱壳，还必须要先修复 IAT 以及阻止模拟 DLL，本文中的脚本并不提供这些功能，请自行解决。  
具体的文章可以看 hyperchem 和 ximo 的文章。hyperchem 大大的文章通过论坛搜索“ZPROTECT”可以找到，基本都是精华帖。  
而 ximo 的文章主要在吾爱破解论坛，还有以下这篇：  
[简单分析下 ZProtect 1.4.9 的 DLL 模拟](http://80dfj.org/article.asp?id=11)  
b.第 8 步只是我所想出来的一些解码 XOR 数值的猜测技巧，上文中的未必能适用全部程序，请在理解原理的前提下，依葫芦画瓢，自己揣摩。  
c.一般而言，阻止模拟 DLL 和正确解码后，IAT 修复就可以正常进行，IAT 修复的结果，正确的一般如下：  
代码:

```
00466000  77DA6C27  ADVAPI32.RegCloseKey
00466004  77DA7852  ADVAPI32.RegOpenKeyExA
00466008  77DAEAE7  ADVAPI32.RegSetValueExA
0046600C  77DCBB8D  ADVAPI32.RegQueryValueA
00466010  77DAE9F4  ADVAPI32.RegCreateKeyExA
00466014  00000000
00466018  5D1803D8  COMCTL32.ImageList\_Destroy
0046601C  5D1765CF  COMCTL32.InitCommonControls
00466020  00000000
00466024  77EF8597  GDI32.SetStretchBltMode
00466028  77EF6AD6  GDI32.GetClipRgn
0046602C  77EFA8BA  GDI32.CreatePolygonRgn
00466030  77EF7AA0  GDI32.SelectClipRgn
00466034  77EF6BFA  GDI32.DeleteObject
00466038  77EFAD23  GDI32.CreateDIBitmap
0046603C  77EFBA9E  GDI32.GetSystemPaletteEntries
00466040  77EFB5EA  GDI32.CreatePalette
00466044  77EFB6D0  GDI32.StretchBlt

```

如果里面还是存在没有直指系统 DLL 的 API 地址的存在。。那么通常都不可能脱壳成功。我也没弄懂，为什么么阻止了模拟 DLL 还是有这样的情况。