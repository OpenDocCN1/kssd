

# 闹钟&笑话 1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1191.htm`](https://www.pediy.com/kssd/pediy03/forum1191.htm)

闹钟&笑话 1.0 破解过程：   
CRACKTOOLS：trw2000 1.23、W32DASM 中文版、language 2000 V4.5   
1、习惯性动作：用 language 2000 V4.5 查文件是否加壳，发现 ASPACK2.1 加壳。UNASPACK 脱之。   
2、习惯性动作：用 W32DASM 反汇编一下，看是否有线索（即注册失败与成功的提示字符串）   
找到如下：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:004822E6(C), :004822F4(C)   
|   
:00482310 8B45FC                  mov eax, dword ptr \[ebp\-04\]    //将假注册码输到 EAX   
:00482313 8B55F4                  mov edx, dword ptr \[ebp\-0C\]    //将真注册码输到 EDX   
:00482316 E8D91BF8FF              call 00403EF4                  //真假注册码对比   
:0048231B 7528                    jne 00482345                  //不相同则跳到失败地方   
:0048231D 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00482320 8BC3                    mov eax, ebx   
:00482322 E829010000              call 00482450   
:00482327 84C0                    test al, al   
:00482329 741A                    je 00482345   
:0048232B C683F402000000          mov byte ptr \[ebx+000002F4\], 00   
  
\* Possible StringData Ref from Code Obj ->"您已注册完成，谢谢您使用闹钟和笑话 1.0!"   
                                  |   
:00482332 B8B8234800              mov eax, 004823B8   
:00482337 E898D3FFFF              call 0047F6D4   
:0048233C 8BC3                    mov eax, ebx   
:0048233E E8D9B0FCFF              call 0044D41C   
:00482343 EB0A                    jmp 0048234F   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:0048231B(C), :00482329(C)   
|   
  
\* Possible StringData Ref from Code Obj ->"注册码错误!"   
                                  |   
:00482345 B8E8234800              mov eax, 004823E8   
:0048234A E885D3FFFF              call 0047F6D4   
  
激活 TRW2000，下 BPX 00482316  F5 返回。   
  
运行闹钟&笑话 1.0 ，填假注册码 78787878。点注册，被拦下。   
  
F8 进这个:00482316 call 00403EF4，如下：   
  
:00403EF4 53                      push ebx   
:00403EF5 56                      push esi   
:00403EF6 57                      push edi   
:00403EF7 89C6                    mov esi, eax   
:00403EF9 89D7                    mov edi, edx   
:00403EFB 39D0                    cmp eax, edx  //呵呵~~   
:00403EFD 0F848F000000            je 00403F92   
:00403F03 85F6                    test esi, esi   
:00403F05 7468                    je 00403F6F   
  
  
做 CRACKCODE 注册机：   
\[Options\]   
CommandLine=joke.exe   
Mode=2   
First\_Break\_Address=482316   
First\_Break\_Address\_Code=E8   
First\_Break\_Address\_Code\_Lenth=5   
Second\_Break\_Address=403EFB   
Second\_Break\_Address\_Code\_Lenth=2   
Save\_Code\_Address=EDX   
  
这个软件的注册有点象美萍。  4dd466d0e1e0cc123afbe504da2462ca