
# Bookup 2000 Express Build30 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-168.htm`](https://www.pediy.com/kssd/pediy08/pediy8-168.htm)

万里长征第四步：Bookup 2000 Express Build30 注册算法分析  

解密者：冲天剑@pediy.com  

工具：`PEID 0.94`，`OllyICE 1.10`  

## 0. 导言  

Bookup 是个用于研究和学习国际象棋开局的软件，据其作者称他可以保证你使用 9 个月之内，国际象棋水平有显著提高，否则全额退款。当然，卖瓜的都是王婆，他的程序究竟怎么样，用的人心里明白。  

网址：`hxxp://www.bookup.com`（自行改正）  

上有简易（Express）版下载，不过要求输入电子邮件之类个人信息，随便捏造个地址就行了。这个程序仍然在不断地重新编译，自从 1 月以来，已经重编译了 3 次，由那时的 `Build27` 更新到现在的 `Build30`。每次编译，注册码都不一样，上一版本的注册码无法在下一版本中使用，这也就是标题中注明是第几次 Build 的原因。如果你下载的是最新的 `Build`，那么注册算法可能会改变，只能自己跟注册码了。好在跟注册码本身比较简单，用不了半个小时的时间，不象这里搞算法分析，两者的难度是不可同日而语的。  

## 1. 解密  

(1) `peid` 查壳，显示 `Borland Delphi 4.0 - 5.0`。（画外音：软柿子又来了……）是吗？那你咬咬看，呵呵！  

(2) 运行原程序，试注册，以便了解注册流程。注册码格式是 16 字符序列号，没有用户名信息，呈现形式：

```
XXXX XXXX XXXX XXXX
```

随便输入一组字符，出现提示信息：

```
The key was incorrect.  The program will continue in Lite mode.
```

然后 `OD` 载入，查找此字串，来到：

```
0050DABE   .  8B45 FC       mov     eax, [ebp-4]  
0050DAC1   .  E8 56F2FFFF   call    0050CD1C  
0050DAC6   .  84C0          test    al, al                            ;  返回非０为注册成功，０为失败  
0050DAC8   .  74 11         je      short 0050DADB                    ;  关键跳转  
0050DACA   .  8B45 FC       mov     eax, [ebp-4]  
0050DACD   .  E8 5EF1FFFF   call    0050CC30  
0050DAD2   .  C605 80235500>mov      byte ptr [552380], 1  
0050DAD9   .  EB 38         jmp     short 0050DB13  
0050DADB   >  8B45 FC       mov     eax, [ebp-4]  
0050DADE   .  E8 05F6FFFF   call    0050D0E8  
0050DAE3   .  84C0          test    al, al  
0050DAE5   .  74 17         je      short 0050DAFE  
0050DAE7   .  6A 00         push    0                                   ; /Arg1 = 00000000  
0050DAE9   .  66:8B0D 34DB5>mov      cx, [50DB34]                       ; |  
0050DAF0   .  B2 02         mov     dl, 2                               ; |  
0050DAF2   .  B8 5CDC5000   mov     eax, 0050DC5C                       ; |ASCII "This key is for an older version.  The program will continue in Lite mode."  
0050DAF7   .  E8 F8DAF4FF   call    0045B5F4                            ; \\Bkup2kE.0045B5F4  
0050DAFC   .  EB 15         jmp     short 0050DB13  
0050DAFE   >  6A 00         push    0                                   ; /Arg1 = 00000000  
0050DB00   .  66:8B0D 34DB5>mov      cx, [50DB34]                       ; |  
0050DB07   .  B2 02         mov     dl, 2                               ; |  
0050DB09   .  B8 B0DC5000   mov     eax, 0050DCB0                       ; |ASCII "The key was incorrect.  The program will continue in Lite mode."  
0050DB0E   .  E8 E1DAF4FF   call    0045B5F4                            ; \\Bkup2kE.0045B5F4  
```

关键跳转似乎挺容易找的，啊！可接下来就犯迷糊了，在 `0050DAC1` 这个 `CALL` 前一句下断点，竟然什么都没有，输入的假码也不知道哪去了。跟进过程 `0050D0E8`，也是什么都没找到。怎么回事？再往前面找找，无意中看到这么一句：

```
0050D9CA   .  BA 9CDB5000   mov     edx, 0050DB9C                       ;  ASCII "5555444433332222"
```

赫赫～～这个 `"5555444433332222"` 倒是挺象注册码的，不管怎么说，至少格式相符嘛。进入原程序输入这个字串，出现提示信息：

```
The temporary code was successfully installed.  Please check your email in the next few days for your permanent code.
```

原来这只是为了等候作者邮寄注册码而略微延长试用期限的临时码。反正暂时跟不到注册码是怎样处理的，不妨先跟跟临时码看看：

