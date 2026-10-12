
# LingoLessons 1.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-116.htm`](https://www.pediy.com/kssd/pediy09/pediy09-116.htm)

> 【文章标题】: LingoLessons 1.0 算法分析  
> 【文章作者】: 坚持到底  
> 【软件名称】: LingoLessons 1.0  
> 【软件大小】: 1093KB  
> 【下载地址】: `http://www.newhua.com/soft/12335.htm`  
> 【加壳方式】: 无  
> 【编写语言】: VC8 -> Microsoft Corporation \*  
> 【使用工具】: `flyODBG`, `PEID`  
> 【操作平台】: `WINXP2`  
> 【软件介绍】: LingoLessons 该软件使用桌面闪铄卡片帮助用户学习....  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  

> -------------------------------------------------------------------------  
> 【详细过程】  

```
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//下 BP MessageBoxW 到这里
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////

00419F73    894424 60            mov dword ptr ss:[esp+60],eax  
00419F77    894424 5C            mov dword ptr ss:[esp+5C],eax  
00419F7B    FF15 84044400        call dword ptr ds:[<&USER32.SendMessageW>]          ; USER32.SendMessageW  
00419F81    85C0                 test eax,eax  
00419F83    74 66                je short LingoLes.00419FEB  
00419F85    8B86 94000000        mov eax,dword ptr ds:[esi+94]  
00419F8B    8B2D 78044400        mov ebp,dword ptr ds:[<&USER32.GetWindowTextLengthW>>; USER32.GetWindowTextLengthW  
00419F91    8DBE 94000000        lea edi,dword ptr ds:[esi+94]  
00419F97    50                   push eax  
00419F98    FFD5                 call ebp  
00419F9A    8D58 01              lea ebx,dword ptr ds:[eax+1]  
00419F9D    53                   push ebx  
00419F9E    8D4C24 54            lea ecx,dword ptr ss:[esp+54]  
00419FA2    E8 299DFEFF          call LingoLes.00403CD0  
00419FA7    53                   push ebx  
00419FA8    50                   push eax  
00419FA9    8BCF                 mov ecx,edi  
00419FAB    E8 C099FEFF          call LingoLes.00403970  
00419FB0    50                   push eax  
00419FB1    8D4C24 54            lea ecx,dword ptr ss:[esp+54]  
00419FB5    E8 969DFEFF          call LingoLes.00403D50  
00419FBA    8B8E 98000000        mov ecx,dword ptr ds:[esi+98]                         ; //email  
00419FC0    8DBE 98000000        lea edi,dword ptr ds:[esi+98]  
00419FC6    51                   push ecx  
00419FC7    FFD5                 call ebp  
00419FC9    8D58 01              lea ebx,dword ptr ds:[eax+1]  
00419FCC    53                   push ebx  
00419FCD    8D4C24 50            lea ecx,dword ptr ss:[esp+50]  
00419FD1    E8 FA9CFEFF          call LingoLes.00403CD0  
00419FD6    53                   push ebx  
00419FD7    50                   push eax  
00419FD8    8BCF                 mov ecx,edi  
00419FDA    E8 9199FEFF          call LingoLes.00403970  
00419FDF    50                   push eax  
00419FE0    8D4C24 50            lea ecx,dword ptr ss:[esp+50]  
00419FE4    E8 679DFEFF          call LingoLes.00403D50                                ; //得到注册码  
00419FE9    EB 25                jmp short LingoLes.0041A010  
00419FEB    8D4E 30              lea ecx,dword ptr ds:[esi+30]  
00419FEE    E8 0DFCFFFF          call LingoLes.00419C00  
00419FF3    85C0                 test eax,eax  
00419FF5    74 19                je short LingoLes.0041A010  
00419FF7    8D5424 4C            lea edx,dword ptr ss:[esp+4C]  
00419FFB    52                   push edx  
00419FFC    8D4424 54            lea eax,dword ptr ss:[esp+54]  
0041A000    50                   push eax  
0041A001    8BCE                 mov ecx,esi  
0041A003    E8 98F8FFFF          call LingoLes.004198A0  
0041A008    84C0                 test al,al  
0041A00A    0F84 B0000000        je LingoLes.0041A0C0  
0041A010    8B6C24 4C            mov ebp,dword ptr ss:[esp+4C]  
0041A014    55                   push ebp  
0041A015    8D4C24 30            lea ecx,dword ptr ss:[esp+30]  
0041A019    E8 42F5FEFF          call LingoLes.00409560  
0041A01E    8B7C24 50            mov edi,dword ptr ss:[esp+50]  
0041A022    50                   push eax  
0041A023    57                   push edi  
0041A024    8D4C24 18            lea ecx,dword ptr ss:[esp+18]  
0041A028    E8 33F5FEFF          call LingoLes.00409560  
0041A02D    50                   push eax  
0041A02E    E8 BD8CFEFF          call LingoLes.00402CF0  
0041A033    8BC8                 mov ecx,eax  
0041A035    E8 768BFEFF          call LingoLes.00402BB0                                ; //关键 call1  
0041A03A    8D4C24 10            lea ecx,dword ptr ss:[esp+10]  
0041A03E    8AD8                 mov bl,al  
0041A040    E8 0B79FEFF          call LingoLes.00401950  
0041A045    8D4C24 2C            lea ecx,dword ptr ss:[esp+2C]  
0041A049    E8 0279FEFF          call LingoLes.00401950  
0041A04E    84DB                 test bl,bl  
0041A050    74 58                je short LingoLes.0041A0AA  
0041A052    55                   push ebp  
0041A053    8D4C24 14            lea ecx,dword ptr ss:[esp+14]  
0041A057    E8 04F5FEFF          call LingoLes.00409560  
0041A05C    50                   push eax  
0041A05D    57                   push edi  
0041A05E    8D4C24 34            lea ecx,dword ptr ss:[esp+34]  
0041A062    E8 F9F4FEFF          call LingoLes.00409560  
0041A067    50                   push eax  
0041A068    E8 E3390000          call LingoLes.0041DA50  
0041A06D    8BC8                 mov ecx,eax  
0041A06F    E8 5C180000          call LingoLes.0041B8D0  
0041A074    8D4C24 2C            lea ecx,dword ptr ss:[esp+2C]  
0041A078    E8 D378FEFF          call LingoLes.00401950  
0041A07D    8D4C24 10            lea ecx,dword ptr ss:[esp+10]  
0041A081    E8 CA78FEFF          call LingoLes.00401950  
0041A086    8B4E 04              mov ecx,dword ptr ds:[esi+4]  
0041A089    6A 40                push 40  
0041A08B    68 9C264400          push LingoLes.0044269C                                ; UNICODE "Registration Successful"  
0041A090    68 90254400          push LingoLes.00442590                                ; UNICODE "Registration complete.  

Thank you for supporting this software.  

Your contribution helps us bring "  
0041A095    51                   push ecx  
0041A096    FF15 78034400        call dword ptr ds:[<&USER32.MessageBoxW>]             ; USER32.MessageBoxW  
0041A09C    8B56 04              mov edx,dword ptr ds:[esi+4]  
0041A09F    6A 01                push 1  
0041A0A1    52                   push edx  
0041A0A2    FF15 0C034400        call dword ptr ds:[<&USER32.EndDialog>]               ; USER32.EndDialog  
0041A0A8    EB 16                jmp short LingoLes.0041A0C0  
0041A0AA    8B46 04              mov eax,dword ptr ds:[esi+4]  
0041A0AD    6A 10                push 10  
0041A0AF    68 5C254400          push LingoLes.0044255C                                ; UNICODE "Registration unsuccessful"  
0041A0B4    68 F0234400          push LingoLes.004423F0                                ; UNICODE "Registration failed.  

Please check the email address and registration code you entered.  

If you co"  
0041A0B9    50                   push eax  
0041A0BA    FF15 78034400        call dword ptr ds:[<&USER32.MessageBoxW>]             ; USER32.MessageBoxW  
0041A0C0    8D4C24 4C            lea ecx,dword ptr ss:[esp+4C]  

  
```



