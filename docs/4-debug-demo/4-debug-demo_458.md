

# 万能五笔 2004 的修改后续（去除 Bug 及自校验,打造绿色五笔）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-91.htm`](https://www.pediy.com/kssd/pediy07/pediy7-91.htm)

【原创】万能五笔 2004 的修改后续（去除 Bug 及自校验,打造绿色五笔）  
  
                         ————手把手系列之七  
  
【破解作者】 jackily  
【作者主页】 [`estudy.ys168.com`](http://estudy.ys168.com)  
             [`jackily.ys168.com`](http://jackily.ys168.com)  
【使用工具】 ollydbg、 stripper、reshacker  
【破解平台】 Win9x/NT/2000/XP  
【加壳方式】 ASPack 2.12 -> Alexey Solodovnikov  
【破解声明】 本破解纯以学习和交流为目的，偶得一点心得，愿与大家分享:)  
\--------------------------------------------------------------------------------  
  
软件名称：万能五笔 2004：EXE 外挂高级 6.41 版  
        
万能五笔 2004 自实行免费以来，一直 BUG 不断。从外挂 6.1 版至 2 月 15 日最新发布的 6.41 版，都存在着强行更改用户主页为 265.com 的问题，而这个 6.4 和 6.41 版更是变本加厉，又出来了个选字搜，每次调用万能五笔，其均自动将“选字搜”selectso.exe 加载，而退出时，却将“选字搜”置之不理，非常不便。于是决定再次拿起手术刀，主要目标“选字搜”。  
  
首先，用 PEID 查壳，发现 ASPack 2.12，用 stripper 去掉外衣。用 ollydbg 加载，先解决了 265.com 的问题。（关于主页修改的问题请参见本人拙文《万能五笔 2004 的修改》，[`bbs.pediy.com/showthread.php?threadid=6168`](http://bbs.pediy.com/showthread.php?threadid=6168)）考虑到每次启动五笔，selectso.exe 都会运行，猜想会用到 CreateProcessA 函数，于是 bpx CreateProcessA。果然在 004018c3 处断下，0040189C 处正是“selectso.exe”。代码分析如下：   
  
00401808   /$  55                   push ebp  
00401809   |.  8BEC                 mov ebp,esp  
0040180B   |.  81EC E0050000        sub esp,5E0  
00401811   |.  53                   push ebx  
00401812   |.  56                   push esi  
00401813   |.  57                   push edi  
00401814   |.  33DB                 xor ebx,ebx  
00401816   |.  68 0C9C4500          push \_wnwb.00459C0C                        ; /MutexName =   
"FinallyIMadeItWorkThisIsMutexForWnwb"  
0040181B   |.  53                   push ebx                                   ; |InitialOwner => FALSE  
0040181C   |.  53                   push ebx                                   ; |pSecurity => NULL  
0040181D   |.  FF15 F4014500        call dword ptr ds:\[<&KERNEL32.CreateMutexA>; \\CreateMutexA  
00401823   |.  8BF0                 mov esi,eax  
00401825   |.  FF15 F8014500        call dword ptr ds:\[<&KERNEL32.GetLastError>; \[GetLastError  
0040182B   |.  3D B7000000          cmp eax,0B7                                ; 判断 wnwb.exe 是否已经加载  
00401830   |.  75 20                jnz short \_wnwb.00401852                   ；否，跳到 401852 继续   
00401832   |.  56                   push esi                                   ; /hObject  
00401833   |.  FF15 FC014500        call dword ptr ds:\[<&KERNEL32.CloseHandle>>; \\CloseHandle  
00401839   |.  53                   push ebx                                   ; /Title => NULL  
0040183A   |.  68 009C4500          push \_wnwb.00459C00                        ; |Class = "WNWB Input"  
0040183F   |.  FF15 B8034500        call dword ptr ds:\[<&USER32.FindWindowA>\]  ; \\FindWindowA  
00401845   |.  3BC3                 cmp eax,ebx                                  
00401847   |.  0F84 6A040000        je \_wnwb.00401CB7  
0040184D   |.  E9 9D030000          jmp \_wnwb.00401BEF  
00401852   |>  6A 10                push 10  
00401854   |.  33C0                 xor eax,eax  
00401856   |.  59                   pop ecx  
00401857   |.  8DBD 6CFFFFFF        lea edi,dword ptr ss:\[ebp-94\]  
0040185D   |.  C785 68FFFFFF 440000>mov dword ptr ss:\[ebp-98\],44  
00401867   |.  6A 40                push 40  
00401869   |.  F3:AB                rep stos dword ptr es:\[edi\]  
0040186B   |.  A0 DC1C4700          mov al,byte ptr ds:\[471CDC\]  
00401870   |.  59                   pop ecx  
00401871   |.  8885 88FCFFFF        mov byte ptr ss:\[ebp-378\],al  
00401877   |.  33C0                 xor eax,eax  
00401879   |.  8DBD 89FCFFFF        lea edi,dword ptr ss:\[ebp-377\]  
0040187F   |.  BE 04010000          mov esi,104  
00401884   |.  F3:AB                rep stos dword ptr es:\[edi\]  
00401886   |.  66:AB                stos word ptr es:\[edi\]  
00401888   |.  AA                   stos byte ptr es:\[edi\]  
00401889   |.  8D85 88FCFFFF        lea eax,dword ptr ss:\[ebp-378\]  
0040188F   |.  56                   push esi  
00401890   |.  50                   push eax  
00401891   |.  E8 2B160200          call \_wnwb.00422EC1  
00401896   |.  8D85 88FCFFFF        lea eax,dword ptr ss:\[ebp-378\]  
0040189C   |.  68 F09B4500          push \_wnwb.00459BF0                    ;  ASCII "\\selectso.exe"  
004018A1   |.  50                   push eax  
004018A2   |.  E8 59E90200          call \_wnwb.00430200  
004018A7   |.  83C4 10              add esp,10  
004018AA   |.  8D45 AC              lea eax,dword ptr ss:\[ebp-54\]  
004018AD   |.  50                   push eax                                   ; /pProcessInfo  
004018AE   |.  8D85 68FFFFFF        lea eax,dword ptr ss:\[ebp-98\]              ; |  
004018B4   |.  50                   push eax                                   ; |pStartupInfo  
004018B5   |.  53                   push ebx                                   ; |CurrentDir  
004018B6   |.  53                   push ebx                                   ; |pEnvironment  
004018B7   |.  53                   push ebx                                   ; |CreationFlags  
004018B8   |.  53                   push ebx                                   ; |InheritHandles  
004018B9   |.  53                   push ebx                                   ; |pThreadSecurity  
004018BA   |.  53                   push ebx                                   ; |pProcessSecurity  
004018BB   |.  8D85 88FCFFFF        lea eax,dword ptr ss:\[ebp-378\]             ; |  
004018C1   |.  53                   push ebx                                   ; |CommandLine  
004018C2   |.  50                   push eax                                   ; |ModuleFileName  
004018C3   |.  FF15 00024500        call dword ptr ds:\[<&KERNEL32.CreateProces>; \\CreateProcessA ，加载程序  
  
004018C9   |.  6A 64                push 64                                    ; /Timeout = 100. ms  
004018CB   |.  FF75 AC              push dword ptr ss:\[ebp-54\]                 ; |hObject  
004018CE   |.  FF15 04024500        call dword ptr ds:\[<&KERNEL32.WaitForSingl>; \\WaitForSingleObject  
004018D4   |.  FF75 B0              push dword ptr ss:\[ebp-50\]                 ; /hObject  
004018D7   |.  8B3D FC014500        mov edi,dword ptr ds:\[<&KERNEL32.CloseHand>; |kernel32.CloseHandle  
004018DD   |.  FFD7                 call edi                                   ; \\CloseHandle  
004018DF   |.  FF75 AC              push dword ptr ss:\[ebp-54\]                 ; /hObject  
004018E2   |.  FFD7                 call edi                                   ; \\CloseHandle  
  
以上是加载 selectso.exe 程序。  
   
....................... 此处代码省略  
  
以下是关键的 BUG 所在：  
00401DC9   |.  FF15 2C024500        call dword ptr ds:\[<&KERNEL32.GetSystemTime>\] ; \\GetSystemTime  
00401DCF   |.  E8 2C120200          call \_wnwb.00423000                     ；此处为修改 265.com 主页之调用，有兴趣  
  
可跟入  
00401DD4   |.  68 B7324200          push \_wnwb.004232B7                           ; /Timerproc = \_wnwb.004232B7  
00401DD9   |.  68 F4010000          push 1F4                                      ; |Timeout = 500. ms  
00401DDE   |.  6A 02                push 2                                        ; |TimerID = 2  
00401DE0   |.  FF35 E0FD4600        push dword ptr ds:\[46FDE0\]                    ; |hWnd = NULL  
00401DE6   |.  FF15 C8034500        call dword ptr ds:\[<&USER32.SetTimer>\]        ; \\SetTimer  
00401DEC   |.  FF35 E0FD4600        push dword ptr ds:\[46FDE0\]  
00401DF2   |.  E8 B8D40200          call \_wnwb.0042F2AF  
00401DF7   |.  59                   pop ecx  
00401DF8   |.  E8 E9090200          call \_wnwb.004227E6  
00401DFD   |.  E8 E3180200          call \_wnwb.004236E5                           ；此处为自校验，跟入可见代码分析  
  
。  
00401E02   |.  8B35 C4034500        mov esi,dword ptr ds:\[<&USER32.PostMessageA>\] ;  USER32.PostMessageA  
  
\-----------------------------------------------------------------------------------  
  
由 00401DFD 跟入，自校验之 call，也就是判断内存中是否存在 selectso.exe，如果是继续，否无条件退出。  
  
004236E5   /$  55                   push ebp  
004236E6   |.  8BEC                 mov ebp,esp  
004236E8   |.  81EC AC030000        sub esp,3AC  
004236EE   |.  53                   push ebx  
004236EF   |.  56                   push esi  
004236F0   |.  57                   push edi  
004236F1   |.  8D85 54FCFFFF        lea eax,dword ptr ss:\[ebp-3AC\]  
004236F7   |.  68 2C010000          push 12C  
004236FC   |.  50                   push eax  
004236FD   |.  C745 EC 38AA4500     mov dword ptr ss:\[ebp-14\],\_wnwb.0045AA38      ;  ASCII "wnwbio.exe"  
00423704   |.  C745 F0 68AB4600     mov dword ptr ss:\[ebp-10\],\_wnwb.0046AB68      ;  ASCII "wnwbio.ime"  
0042370B   |.  C745 F4 5CAB4600     mov dword ptr ss:\[ebp-C\],\_wnwb.0046AB5C       ;  ASCII "loadwb.exe"  
00423712   |.  C745 F8 C0A94600     mov dword ptr ss:\[ebp-8\],\_wnwb.0046A9C0       ;  ASCII "search.exe"  
00423719   |.  E8 A3F7FFFF          call \_wnwb.00422EC1  
0042371E   |.  8B3D 8C034500        mov edi,dword ptr ds:\[<&USER32.PostQuitMessag>;  USER32.PostQuitMessage  
00423724   |.  59                   pop ecx  
00423725   |.  59                   pop ecx  
00423726   |.  8D75 EC              lea esi,dword ptr ss:\[ebp-14\]  
00423729   |.  C745 FC 04000000     mov dword ptr ss:\[ebp-4\],4  
00423730   |.  33DB                 xor ebx,ebx  
00423732   |>  8D85 54FCFFFF        /lea eax,dword ptr ss:\[ebp-3AC\]  
00423738   |.  50                   |push eax  
00423739   |.  8D85 C0FEFFFF        |lea eax,dword ptr ss:\[ebp-140\]  
0042373F   |.  50                   |push eax  
00423740   |.  E8 ABCA0000          |call \_wnwb.004301F0  
00423745   |.  8D85 C0FEFFFF        |lea eax,dword ptr ss:\[ebp-140\]  
0042374B   |.  68 28994500          |push \_wnwb.00459928  
00423750   |.  50                   |push eax  
00423751   |.  E8 AACA0000          |call \_wnwb.00430200  
00423756   |.  FF36                 |push dword ptr ds:\[esi\]  
00423758   |.  8D85 C0FEFFFF        |lea eax,dword ptr ss:\[ebp-140\]  
0042375E   |.  50                   |push eax  
0042375F   |.  E8 9CCA0000          |call \_wnwb.00430200  
00423764   |.  83C4 18              |add esp,18  
00423767   |.  8D85 C0FEFFFF        |lea eax,dword ptr ss:\[ebp-140\]  
0042376D   |.  50                   |push eax                                     ; /FileName  
0042376E   |.  FF15 A0004500        |call dword ptr ds:\[<&KERNEL32.GetFileAttribu>; \\GetFileAttributesA  
00423774   |.  83F8 FF              |cmp eax,-1  
00423777   |.  75 03                |jnz short \_wnwb.0042377C  
00423779   |.  53                   |push ebx  
0042377A   |.  FFD7                 |call edi  
0042377C   |>  83C6 04              |add esi,4  
0042377F   |.  FF4D FC              |dec dword ptr ss:\[ebp-4\]  
00423782   |.^ 75 AE                \\jnz short \_wnwb.00423732  
00423784   |.  8D85 54FCFFFF        lea eax,dword ptr ss:\[ebp-3AC\]  
0042378A   |.  50                   push eax  
0042378B   |.  8D85 C0FEFFFF        lea eax,dword ptr ss:\[ebp-140\]  
00423791   |.  50                   push eax  
00423792   |.  E8 59CA0000          call \_wnwb.004301F0  
00423797   |.  8D85 C0FEFFFF        lea eax,dword ptr ss:\[ebp-140\]  
0042379D   |.  68 50AB4600          push \_wnwb.0046AB50                           ;  ASCII "\\wnwb.exe"  
004237A2   |.  50                   push eax  
004237A3   |.  E8 58CA0000          call \_wnwb.00430200  
004237A8   |.  83C4 10              add esp,10  
004237AB   |.  8D85 80FDFFFF        lea eax,dword ptr ss:\[ebp-280\]  
004237B1   |.  50                   push eax                                      ; /pFindFileData  
004237B2   |.  8D85 C0FEFFFF        lea eax,dword ptr ss:\[ebp-140\]                ; |  
004237B8   |.  50                   push eax                                      ; |FileName  
004237B9   |.  FF15 9C004500        call dword ptr ds:\[<&KERNEL32.FindFirstFileA>>; \\FindFirstFileA ，查找关键  
  
004237BF   |.  50                   push eax                                      ; /hSearch  
004237C0   |.  FF15 98004500        call dword ptr ds:\[<&KERNEL32.FindClose>\]     ; \\FindClose  
004237C6   |.  399D 98FDFFFF        cmp dword ptr ss:\[ebp-268\],ebx  
004237CC   |.  74 19                je short \_wnwb.004237E7  
004237CE   |.  399D 94FDFFFF        cmp dword ptr ss:\[ebp-26C\],ebx  
004237D4   |.  74 11                je short \_wnwb.004237E7  
004237D6   |.  8D45 EC              lea eax,dword ptr ss:\[ebp-14\]  
004237D9   |.  50                   push eax                                      ; /pSystemTime  
004237DA   |.  8D85 94FDFFFF        lea eax,dword ptr ss:\[ebp-26C\]                ; |  
004237E0   |.  50                   push eax                                      ; |pFileTime  
004237E1   |.  FF15 94004500        call dword ptr ds:\[<&KERNEL32.FileTimeToSyste>; \\FileTimeToSystemTime  
004237E7   |>  68 28AB4600          push \_wnwb.0046AB28                           ; /MutexName =   
"IsItNecessaryToDisallowMultipleInstance"  
004237EC   |.  6A 01                push 1                                        ; |InitialOwner = TRUE  
004237EE   |.  53                   push ebx                                      ; |pSecurity  
004237EF   |.  FF15 F4014500        call dword ptr ds:\[<&KERNEL32.CreateMutexA>\]  ; \\CreateMutexA  
004237F5   |.  8BF0                 mov esi,eax  
004237F7   |.  FF15 F8014500        call dword ptr ds:\[<&KERNEL32.GetLastError>\]  ; \[GetLastError  
004237FD   |.  3D B7000000          cmp eax,0B7                                   ; 关键比较  
00423802   |.  74 03                je short \_wnwb.00423807                    ; 内存中已经加载选字搜，跳至 00423807 继续，否则闪人  
00423804   |.  53                   push ebx  
00423805   |.  FFD7                 call edi                        ;USER32!POSTQUITMESSAGE,发送退出消息，准备走人了  
423807   |>  56                   push esi                                      ; /hObject  
00423808   |.  FF15 FC014500        call dword ptr ds:\[<&KERNEL32.CloseHandle>\]   ; \\CloseHandle  
0042380E   |.  5F                   pop edi  
0042380F   |.  5E                   pop esi  
00423810   |.  5B                   pop ebx  
00423811   |.  C9                   leave  
00423812   \\.  C3                   retn  
  
  
  
\--------------------------------------------------------------------------------  
  
【总结】  
有人说，万能五笔无碍乎就是去主页和时间，可这次不同，去主页很简单，而现在是免费版，虽有过期验证代码，但没有接上，因此无时间限制，关键是 selectso.exe 的自校验。  
  
因为调试过程中发现，无论是将 selectso.exe 删除，还是将 004018c3 中的 createprocessa 去掉，wnwb.exe 就一闪而过了。自校验就在 004236E5call 中的 004237FD 的 CMP，因此，完全去除 bug,此版需要三步：  
1. 004018C3 处，改为 6 个 90，去 selectso.exe；  
2. 00401DCF 处，改为 5 个 90，去 265.com 主页；  
3. 00401DFD 处，改为 5 个 90，去 selectso.exe 的自校验 call，或将 00423804 处改为 3 个 90，去除发送退出消息的 call；  
这一节中，我们了解了 KERNEL32.CreateProcessA、KERNEL32.CreateMutexA 和 USER32!POSTQUITMESSAGE 三个和进程有关的函数，它们是此次破解的关键。  
最后，对于控制条中的功能提示和“关于”中的提示，用 reshacker 将资源修改即可。  
  
                                    jackily  
                                    二零零五年二月十六日  
  
本人空间 http://esudy.ys168.com 和 http://jackily.ys168.com 中提供全新绿色安装版，欢迎试用！  
\---------------------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

 4dd466d0e1e0cc123afbe504da2462ca