```
0050DFBC  /$  55             push    ebp  
0050DFBD  |.  8BEC           mov     ebp, esp  
0050DFBF  |.  83C4 D0        add     esp, -30                          ;  局部变量空间 dword*12  
0050DFC2  |.  33D2           xor     edx, edx  
0050DFC4  |.  8955 F8       mov     [ebp-8], edx  
0050DFC7  |.  8945 FC       mov     [ebp-4], eax  
0050DFCA  |.  33C0           xor     eax, eax  
//此段代码略  
0050E079  |> \BA 18E25000    mov     edx, 0050E218                       ;  ASCII "TemporaryCode build30"  
0050E07E  |.  8B45 E8       mov     eax, [ebp-18]  
0050E081  |.  E8 762AF5FF   call    00460AFC  
0050E086  |.  84C0           test    al, al                            ;  判别是否首次输入临时码  
0050E088  |.  0F85 AA000000 jnz     0050E138  
0050E08E  |.  E8 B9C3EFFF   call    0040A44C                           ;  生成浮点数  
0050E093  |.  DD5D D8       fstp    qword ptr [ebp-28]  
0050E096  |.  9B             wait  
0050E097  |.  DD45 D8       fld     qword ptr [ebp-28]  
0050E09A  |.  D805 30E25000 fadd    dword ptr [50E230]                 ;  ds:[0050E230]=10.00000  
0050E0A0  |.  DD5D D0       fstp    qword ptr [ebp-30]  
0050E0A3  |.  9B             wait  
0050E0A4  |.  FF75 DC       push    dword ptr [ebp-24]                 ; /Arg4  
0050E0A7  |.  FF75 D8       push    dword ptr [ebp-28]                 ; |Arg3  
0050E0AA  |.  FF75 D4       push    dword ptr [ebp-2C]                 ; |Arg2  
0050E0AD  |.  FF75 D0       push    dword ptr [ebp-30]                 ; |Arg1  
0050E0B0  |.  8D55 F0       lea     edx, [ebp-10]                       ; |  
0050E0B3  |.  A1 BC805400   mov     eax, [5480BC]                       ; |此过程为生成注册码的过程  
0050E0B8  |.  E8 37E6F5FF   call    0046C6F4                            ; \\Bkup2kE.0046C6F4  
0050E0BD  |.  8D4D F8       lea     ecx, [ebp-8]  
0050E0C0  |.  8D45 F0       lea     eax, [ebp-10]  
0050E0C3  |.  BA 08000000   mov     edx, 8  
0050E0C8  |.  E8 D731F5FF   call    004612A4                            ;  将注册码由 16 进制数值转化为 ASCII 串  
0050E0CD  |.  8B45 FC       mov     eax, [ebp-4]  
0050E0D0  |.  05 B4080000   add     eax, 8B4  
0050E0D5  |.  8B55 F8       mov     edx, [ebp-8]                         ;  此处出现注册码  
0050E0D8  |.  E8 275DEFFF   call    00403E04  
```

当程序运行到 `0050E0D5` 这一句时，`edx` 中出现了一个奇怪的字符串，跟注册码格式一样。用这个串去给程序注册，竟然成功了！怎么会这样？按照常理，这种注册方式的原理都是判定输入的内容经过某一特定变换后，是否满足某一组特定规则。莫不成这个程序更绝，竟然自带注册机？  

然而这也只是推测的一方面，毕竟上面这段代码只有输入的是临时码 `"5555444433332222"` 时才会被执行到，如果输入其他假码，早在进入这个过程以前，流程就已经跳到别处了。因而这里出现的也有可能只是一个常字符串而已。为了弄明白这一点，输入其他假码的时候通过修改标志位让流程进入此段代码，结果是出现的真码不变。（注：在寄存器窗口选中某一标志位的值右击即可修改）  

这么说来，真码并不依赖于输入，那么它依赖于什么呢？跟进 `004612A4` 这个过程，发现它是把某一内存地址处的 16 进制数值转换成 `ASCII` 字符，便形成了在 `edx` 中所看到的注册码。记下这个内存地址：`qword ptr ss:[0012F9D8]`，在数据窗口中（`ds` 和 `ss` 共用一个选择子）转到这个地址，（注：右击选择“转到——＞表达式”输入 `0012F9D8`）看程序是何时往这个地址中写入数据的，结果来到：

