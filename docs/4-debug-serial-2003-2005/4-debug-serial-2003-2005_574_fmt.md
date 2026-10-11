
# Axman 3.12r

> 原文：[`www.pediy.com/kssd/pediy06/pediy6641.htm`](https://www.pediy.com/kssd/pediy06/pediy6641.htm)

## 手把手系列之二—Axman 3.12rGB 的注册算法详解及注册机

**【破解作者】** jackily

**【作者邮箱】** jackily_zhang@msn.com；jackily_zhang@yahoo.com.cn

**【作者主页】** [`estudy.ys168.com`](http://estudy.ys168.com)

**【使用工具】** `ollydbg` 1.10

**【破解平台】** Win9x/NT/2000/XP

**【软件名称】** `axman` 3.12rGB

**【下载地址】** `http://www.pc123.cn/SoftView/SoftView_5625.html`

**【软件简介】** 一个非常不错的文件分割工具，该工具的主要作用是可以将大文件切成几个小文件便于拷贝到软盘或作为电子邮件附件发送。对于分割的文件，程序提供自动合并功能，同时该工具也提供非常简单的操作界面和操作方式以方便用户使用。未注册版本有 30 天的使用限制，并在开始程序时有 nag 窗口。

**【破解声明】** 本破解纯属以学习为目的，偶得一点心得，愿与大家分享：） xiA Qin 曾破解过 3.10 版（具体参见看雪精华 II），但只是给了一个注册码而已，并没有给出算法和注册机，3.12 版程序稍有变化，我们的目的是要找到算法并写出注册机。

---

## 破解内容

安装完毕后主文件为 `axman.exe`，用 `peid` 检查无壳。用 `ollydbg` 载入，运行，出现提示注册窗口，点击注册，要求输入 `name`、`company` 和注册码。这里我们分别输入 `jackily`，`http://estudy.ys168.com` 和 `123456`。点确定，弹出“Invalid RegisterInformation”。好了，重新载入，点右键—“搜索”—“字符参考”，找到刚才的 ASCII 码，地址为 `00406933`，向上看，`00406923` 处有一 `call`，为典型的注册关键调用。在此处下断，跟进。代码分析如下：

```assembly
004068DA    .   8D45 F0             lea     eax,dword ptr ss:[ebp-10]
004068DD    .   8D8E C8020000       lea     ecx,dword ptr ds:[esi+2C8]
004068E3    .   50                  push    eax
004068E4    .   C645 FC 02          mov     byte ptr ss:[ebp-4],2
004068E8    .   E8 B5B00000         call    <jmp.&MFC42.#3874>       ;将"jackily"放入地址 00674660
004068ED    .   8D45 E8             lea     eax,dword ptr ss:[ebp-18]
004068F0    .   8D8E 08030000       lea     ecx,dword ptr ds:[esi+308]
004068F6    .   50                  push    eax
004068F7    .   E8 A6B00000         call    <jmp.&MFC42.#3874>       ; 将"http://estudy.ys168.com"放入地址 006747a0
004068FC    .   8D45 EC             lea     eax,dword ptr ss:[ebp-14]
004068FF    .   8D8E 88020000       lea     ecx,dword ptr ds:[esi+288]
00406905    .   50                  push    eax
00406906    .   E8 97B00000         call    <jmp.&MFC42.#3874>       ; 将 ASCII "123456")放入地址 00674840
0040690B    .   8D4D E4             lea     ecx,dword ptr ss:[ebp-1C]
0040690E    .   E8 25FBFFFF         call    AXMAN.00406438
00406913    .   FF75 EC             push    dword ptr ss:[ebp-14]    ; /Arg3 = 00674840 ASCII "123456"
00406916    .   8D4D E4             lea     ecx,dword ptr ss:[ebp-1C] ; |
00406919    .   C645 FC 03          mov     byte ptr ss:[ebp-4],3     ; |
0040691D    .   FF75 E8             push    dword ptr ss:[ebp-18]    ; |Arg2 = 006747A0 ASCII "http://estudy.ys168.com"
00406920    .   FF75 F0             push    dword ptr ss:[ebp-10]    ; |Arg1 = 00674660 ASCII "jackily"
00406923    .   E8 3CFBFFFF         call    AXMAN.00406464           ；典型的注册模式，有点意思，跟进
00406928    .   84C0                test    al,al                    ; 检验注册标志，这里成功应是非 0
0040692A    .   75 18               jnz     short AXMAN.00406944     ; 成功则到 00406944
0040692C    .   6A 30               push    30
0040692E    .   68 10AC4100         push    AXMAN.0041AC10          ;  ASCII "AxMan - Error 400"
00406933    .   68 ECAB4100         push    AXMAN.0041ABEC          ;  ASCII "Invalid Registration Information"
00406938    .   8BCE                mov     ecx,esi
0040693A    .   E8 21B00000         call    <jmp.&MFC42.#4224>
0040693F    .   E9 8E000000         jmp     AXMAN.004069D2
00406944    >   8D85 54FEFFFF       lea     eax,dword ptr ss:[ebp-1AC]
0040694A    .   68 BCAB4100         push    AXMAN.0041ABBC          ; /src = "Thank you for registering your copy of AxMan
"
0040694F    .   50                  push    eax                     ; |dest
00406950    .   E8 CBB50000         call    <jmp.&MSVCRT.strcpy>    ; \strcpy
00406955    .   8D85 54FEFFFF       lea     eax,dword ptr ss:[ebp-1AC]
0040695B    .   68 68AB4100         push    AXMAN.0041AB68          ; /src = "You will need to stop and restart AxMan to remove  the banner ads from displaying."
00406960    .   50                  push    eax                     ; |dest
00406961    .   E8 C0B50000         call    <jmp.&MSVCRT.strcat>    ; \strcat
00406966    .   83C4 10             add     esp,10
00406969    .   8D85 54FEFFFF       lea     eax,dword ptr ss:[ebp-1AC]
0040696F    .   8BCE                mov     ecx,esi
00406971    .   6A 30               push    30
00406973    .   68 D8A64100         push    AXMAN.0041A6D8          ;  ASCII "AxMan"
00406978    .   50                  push    eax
00406979    .   E8 E2AF0000         call    <jmp.&MFC42.#4224>      ; 解除程序限制，以下省略
```

由 `00406923` 跟进，一级调用：

```assembly
00406464    /$  55                  push    ebp
00406465    |.  8BEC                mov     ebp,esp
00406467    |.  83EC 18             sub     esp,18
0040646A    |.  8D45 E8             lea     eax,dword ptr ss:[ebp-18]
0040646D    |.  50                  push    eax
0040646E    |.  FF75 0C             push    dword ptr ss:[ebp+C]    ; (ASCII "http://estudy.ys168.com")
00406471    |.  FF75 08             push    dword ptr ss:[ebp+8]    ; (ASCII "jackily")
00406474    |.  E8 18000000         call    AXMAN.00406491          ；关键 call，数据处理，为计算做准备，跟进
00406479    |.  FF75 10             push    dword ptr ss:[ebp+10]   ; /s2 = "123456" 我们输入的假码
0040647C    |.  8D45 E8             lea     eax,dword ptr ss:[ebp-18]; |
0040647F    |.  50                  push    eax                     ; |s1 = "156-936-773" ,真实的注册码，但不要忘记我们的目的是找算法，不应满足在找到注册码的水平上
00406480    |.  E8 51BA0000         call    <jmp.&MSVCRT.strcmp>    ; \strcmp 比较
00406485    |.  F7D8                neg     eax
00406487    |.  59                  pop     ecx
00406488    |.  1AC0                sbb     al,al
0040648A    |.  59                  pop     ecx
0040648B    |.  FEC0                inc     al
0040648D    |.  C9                  leave
0040648E    |\.  C2 0C00            retn    0C
```

---

由 `00406474` 跟进，以下为数据处理，为计算做准备，把 `name` 和 `company` 字符串小写，再将 `"My Love"`(注意大小写和空格) `strcat` 在一起，也就是连接在一起。

```assembly
00406491    /$  B8 C42B4100         mov     eax,AXMAN.00412BC4
00406496    |.  E8 15BA0000         call    AXMAN.00411EB0
0040649B    |.  81EC F8000000       sub     esp,0F8
004064A1    |.  53                  push    ebx
004064A2    |.  56                  push    esi
004064A3    |.  57                  push    edi
004064A4    |.  8BF9                mov     edi,ecx
004064A6    |.  FF75 08             push    dword ptr ss:[ebp+8]    ; /s = "jackily"
004064A9    |.  E8 22BA0000         call    <jmp.&MSVCRT.strlen>    ; \strlen 取字符串长度
004064AE    |.  8B35 10464100       mov     esi,dword ptr ds:[<&MSVCRT.mallo>;  MSVCRT.malloc
004064B4    |.  40                  inc     eax
004064B5    |.  50                  push    eax                     ; /size
004064B6    |.  FFD6                call    esi                     ; \malloc
004064B8    |.  59                  pop     ecx
004064B9    |.  8BD8                mov     ebx,eax
004064BB    |.  59                  pop     ecx
004064BC    |.  53                  push    ebx
004064BD    |.  8BCF                mov     ecx,edi
004064BF    |.  FF75 08             push    dword ptr ss:[ebp+8]
004064C2    |.  E8 63020000         call    AXMAN.0040672A
004064C7    |.  FF75 0C             push    dword ptr ss:[ebp+C]    ; /s = "http://estudy.ys168.com"
004064CA    |.  E8 01BA0000         call    <jmp.&MSVCRT.strlen>    ; \strlen 取字符串长度
```
`004064CF` 至 `0040651A` 非关键部分省略

```assembly
0040651F    |.  FF75 EC             push    dword ptr ss:[ebp-14]   ; /src = "jackily"
00406522    |.  8D85 FCFEFFFF       lea     eax,dword ptr ss:[ebp-104]; |(ASCII "http://estudy.ys168.com")
00406528    |.  50                  push    eax                     ; |dest = 0065EB98
```



```
00406529   |.  E8 F2B90000       call <jmp.&MSVCRT.strcpy>        ; \strcpy  
0040652E   |.  FF75 F0          push dword ptr ss:[ebp-10]       ; /src = "http://estudy.ys168.com"  
00406531   |.  8D85 FCFEFFFF    lea eax,dword ptr ss:[ebp-104]    ; |(ASCII "jackily")  
00406537   |.  50              push eax                           ; |dest "jackily"  
00406538   |.  E8 E9B90000     call <jmp.&MSVCRT.strcat>        ; \strcat  
0040653D   |.  8D85 FCFEFFFF    lea eax,dword ptr ss:[ebp-104]  
00406543   |.  68 60AB4100     push AXMAN.0041AB60               ; /src = "My Love"  
00406548   |.  50              push eax                           ; |dest  
00406549   |.  E8 D8B90000     call <jmp.&MSVCRT.strcat>        ; \strcat  
0040654E   |.  83C4 18         add esp,18  
00406551   |.  8D45 E8         lea eax,dword ptr ss:[ebp-18]     ; (ASCII "jackilyhttp://estudy.ys168.comMy Love")  
00406554   |.  8BCF           mov ecx,edi                        ；以上为 把 name 和 company 字符串所有字符小写并把“My Love”连接在一起  
00406556   |.  50              push eax  
00406557   |.  8D85 FCFEFFFF    lea eax,dword ptr ss:[ebp-104]  
0040655D   |.  50              push eax  
0040655E   |.  E8 8D010000     call AXMAN.004066F0               ；关键之关键，算法，跟进  
00406563   |.  FF75 E8         push dword ptr ss:[ebp-18]       ; /<%u> = 95AAA45(注册码的 HEX 值,十进制为 156936773)  
00406566   |.  8D45 D0         lea eax,dword ptr ss:[ebp-30]     ; |  
00406569   |.  68 40A04100     push AXMAN.0041A040             ; |format = "%u" (c 语言中代表无符号十进制值）  
0040656E   |.  50              push eax                           ; |s  
0040656F   |.  FF15 30464100    call dword ptr ds:[<&MSVCRT.sprintf>>; \sprintf  
00406575   |.  83C4 0C         add esp,0C  
00406578   |.  8D45 D0         lea eax,dword ptr ss:[ebp-30]  
0040657B   |.  8BCF           mov ecx,edi  
0040657D   |.  6A 01           push 1  
0040657F   |.  6A 09           push 9  
00406581   |.  6A 30           push 30  
00406583   |.  50              push eax  
00406584   |.  E8 73000000     call AXMAN.004065FC  
00406589   |.  8B75 10         mov esi,dword ptr ss:[ebp+10]     ; 以下是将 156936773 每三个数字间加“-”  
0040658C   |.  8D45 D0         lea eax,dword ptr ss:[ebp-30]     ; ASII "156936773"  
0040658F   |.  6A 03           push 3                              ; /maxlen = 3 复制前 3 个字符  
00406591   |.  50              push eax                           ; |src = "156936773"  
00406592   |.  56              push esi                           ; |dest  
00406593   |.  FF15 D4454100    call dword ptr ds:[<&MSVCRT.strncpy>>; \strncpy  
00406599   |.  8066 03 00      and byte ptr ds:[esi+3],0  
0040659D   |.  BB 5CAB4100     mov ebx,AXMAN.0041AB5C  
004065A2   |.  53              push ebx                           ; /src => "-"  
004065A3   |.  56              push esi                           ; |dest = "156"  
004065A4   |.  E8 7DB90000     call <jmp.&MSVCRT.strcat>        ; \strcat  
004065A9   |.  8B3D D8454100    mov edi,dword ptr ds:[<&MSVCRT.strnc>;  MSVCRT.strncat  
004065AF   |.  8D45 D3         lea eax,dword ptr ss:[ebp-2D]  
004065B2   |.  6A 03           push 3                              ; /maxlen = 3  
004065B4   |.  50              push eax                           ; |src = "936773"  
004065B5   |.  56              push esi                           ; |dest = "156-"  
004065B6   |.  FFD7           call edi                           ; \strncat  
004065B8   |.  8066 07 00      and byte ptr ds:[esi+7],0  
004065BD   |.  56              push esi                           ; |dest = "156-936"  
004065BD   |.  56              push esi                           ; |dest  
004065BE   |.  E8 63B90000     call <jmp.&MSVCRT.strcat>        ; \strcat  
004065C3   |.  8D45 D6         lea eax,dword ptr ss:[ebp-2A]  
            .
            .                                             ;004065C6 至 004065EB 非关键部分省略  
            .
            .  
004065EE   |.  5F              pop edi  
004065EF   |.  5E              pop esi  
004065F0   |.  5B              pop ebx  
004065F1   |.  64:890D 00000000  mov dword ptr fs:[0],ecx  
004065F8   |.  C9              leave  
004065F9   \.  C2 0C00         retn 0C  
```

由 `0040655E` 跟进，以下为此次重点，关键算法，到这里了，有一种自豪感吧。

```
004066F4   |.  33D2           xor edx,edx             ; edx 清 0  
004066F6   |.  8A01           mov al,byte ptr ds:[ecx]    ；第一个字符“j”，HEX 值为 61，放入 al  
004066F8   |.  84C0           test al,al                
004066FA   |.  74 25          je short AXMAN.00406721   ;是 0 的话跳走  
004066FC   |.  56              push esi                 ；开始计算  
004066FD   |>  0FBEC0          /movsx eax,al             ；eax=al  
00406700   |.  C1E2 04        |shl edx,4               ; edx 左移 0x4  
00406703   |.  03D0           |add edx,eax             ；edx=edx+eax  
00406705   |.  41              |inc ecx                ; ecx 加 1，也就是上面指向字符串地址向后移一位，取下一个字符  
00406706   |.  8BC2           |mov eax,edx             ；把 edx 值给 eax  
00406708   |.  25 000000F0    |and eax,F0000000        ；与 0xF0000000  
0040670D   |.  74 07          |je short AXMAN.00406716  ；为 0 的话，至 00406716  
0040670F   |.  8BF0           |mov esi,eax  
00406711   |.  C1EE 18        |shr esi,18  
00406714   |.  33D6           |xor edx,esi             ；异或  
00406716   |>  F7D0           |not eax                 ；取反  
00406718   |.  23D0           |and edx,eax             ；  
0040671A   |.  8A01           |mov al,byte ptr ds:[ecx]   ； 取下一个字符放入 al 里  
0040671C   |.  84C0           |test al,al             ；检查是否为 0，也就是是否取完字符串中的字符  
0040671E   |.^ 75 DD         \jnz short AXMAN.004066FD   ；满足条件的话，继续计算  
00406720   |.  5E              pop esi  
00406721   |>  8B4424 08     mov eax,dword ptr ss:[esp+8]  
00406725   |.  8910           mov dword ptr ds:[eax],edx  ;最终结果放 EAX 指向的地址里  
00406727   \.  C2 0800        retn 8  
```

### 破解总结

由此向上追溯到 `00406480` 处，为比较真假注册码。此注册代码部分有三个关键 `call`，分别是 `00406464`、`00406491` 和 `004066F0`。个人认为关键算法倒不是很难，难点在于要理解“数据处理”，也就是把 `name` 和 `company` 字符串和 `"My Love"` `strcat` 在一起的部分，和将注册码每三个数字间加 `"-"` 这两个部分运算。好在 `ollydbg` 调试结构原理让我们能清晰、直观地分析这些调用。该注册机在本人文件空间 [`estudy.ys168.com`](http://estudy.ys168.com) 可以下载。

### 用户名、密码

提供一组注册码：

`name:jackily`
`company:http://estudy.ys168.com`
`code:156-936-773`

### 算法注册机

```c
/* axman 3.12rGB c 语言注册机 */
/* 在 Turboc 2.0 下调试通过  */

#include "stdio.h"
#include "string.h"
#include "stdlib.h"

main()
{ char name[100]={0},company[50]={0},code[50]={0};
  int i;
  unsigned long l=0,k,m;
  printf("Axman 3.12rGB KeyGen by jackily 2004-12-15\n");
  printf("Email:jackily_zhang@msn.com or jackily_zhang@yahoo.com.cn\n");
  printf("please input name:");
  scanf("%s",name);
  printf("\nplease input company:");
  scanf("%s",company);
  strcat(name,company);
  strlwr(name);
  strcat(name,"My Love");
  for (i=0;i<strlen(name);i++)
     { k=name[i];l<<=0x4;l+=name[i];k=l;
     if ((k&=0xf0000000)!=0) { m=k; m>>=0x18; l^=m;}
     k=~k; /*0xffffffff-k;*/  /*not eax*/
     l&=k;
     }
  ltoa(l,code,10);  /* change "l" into string */
  printf("\nserial number is:");
  for (i=0;i<3;i++) printf("%c",code[i]);
  printf("-");
  for (i=3;i<6;i++) printf("%c",code[i]);
  printf("-");
  for (i=6;i<strlen(code);i++)
  printf("%c",code[i]);
}
```



```
if (9-strlen(code)==1) printf("0");  /* 注册码长度小于 9 时，补 0 */

if (9-strlen(code)==2) printf("00");

}
```
--------------------------------------------------------------------------------

**【版权声明】** 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`2004-11-15`

`4dd466d0e1e0cc123afbe504da2462ca`