### 跟进关键 call1

```
00402BBB    50                     push eax
00402BBC    8D4C24 28              lea ecx,dword ptr ss:[esp+28]
00402BC0    51                     push ecx
00402BC1    E8 1A1C0200            call LingoLes.004247E0
00402BC6    8B5424 4C              mov edx,dword ptr ss:[esp+4C]
00402BCA    83C4 08                add esp,8
00402BCD    50                     push eax
00402BCE    52                     push edx
00402BCF    8D4424 10              lea eax,dword ptr ss:[esp+10]
00402BD3    50                     push eax
00402BD4    E8 071C0200            call LingoLes.004247E0
00402BD9    83C4 08                add esp,8
00402BDC    50                     push eax
00402BDD    8BCE                   mov ecx,esi
00402BDF    E8 0CFDFFFF            call LingoLes.004028F0                 ; //关键 call2
00402BE4    BE 10000000            mov esi,10
00402BE9    397424 20              cmp dword ptr ss:[esp+20],esi
00402BED    8AD8                   mov bl,al
00402BEF    72 0D                  jb short LingoLes.00402BFE
00402BF1    8B4C24 0C              mov ecx,dword ptr ss:[esp+C]
00402BF5    51                     push ecx
00402BF6    E8 EF480200            call LingoLes.004274EA
00402BFB    83C4 04                add esp,4
```