```
0046C6F4  /$  55             push    ebp  
0046C6F5  |.  8BEC           mov     ebp, esp  
0046C6F7  |.  83C4 F8        add     esp, -8                           ;  局部变量空间 dword*2  
0046C6FA  |.  8955 F8       mov     [ebp-8], edx  
0046C6FD  |.  8945 FC       mov     [ebp-4], eax  
0046C700  |.  8B45 F8       mov     eax, [ebp-8]                         ;  EAX=注册码存放地址  
0046C703  |.  66:C700 CBA4  mov      word ptr [eax], 0A4CB  
0046C708  |.  8B45 F8       mov     eax, [ebp-8]  
0046C70B  |.  66:C740 02 00>mov      word ptr [eax+2], 0  
0046C711  |.  FF75 14       push    dword ptr [ebp+14]                 ; /Arg2  
0046C714  |.  FF75 10       push    dword ptr [ebp+10]                 ; |Arg1  
0046C717  |.  E8 74FFFFFF   call    0046C690                            ; \\Bkup2kE.0046C690  
0046C71C  |.  8B55 F8       mov     edx, [ebp-8]  
0046C71F  |.  66:8942 04    mov     [edx+4], ax  
0046C723  |.  FF75 0C       push    dword ptr [ebp+C]                  ; /Arg2  
0046C726  |.  FF75 08       push    dword ptr [ebp+8]                  ; |Arg1  
0046C729  |.  E8 62FFFFFF   call    0046C690                            ; \\Bkup2kE.0046C690  
0046C72E  |.  8B55 F8       mov     edx, [ebp-8]  
0046C731  |.  66:8942 06    mov     [edx+6], ax  
0046C735  |.  8B55 F8       mov     edx, [ebp-8]                         ;  入口参数 1:[ebp-8]=0x0012f9d8=存放注册码的内存地址  
0046C738  |.  8B45 FC       mov     eax, [ebp-4]                         ;  入口参数 2:[ebp-4]=00546E24  
0046C73B  |.  B1 01          mov     cl, 1  
0046C73D  |.  E8 0AF7FFFF   call    0046BE4C  
0046C742  |.  59             pop     ecx  
0046C743  |.  59             pop     ecx  
0046C744  |.  5D             pop     ebp  
```



0046C745&nbsp;&nbsp;\\.&nbsp;&nbsp;C2&nbsp;1000&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;retn&nbsp;&nbsp;&nbsp;&nbsp;10

////////////////&nbsp;&nbsp;&nbsp;&nbsp;以上是代码&nbsp;&nbsp;&nbsp;&nbsp;///////////////

在调用 `004612A4` 处的字符转换子程序前最后一次对 `qword ptr [0012F9D8]`

写入数据是在 `0046C73D` 处的 `call 0046BE4C` 语句，跟进这个过程中一看：

////////////////&nbsp;&nbsp;&nbsp;&nbsp;以下是代码&nbsp;&nbsp;&nbsp;&nbsp;///////////////

