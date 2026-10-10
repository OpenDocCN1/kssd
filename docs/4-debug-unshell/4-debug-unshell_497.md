

# ASPROTECT V2.x 版本的手工识别

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-685.htm`](https://www.pediy.com/kssd/pediy07/pediy7-685.htm)

ASPROTECT V2.x 版本的手工识别  
  
  
【任    务】:手工识别 aspr v2.x 的那三个版本  
【操作平台】:Windows 2003 server   
【作    者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【简要说明】:写在前面的，这篇文章主要写给初学朋友的，没有别的意思。asprotect v2.x 已经公布的版本有三个按时间顺序分别为 v2.0   
  
06.23、v2.0 01.13 和最近网上出的 v2.1 02.19，  
用 PEID 无法直接区分出来,如果是 v2.1 02.19 的话，PEID 会误报为 ASProtect 1.2x - 1.3x.于是动手写下这篇文章，方便以后自己区分:-)  
  
【详细过程】:  
我分别按时间的来去分析(注明一点，加密选项选择了保护 iat、oep 混乱和检测调试的选项，我想没有几个人加密是不选吧):  
  
Asprotect v2.06.23:  
   这个版本现在已经很容易脱了,OD 设置:忽略除 INT3 之外的全部异常项，隐藏调试器，载入目标后，F9 运行，这样便会中断在 int3 异常处  
0097341F    90              NOP                                      ; 中断在这里  
00973420    EB 01           JMP SHORT 00973423  
00973422    6966 81 FE47467>IMUL ESP,DWORD PTR DS:\[ESI-7F\],744647FE  
......  
009734C4    59              POP ECX  
009734C5    C3              RETN                                     ; 直接在这里下断  
在 009734C5 处下断，几次 int3 异常后，断在 009734c5 处后，取消断点，然后设置忽略 int 3 异常，不忽略内存访问异常。  
按 F9 再次运行触发内存访问异常：  
00976807    C700 7F0677B9   MOV DWORD PTR DS:\[EAX\],B977067F          ; 内存访问异常  
0097680D    FB              STI  
异常发生后,向下看到一个远程条件跳转：  
  00976882    833D 10249800 0>CMP DWORD PTR DS:\[982410\],0  
  00976889    74 14           JE SHORT 0097689F  
  0097688B    6A 04           PUSH 4  
  0097688D    B9 10249800     MOV ECX,982410  
  00976892    8D45 F4         LEA EAX,DWORD PTR SS:\[EBP-C\]  
  00976895    BA 04000000     MOV EDX,4  
  0097689A    E8 31ADFEFF     CALL 009615D0  
  0097689F    B8 02000000     MOV EAX,2  
  009768A4    E8 1BC0FDFF     CALL 009528C4  
  009768A9    85C0            TEST EAX,EAX  
  009768AB    0F85 81010000   JNZ 00976A32                             ; 找到这里  
在按回车跟随到 00976A32 处，然后向下找到第一个 RETN 处就是跳去 Stolen code 的开始处:  
  00976AEB   /74 0B           JE SHORT 00976AF8  
  00976AED   |8B65 F8         MOV ESP,DWORD PTR SS:\[EBP-8\]  
  00976AF0   |FF35 40249800   PUSH DWORD PTR DS:\[982440\]               ; 这里 push 跳去 stolen code 的地址  
  00976AF6   |C3              RETN                                     ; 向下找到这里  
  00976AF7  -|E9 FF75F8FF     JMP 008FE0FB  
......  
  00B30192    55              PUSH EBP                                 ; stolen code 开始处  
  00B30193    C1D5 29         RCL EBP,29                               ; Shift constant out of range 1..31  
在 076AF0 处 F2 断点，然后 Shift+F9,中断后 2 次 f8 就到了 stolen code 处。Asprotect v2.06.23 分析完毕。  
  
  
ASPROTECT V2.0.01.13:  
    这个版本和上一个版本又有所不同,STOLEN CODE 更加难以修复了，我自己是没有多少时间和兴趣做这类苦力活。OD 设置:忽略除 INT3 之  
  
外的全部异常项，隐藏调试器，载入目标后，F9 运行，这样便会中断在 int3 异常处  
0099B497    90              NOP                                      ; 中断在这里  
0099B498    EB 01           JMP SHORT 0099B49B  
0099B49A    6966 81 FE47467>IMUL ESP,DWORD PTR DS:\[ESI-7F\],744647FE  
......  
0099B515    C7              ???                                      ; Unknown command  
0099B516    5B              POP EBX  
0099B517    5A              POP EDX  
0099B518    59              POP ECX  
0099B519    C3              RETN                                     ; 直接在这里下断然后 Shift+f9 运行到这里  
......  
取消断点，然后设置忽略 int 3 异常，不忽略内存访问异常。  
按 F9 再次运行触发内存访问异常，这个版本过了 int3 后，还有两个内存访问异常：  
0099D181    C601 46         MOV BYTE PTR DS:\[ECX\],46                 ; 异常一  
0099D184    67:64:8F06 0000 POP DWORD PTR FS:\[0\]  
0099D18A    83C4 04         ADD ESP,4  
......  
0099E8F9    C601 9E         MOV BYTE PTR DS:\[ECX\],9E                 ; 异常二  
0099E8FC    BB 71FE8A67     MOV EBX,678AFE71  
0099E901    64:8F06         POP DWORD PTR FS:\[ESI\]  
第二次异常后，直接向下找到第一个 RETN：  
0099EA4C    E8 23D0FFFF     CALL 0099BA74                            ; 直接在这里下断，然后 Shift+f9,断下后，取消断点  
0099EA51    83C4 2C         ADD ESP,2C  
0099EA54    5D              POP EBP  
0099EA55    5F              POP EDI  
0099EA56    5E              POP ESI  
0099EA57    5B              POP EBX  
0099EA58    C3              RETN                                     ; 找到这里  
在 0099EA4C 处下断，并运行到那里，中断后，取消断点，F7 一次跟进，进去后，按 CTRL+F9 运行到返回：  
  
0099BA74    8D4451 41       LEA EAX,DWORD PTR DS:\[ECX+EDX\*2+41\]      ; F7 进这里,然后 CTRL+F9 执行到 RETN  
0099BA78    83E8 41         SUB EAX,41  
......  
0099BAFB    8D40 46         LEA EAX,DWORD PTR DS:\[EAX+46\]  
0099BAFE    C3              RETN                                     ; 执行到这里  
  
......  
00B60250    55              PUSH EBP                               ;Stolen code 开始处  
00B60251    81E5 E2C34062   AND EBP,6240C3E2  
00B60257    F2:             PREFIX REPNE:                            ; Superfluous prefix  
  
执行到返回后，再次 F7 一次便到了 Stolen code 的开始处。  
  
ASPROTECT V2.1.02.19:  
  这个版本和 v2.0.01.13 差不多(指识别方法,不是指加密强度),所以前面的操作我就不再重复，最后两个内存异常代码不同而已:  
0099DA0A    0156 00         ADD DWORD PTR DS:\[ESI\],EDX               ; 第一次异常  
0099DA0D    1ACB            SBB CL,BL  
......  
0099F569    0156 00         ADD DWORD PTR DS:\[ESI\],EDX               ; 第二次异常  
0099F56C    EE              OUT DX,AL                                ; I/O command  
0099F56D    B7 D8           MOV BH,0D8  
0099F56F    855F E3         TEST DWORD PTR DS:\[EDI-1D\],EBX  
找 Stolen code 开始处和 01.13 是一样的。  
  
好了，现在简单一点的总结：  
  
asprotect v2.06.23-->最后一次内存异常-->然后向下找 push \[addr\];ret  
asprotect v2.01.13-->最后一次内存异常-->向下找到最 ret 最近的一个 CALL.  
asprotect v2.02.19 和 asprotect v2.01.13 的识别为最后两次异常代码不同.  
  
        END 

 4dd466d0e1e0cc123afbe504da2462ca