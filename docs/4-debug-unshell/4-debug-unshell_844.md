

# 五笔打字通 5.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6078.htm`](https://www.pediy.com/kssd/pediy06/pediy6078.htm)

对象：五笔打字通 5.0  
作者：lordor\[BCG\]  
声明：属技术交流，无其它目的，请不要任意散布或用作商业用途。初学破解,如有不对的地方欢迎批评指出。  
工具：ollydbg1.09B,插件 ollyDump V2.11.108 

基本操作：F8－单步执行，遇到 call 不进入。F7-单步执行，遇到 call 进入。F4－执行到光标所在行。F2－设断  
手动脱壳要把握两点：  
1、单步往前走，不要回头。  
2、观察。注意 poshad、poshfd,popad、popfd 等，注意地址发生大的变化。 

  
程序用 PECompact V1.40-45 加的壳，没见过的，在这里只好手动脱壳。 

0054DC00 > /EB 06           JMP SHORT wb86.0054DC08  
0054DC02   |68 84370000     PUSH 3784  
0054DC07   |C3              RETN  
0054DC08   \\9C              PUSHFD  
0054DC09    60              PUSHAD  
0054DC0A    E8 02000000     CALL wb86.0054DC11  =>单步走到这里，F8 过的话程序就运行，所以要 F7 跟入 

  
0054DC11    8BC4            MOV EAX,ESP　　　　＝＞F7 后来到这，继续单步运行  
0054DC13    83C0 04         ADD EAX,4  
0054DC16    93              XCHG EAX,EBX  
0054DC17    8BE3            MOV ESP,EBX  
0054DC19    8B5B FC         MOV EBX,DWORD PTR DS:\[EBX\-4\]  
0054DC1C    81EB 0FA04000   SUB EBX,wb86.0040A00F  
0054DC22    87DD            XCHG EBP,EBX  
0054DC24    8B85 A6A04000   MOV EAX,DWORD PTR SS:\[EBP+40A0A6\]  
0054DC2A    0185 03A04000   ADD DWORD PTR SS:\[EBP+40A003\],EAX  
0054DC30    66:C785 00A0400>MOV WORD PTR SS:\[EBP+40A000\],9090  
0054DC39    0185 9EA04000   ADD DWORD PTR SS:\[EBP+40A09E\],EAX  
0054DC3F    BB C3110000     MOV EBX,11C3  
0054DC44    039D AAA04000   ADD EBX,DWORD PTR SS:\[EBP+40A0AA\]  
0054DC4A    039D A6A04000   ADD EBX,DWORD PTR SS:\[EBP+40A0A6\]  
0054DC50    53              PUSH EBX  
0054DC51    53              PUSH EBX 

  
...............(一直往前走，省略)..................... 

0054F25E    57              PUSH EDI  
0054F25F    AD              LODS DWORD PTR DS:\[ESI\]  
0054F260    0BC0            OR EAX,EAX  
0054F262    74 6C           JE SHORT wb86.0054F2D0  
0054F264    8BD0            MOV EDX,EAX  
0054F266    0395 A6A04000   ADD EDX,DWORD PTR SS:\[EBP+40A0A6\]  
0054F26C    AD              LODS DWORD PTR DS:\[ESI\]  
0054F26D    56              PUSH ESI  
0054F26E    8BC8            MOV ECX,EAX  
0054F270    57              PUSH EDI  
0054F271    52              PUSH EDX  
0054F272    8BF2            MOV ESI,EDX  
0054F274    8B85 15A64000   MOV EAX,DWORD PTR SS:\[EBP+40A615\]  
0054F27A    8B9D 19A64000   MOV EBX,DWORD PTR SS:\[EBP+40A619\]  
0054F280    E8 910A0000     CALL wb86.0054FD16  
0054F285    5A              POP EDX  
0054F286    5F              POP EDI  
0054F287    52              PUSH EDX  
0054F288    57              PUSH EDI  
0054F289    FF95 9EA04000   CALL DWORD PTR SS:\[EBP+40A09E\]  
0054F28F    0BC0            OR EAX,EAX  
0054F291    74 07           JE SHORT wb86.0054F29A  
0054F293    8BC8            MOV ECX,EAX  
0054F295    5E              POP ESI  
0054F296    5F              POP EDI  
0054F297  ^ EB C5           JMP SHORT wb86.0054F25E　　＝＝＞走到这里会跳到前面，把光标移动到下一 