```
0046BE4C       55            push    ebp

0046BE4D       8BEC          mov     ebp, esp

0046BE4F       83C4 D4       add     esp, -2C                ;  局部变量空间 dword*11

0046BE52  |.  884D F7       mov     [ebp-9], cl             ;  cl=1

0046BE55  |.  8955 F8       mov     [ebp-8], edx            ;  loc_2

0046BE58  |.  8945 FC       mov     [ebp-4], eax            ;  loc_1

0046BE5B  |.  8B45 F8       mov     eax, [ebp-8]

0046BE5E  |.  8B00          mov     eax, [eax]

0046BE60  |.  8945 EC       mov     [ebp-14], eax           ;  loc_5 = 注册码首个双字（现为 0x0000A4CB）

0046BE63  |.  8B45 F8       mov     eax, [ebp-8]

0046BE66  |.  8B40 04       mov     eax, [eax+4]

0046BE69  |.  8945 E8       mov     [ebp-18], eax           ;  loc_6 = 注册码第二个双字

0046BE6C  |.  33C0          xor     eax, eax

0046BE6E  |.  8945 E4       mov     [ebp-1C], eax           ;  loc_7 = 0;

0046BE71  |>  8B45 EC      /mov     eax, [ebp-14]           ;  do

0046BE74  |.  8945 E0       |mov    [ebp-20], eax           ;  loc_8 = loc_5;

0046BE77  |.  8B45 E4       |mov    eax, [ebp-1C]

0046BE7A  |.  8D0440        |lea    eax, [eax+eax*2]        ;  eax=eax*3

0046BE7D  |.  33D2          |xor    edx, edx

0046BE7F  |.  8A55 F7       |mov    dl, [ebp-9]

0046BE82  |.  03D2          |add    edx, edx

0046BE84  |.  8D1452        |lea    edx, [edx+edx*2]        ;  edx=6

0046BE87  |.  8D14D5 B46D54>|lea    edx, [edx*8+546DB4]    ;  edx=0x546de4

0046BE8E  |.  8B0482        |mov    eax, [edx+eax*4]        ;  [546de4]=3

0046BE91  |.  8B55 FC       |mov    edx, [ebp-4]            ;  edx=0x546e24

0046BE94  |.  8B0482        |mov    eax, [edx+eax*4]

0046BE97  |.  8945 DC       |mov    [ebp-24], eax           ;  loc_9 = ?

0046BE9A  |.  8B45 E4       |mov    eax, [ebp-1C]

0046BE9D  |.  8D0440        |lea    eax, [eax+eax*2]

0046BEA0  |.  33D2          |xor    edx, edx

0046BEA2  |.  8A55 F7       |mov    dl, [ebp-9]

0046BEA5  |.  03D2          |add    edx, edx

0046BEA7  |.  8D1452        |lea    edx, [edx+edx*2]

0046BEAA  |.  8D14D5 B46D54>|lea    edx, [edx*8+546DB4]

0046BEB1  |.  8B4482 04     |mov    eax, [edx+eax*4+4]

0046BEB5  |.  8B55 FC       |mov    edx, [ebp-4]

0046BEB8  |.  8B0482        |mov    eax, [edx+eax*4]

0046BEBB  |.  8945 D8       |mov    [ebp-28], eax           ;  loc_10 = ?

0046BEBE  |.  8B45 E4       |mov    eax, [ebp-1C]

0046BEC1  |.  8D0440        |lea    eax, [eax+eax*2]

0046BEC4  |.  33D2          |xor    edx, edx

0046BEC6  |.  8A55 F7       |mov    dl, [ebp-9]

0046BEC9  |.  03D2          |add    edx, edx

0046BECB  |.  8D1452        |lea    edx, [edx+edx*2]

0046BECE  |.  8D14D5 B46D54>|lea    edx, [edx*8+546DB4]

0046BED5  |.  8B4482 08     |mov    eax, [edx+eax*4+8]

0046BED9  |.  8B55 FC       |mov    edx, [ebp-4]

0046BEDC  |.  8B0482        |mov    eax, [edx+eax*4]

0046BEDF  |.  8945 D4       |mov    [ebp-2C], eax           ;  loc_11 = ?

0046BEE2  |.  8B45 D4       |mov    eax, [ebp-2C]

0046BEE5  |.  0145 E0       |add    [ebp-20], eax           ;  loc_8 = loc_8 + loc_11;

0046BEE8  |.  8B45 E0       |mov    eax, [ebp-20]

0046BEEB  |.  0145 D4       |add    [ebp-2C], eax           ;  loc_11 = loc_11 + loc_8;

0046BEEE  |.  8B45 E0       |mov    eax, [ebp-20]

0046BEF1  |.  C1E8 07       |shr    eax, 7

0046BEF4  |.  3145 E0       |xor    [ebp-20], eax           ;  loc_8 = loc_8 ^ (loc_8 >> 7);

0046BEF7  |.  8B45 E0       |mov    eax, [ebp-20]

0046BEFA  |.  0145 DC       |add    [ebp-24], eax           ;  loc_9 = loc_9 + loc_8;

0046BEFD  |.  8B45 DC       |mov    eax, [ebp-24]

0046BF00  |.  0145 E0       |add    [ebp-20], eax           ;  loc_8 = loc_8 + loc_9;

0046BF03  |.  8B45 DC       |mov    eax, [ebp-24]

0046BF06  |.  C1E0 0D       |shl    eax, 0D

0046BF09  |.  3145 DC       |xor    [ebp-24], eax           ;  loc_9 = loc_9 ^ (loc_9 << 13);

0046BF0C  |.  8B45 DC       |mov    eax, [ebp-24]

0046BF0F  |.  0145 D8       |add    [ebp-28], eax           ;  loc_10 = loc_10 + loc_9;

0046BF12  |.  8B45 D8       |mov    eax, [ebp-28]

0046BF15  |.  0145 DC       |add    [ebp-24], eax           ;  loc_9 = loc_9 + loc_10;

0046BF18  |.  8B45 D8       |mov    eax, [ebp-28]

0046BF1B  |.  C1E8 11       |shr    eax, 11

0046BF1E  |.  3145 D8       |xor    [ebp-28], eax           ;  loc_10 = loc_10 ^ (loc_10 >> 17);

0046BF21  |.  8B45 D8       |mov    eax, [ebp-28]

0046BF24  |.  0145 D4       |add    [ebp-2C], eax           ;  loc_11 = loc_11 + loc_10;

0046BF27  |.  8B45 D4       |mov    eax, [ebp-2C]

0046BF2A  |.  0145 D8       |add    [ebp-28], eax           ;  loc_10 = loc_10 + loc_11;

0046BF2D  |.  8B45 D4       |mov    eax, [ebp-2C]

0046BF30  |.  C1E0 09       |shl    eax, 9

0046BF33  |.  3145 D4       |xor    [ebp-2C], eax           ;  loc_11 = loc_11 ^ (loc_11 << 9);

0046BF36  |.  8B45 D4       |mov    eax, [ebp-2C]

0046BF39  |.  0145 E0       |add    [ebp-20], eax           ;  loc_8 = loc_8 + loc_11;

0046BF3C  |.  8B45 E0       |mov    eax, [ebp-20]

0046BF3F  |.  0145 D4       |add    [ebp-2C], eax           ;  loc_11 = loc_11 + loc_8;

0046BF42  |.  8B45 E0       |mov    eax, [ebp-20]

0046BF45  |.  C1E8 03       |shr    eax, 3

0046BF48  |.  3145 E0       |xor    [ebp-20], eax           ;  loc_8 = loc_8 ^ (loc_8 >> 3);

0046BF4B  |.  8B45 E0       |mov    eax, [ebp-20]

0046BF4E  |.  0145 DC       |add    [ebp-24], eax           ;  loc_9 = loc_9 + loc_8;

0046BF51  |.  8B45 DC       |mov    eax, [ebp-24]

0046BF54  |.  C1E0 07       |shl    eax, 7

0046BF57  |.  3145 DC       |xor    [ebp-24], eax           ;  loc_9 = loc_9 ^ (loc_9 << 7);

0046BF5A  |.  8B45 DC       |mov    eax, [ebp-24]

0046BF5D  |.  0145 D8       |add    [ebp-28], eax           ;  loc_10 = loc_10 + loc_9;

0046BF60  |.  8B45 D4       |mov    eax, [ebp-2C]

0046BF63  |.  C1E8 0F       |shr    eax, 0F

0046BF66  |.  3145 D8       |xor    [ebp-28], eax           ;  loc_10 = loc_10 ^ (loc_11 >> 15);

0046BF69  |.  8B45 D8       |mov    eax, [ebp-28]

0046BF6C  |.  0145 D4       |add    [ebp-2C], eax           ;  loc_11 = loc_11 + loc_10;

0046BF6F  |.  8B45 D4       |mov    eax, [ebp-2C]

0046BF72  |.  C1E0 0B       |shl    eax, 0B

0046BF75  |.  3145 D4       |xor    [ebp-2C], eax           ;  loc_11 = loc_11 ^ (loc_11 << 11);

0046BF78  |.  8B45 E8       |mov    eax, [ebp-18]

0046BF7B  |.  3345 D4       |xor    eax, [ebp-2C]

0046BF7E  |.  8945 F0       |mov    [ebp-10], eax           ;  loc_4 = loc_6 ^ loc_11;

0046BF81  |.  8B45 EC       |mov    eax, [ebp-14]

0046BF84  |.  8945 E8       |mov    [ebp-18], eax           ;  loc_6 = loc_5;

0046BF87  |.  8B45 F0       |mov    eax, [ebp-10]

0046BF8A  |.  8945 EC       |mov    [ebp-14], eax           ;  loc_5 = loc_4;

0046BF8D  |.  FF45 E4       |inc    dword ptr [ebp-1C]      ;  loc_7 = loc_7 + 1;
```



