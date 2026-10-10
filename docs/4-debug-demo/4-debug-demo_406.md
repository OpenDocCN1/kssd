

# Resource Grabber v2.68c

> 原文：[`www.pediy.com/kssd/pediy03/forum668.htm`](https://www.pediy.com/kssd/pediy03/forum668.htm)

http://www.newhua.com/RESOURCEGRABBER.HTM   
  
试用版有文件数目限制，即每个目录下只能提取一定数目的资源出来。   
反汇编后查找"Demo limit"，找到好多处，每一处对应一种资源格式。如下。把 004E402F 处的 jle 改为 jmp 就 ok 了，总共大概要改 20 处左右。其它的受限制的功能在试用版中没有。   
  
:004E4012 8B154CBD5000            mov edx, dword ptr \[0050BD4C\]   
:004E4018 0302                    add eax, dword ptr \[edx\]   
:004E401A 3B0518D35000            cmp eax, dword ptr \[0050D318\]   
:004E4020 7FDB                    jg 004E3FFD   
:004E4022 A1CCB95000              mov eax, dword ptr \[0050B9CC\]   
:004E4027 8B00                    mov eax, dword ptr \[eax\]   
:004E4029 3B0518D35000            cmp eax, dword ptr \[0050D318\]   
:004E402F 7E39                    jle 004E406A   
:004E4031 A178BC5000              mov eax, dword ptr \[0050BC78\]   
:004E4036 FF00                    inc dword ptr \[eax\]   
  
\* Possible StringData Ref from Code Obj ->"skipped ""   
                                  |   
:004E4038 6868704E00              push 004E7068   
:004E403D 8D55A8                  lea edx, dword ptr \[ebp\-58\]   
:004E4040 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:004E4043 8B08                    mov ecx, dword ptr \[eax\]   
:004E4045 FF5134                  call \[ecx+34\]   
:004E4048 FF75A8                  push \[ebp\-58\]   
  
\* Possible StringData Ref from Code Obj ->"": Demo limit .."   
                                  |   
:004E404B 687C704E00              push 004E707C   
:004E4050 8D45AC                  lea eax, dword ptr \[ebp\-54\]   
:004E4053 BA03000000              mov edx, 00000003   
:004E4058 E8EF00F2FF              call 0040414C   
:004E405D 8B55AC                  mov edx, dword ptr \[ebp\-54\]   
:004E4060 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:004E4063 E864520000              call 004E92CC   
:004E4068 33DB                    xor ebx, ebx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004E402F(U)   
|   
:004E406A B201                    mov dl, 01   
  
\* Possible StringData Ref from Code Obj ->"怓 A"   
                                  |   
:004E406C A190094100              mov eax, dword ptr \[00410990\]   
:004E4071 E8FEEFF1FF              call 00403074   
:004E4076 8BF0                    mov esi, eax   
:004E4078 8BD6                    mov edx, esi   
:004E407A 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:004E407D 8B08                    mov ecx, dword ptr \[eax\]   
:004E407F FF514C                  call \[ecx+4C\]   
:004E4082 84DB                    test bl, bl   
:004E4084 744D                    je 004E40D3   
  
\* Possible StringData Ref from Code Obj ->"writing "   
                                  |   
:004E4086 6898704E00              push 004E7098   
  
  
BF  4dd466d0e1e0cc123afbe504da2462ca