### 跟进关键 call2

```
00402902    83C0 04                add eax,4
00402905    50                     push eax
00402906    8D4424 04              lea eax,dword ptr ss:[esp+4]
0040290A    50                     push eax
0040290B    E8 F0FEFFFF            call LingoLes.00402800                 ; //关键 call3
00402910    8B5424 24              mov edx,dword ptr ss:[esp+24]
00402914    837A 18 10             cmp dword ptr ds:[edx+18],10
00402918    8B4A 14                mov ecx,dword ptr ds:[edx+14]
0040291B    72 05                  jb short LingoLes.00402922
0040291D    8B52 04                mov edx,dword ptr ds:[edx+4]
00402920    EB 03                  jmp short LingoLes.00402925
00402922    83C2 04                add edx,4
00402925    53                     push ebx
00402926    51                     push ecx
00402927    8B48 14                mov ecx,dword ptr ds:[eax+14]
0040292A    52                     push edx
0040292B    51                     push ecx
0040292C    6A 00                  push 0
0040292E    8BC8                   mov ecx,eax
00402930    E8 CBE9FFFF            call LingoLes.00401300                 ; //比较真假注册码
00402935    85C0                   test eax,eax
00402937    0F94C3                 sete bl
0040293A    837C24 1C 10           cmp dword ptr ss:[esp+1C],10
0040293F    72 0D                  jb short LingoLes.0040294E
00402941    8B5424 08              mov edx,dword ptr ss:[esp+8]
00402945    52                     push edx
00402946    E8 9F4B0200            call LingoLes.004274EA
0040294B    83C4 04                add esp,4
0040294E    8AC3                   mov al,bl
00402950    5B                     pop ebx
00402951    83C4 1C                add esp,1C
00402954    C2 0800                retn 8
```

### 跟进关键 call3