```
0046BF90  |.  837D E4 04    |cmp     dword ptr [ebp-1C], 4           ;  while(loc7 < 4);  
0046BF94  |.^ 0F85 D7FEFFFF \jnz     0046BE71  
0046BF9A  |.  8B45 F8       mov     eax, [ebp-8]  
0046BF9D  |.  8B55 E8       mov     edx, [ebp-18]                     ;  loc_6  
0046BFA0  |.  8910          mov     [eax], edx  
0046BFA2  |.  8B45 F8       mov     eax, [ebp-8]  
0046BFA5  |.  8B55 EC       mov     edx, [ebp-14]                     ;  loc_5  
0046BFA8  |.  8950 04       mov     [eax+4], edx  
0046BFAB  |.  8BE5          mov     esp, ebp  
0046BFAD  |.  5D            pop     ebp  
0046BFAE  \.  C3            retn  
```

老天，我看到这堆运算的第一个反应是几乎吐血，虽然注册码的值最后是由 `loc_5` 和 `loc_6` 两个变量写入的，但它的运算过程中用到了 `loc_7`（循环控制变量）到 `loc_11` 中所有的值，没有办法精简掉一些指令！如果仅仅是指令多些也还罢了，更恼人的是读入 `loc_9` 到 `loc_11` 的值的时候用了两级寄存器间接寻址，要知道，指针值随便修改一点，所指向的内容就可能相差十万八千里，何况还是两级指针！这要从何跟起？到这里已经用了我一整天时间，真有点想就此放弃，索性先去睡觉了。

第二天回来，再按头天的方法跟注册码，发现注册码变了。看来软件中确实有个 keygen 过程，而不仅仅是常值注册码那么简单！同时经过一夜的整理思路，对于那个寄存器二级寻址的问题好歹有了点头绪。首先找到一级间址所在的内存地址：

```
loc_9  = [546e24+[546de4+loc_7*0C]*4]
loc_10 = [546e24+[546de4+loc_7*0C+4]*4]
loc_11 = [546e24+[546de4+loc_7*0C+8]*4]
```

由于循环控制变量 `loc_7` 从 0 到 3，一级间址就是 `[546de4]` 到 `[546de4+2C]`，在数据窗口中找到这片内存：

```
00546DE4  03 00 00 00 02 00 00 00 00 00 00 00 01 00 00 00  ............  
00546DF4  00 00 00 00 02 00 00 00 02 00 00 00 01 00 00 00  ............  
00546E04  03 00 00 00 00 00 00 00 03 00 00 00 01 00 00 00  ............  
```

设置内存写入断点，没有发现程序往这片内存中写入数据，也就意味着这片内存的值是一些常数！再一看每个 `dword` 的值都不超过 3，代入到二级间址的表达式中，也就意味着 `loc_9` 到 `loc_11` 所读取的内容是在从 `[546e24]` 起始的 4 个 `dword` 中！同样在数据窗口中找到这片内存：

```
00546E24  CC 20 0C 71 C5 18 27 FF 20 7F 86 F3 DC 15 A0 ED  ?.q?'...?
```

