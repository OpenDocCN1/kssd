
# CD to MP3 Maker

> 原文：[`www.pediy.com/kssd/pediy06/pediy6164.htm`](https://www.pediy.com/kssd/pediy06/pediy6164.htm)

破解 CD to MP3 Maker

调试环境：`win98se`

破解工具：`SoftICE`, `win32dasm` 黄金版

难度：比较简单

软件没有加壳，用 `win32dasm` 加载，在串式参考中可看到许多有用信息

运行软件，进入注册窗口，注册名处填 `008liwei`(任意),注册码处填 `123456789`(任意)

现在用 `Ctrl－d` 调出 `softice`，`bpx hmemcpy,` 然后按 `F5` 退出 ICE，点击注册按钮，被 ICE 拦截，键入 `bd 0` 关闭断点狂按 `F12`，按到第 13 下时，弹出错误窗口，重新打开 ICE 按照上面的步骤，这回狂按 12 下 `F12`,代码停留在下面的地方

```
:0041171E 6A01                 push 00000001

...........一边下 d 命令一边看寄存器窗口的值，发现并无有用信息，所以这儿的代码略

...........

:00411791 8D4C2438             lea ecx, dword ptr [esp+38] --------在此键入 d ecx，数据窗口中发现 008liwei，说明要进入雷区了

:00411795 51                   push ecx                   --------将注册名 008liwei 压入堆栈保存起来

:00411796 8D4C241C             lea ecx, dword ptr [esp+1C]

:0041179A E877EC0100           call 00430416

:0041179F 8B542410             mov edx, dword ptr [esp+10]

:004117A3 8B07                 mov eax, dword ptr [edi]    --------此处 eax 内容为 008liwei

* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:027Fh --------准备将用户名 008liwei 写入"c:\windows\repenc.ini"文件中

:004117A5 8B1DB49A4500         mov ebx, dword ptr [00459AB4] --------此处 edx 内容为"c:\windows\ripenc.ini"

:004117AB 52                   push edx

:004117AC 50                   push eax                  ---------                                eax=008liwei

* Possible StringData Ref from Data Obj ->"Name"

:004117AD 68FCC44400           push 0044C4FC             ----------将字符"name"保存

* Possible StringData Ref from Data Obj ->"Mp3Option"

:004117B2 6844C34400           push 0044C344             ---------将字符"mp3option"保存

:004117B7 FFD3                 call ebx                  ---------将注册名信息写入文件

:004117B9 8B4C2410             mov ecx, dword ptr [esp+10]  ---------ecx 为字符"c:\windows\ripenc.ini"

:004117BD 8D542438             lea edx, dword ptr [esp+38]  ---------键入 d edx 内容为"008liwei"

:004117C1 51                   push ecx

* Possible Reference to Dialog: DialogID_00D2, CONTROL_ID:00FF, ""

:004117C2 68FF000000           push 000000FF

:004117C7 52                   push edx

:004117C8 686C3C4500           push 00453C6C

* Possible StringData Ref from Data Obj ->"Name"

:004117CD 68FCC44400           push 0044C4FC

* Possible StringData Ref from Data Obj ->"Mp3Option"

:004117D2 6844C34400           push 0044C344

:004117D7 FFD5                 call ebp

    －－－－－－－注意了！雷区就在眼前，不怕死的跟我来 >:-()－－－－－－

:004117D9 8B07                 mov eax, dword ptr [edi]  -------d eax 内容为"008liwei"  ):-()好兄弟够义气，怎么?只有你一个人  ~@#$%^&*

:004117DB BA01000000           mov edx, 00000001           -------将 edx 置入 1

:004117E0 8B48F8               mov ecx, dword ptr [eax-08] -------将注册名个数 8 放入 ecx 中

:004117E3 33C0                 xor eax, eax                -------将 eax 清零

:004117E5 85C9                 test ecx, ecx               -------检查注册名个数是否为零

:004117E7 7E0C                 jle 004117F5                -------当为零时跳到 004117F5 继续执行，当然不会跳转

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004117F3(C)

:004117E9 0FBE6C0438           movsx ebp, byte ptr [esp+eax+38]  -------在此处键入 d esp＋38   发现数据窗口中为"008liwei"   在寄存器窗口中可看见 eax＝00000

000     由汇编语言知识可知此处代码的意思是将"008liwei"的第一个字符的十六进制数送入 ebp 中.由于第一个数是 0，所以将会把 30 这个数放入 ebp 中

:004117EE 03D5                 add edx, ebp                  -------将 30(ebp 的值)＋1(edx 的值)放入 edx 中

:004117F0 40                    inc eax                        -------eax 自动加 1

:004117F1 3BC1                  cmp eax, ecx                   -------由于 ecx 的值是注册名个数 8，所以这句的实际意思是判断注册名每位数是否都进行了上面的运算

:004117F3 7CF4                  jl 004117E9                    -------如果没有则跳到 004117E9 处,依次进行第 2，3，4，5，6，7，8 位的计算，直至计算完成，所以在此处要循环 7 次，方能执行下面的代码

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004117E7(C)

:004117F5 8BC2                  mov eax, edx                   -------此时的 edx 值为: 30(注册名第一位)+1+30(第二位)+38(第三位)+6c+69+77+65+69=2B3 所以 eax＝2B3

:004117F7 C1E006                shl eax, 06                      -------将 eax 的值乘以 2 的 6 次方  eax=ACC6

:004117FA 03C2                  add eax, edx                   -------

                                                                        eax=eax+edx=ACC6+2b3=AF73

:004117FC 8D0CC0                lea ecx, dword ptr [eax+8*eax]    -------   ecx=AF73(eax)*8+AF73(eax)=62b0b   键入 ？ecx 可看到此值

:004117FF 8DAC8A155E0100        lea ebp, dword ptr [edx+4*ecx+00015E15]  --------ebp＝62b0b(ecx)*4+2b3(ebx)+15e15＝1a0cf4  你将会在后面发现它的十进制数就是正确的注册码

:00411806 8D4C241C             lea ecx, dword ptr [esp+1C]

:0041180A E874E90100           call 00430183

:0041180F 8B86A0030000         mov eax, dword ptr [esi+000003A0] --------寄存器窗口中 eax=75bcd15

:00411815 C684244002000004     mov byte ptr [esp+00000240], 04

:0041181D 3BE8                 cmp ebp, eax                     --------下？ebp 可在命令窗口看见 1707252  下？eax 同样可看到 123456789   由这可知 4117ff 处的数为正确的注册码

:0041181F 756C                 jne 0041188D                     --------命运现在掌握在你的手中  下 r fl z 改变标志位不叫它跳，跳则 Game Over

:00411821 8B542410             mov edx, dword ptr [esp+10]

:00411825 8B07                 mov eax, dword ptr [edi]

:00411827 52                   push edx

:00411828 50                   push eax

* Possible StringData Ref from Data Obj ->"Name"

:00411829 68FCC44400           push 0044C4FC

* Possible StringData Ref from Data Obj ->"Mp3Option"

:0041182E 6844C34400           push 0044C344

:00411833 FFD3                 call ebx

:00411835 8B8EA0030000         mov ecx, dword ptr [esi+000003A0]

:0041183B 8D542424             lea edx, dword ptr [esp+24]

:0041183F 51                   push ecx

* Possible StringData Ref from Data Obj ->"%d"

:00411840 6854C44400           push 0044C454

:00411845 52                   push edx

:00411846 E845910000           call 0041A990

:0041184B 83C40C               add esp, 0000000C

:0041184E 8D442424             lea eax, dword ptr [esp+24]

:00411852 8D4C241C             lea ecx, dword ptr [esp+1C]

:00411856 50                   push eax

:00411857 E8BAEB0100           call 00430416

:0041185C 8B4C2410             mov ecx, dword ptr [esp+10]

:00411860 8B54241C             mov edx, dword ptr [esp+1C]

:00411864 51                   push ecx

:00411865 52                   push edx

* Possible StringData Ref from Data Obj ->"Code"

:00411866 68F4C44400           push 0044C4F4

* Possible StringData Ref from Data Obj ->"Mp3Option"

:0041186B 6844C34400           push 0044C344

:00411870 FFD3                 call ebx

:00411872 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"congratulation!"

:00411874 683CD04400           push 0044D03C

* Possible StringData Ref from Data Obj ->"You have registered.Please quit "
                                ->"and execute it again!"

:00411879 6804D04400           push 0044D004

:0041187E 8BCE                 mov ecx, esi
```



```
:00411880 E8592F0200              call 004347DE  ------------(:-)好兄弟！我们通过雷区了~~~ 人怎么不见了？哇！你怎么跑到下面去了！oh my God!

:00411885 3BAEA0030000           cmp ebp, dword ptr [esi+000003A0]

:0041188B 7429                   je 004118B6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041181F(C)

|

:0041188D 6A00                   push 00000000   -----------

                                  <You are terminated. blaste>

                   ------<I will be back>--------
```

### 算法总结
设注册名对应的正确注册码为`x`，则：

1. `a = 注册名的每一位(hex)相加再+1`
2. `b = a * 2⁶ + a`
3. `c = b + b * 8`
4. `x = c * 4 + a + 15E15`

### 注册机

在`TC2.0`下通过：

```c
#include "math.h"

#include "stdio.h"

main()

{

char name[50];int i,n=0,m;

unsigned long int b,a,c,d;

printf("========================\n");

printf("CD to MP3 1.15 Keygen\n");

printf("Made by 008liwei \n");

printf("2003.7.25\n");

printf("========================\n");

printf("\n");

printf("Pleaes enter your name:");

scanf("%s",name);

m=strlen(name);

for(i=0;i<=m;i++)

 {n=n+name[i];}

 n=n+1;

b=pow(2,6);

a=n*b+n;

c=a+a*8;

d=c*4+n+89621;

printf("Your registration code is %ld\n",d);

printf("Thanke You !!");

getch();

}
```

请大家多多指教!!

我是一个初学者，这是我的第一篇文章，错误一定大大的有，意见也请多多的提。我很希望得到你们的指导和帮助。在这里我先请教一个问题：从`4117c1`到`4117d7`处，程序在干什么？我怎么看不懂,还有`411806`处？

学破解也有一年了，想谈一点点自己的学习心得：

回想第一次认识破解，是在无意中发现光盘中的《看学论坛精华 III》，平时也用过破解补丁或者注册机什么的，却从没有想过它们是怎么做出来的，但看过了《论坛精华》序言后，就立刻产生了浓厚的兴趣，虽然当时还看不懂各位前辈写的文章(当然现在有的也看不懂 :-> ) ,但是兴趣已经有了，所以立刻就到书店买了看雪老师编的《加密与解密－软件保护及完全解决方案》，这本书的确不错，内容由浅入深使我了解了许多,比如:什么是寄存器，什么是`API`，`PE`文件格式是什么东西，把我从软件的表层带进了它的核心 ……，这些都是原来不可想象的。所以在这里我建议想学破解的朋友，一定要看此书。

我认为学破解并不只是一味地找软件破软件，而是当你破解的时候，会发现需要用到方方面面的知识，比如汇编语言（当然这是基础），`windowsAPI`，注册表，计算机编程（做注册机）……，所以我感觉不应为了破解而去破解，而应在这一过程中学习更多其他方面的电脑知识，我已受益匪浅，不知大家是否有同感？

好了，就现写到这儿。我的文笔不怎么样，凑活着看吧！:-) 最后谢谢各位看完我的文章。

`Email:008liwei@sina.com`

`QQ：3236738`

`4dd466d0e1e0cc123afbe504da2462ca`
