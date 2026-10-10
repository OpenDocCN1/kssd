

# ASProtect 1.3stolen code

> 原文：[`www.pediy.com/kssd/pediy06/pediy6459.htm`](https://www.pediy.com/kssd/pediy06/pediy6459.htm)

也谈 ASProtect 1.3 之 stolen code 的修复    
  
    ASProtect 1.23RC4 因为 stolen code 没有变形，可以不用分析得到，1.3 开始变形了。blowfish 把程序 email 给 me 后，对于其 stolen code,偶不是去猜测，而是采用到壳中去找的办法，现把分析报告如下：    
1、设置 OllyDBG 异常除“内存访问异常”外全部钩上。记下入口 esp 值，以备后用。    
2、25 次异常后按原来的方法在 ret 处按 F2 设断，断下后然后根据入口 esp 在命令行设置 tc ebp\>=12ffc0 && ebp<12ffc4,停下后，即到变形的 stolen code 处。    
F7 一步一步跟，并分析见每行尾部。    
0B22424B  PUSH -1                        //此处 3 行代码没变，照抄     
0B22424D  PUSH 54E948    
0B224252  PUSH 542B94    
0B224257  MOV EAX,DWORD PTR FS:\[0\]    
0B22425D  JMP SHORT 0B224261    
  
0B22426E  PUSH 34    
0B224270  JMP SHORT 0B224275                      ; 多余的前缀    
  
0B224275  PUSH DWORD PTR FS:\[0\]    
0B22427C  MOV DWORD PTR SS:\[ESP+4\],EAX  //\[esp+4\]=eax 相当于把前面 push 34 变为 push eax     
  
0B22428C  LEA ESP,DWORD PTR SS:\[ESP+4\]  //add esp,4 相当于去掉 PUSH DWORD PTR FS:\[0\]    
0B224290  MOV DWORD PTR FS:\[0\],ESP    //设置 SEH,前面几行可以简化为 push fs:\[0\],mov fs\[0\],esp     
0B224297  SUB ESP,58                  //不变，照抄     
0B22429A  JMP SHORT 0B22429E    
  
0B2242AB  PUSH 34    
0B2242AD  JMP SHORT 0B2242B2                      ; 多余的前缀    
  
0B2242B2  PUSH DWORD PTR FS:\[0\]    
0B2242B9  MOV DWORD PTR SS:\[ESP+4\],EBX  //\[esp+4\]=ebx 相当于前面 push 34 变为 push ebx     
  
0B2242C9  LEA ESP,DWORD PTR SS:\[ESP+4\]  //add esp,4 相当于去掉 PUSH DWORD PTR FS:\[0\]，下同     
0B2242CD  JMP SHORT 0B2242D1    
  
0B2242DE  PUSH 34    
0B2242E0  JMP SHORT 0B2242E5                      ; 多余的前缀    
  
0B2242E5  PUSH DWORD PTR FS:\[0\]    
0B2242EC  MOV DWORD PTR SS:\[ESP+4\],ESI  // 相当于把前面 push 34 变为 push esi     
  
0B2242FC  LEA ESP,DWORD PTR SS:\[ESP+4\]    
0B224300  JMP SHORT 0B224304    
  
0B224311  PUSH 34    
0B224313  JMP SHORT 0B224318                      ; 多余的前缀    
  
0B224318  PUSH DWORD PTR FS:\[0\]    
0B22431F  MOV DWORD PTR SS:\[ESP+4\],EDI  // 相当于把前面 push 34 变为 push edi     
  
0B22432F  LEA ESP,DWORD PTR SS:\[ESP+4\]    
0B224333  MOV DWORD PTR SS:\[EBP\-18\],ESP    
0B224336  JMP SHORT 0B22433A                      ; 多余的前缀    
  
0B22433E  PUSH 53F07B  //不变，照抄    
0B224343  PUSH 0B224131    
0B224348  RETN    
  
0B224131  JMP SHORT 0B224134    
以下打扫战场：    
0B224134  PUSH ECX    
0B224135  PUSH EDI    
0B224136  PUSHFD    
0B224137  CLD    
0B224138  MOV EDI,0B224175    
0B22413D  MOV ECX,145E    
0B224142  REP STOS BYTE PTR ES:\[EDI\]    
0B224144  POPFD    
0B224145  POP EDI    
0B224146  POP ECX    
0B224147  RETN     
回到 fake OEP:    
0053F07B  CALL DWORD PTR DS:\[54D1DC\]    
0053F081  XOR EDX,EDX    
0053F083  MOV DL,AH    
0053F085  MOV DWORD PTR DS:\[88BAF10\],EDX    
0053F08B  MOV ECX,EAX    
0053F08D  AND ECX,0FF    
0053F093  MOV DWORD PTR DS:\[88BAF0C\],ECX    
0053F099  SHL ECX,8    
0053F09C  ADD ECX,EDX    
0053F09E  MOV DWORD PTR DS:\[88BAF08\],ECX    
0053F0A4  SHR EAX,10    
  
3.通过以上跟踪分析，得出正确的 stolen code 如下：(而且长度正好一致)    
0053F055 >PUSH EBP    
0053F056  MOV EBP,ESP    
0053F058  PUSH -1    
0053F05A  PUSH 0054E948    
0053F05F  PUSH 00542B94    
0053F064  PUSH DWORD PTR FS:\[0\]    
0053F06A  MOV DWORD PTR FS:\[0\],ESP    
0053F070  SUB ESP,58    
0053F073  PUSH EBX    
0053F074  PUSH ESI    
0053F075  PUSH EDI    
0053F076  PUSH 0053F07B    
0053F07B  CALL DWORD PTR DS:\[54D1DC\]  
  
 4dd466d0e1e0cc123afbe504da2462ca