也没有发现程序往其中写入数据！尝试把这 4 个 `dword` 的数值修改，跟出的注册码便无效了。这么说，这串数值可能是某种密钥。再看 `0046BE60` 和 `0046BE69` 两句，说明了这整个过程就是用这个密钥对注册码存放的内存进行变换。回到过程 `0046C6F4` 中看变换前的数据，`dword ptr [0012F9D8]` 的内容很明白是 `0000A4CB`，而 `dword ptr [0012F9DC]` 由两次过程调用 `call 0046C690` 给其赋值，过程的参数分别是 `qword ptr [0012F998]` 和 `qword ptr [0012F990]`。这两个 `qword` 是父过程传递进来的参数，回到父过程 `0050DFBC` 中，得知它们原来的地址是 `qword ptr ss:[0012F9C0]` 和 `qword ptr ss:[0012F9B8]`。而 `0050E09A` 一句明显地表明 `qword ptr [0012F9B8]` 所表示的浮点数是 `qword ptr [0012F9C0]` 加上 `10.00000`（注意，`[0050E230]` 仍属于代码段，其中的内容不可能被改写，无疑是常量。）所以关键就在 `qword ptr [0012F9C0]` 的内容是从什么地方来的。任何事物的产生总要有个源头，总不至于凭空变出一个注册码来吧！说了那么多，还是先把变量之间的依赖关系小结一下：

- 注册码 依赖于 `qword ptr ds:[0012F9D8]`
- `qword ptr ds:[0012F9D8]` 经过程 `0046BE4C` 变换，变换前：
  - `dword ptr ds:[0012F9D8] = 0000A4CB`
  - `word ptr ds:[0012F9DC]` 依赖于 `qword ptr ss:[0012F998]`（经过程 `0046C690` 变换）
  - `word ptr ds:[0012F9DE]` 依赖于 `qword ptr ss:[0012F990]`（经过程 `0046C690` 变换）
- `qword ptr ss:[0012F998] = qword ptr ss:[0012F9C0]`
- `qword ptr ss:[0012F990] = qword ptr ss:[0012F9B8]`
- `qword ptr ss:[0012F9B8] = qword ptr ss:[0012F9C0] fadd 10.00000`

接着往上看，`qword ptr [0012F9C0]` 的值是从浮点堆栈中拉出来的，而在 `0050E08E` 这个 `call` 之前，浮点堆栈还是空的。这说明 `qword ptr [0012F9C0]` 的值就是由这个 `call` 生成的。这个 `call` 我已经跟进去过，具体过程就不写了。其大致操作是先调用 `GetLocalTime` 取得本地机时间，然后计算出自从公元 1901 年元月 1 日 0 时 0 分 0 秒以来到这个时间为止所经历的天数（不足一天的部分以浮点小数表示）返回到浮点堆栈中。

到此为止，这个注册码生成的流程基本如下：

```
qword_1 = (double)公元 1901 年元月 1 日 0 时 0 分 0 秒以来到本地机时间为止所经历的天数
qword_2 = qword_1 + 10.00000
dword ptr qword_3 = 0000A4CB
word ptr qword_3+4 = sub_0046C690(qword_1)
word ptr qword_3+6 = sub_0046C690(qword_2)
sub_00468A4C(qword_3)
返回 qword_3——注册码
```

然而我对 `0046C690` 这个过程的处理是怎么回事还是不了解，兹列代码如下：

```
0046C690  /$  55            push    ebp  
0046C691  |.  8BEC          mov     ebp, esp  
0046C693  |.  51            push    ecx  
0046C694  |.  DD45 08       fld     qword ptr [ebp+8]  
0046C697  |.  E8 E464F9FF   call    00402B80  
0046C69C  |.  83FA 00       cmp     edx, 0  
0046C69F  |.  75 03         jnz     short 0046C6A4  
0046C6A1  |.  83F8 00       cmp     eax, 0  
0046C6A4  |>  74 29         je      short 0046C6CF                     ;  EDX:EAX != 0  
0046C6A6  |.  DD45 08       fld     qword ptr [ebp+8]  
0046C6A9  |.  E8 D264F9FF   call    00402B80  
0046C6AE  |.  52            push    edx  
0046C6AF  |.  50            push    eax  
0046C6B0  |.  A1 B06D5400   mov     eax, [546DB0]                     ;  ds:[00546DB0]=000088F9  
0046C6B5  |.  99            cdq  
0046C6B6  |.  290424        sub     [esp], eax  
0046C6B9  |.  195424 04     sbb     [esp+4], edx  
0046C6BD  |.  58            pop     eax  
0046C6BE  |.  5A            pop     edx  
0046C6BF  |.  83FA 00       cmp     edx, 0  
0046C6C2  |.  75 09         jnz     short 0046C6CD  
0046C6C4  |.  3D FFFF0000   cmp     eax, 0FFFF  
0046C6C9  |.  76 0C         jbe     short 0046C6D7  
0046C6CB  |.  EB 02         jmp     short 0046C6CF                     ;  花指令  
0046C6CD  |>  7E 08         jle     short 0046C6D7  
0046C6CF  |>  66:C745 FE 0>mov      word ptr [ebp-2], 0  
0046C6D5  |.  EB 13         jmp     short 0046C6EA  
0046C6D7  |>  DD45 08       fld     qword ptr [ebp+8]  
0046C6DA  |.  E8 A164F9FF   call    00402B80  
0046C6DF  |.  66:2B05 B06D5>sub      ax, [546DB0]  
0046C6E6  |.  66:8945 FE    mov     [ebp-2], ax  
0046C6EA  |>  66:8B45 FE    mov     ax, [ebp-2]  
0046C6EE  |.  59            pop     ecx  
0046C6EF  |.  5D            pop     ebp  
0046C6F0  \.  C2 0800       retn    8  
```

