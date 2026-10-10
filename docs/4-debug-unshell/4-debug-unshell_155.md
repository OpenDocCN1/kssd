

# 

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-483.htm`](https://www.pediy.com/kssd/pediy07/pediy7-483.htm)

ACProtect V.1.3x-1.41 的 CodeReplace 都长成这样!  
你在 009D0612 说代码解密后放到 143270 这个内存块.OK!那么你认为你 Dump 出来的程式会有 143270 这个内存块吗?我想不  
太可能,你可以直接 Patch 它.例如  
  
009D0416    PUSHAD  
009D0417    Jmp 009D05C1  
                 \\  
                 \\  
009D05C1    Call 009DF4D2                        ; 取下面要用到的 EBP 的值  
                 \\  
                 \\         
009D060A    CMP  AL,20                           ; 等于 20h 会到另外一处解码  
009D060C    JNZ  009D0621                        ; 改 JMP 让它跳至固定地址来解码  
009D0612    MOV  EDI,DWORD PTR SS:\[EBP+40FC23\]   ; 找适当的地址来解码吧!  
                 \\  
                 \\  
009D0673    PUSHAD  
009D0674    JMP  009D06AE                        ;不让它破坏代码  
                 \\  
                 \\  
009D06AE    POPAD  
009d06AF    POPAD  
009d06BO    RETN  
  
既然要找到一个适当的地方来解码，不如我们就来还原 Code Replace.如何做呢？你应该知道这里有两个 Table.  
  Table1 =返回地址的 RVA   
  Table2 =相对于返回地址的编码，一个地址对应 OAh bytes.  
查看 Table2 的 SIZE，例如 6000，OK.在 DUMP 出来的程式增加一个新的 Section,大小为 6000h,这时你需要 4 个 Buffer.  
  
 Buffer1=用来存放返回地址  
 Buffer2=用来存放解码后的地址  
 Buffer3=用来存放有错误发生时写入 CALL 地址的指针(lpErrorAddr)  
 Buffer4=计数器  
  
原程式可 Patch 成  
  
009D05C1     E8 0CEF0000           CALL WinMain1.009DF4D2  
////////////////////////////////////////////////////////////////////////////                  
009D05CA     33C9                  XOR ECX,ECX  
009D05CC     8B9C8D 69324000       MOV EBX,DWORD PTR SS:\[EBP+ECX\*4+403269\]   
                                   ADD EBX,00400000                   
                                   MOV DWORD PTR DS:\[Buffer1\],EBX  
////////////////////////////////////////////////////////////////////////////  
009D05E4     8DB5 49614000         LEA ESI,DWORD PTR SS:\[EBP+406149\]         
009D05EA     B8 0A000000           MOV EAX,0A                                 
009D05EF     F7E1                  MUL ECX  
009D05F1     03F0                  ADD ESI,EAX                                
009D05F3     8DBD EF1B4000         LEA EDI,DWORD PTR SS:\[EAX+YourNewSectionStartAddr\]  
                                   MOV DWORD PTR DS:\[Buffer2\],EDI  
                                   MOV DWORD PTR DS:\[Buffer4\],ECX  
                                   JMP SHORT WinMain1.009D0621  
/////////////////////////////////////////////////////////////////////////////  
009D0621     8A9D 06244000         MOV BL,BYTE PTR SS:\[EBP+402406\]  
009D0627     B9 0A000000           MOV ECX,0A  
009D062C     AC                    LODS BYTE PTR DS:\[ESI\]  
009D062D     32C3                  XOR AL,BL  
009D062F     AA                    STOS BYTE PTR ES:\[EDI\]  
009D0630   ^ E2 FA                 LOOPD SHORT WinMain1.009D062C  
  
                                   CALL WriteCallAddr  
                                   MOV ECX,DWORD PTR DS:\[Buffer4\]  
                                   INC ECX  
                                   CMP DWORD PTR DS:\[EBP+ECX\*4+403269\],0  
                                   JNZ SHORT WinMain1.009D05CC  
  
WriteCallAddr:  
                                    
                                   PUSHAD  
                                   MOV EBX,DWORD PTR DS:\[Buffer1\]  
                                   SUB EBX,5  
                                   CMP BYTE PTR DS:\[EBX\],0E8  
                                   JNZ WritErrorAddr  
                                   MOV EAX,DWORD PTR DS:\[Buffer2\]  
                                   SUB EAX,5  
                                   SUB EAX,EBX  
                                   ADD EBX,1  
                                   MOV DWORD PTR DS:\[EBX\],EAX  
                                   POPAD  
                                   RETN  
WritErrorAddr                      MOV EDX,DWORD PTR DS:\[Buffer3\]  
                                   MOV DWORD PTR DS:\[EDX\],EBX  
                                   ADD DWORD PTR DS:\[Buffer3\],4  
                                   POPAD  
                                   RETN   
  
如此处理后,你会看到的 CALL 009D0416 变成 CALL XXXXXXXX(你的新地址)记得先用 0D 去载入你的 dumped 程序,下  
da 009D0416 或 ctrl+G 输入 009D0416 再去 Patch,在 009D0416 去重建 EIP,修复后再 dump 出来.  
  
其实 OEP obfuscation,Code Replace 和 API random redirection 并非是 ACProtect 最困难的,你修复好 Code Replace  
后接下来你可能要面对 HookMessageBoxA,和 Embedded Protector,  
HookMessageBoxA 好解决,最麻烦的是  
Embedded Protector(嵌套加密)所谓的 Embedded Protector 就是 ACProtect 的用户能指定某段代码.例如,注册码  
比较的地方,主要功能的地方等等...把它重新编码,你必须通过 ACProtec 的检验,这些检验包括:  
    文件的大小(SIZE)  
    目前的 OEP 是否等于加壳后的 OEP,  
    目前的 OEP 是否等于加壳前的 OEP.(嘻!!告诉我们的原始 OEP 在那里,真是可爱),  
    检查调试器的 Driver 是否存在,  
    检查 PE 部首的数据等...  
  
通过重重的检验它才会给你正确的解码.麻烦的是这不止一处两处,而可能会  
有十几处.  
  
至于你的第二个问题,ACProtect 它会留下 Import Table 的完美指针只清掉函数名而已(令我想起旧版的 ASProtect)丢掉  
的数据,在 IAT 的编码处能找回,若你想脱壳跨平台,修复原始的 Import Table 会为你省却不少麻烦,fly 的教程有,  
拉拉杂杂写了这么多,祝好运!![](img/0cf410da109ac47387dd713fcee2cd79.jpg)

 4dd466d0e1e0cc123afbe504da2462ca