行，F4 跳过时程序会直接运行，所以还得单步运行，走到上面的 0054F262 处会跳到后面去了  
0054F299    B9 8D9D97A5     MOV ECX,A5979D8D  
0054F29E    40              INC EAX  
0054F29F    0053 FF         ADD BYTE PTR DS:\[EBX\-1\],DL  
0054F2A2    95              XCHG EAX,EBP  
0054F2A3    15 A640008D     ADC EAX,8D0040A6  
0054F2A8    9D              POPFD 

...............(一直往前走，省略)..................... 

0054F2CF    24 58           AND AL,58　　＝＝＞从上面跳到这，继续单步走  
0054F2D1    8DB5 C3A64000   LEA ESI,DWORD PTR SS:\[EBP+40A6C3\]  
0054F2D7    AD              LODS DWORD PTR DS:\[ESI\]  
0054F2D8    0BC0            OR EAX,EAX  
0054F2DA    74 74           JE SHORT wb86.0054F350  
0054F2DC    0385 A6A04000   ADD EAX,DWORD PTR SS:\[EBP+40A0A6\] 

...............(一直往前走，省略)..................... 

0054F36E   /74 72           JE SHORT wb86.0054F3E2　　　  
0054F36D    49              DEC ECX  
0054F36E    74 72           JE SHORT wb86.0054F3E2  
0054F370    78 70           JS SHORT wb86.0054F3E2  
0054F372    66:8B07         MOV AX,WORD PTR DS:\[EDI\]  
0054F375    2C E8           SUB AL,0E8  
0054F377    3C 01           CMP AL,1  
0054F379    76 38           JBE SHORT wb86.0054F3B3  
0054F37B    66:3D 1725      CMP AX,2517  
0054F37F    74 51           JE SHORT wb86.0054F3D2  
0054F381    3C 27           CMP AL,27  
0054F383    75 0A           JNZ SHORT wb86.0054F38F  
0054F385    80FC 80         CMP AH,80  
0054F388    72 05           JB SHORT wb86.0054F38F  
0054F38A    80FC 8F         CMP AH,8F  
0054F38D    76 05           JBE SHORT wb86.0054F394  
0054F38F    47              INC EDI  
0054F390    43              INC EBX  
0054F391  ^ EB DA           JMP SHORT wb86.0054F36D　＝＝＞这里又跳到前面，看一下前面那一句会跳到 

后面的，是 JE SHORT 0054F3E2，JS SHORT 0054F3E2，JBE SHORT wb86.0054F3B3，JE SHORT 0054F3D2，依次 

在其跳往的地方设断。F9 运行，会在设断的地方停，最后确定 0054F3E2 才是正确的设断地方  
0054F393    B8 8B47023C     MOV EAX,3C02478B 

  
...............(一直往前走，省略)..................... 

0054F476    8BB5 15A64000    MOV ESI,DWORD PTR SS:\[EBP+40A615\]  
0054F47C    8BBD 19A64000    MOV EDI,DWORD PTR SS:\[EBP+40A619\]  
0054F482    E8 8F0C0000      CALL wb86.00550116  
0054F487    61               POPAD   ==＞看到希望了，继续单步走  
0054F488    9D               POPFD    
0054F489    50               PUSH EAX  
0054F48A    68 84374000      PUSH wb86.00403784  
0054F48F    C2 0400          RETN 4　＝＝＞走过这里，地址会有很大变化，可以确定，壳已脱完了。  
0054F492    8BB5 37A64000    MOV ESI,DWORD PTR SS:\[EBP+40A637\] 

  
00403781      00             DB 00  
00403782   >  0000           ADD BYTE PTR DS:\[EAX\],AL  
00403784   .  68 94FF4300    PUSH wb86.0043FF94　　＝＝＝＞由 0054F48F 处跳来，在这里运行 ollyDump 把 

程序 dump 下来。到此手动脱壳结束。  
00403789      E8             DB E8  
0040378A      EE             DB EE  
0040378B      FF             DB FF  
0040378C      FF             DB FF  
0040378D      FF             DB FF  
0040378E      00             DB 00  
0040378F      00             DB 00  
00403790      00             DB 00  
00403791      00             DB 00  
00403792      00             DB 00 

  
脱完后可以用侦壳工具看，是用 VB 写的。其它壳（如 Aspack 等）都可以用此法配合 OLLYDUMP 来手动脱壳 

\---------------------------- 

  
看雪这里高手如云，发表此文，谨在于希望知识在共享中增值，如有不对的地方，请大家指正！ 

  
cracked by lordor\[BCG\]  
03.5.30

  
 4dd466d0e1e0cc123afbe504da2462ca