其中调用的 `00402B80` 过程：

```
00402B80  /$  83EC 0C       sub     esp, 0C  
00402B83  |.  9B            wait  
00402B84  |.  D93C24        fstcw   [esp]  
00402B87  |.  9B            wait  
00402B88  |.  D92D 28605400 fldcw   [546028]                            ;  ds:[00546028]=1F32  
00402B8E  |.  DF7C24 04     fistp   qword ptr [esp+4]  
00402B92  |.  9B            wait  
00402B93  |.  D92C24        fldcw   [esp]  
00402B96  |.  59            pop     ecx  
00402B97  |.  58            pop     eax  
00402B98  |.  5A            pop     edx                                   ;  EDX:EAX=变换后的数值  
00402B99  \.  C3            retn  
```

进入这个过程的时候浮点堆栈的 `st(0)` 还是好好的那个天数的数值，中途只是把控制字改为 `1F32`，输出结果就面目全非，怎么搞的。鄙人不懂 FPU 的工作原理，有没有哪位给解释一下？

上面所总结的算法只是程序“自带”的注册算法，至于任意输入一个注册码，判定它是否合法的代码则没有被涉及。（五天当中我有两天是在研究这事，但令人失望地一无所获。）所以可能会有挂一漏万的事发生。

### (3) 注册机

这个注册算法描述起来太拖泥带水，鄙人以为不如直接抽取上面所列的汇编代码来实现一个汇编注册机。（兹从略）

### 2. 感言

(1) 不知不觉，五天过去了……

(2) 个人认为除了跟自己的工作相关的软件以外，具体软件还是少解为妙。一个软件被解多了，保护措施必然相应增强，增加了后来的人解密的难度。如果这个软件跟你工作无关，只是出于兴趣而破解，那你更是在破坏资源。最近论坛上又有不少人在解国产软件，因此说这句话表明意见。

没人回贴，也没人加精么？可能是我说得不够清楚。但是涉及 FPU 的部分我确实不懂，只有直接抽取原程序反汇编的代码。这样吧，先贴出注册机源程序的主要部分。那个计算天数的过程大家应该都懂怎么回事，这里就略了吧。好歹搞了五天啊。

```
                    .586
      .model flat
      .option casemap:none
      include windows.inc
      include kernel32.inc
      includelib kernel32.lib
  Unk_Proceed     MACRO
                  ;这段代码是从反汇编代码的 00402B80 处抽取出来的
      ;不清楚它完成什么功能
      ;在原来的代码中是一个子程序，由于短小，现把它定义为一个宏
                  sub     esp, 0Ch
                  wait
                  fstcw   [esp]
                  wait
                  fldcw   magiccw                
                  fistp   qword ptr [esp+4]
                  wait
                  fldcw   [esp]
                  pop     ecx
                  pop     eax
                  pop     edx    
  Unk_Proceed     ENDM
      .const
  CryptKey        dd      EDA015DCh, F3867F20h, 710C20CCh, FF2718C5h 
                  dd      710C20CCh, F3867F20h, F3867F20h, FF2718C5h
      dd      EDA015DCh, 710C20CCh, EDA015DCh, FF2718C5h
  magicinc        real4   10.00000
  magiccw         dw      1F32h
      .data
  Regcode         dq      ?               ;注册码的内存区
```



`uDaysFrom20Cen dd ?` ; 这里存放的值是自 20 世纪初以来所经过的整数天数
`rDF20Cinfloat dq ?`
`stLocalTime SYSTEMTIME <>`

