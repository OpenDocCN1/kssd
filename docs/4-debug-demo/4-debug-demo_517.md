

# FOXIT PDF Editor 2.1 去水印

> 原文：[`www.pediy.com/kssd/pediy10/92928.html`](https://www.pediy.com/kssd/pediy10/92928.html)

本来水平很菜，高手不要笑话我，发这个帖子一是为了作个记录，二是给和我一样菜的人一起学习，共同进步。不说了，直接发我调试的时候的记录  
  
0040863A  |.  68 20C87300   push    0073C820                         ;  edited by foxit pdf editor  
0040863F  |.  8D4C24 20     lea     ecx, dword ptr \[esp+20\]  
00408643  |.  E8 F3F21F00   call    0060793B  
00408648  |.  68 64C87300   push    0073C864                         ;  copyright (c) by foxit software company, 2003 - 2009  
0040864D  |.  8D4C24 14     lea     ecx, dword ptr \[esp+14\]  
00408651  |.  895C24 74     mov     dword ptr \[esp+74\], ebx  
00408655  |.  E8 E1F21F00   call    0060793B  
0040865A  |.  68 D0C77300   push    0073C7D0                         ;  for evaluation only.  
  
\-----------------------------------------  
上面就是水印了(在字符串中查找上面注释中的字符串，真的找到了)  
  
现在就要想办法不执行上面的这段代码了，在 OD 窗口中往上找，看有没有 跳转之类的跳过这段代码不执行它  
于是找到了这段代码(发现下面的代码中第一行代码就是一个跳转，跳到 00408637 就完蛋了，就会执行水印的那段代码，那就直接 NOP)  
  
  
00408621     /75 14         jnz     short 00408637                   ;  jnz     short 00408637 改为 nop  
00408623  |. |33C0          xor     eax, eax  
00408625  |. |5B            pop     ebx  
00408626  |. |8B4C24 58     mov     ecx, dword ptr \[esp+58\]  
0040862A  |. |64:890D 00000>mov     dword ptr fs:\[0\], ecx  
00408631  |. |83C4 64       add     esp, 64  
00408634  |. |C2 0400       retn    4  
  
有这么一段东西  
  
现在还有一个问题，就是程序启动的时候的提示框，下面把它屏蔽掉  
  
  
把跳转 jnz     short 00408637  改为 NOP，就会继续执行下面的代码，遇到 retn 返回了。不会执行到水印的代码里面去  
  
  
  
00407521  |.  50            push    eax  
00407522  |.  51            push    ecx  
00407523  |.  56            push    esi  
00407524  |.  52            push    edx  
00407525  |.  6A 00         push    0  
00407527  |.  53            push    ebx  
00407528  |.  68 A4C67300   push    0073C6A4                         ;  ASCII "2008/01/19"  
0040752D  |.  68 A0C67300   push    0073C6A0                         ;  ASCII "PED"  
00407532  |.  57            push    edi  
00407533  |.  E8 B8DD1A00   call    005B52F0  
00407538  |.  83C4 24       add     esp, 24  
0040753B  |.  8985 A8040000 mov     dword ptr \[ebp+4A8\], eax  
00407541  |.  85C0          test    eax, eax  
00407543      E9 C0000000   jmp     00407608                         ;  这里改一下跳转 原先是 jnz     00407608  
  
  
;为什么会想到这里要跳转呢？原因就是看到下面这段代码的注释中的字符串，在调试的时候改了一下跳转，发现果然好了  
  
00407548      0080 3B00740E add     byte ptr \[eax+E74003B\], al  
0040754E  |.  6A FF         push    -1  
00407550  |.  6A 00         push    0  
00407552  |.  68 92130000   push    1392  
00407557  |.  E8 7CA12000   call    006116D8  
0040755C  |>  B8 80C67300   mov     eax, 0073C680                    ;  ASCII "Unregistered Retail Customer"  
00407561  |>  8A16          /mov     dl, byte ptr \[esi\]  
00407563  |.  8A18          |mov     bl, byte ptr \[eax\]  
00407565  |.  8ACA          |mov     cl, dl  
00407567  |.  3AD3          |cmp     dl, bl  
00407569  |.  75 1E         |jnz     short 00407589  
0040756B  |.  84C9          |test    cl, cl  
0040756D  |.  74 16         |je      short 00407585  
0040756F  |.  8A56 01       |mov     dl, byte ptr \[esi+1\]  
00407572  |.  8A58 01       |mov     bl, byte ptr \[eax+1\]  
00407575  |.  8ACA          |mov     cl, dl  
00407577  |.  3AD3          |cmp     dl, bl  
00407579  |.  75 0E         |jnz     short 00407589  
0040757B  |.  83C6 02       |add     esi, 2  
0040757E  |.  83C0 02       |add     eax, 2  
00407581  |.  84C9          |test    cl, cl  
00407583  |.^ 75 DC         \\jnz     short 00407561  
00407585  |>  33C0          xor     eax, eax  
00407587  |.  EB 05         jmp     short 0040758E  
00407589  |>  1BC0          sbb     eax, eax  
0040758B  |.  83D8 FF       sbb     eax, -1  
0040758E  |>  85C0          test    eax, eax  
00407590  |.  75 0D         jnz     short 0040759F  
00407592  |.  57            push    edi  
00407593  |.  8BCD          mov     ecx, ebp  
00407595  |.  E8 76030000   call    00407910  
0040759A  |.  E9 F5000000   jmp     00407694  
0040759F  |>  6A 00         push    0  
004075A1  |.  8D4C24 14     lea     ecx, dword ptr \[esp+14\]  
004075A5  |.  E8 E6700100   call    0041E690  
004075AA  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]  
004075AE  |.  C78424 E00200>mov     dword ptr \[esp+2E0\], 0  
004075B9  |.  E8 7DF31F00   call    0060693B  
004075BE  |.  8D4C24 74     lea     ecx, dword ptr \[esp+74\]  
004075C2  |.  C78424 E00200>mov     dword ptr \[esp+2E0\], 3  
004075CD  |.  E8 FB022000   call    006078CD  
004075D2  |.  8D4C24 70     lea     ecx, dword ptr \[esp+70\]  
004075D6  |.  C68424 E00200>mov     byte ptr \[esp+2E0\], 2  
004075DE  |.  E8 EA022000   call    006078CD  
004075E3  |.  8D4C24 6C     lea     ecx, dword ptr \[esp+6C\]  
004075E7  |.  C68424 E00200>mov     byte ptr \[esp+2E0\], 1  
004075EF  |.  E8 D9022000   call    006078CD  
004075F4  |.  C78424 E00200>mov     dword ptr \[esp+2E0\], -1  
004075FF  |.  8D4C24 10     lea     ecx, dword ptr \[esp+10\]  
00407603  |.  E9 87000000   jmp     0040768F  
00407608  |>  B8 80C67300   mov     eax, 0073C680                    ;  ASCII "Unregistered Retail Customer"