

# ASProtect SKE 2.4 0226 之 98 记事本

> 原文：[`www.pediy.com/kssd/pediy10/70102.html`](https://www.pediy.com/kssd/pediy10/70102.html)

【文章标题】: ASProtect SKE 2.4 0226 之 98 记事本  
【文章作者】: yangjt  
【作者邮箱】: yangjietao123@163.com  
【作者主页】: http://blog.sina.com.cn/yangjt  
【作者 QQ 号】: 325002492  
【下载地址】: 自己搜索下载  
【保护方式】: ASProtect SKE 2.4 0226  
【使用工具】: OllyICE,PhantOm  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  /\*  
  保护时选择了一下项目：  
  Resources Protection  
  Preserve Extra Data  
  Anti-Debugger Protection  
  CheckSum Protection  
  \*/  
  
![](img/5483ca54b538354500d2adef1eaa2beb.jpg)  
    
  //入口：  
    
  00401000 >  68 01D04000     push    0040D001  
  00401005    E8 01000000     call    0040100B  
  0040100A    C3              retn  
  0040100B    C3              retn  
    
    
  //PhantOm 全选以后 Ctrl+G 来到 GetSystemTime,F2 设置断点，然后 F9 运行程序，中断两次取消断点返回。  
  7C80176F kernel32.GetSystemTime       8BFF            mov     edi, edi    //F2  
  7C801771                              55              push    ebp  
  7C801772                              8BEC            mov     ebp, esp  
  7C801774                              83EC 18         sub     esp, 18  
  7C801777                              A1 1800FE7F     mov     eax, dword ptr \[7FFE0018\]  
    
    
  //返回到此处  
  00AC2777    0FB745 F0       movzx   eax, word ptr \[ebp\-10\]  
  00AC277B    6BC0 3C         imul    eax, eax, 3C  
  00AC277E    66:0345 F2      add     ax, word ptr \[ebp\-E\]  
  00AC2782    6BC0 3C         imul    eax, eax, 3C  
  00AC2785    31D2            xor     edx, edx  
  00AC2787    66:8B55 F4      mov     dx, word ptr \[ebp\-C\]  
  00AC278B    01D0            add     eax, edx  
  00AC278D    69C0 E8030000   imul    eax, eax, 3E8  
  00AC2793    66:8B55 F6      mov     dx, word ptr \[ebp\-A\]  
  00AC2797    01D0            add     eax, edx  
  00AC2799    8905 3C30B000   mov     dword ptr \[B0303C\], eax  
  00AC279F    8BE5            mov     esp, ebp  
  00AC27A1    5D              pop     ebp  
  00AC27A2    C3              retn        //此处 F2 然后 F9 运行到这里，之后 F8 单步返回  
    
  00AE15D0    8BC7            mov     eax, edi       //返回到此处  
  00AE15D2    E8 39FAFFFF     call    00AE1010  
  00AE15D7    C747 18 FFFFFFF>mov     dword ptr \[edi+18\], -1  
  00AE15DE    A1 482BB000     mov     eax, dword ptr \[B02B48\]  
  00AE15E3    83C0 0D         add     eax, 0D  
  00AE15E6    8947 0C         mov     dword ptr \[edi+C\], eax  
  00AE15E9    33D2            xor     edx, edx  
  .......  
  .......  
  00AE1683    83C4 0C         add     esp, 0C  
  00AE1686    8BC7            mov     eax, edi  
  00AE1688    5F              pop     edi  
  00AE1689    5E              pop     esi  
  00AE168A    5B              pop     ebx  
  00AE168B    C3              retn        //段尾 Ret 处 F2 然后 F9，F8  
    
  00B00226    8B15 982BB000   mov     edx, dword ptr \[B02B98\]  //返回到这里  
  00B0022C    8902            mov     dword ptr \[edx\], eax  
  00B0022E    A1 982BB000     mov     eax, dword ptr \[B02B98\]  
  00B00233    8B00            mov     eax, dword ptr \[eax\]  
  00B00235    E8 462BFEFF     call    00AE2D80  
  .......  
  .......  
  00B002C5    59              pop     ecx  
  00B002C6    5A              pop     edx  
  00B002C7    5B              pop     ebx  
  00B002C8    C3              retn        //段尾 Ret 处 F2 然后 F9，F8  
    
    
    
  00ACED60    68 A75915E2     push    E21559A7                 //返回到这里……  
  00ACED65    68 F0290000     push    29F0  
  00ACED6A    68 B4150200     push    215B4  
  00ACED6F    68 C0200000     push    20C0  
  00ACED74    68 9CCC0000     push    0CC9C  
  00ACED79    68 00500500     push    55000  
  00ACED7E    FF35 D434B000   push    dword ptr \[B034D4\]  
  00ACED84    E8 23D1FFFF     call    00ACBEAC  
  00ACED89    310424          xor     dword ptr \[esp\], eax  
  00ACED8C    8B05 D434B000   mov     eax, dword ptr \[B034D4\]  
  00ACED92    010424          add     dword ptr \[esp\], eax  
  00ACED95    C3              retn                            //段尾 Ret 处 F2 然后 F9，F8  
    
    
  00B002E0    68 9B311C0B     push    0B1C319B                 //返回到这里……  
  00B002E5    68 E0020000     push    2E0  
  00B002EA    68 6C590100     push    1596C  
  00B002EF    68 5C140000     push    145C  
  00B002F4    68 80EE0300     push    3EE80  
  00B002F9    68 00500500     push    55000  
  00B002FE    FF35 D434B000   push    dword ptr \[B034D4\]  
  00B00304    E8 01000000     call    00B0030A  
  00B00309    8183 C404E89A B>add     dword ptr \[ebx+9AE804C4\], E8FFFC>  
  00B00313    0100            add     dword ptr \[eax\], eax  
  00B00315    0000            add     byte ptr \[eax\], al  
  00B00317    8183 C4043104 2>add     dword ptr \[ebx+43104C4\], 1E824  
  00B00321    0000            add     byte ptr \[eax\], al  
  00B00323    68 83C4048B     push    8B04C483  
  00B00328    05 D434B000     add     eax, 0B034D4  
  00B0032D    E8 02000000     call    00B00334  
  00B00332    E8 6883C404     call    0574869F  
  00B00337    010424          add     dword ptr \[esp\], eax  
  00B0033A    C3              retn                           //段尾 Ret 处 F2 然后 F9，F8  
    
    
  00AFFC8A    E8 61F1FFFF     call    00AFEDF0  //在离返回出最近的一处 Call 处 F2 然后 F9，F7  
  00AFFC8F    83C4 24         add     esp, 24  
  00AFFC92    5F              pop     edi  
  00AFFC93    5E              pop     esi  
  00AFFC94    5B              pop     ebx  
  00AFFC95    C3              retn  
  00AFFC96    8BC0            mov     eax, eax  
  00AFFC98    E8 E7FEFFFF     call    00AFFB84   //返回到这里  
  00AFFC9D    C3              retn  
    
  00AFEDF0    BF 02DB4A00     mov     edi, 4ADB02       //到这里，往下单步跟踪到返回  
  00AFEDF5    337C24 28       xor     edi, dword ptr \[esp+28\]  
  ......  
  ......  
  00AFEF17    037C24 18       add     edi, dword ptr \[esp+18\]  
  00AFEF1B    C3              retn                     //返回处  
    
    
    
  01B70000    80F2 23         xor     dl, 23                  //返回到这里  
  01B70003    E8 05000000     call    01B7000D  
  01B70008    D99E 7F4C95B6   fstp    dword ptr \[esi+B6954C7F\]  
  01B7000E    DB58 BE         fistp   dword ptr \[eax\-42\]  
  01B70011    1312            adc     edx, dword ptr \[edx\]  
  01B70013    696D 81 C0E2000>imul    ebp, dword ptr \[ebp\-7F\], 0E2C0  
  01B7001A    0068 0A         add     byte ptr \[eax+A\], ch  
    
  //然后往下拉……  
    
  01B700B5    15 2A1BB891     adc     eax, 91B81B2A  
    
  01B7008B   /E9 36000000     jmp     01B700C6             //上面最近一处 Jmp，Enter 跟随  
  01B70090   |3B58 B1         cmp     ebx, dword ptr \[eax\-4F\]  
  01B70093   |96              xchg    eax, esi  
  01B70094   |17              pop     ss  
  01B70095   |04 ED           add     al, 0ED  
  01B70097   |22B3 70E96E0F   and     dh, byte ptr \[ebx+F6EE970\]  
  01B7009D   |9C              pushfd  
  01B7009E   |A5              movs    dword ptr es:\[edi\], dword ptr \[e>  
  01B7009F   |0F8C 09000000   jl      01B700AE  
  01B700A5   |68 07547301     push    1735407  
  01B700AA   |66:8BD8         mov     bx, ax  
  01B700AD   |5B              pop     ebx  
  01B700AE  ^\\E9 7AFFFFFF     jmp     01B7002D         //此处有个向回跳的大跳转，判断上面的代码用于返回代码的解压缩  
  01B700B3    FFCC            dec     esp  
  .....  
  .....  
  01B700C6   /EB 51           jmp     short 01B70119    //跟随到此处……硬件执行断点  
  01B700C8   |34 E2           xor     al, 0E2  
  01B700CA   |3006            xor     byte ptr \[esi\], al  
  01B700CC   |AE              scas    byte ptr es:\[edi\]  
  01B700CD   |BC 485E755F     mov     esp, 5F755E48  
  01B700D2   |8890 D6B0FB53   mov     byte ptr \[eax+53FBB0D6\], dl  
    
  //下面代码漏出真实面目……  
    
  01B700C6    BB 4EE24900     mov     ebx, 49E24E  
  01B700CB    035C24 18       add     ebx, dword ptr \[esp+18\]  
  01B700CF    5B              pop     ebx  
  01B700D0    23C7            and     eax, edi  
  01B700D2    58              pop     eax  
  01B700D3    8D8418 CB50AFF9 lea     eax, dword ptr \[eax+ebx+F9AF50CB>  
  01B700DA    2BC3            sub     eax, ebx  
  01B700DC    03C3            add     eax, ebx  
  01B700DE    5C              pop     esp  
  01B700DF    FFE0            jmp     eax            //飞向光明之巅……  
    
    
  004010CC    55              push    ebp                //到达 OEP  
  004010CD    8BEC            mov     ebp, esp  
  004010CF    83EC 44         sub     esp, 44  
  004010D2    56              push    esi  
  004010D3    E8 28EF7801     call    01B90000           //次出被 ASPr 保护……现在想办法恢复回来……硬件执行断点  
  004010D8    AE              scas    byte ptr es:\[edi\]  
  004010D9    8BF0            mov     esi, eax  
  004010DB    8A00            mov     al, byte ptr \[eax\]  
  004010DD    3C 22           cmp     al, 22  
  004010DF    75 1B           jnz     short 004010FC  
  004010E1    56              push    esi  
  004010E2    FF15 F4644000   call    dword ptr \[4064F4\]               ; USER32.CharNextA  
    
    
    
  //中断两次以后此处变成这个样……  
  004010D3    E8 2CEF9900     call    00DA0004   //F7 进  
    
  //ASPr 很狡猾……把系统 Call 代码抽到自己家里运行去了  
    
  00DA0004    FF0424          inc     dword ptr \[esp\]  
  00DA0007  - E9 F4FFF4FF     jmp     00CF0000   //到这里看看……  
    
    
  00CF0000    A1 F455887C     mov     eax, dword ptr \[7C8855F4\]         //此处正是 kernel32.GetCommandLineA 的入口代码  
  00CF0005    C3              retn  
  //对比一下  
  7C812FAD kernel32.GetCommandLineA     A1 F455887C     mov     eax, dword ptr \[7C8855F4\]  
  7C812FB2                              C3              retn  
  7C812FB3                              90              nop  
    
  //其他处代码恢复诸如此类……形势一样……  
  //修复好了就可以 Dump，然后 ImportREC 伺候……因为没有选择高技输入表保护，所以修复起来就简单了  
    
\--------------------------------------------------------------------------------  
【经验总结】  
  总结一下，感觉 ASPr2.4SKE 的确狡猾了很多……系统标准 DLL 代码都抽到自己空间运行给恢复代码带来很大困难，即便要恢  
  复，也对代码进行恢复的人员要求很高，必须熟悉 WindowsDll 的标准调用，所以个人感觉还是 Volx 的脚本方便些……呵呵…  
  …本教程只作娱乐，看到没有手脱 2.4SKE 的教程……特此写一篇……当脚本不好用的时候……我想手工可能会好些  
  吧![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")   
                                                       2008 年 08 月 06 日