```
.code
CryptTransform proc stdcall public lpRegcode:dword
    local loopctrl:dword
    local RegCodeFirstDword:dword
    local RegcodeSecondDword:dword
    local Var1:dword, Var2:dword, Var3:dword, Var4:dword, Var5:dword
    ; 这个过程是从反汇编代码的 0046BE4C 处抽取出来的
    ; 作用是对 lpRegcode 指向的内存区域进行变换，以得到注册码
    mov     eax, lpRegcode
    mov     eax, [eax]
    mov     RegCodeFirstDword, eax
    mov     eax, lpRegcode
    mov     eax, [eax+4]
    mov     RegCodeSecondDword, eax
    xor     eax, eax
    mov     loopctrl, eax
            mov     edx, offset CryptKey
L011:
    mov     eax, RegCodeFirstDword
    mov     Var2, eax
    mov     eax, loopctrl
    mov     eax, [edx+eax*4]
    mov     Var3, eax
    mov     eax, loopctrl
    mov     eax, [edx+eax*4+4]
    mov     Var4, eax
    mov     eax, loopctrl
    mov     eax, [edx+eax*4+8]
    mov     Var5, eax
    mov     eax, Var5
    add     Var2, eax
    mov     eax, Var2
    add     Var5, eax
    mov     eax, Var2
    shr     eax, 7
    xor     Var2, eax
    mov     eax, Var2
    add     Var3, eax
    mov     eax, Var3
    add     Var2, eax
    mov     eax, Var3
    shl     eax, 0Dh
    xor     Var3, eax
    mov     eax, Var3
    add     Var4, eax
    mov     eax, Var4
    add     Var3, eax
    mov     eax, Var4
    shr     eax, 11h
    xor     Var4, eax
    mov     eax, Var4
    add     Var5, eax
    mov     eax, Var5
    add     Var4, eax
    mov     eax, Var5
    shl     eax, 9
    xor     Var5, eax
    mov     eax, Var5
    add     Var2, eax
    mov     eax, Var2
    add     Var5, eax
    mov     eax, Var2
    shr     eax, 3
    xor     Var2, eax
    mov     eax, Var2
    add     Var3, eax
    mov     eax, Var3
    shl     eax, 7
    xor     Var3, eax
    mov     eax, Var3
    add     Var4, eax
    mov     eax, Var5
    shr     eax, 0Fh
    xor     Var4, eax
    mov     eax, Var4
    add     Var5, eax
    mov     eax, Var5
    shl     eax, 0Bh
    xor     Var5, eax
    mov     eax, RegCodeSecondDword
    xor     eax, Var5
    mov     Var1, eax
    mov     eax, RegCodeFirstDword
    mov     RegCodeSecondDword, eax
    mov     eax, Var1
    mov     RegCodeFirstDword, eax
    add     loopctrl, 3
    cmp     loopctrl, 0Ch
    jnz     L011
    mov     eax, lpRegcode
    mov     edx, RegCodeSecondDword
    mov     [eax], edx
    mov     eax, lpRegcode
    mov     edx, RegCodeFirstDword
    mov     [eax+4], edx
    ret
CryptTransform endp

DaysFromAD     proc    stdcall public uses ebx esi edi, lpSystemTime:dword
            ; 根据 lpSystemTime 所指向的 SYSTEMTIME 结构计算出自公元以来所经历的天数, 返回在 eax 中
    ; 代码略
DaysFromAD     endp

FracDays       proc    stdcall public uses ebx esi edi, lpSystemTime:dword
            ; 将 lpSystemTime 所指向的 SYSTEMTIME 结构中的时、分、秒、毫秒等字段全部换算成毫秒
    ; 再除以一整天的毫秒数（8.64e+07）
    ; 结果返回在 st(0)中
    ; 代码略
FracDays       endp 

Unk_Transform  proc    stdcall public D1:dword,D2:dword
            local   at1:word
            ; 这段代码是从反汇编代码的 0046C690 处抽取出来的
    ; 不清楚它所做的是什么变换
    ; 返回的值在 ax 中
            push    ecx
            fld     qword ptr D1
            Unk_Proceed
            cmp     edx, 0
            jnz     @F
            cmp     eax, 0
      @@:
            je      L025
            fld     qword ptr D1
            Unk_Proceed
            push    edx
            push    eax
            mov     eax, 88F9h
            cdq
            sub     [esp], eax
            sbb     [esp+4], edx
            pop     eax
            pop     edx
            cmp     edx, 0
            jnz     @F
            cmp     eax, 0FFFFh
            jbe     L027
            jmp     L025
      @@:
            jle     L027
      L025:
            mov     at1, 0
            jmp     @F
      L027:     
            fld     qword ptr D1
            Unk_Proceed
            sub     ax, 88F9h
            mov     at1, ax
      @@:
            mov     ax, at1
            pop     ecx
    ret
Unk_Transform  endp

START:
    invoke  GetLocalTime, offset stLocalTime
    invoke  DaysFromAD, offset stLocalTime ; 这是计算出自公元以来所经历的天数, 返回在 eax 中
    sub     eax, 0A955Ah
    mov     uDaysFrom20Cen, eax
    fild    uDaysFrom20Cen               ;
    fstp    rDF20Cinfloat              
    invoke  FracDays, offset stLocalTime   ; 计算出不足一天的部分，返回在 st(0)中
    fadd    rDF20Cinfloat
    fstp    rDF20Cinfloat
    mov     dword ptr Regcode, 0A4CBh
    invoke  Unk_Transform, dword ptr rDF20Cinfloat, dword ptr [4 + offset rDF20Cinfloat]
            mov     word ptr [4 + offset Regcode], ax
    fld     rDF20Cinfloat
    fadd    magicinc
    fstp    rDF20Cinfloat
    invoke  Unk_Transform, dword ptr rDF20Cinfloat, dword ptr [4 + offset rDF20Cinfloat]
            mov     word ptr [6 + offset Regcode], ax
    invoke  CryptTransform, offset Regcode 
    ; 此处可插入将注册码的内存值转换为 ASCII 并显示的代码
    end     START
```
