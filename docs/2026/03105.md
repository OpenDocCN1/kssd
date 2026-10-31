
# 键盘乐器2.0

## 简单算法――键盘乐器2.0

呵呵，从《软件王》的光盘里看到这个软件，安装，不仅为了试用，更为了CRACK。呵呵，CRACK就是需要不断的练手，“惟手熟而”，不断的训练会使我们学到更多的知识和技巧。

这个软件很简单，大侠见笑了。

### 软件简介

也许你曾经有过演奏各种乐器的梦想，但由于各种原因不能实现。

<<键盘乐器>>将实现你的梦想！

<<键盘乐器>>是一个利用电脑键盘演奏乐器的软件。用他你可以演奏包括钢琴、小提琴、吉他、萨克斯、古筝等一百多种乐器和音效！

欢迎进入乐器的世界！

### 软件限制

次数限制

### 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 破解工具

TRW2000娃娃修改版、W32Dasm8.93黄金版

―――――――――――――――――――――――――――――

### 过程

机器码：1330515

试炼码：13572468

程序无壳，VC++5.0编写。

反汇编后查找关键提示。呵呵，很好，该有的都有了。

"注册成功"在`00403C2D`，上面就是运算过程以及关键比较！

填好试炼信息。直接`BPX 403BF0`，开始我们的简单算法追踪之旅！

―――――――――――――――――――――――――――――

```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403BD8(C)
|
:00403BE4
53                      push
ebx
:00403BE5 55
   push ebp
:00403BE6 6A00
       push 00000000
:00403BE8 6A00
             push 00000000
\*
Possible Reference to Dialog: REGISTBOX, CONTROL\_ID:0102, ""
       |
:00403BEA 6802010000
     push 00000102
:00403BEF 56
             push esi
\*
Reference To: USER32.GetDlgItemInt, Ord:0103h
 |
:00403BF0 FF15C4A14000            Call
dword ptr [0040A1C4]
                 ====>拦下！
:00403BF6
8BC8                    mov
ecx, eax
             ====>?ECX=CB06AB
    十进制值1330515  即：显示的机器码
:00403BF8
8BC7                    mov
eax, edi
:00403BFA 99
     cdq
:00403BFB BB0F000000
   mov ebx, 0000000F
                   ====>F入EBX
:00403C00
BDFBE80400              mov ebp, 0004E8FB
         ====>4E8FB入EBP
:00403C05
F7FB                    idiv
ebx
第１步       ====>EAX=CB06AB/F=D88FA余5，余数入DL
:00403C07
8BC7                    mov
eax, edi
             ====>?EAX=CB06AB
:00403C09
8BDA                    mov
ebx, edx
             ====>?EBX=5  即：上面的余数
:00403C0B
99                      cdq
:00403C0C
69DB9961C610            imul ebx, 10C66199
                  第２步
  ====>EBX=5\*10C66199=53DFE7FD
:00403C12
F7FD                    idiv
ebp
第３步       ====>EAX=CB06AB/4E8FB=29余1B678
:00403C14
5D                      pop
ebp
:00403C15 2BDA
   sub ebx, edx
      第４步       ====>53DFE7FD-1B678=53DE3185
:00403C17
03DF                    add
ebx, edi
第５步       ====>EBX=53DE3185+CB06AB=54A93830
:00403C19
81F3410B7300            xor ebx, 00730B41
                  第６步
  ====>EBX=54A93830 XOR 00730B41
       =54DA3371
:00403C1F F7D3
           not ebx
            第７步       ====>各位变反EBX=AB25CC8E
   AB25CC8E的十进制值就是我们的真码！
:00403C21
3BD9                    cmp
ebx, ecx
             ====>真假码比较！
     ====>？EBX=2871381134   真码！！
   ====>？EAX=13572468    试炼码！
:00403C23
5B                      pop
ebx
:00403C24 7530
   jne 00403C56
                   ====>跳则OVER！
:00403C26
6A40                    push
00000040
\* Possible
StringData Ref from Data Obj ->"欢迎使用！"
 |
:00403C28 6828D44000              push
0040D428
\* Possible
StringData Ref from Data Obj ->"注册成功！"
       ====>呵呵，胜利女神！
:00403C2D
681CD44000              push 0040D41C
:00403C32
56                      push
esi
:00403C33 C70548E6400001000000    mov dword ptr [0040E648], 00000001
\*
Reference To: USER32.MessageBoxA, Ord:01BEh
                       |
:00403C3D
FF15A4A14000            Call dword ptr [0040A1A4]
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403BDB(C)
|
:00403C43
6A00                    push
00000000
:00403C45 56
     push esi
\*
Reference To: USER32.EndDialog, Ord:00B9h
                       |
:00403C46
FF15C8A14000            Call dword ptr [0040A1C8]
:00403C4C
5F                      pop
edi
:00403C4D B801000000              mov
eax, 00000001
:00403C52 5E
       pop esi
:00403C53 C21000
         ret 0010
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403C24(C)
|
:00403C56
6A40                    push
00000040
\* Possible
StringData Ref from Data Obj ->"请检查注册码"
   |
:00403C58 680CD44000
 push 0040D40C
\*
Possible StringData Ref from Data Obj ->"注册失败！"
                     ====>BAD
BOY！
:00403C5D 6800D44000
           push 0040D400
```

―――――――――――――――――――――――――――――

### KeyMake之内存注册机

中断地址：`403C21`

中断次数：1

第一字节：`3B`

指令长度：2

寄存器方式：`EBX`

十进制

―――――――――――――――――――――――――――――

### 注册信息保存

`[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\WebCheck]`

`"Window"=dword:01ead4cf`

呵呵，保存的地方“挺狡猾”。

―――――――――――――――――――――――――――――

### 整理

机器码：1330515

注册码：2871381134

有点疑问，其机器码是如何得出的？好象还会变化。

―――――――――――――――――――――――――――――

### 总结

1、取机器码`CB06AB`与程序自给的`F`求模，余数是5。

2、余数5与程序自给的`10C66199`相乘=`53DFE7FD`。

3、取机器码`CB06AB`与程序自给的`4E8FB`求模，余数是`1B678`。

4、再拿第2步所得的`53DFE7FD`-`1B678`=`53DE3185`。

5、机器码`CB06AB`+第四步所得的`53DE3185`=`54A93830`。

6、`54A93830`与程序自给的`00730B41`异或=`54DA3371`。

7、`54DA3371`各位变反=`AB25CC8E`。这就是真码是十六进制值！

转化为十进制真码=2871381134

―――――――――――――――――――――――――――――

Cracked By 巢水工作坊――fly【OCN】

2003-2-10   0:00
