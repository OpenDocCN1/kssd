

# aspr 1.3x 注册笔记

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-822.htm`](https://www.pediy.com/kssd/pediy07/pediy7-822.htm)

aspr 1.3x 注册笔记  
  
  
【目    标】:N/A  
【工    具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F  
【任    务】:aspr 1.3x 注册分析  
【操作平台】:Windows XP sp2  
【作    者】: LOVEBOOM\[DFCG\]\[FCG\]\[CUG\]  
【相关链接】: N/A  
【简要说明】: 以前记下来的关于 aspr 1.3x 注册的笔记，自己今天整理硬盘时无意中发现的:-),总的来说 aspr 的 inline patch 还是比较简单的 。相对来说，现在 ASPR2.x 的脱壳就复杂多了。因此如果只是为了破解的话，相信这篇文章多少会给你带来点作用:-).  
  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第一种情况\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
打开内存异常项，shift+f9 直到出现识别码:  
0012FF40   00000000  
0012FF44   00C69360  ASCII "Qwi0PABwYqk="  
0012FF48   00C6932C  ASCII "A3A8754B-A0E0"  
  
出现后再按 2 次 shift+f9，然后在 code 段处下 F2 断点 shift+f9 再次运行后中断在如下地址:  
005321B0    55              PUSH EBP  
005321B1    8BEC            MOV EBP,ESP  
005321B3    8B45 08         MOV EAX,DWORD PTR SS:\[EBP+8\]    ;这里就是注册名信息  
\[EBP+8\]时有两种情况一种是\[ebp+8\]对应的地址为注册名地址，另一种就是\[\[ebp+8\]\]标志为 1 则表示为注册,0 未注册  
005321B6    85C0            TEST EAX,EAX  
005321B8    74 0C           JE SHORT 005321C6  
005321BA    8038 00         CMP BYTE PTR DS:\[EAX\],0  
005321BD    74 07           JE SHORT 005321C6  
005321BF    C605 686A5700 0>MOV BYTE PTR DS:\[576A68\],1  
005321C6    8B15 5C6A5700   MOV EDX,DWORD PTR DS:\[576A5C\]  
005321CC    8915 586A5700   MOV DWORD PTR DS:\[576A58\],EDX  
005321D2    A3 5C6A5700     MOV DWORD PTR DS:\[576A5C\],EAX  
005321D7    5D              POP EBP  
005321D8    C2 0400         RETN 4  
  
返回后看到:  
  
  
00C3CAB9    50              PUSH EAX                                 ; 这里 push 注册名地址或标记为注册的地址  
00C3CABA    8B47 04         MOV EAX,DWORD PTR DS:\[EDI+4\]  
00C3CABD    FFD0            CALL EAX                                 ; 这里就是 call 检测的代码 EAX==005321B0  
00C3CABF    A1 E097C400     MOV EAX,DWORD PTR DS:\[C497E0\]  
00C3CAC4    8B40 48         MOV EAX,DWORD PTR DS:\[EAX+48\]  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*END\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第二种情况\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
打开内存异常项,shift+f9 直到出现识别码:  
  
0012FF40   00000000  
0012FF44   00B39534  ASCII "Qwi0PABwYqk="  
0012FF48   00B39500  ASCII "A3A8754B-A0E0"  
  
出现后再按 2 次 shift+f9,然后在 code 段处下断，下断后 Shift+f9 运行后中断如下地址:  
  
00424C17    8B4424 04       MOV EAX,DWORD PTR SS:\[ESP+4\]              ; 中断在这里,这里\[esp+4\]就是保存 RegName 的地址  
00424C1B    81EC 08020000   SUB ESP,208  
00424C21    53              PUSH EBX  
00424C22    33DB            XOR EBX,EBX  
00424C24    55              PUSH EBP  
00424C25    881D 28DE4900   MOV BYTE PTR DS:\[49DE28\],BL  
00424C2B    881D 2CDF4900   MOV BYTE PTR DS:\[49DF2C\],BL               ; 这里 RegName save as 地址\[49DF2C\]  
00424C31    881D 30E04900   MOV BYTE PTR DS:\[49E030\],BL  
00424C37    8A08            MOV CL,BYTE PTR DS:\[EAX\]  
00424C39    3ACB            CMP CL,BL  
00424C3B    56              PUSH ESI  
00424C3C    57              PUSH EDI  
  
返回后:  
00AED18A    50              PUSH EAX                                  ; 这里 push 注册名地址  
00AED18B    8B47 04         MOV EAX,DWORD PTR DS:\[EDI+4\]  
00AED18E    FFD0            CALL EAX          ;这里就是 call 424c17  
00AED190    A1 E0A7AF00     MOV EAX,DWORD PTR DS:\[AFA7E0\]  
00AED195    8B40 48         MOV EAX,DWORD PTR DS:\[EAX+48\]  
00AED198    50              PUSH EAX  
  
对于第二种情况的 Inline Patch 可以在返回后直接向下找到类似这样的代码:  
  
00AED5AB    33C9            XOR ECX,ECX  
00AED5AD    BA 54D7AE00     MOV EDX,0AED754                           ; ASCII ".key"  
00AED5B2    B8 00000080     MOV EAX,80000000  
00AED5B7    E8 CCC5FEFF     CALL 00AD9B88  
00AED5BC    68 64D7AE00     PUSH 0AED764                              ; ASCII "regfile"  
00AED5C1    33C9            XOR ECX,ECX  
00AED5C3    BA 54D7AE00     MOV EDX,0AED754                           ; ASCII ".key"  
00AED5C8    B8 00000080     MOV EAX,80000000  
00AED5CD    E8 E6C5FEFF     CALL 00AD9BB8  
......  
再向下不远处就可以 patch 了  
00AED68F    E8 3A000000     CALL 00AED6CE  
00AED694    52              PUSH EDX  
00AED695    81DA 60BB3D58   SBB EDX,583DBB60                          ; 这里可以作为 patch 点  
这里 patch 的话，就这样写:  
MOV \[49DF2C\],'evol'  
  
00AED69B    8B5424 10       MOV EDX,DWORD PTR SS:\[ESP+10\]  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*END\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
                ★★★★★LoVe Wen★★★★★  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my   
  
friends and you!  
  
          By loveboom\[DFCG\]\[FCG\]\[US\]  
                      http://blog.csdn.net/bmd2chen  
                  
  
                                        Email:loveboom#163.com 

 4dd466d0e1e0cc123afbe504da2462ca