```
00402813    50                     push eax
00402814    6A 17                  push 17                                  //参数 0x17 入栈(就是下面的[ESP+40]值)
00402816    8BF9                   mov edi,ecx
00402818    33DB                   xor ebx,ebx
0040281A    55                     push ebp
0040281B    8D4C24 20              lea ecx,dword ptr ss:[esp+20]
0040281F    51                     push ecx
00402820    C746 18 0F000000       mov dword ptr ds:[esi+18],0F
00402827    895E 14                mov dword ptr ds:[esi+14],ebx
0040282A    8BCF                   mov ecx,edi
0040282C    885E 04                mov byte ptr ds:[esi+4],bl
0040282F    895C24 20              mov dword ptr ss:[esp+20],ebx
00402833    E8 B8FEFFFF            call LingoLes.004026F0                 ; //计算注册码的第一部分
00402838    6A FF                  push -1
0040283A    53                     push ebx
0040283B    50                     push eax
0040283C    8BCE                   mov ecx,esi
0040283E    E8 BDF5FFFF            call LingoLes.00401E00
00402843    BB 10000000            mov ebx,10
00402848    395C24 2C              cmp dword ptr ss:[esp+2C],ebx
0040284C    72 0D                  jb short LingoLes.0040285B
0040284E    8B5424 18              mov edx,dword ptr ss:[esp+18]
00402852    52                     push edx
00402853    E8 924C0200            call LingoLes.004274EA
00402858    83C4 04                add esp,4
0040285B    6A 01                  push 1
0040285D    68 6C074400            push LingoLes.0044076C
00402862    8BCE                   mov ecx,esi
00402864    E8 97F6FFFF            call LingoLes.00401F00
00402869    8D4424 10              lea eax,dword ptr ss:[esp+10]
0040286D    50                     push eax
0040286E    6A 47                  push 47                                  //参数 0x47 入栈(就是下面的[ESP+40]值)
00402870    55                     push ebp
00402871    8D4C24 20              lea ecx,dword ptr ss:[esp+20]
00402875    51                     push ecx
00402876    8BCF                   mov ecx,edi
00402878    E8 73FEFFFF            call LingoLes.004026F0                 ; //计算注册码的第二部分
0040287D    6A FF                  push -1
0040287F    6A 00                  push 0
00402881    50                     push eax
00402882    8BCE                   mov ecx,esi
00402884    E8 77F5FFFF            call LingoLes.00401E00
00402889    395C24 2C              cmp dword ptr ss:[esp+2C],ebx
0040288D    72 0D                  jb short LingoLes.0040289C
0040288F    8B5424 18              mov edx,dword ptr ss:[esp+18]
00402893    52                     push edx
00402894    E8 514C0200            call LingoLes.004274EA
00402899    83C4 04                add esp,4
0040289C    6A 01                  push 1
0040289E    68 6C074400            push LingoLes.0044076C
004028A3    8BCE                   mov ecx,esi
004028A5    E8 56F6FFFF            call LingoLes.00401F00
004028AA    8D4424 10              lea eax,dword ptr ss:[esp+10]
004028AE    50                     push eax
004028AF    6A 09                  push 9                                   //参数 0x9 入栈(就是下面的[ESP+40]值)
004028B1    55                     push ebp
004028B2    8D4C24 20              lea ecx,dword ptr ss:[esp+20]
004028B6    51                     push ecx
004028B7    8BCF                   mov ecx,edi
004028B9    E8 32FEFFFF            call LingoLes.004026F0                 ; //计算注册码的第三部分
```



**004028BE** `6A FF` `push -1`

**004028C0** `6A 00` `push 0`

**004028C2** `50` `push eax`

**004028C3** `8BCE` `mov ecx,esi`

**004028C5** `E8 36F5FFFF` `call LingoLes.00401E00`

**004028CA** `395C24 2C` `cmp dword ptr ss:[esp+2C],ebx`

**004028CE** `72 0D` `jb short LingoLes.004028DD`

**004028D0** `8B5424 18` `mov edx,dword ptr ss:[esp+18]`

```
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////
//计算注册码的算法
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////

00402712    8D48 01               lea ecx,dword ptr ds:[eax+1]
00402715    C745 18 0F000000      mov dword ptr ss:[ebp+18],0F
0040271C    8955 14               mov dword ptr ss:[ebp+14],edx
0040271F    896C24 14             mov dword ptr ss:[esp+14],ebp
00402723    33DB                  xor ebx,ebx
00402725    8855 04               mov byte ptr ss:[ebp+4],dl
00402728    C64424 18 0C          mov byte ptr ss:[esp+18],0C    ; //初始化数组 记为 A
0040272D    C64424 19 E8          mov byte ptr ss:[esp+19],0E8   
00402732    C64424 1A 2B          mov byte ptr ss:[esp+1A],2B    
00402737    C64424 1B 4D          mov byte ptr ss:[esp+1B],4D    
0040273C    C64424 1C 65          mov byte ptr ss:[esp+1C],65    
00402741    C64424 1D FD          mov byte ptr ss:[esp+1D],0FD   
00402746    C64424 1E A5          mov byte ptr ss:[esp+1E],0A5   
0040274B    C64424 1F 7B          mov byte ptr ss:[esp+1F],7B    
00402750    894C24 10             mov dword ptr ss:[esp+10],ecx
00402754    8A08                  mov cl,byte ptr ds:[eax]
00402756    83C0 01               add eax,1
00402759    84C9                  test cl,cl
0040275B    75 F7                 jnz short LingoLes.00402754
0040275D    2B4424 10             sub eax,dword ptr ss:[esp+10]       ; //得到 Email 的位数
00402761    74 41                 je short LingoLes.004027A4
00402763    8B06                  mov eax,dword ptr ds:[esi]
00402765    8A0C17                mov cl,byte ptr ds:[edi+edx]        ; //循环取 Email 的各位到 cl
00402768    0FB66C04 18           movzx ebp,byte ptr ss:[esp+eax+18]   // 从[ESI]＋1 位开始循环取数组中的元素到 EBP
0040276D    0FB6C9                movzx ecx,cl
00402770    33E9                  xor ebp,ecx                         ; // Email 的各位^EBP
00402772    0FAF6C24 40           imul ebp,dword ptr ss:[esp+40]      ; // (Email 的各位^EBP)*[ESP+40]
00402777    83C0 01               add eax,1
0040277A    03DD                  add ebx,ebp                         ; // 累加在 EBX
0040277C    83F8 08               cmp eax,8  
0040277F    8906                  mov dword ptr ds:[esi],eax          // [ESI]=EAX  保存 EAX 的值
00402781    7C 06                 jl short LingoLes.00402789          // 取完数组 A 的最后一位再从第一位开始取  
00402783    C706 00000000         mov dword ptr ds:[esi],0
00402789    8BC7                  mov eax,edi
0040278B    83C2 01               add edx,1
0040278E    8D68 01               lea ebp,dword ptr ds:[eax+1]
00402791    8A08                  mov cl,byte ptr ds:[eax]
00402793    83C0 01               add eax,1
00402796    84C9                  test cl,cl
00402798    75 F7                 jnz short LingoLes.00402791
0040279A    2BC5                  sub eax,ebp
0040279C    3BD0                  cmp edx,eax
0040279E    72 C3                 jb short LingoLes.00402763
004027A0    8B6C24 14             mov ebp,dword ptr ss:[esp+14]
004027A4    81E3 FFFF0000         and ebx,0FFFF                         ; //计算结果^=0xffff
004027AA    53                    push ebx
004027AB    8D5424 24             lea edx,dword ptr ss:[esp+24]
004027AF    68 64074400           push LingoLes.00440764                ; ASCII "%04x"
004027B4    52                    push edx
004027B5    E8 C0550200          call LingoLes.00427D7A
```

**【经验总结】**

总结：

算法蛮简单的，`keygen` 代码

刚学 `C` 不久，代码写得很乱。。。。。。。。

```c
#include<stdio.h>
#include<string.h>

void main()
{    
    char email[64]={0};
    long sn[3]={0};
    int esp, esi=0, i, j, z, len, k[]={0x17,0x47,9};
    int a[]={0xc,0xe8,0x2b,0x4d,0x65,0xfd,0xa5,0x7b};  //初始化数组
    printf("请输入 Email 地址:\n");
    gets(email);
    len=strlen(email);
    for(z=0, j=esi; z<3; z++)  //循环计算三次
    {  
        for(i=0; i<len; i++)
        {  
            sn[z]+= (email[i]^a[j]) * k[z];
            j++;
            esi=j;
            if(j>=8)
                j=0;
        }  
        sn[z] &= 0xffff;
    }
    printf("序列号:\n");
    printf("%lx-%lx-%lx", sn[0], sn[1], sn[2]);
    getch();
}
```

在 `Dev-C++` 和 `winxp2` 下编译成功！

**【版权声明】**: 本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

2007 年 04 月 10 日 23:38:33
