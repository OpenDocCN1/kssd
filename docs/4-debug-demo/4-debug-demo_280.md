

# TRW2000 1.11

> 原文：[`www.pediy.com/kssd/pediy01/forum012.htm`](https://www.pediy.com/kssd/pediy01/forum012.htm)

修改 trw2000.sys:   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00029137(C)   
|   
:000292F0 8D7E70                  lea edi, dword ptr \[esi+70\]   
:000292F3 57                      push edi   
:000292F4 E815B3FFFF              call 0002460E   
:000292F9 57                      push edi   
:000292FA E86DB3FFFF              call 0002466C   
:000292FF 803F00                  cmp byte ptr \[edi\], 00   
:00029302 7470                    je 00029374   
:00029304 E895BCFFFF              call 00024F9E   
:00029309 6A06                    push 00000006   
:0002930B 33D2                    xor edx, edx   
:0002930D 59                      pop ecx   
:0002930E F7F1                    div ecx   
:00029310 85D2                    test edx, edx   
:00029312 7505                    jne 00029319   
          ^^^^   
          EB05   
:00029314 E8663EFFFF              call 0001D17F   
:00029319 .......   
  
  
  
  
\* Referenced by a CALL at Address:   
|:00023306    
|   
:000377F6 68C5770300              push 000377C5   
:000377FB 68E4C90500              push 0005C9E4   
:00037800 E833BDFEFF              call 00023538   
:00037805 817C2404D0070000        cmp dword ptr \[esp+04\], 000007D0   
:0003780D 7515                    jne 00037824   
          ^^^^   
          9090   
:0003780F 837C240802              cmp dword ptr \[esp+08\], 00000002   
:00037814 7413                    je 00037829   
          ^^^^    
          EB13   
:00037816 837C240803              cmp dword ptr \[esp+08\], 00000003   
:0003781B 740C                    je 00037829   
          ^^^^   
          EB0C   
:0003781D 837C240804              cmp dword ptr \[esp+08\], 00000004   
:00037822 7405                    je 00037829   
          ^^^^   
          EB05   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0003780D(C)   
|   
:00037824 E865FFFFFF              call 0003778E   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00037814(C), :0003781B(C), :00037822(C)   
|   
:00037829 C20800                  

“TR 系列和 av98 是本人最喜爱的软件。”   
  
去除 trw2k 的注册提示框纯属偶然。本人的机器为 486DX66/8900C/270M,   
  
trw2k 使用时返回 windows 窗口则花屏。直到 1.11 版，突然发现可以凑合   
  
用了--不花屏了。本机器时间不对，trw 显示 trw2000 expired!后退出。   
  
用 W32DASM 打开 TRW2000.SYS 见下面，   
  
  
:0003775A 54                      push esp   
:0003775B 52                      push edx   
:0003775C 57                      push edi   
:0003775D 3230                    xor dh, byte ptr \[eax\]   
:0003775F 3030                    xor byte ptr \[eax\], dh   
:00037761 206578                  and byte ptr \[ebp+78\], ah   
:00037764 7069                    jo 000377CF   
:00037766 7265                    jb 000377CD   
:00037768 642100                  and dword ptr fs:\[eax\], eax   
:0003776B 000000                  BYTE  3 DUP(0)   
  
  
:0003776E 53                      push ebx   
:0003776F 54                      push esp   
:00037770 4F                      dec edi   
:00037771 50                      push eax   
:00037772 006A00                  add byte ptr \[edx+00\], ch   
:00037775 6A64                    push 00000064   
:00037777 E80155FEFF              call 0001CC7D   
  
\* Possible StringData Ref from Code Obj ->"TRW2000 expired!"   
                                  |   
:0003777C 685A770300              push 0003775A   
  
\* Possible StringData Ref from Code Obj ->"STOP"   
                                  |   
:00037781 686E770300              push 0003776E   
:00037786 E88F0DFFFF              call 0002851A   
:0003778B B001                    mov al, 01   
:0003778D C3                      ret   
  
  
该框和注册提示框使用同一方法，即   
  
:00037786 E88F0DFFFF              call 0002851A   
  
注：此处本人并未修改，不知是否可以。   
  
此处的 CALL 将'STOP'为标题，'trw2000 expired!'为内容，显示提示框。   
  
然后查找调用此 CALL 的地方   
  
\* Referenced by a CALL at Address:   
|:00029314    
|   
:0001D17F 60                      pushad   
  
\* Possible StringData Ref from Data Obj ->"6+"   
                                  |   
:0001D180 BBCE3D0400              mov ebx, 00043DCE   
:0001D185 8D732C                  lea esi, dword ptr \[ebx+2C\]   
:0001D188 53                      push ebx   
:0001D189 33DB                    xor ebx, ebx   
:0001D18B B460                    mov ah, 60   
:0001D18D 33D2                    xor edx, edx   
:0001D18F FC                      cld   
:0001D190 BF48D10100              mov edi, 0001D148   
:0001D195 B90A000000              mov ecx, 0000000A   
:0001D19A AC                      lodsb   
:0001D19B 0FB6D0                  movzx edx, al   
:0001D19E 03DA                    add ebx, edx   
:0001D1A0 2AC4                    sub al, ah   
:0001D1A2 FEC4                    inc ah   
:0001D1A4 AA                      stosb   
:0001D1A5 E2F3                    loop 0001D19A   
:0001D1A7 B460                    mov ah, 60   
:0001D1A9 B92C000000              mov ecx, 0000002C   
:0001D1AE AC                      lodsb   
:0001D1AF 0FB6D0                  movzx edx, al   
:0001D1B2 03DA                    add ebx, edx   
:0001D1B4 2AC4                    sub al, ah   
:0001D1B6 FEC4                    inc ah   
:0001D1B8 AA                      stosb   
:0001D1B9 E2F3                    loop 0001D1AE   
:0001D1BB 8BD3                    mov edx, ebx   
:0001D1BD 5B                      pop ebx   
:0001D1BE 3913                    cmp dword ptr \[ebx\], edx   
:0001D1C0 7408                    je 0001D1CA                  ；这里   
:0001D1C2 81C2E9D10100            add edx, 0001D1E9   
:0001D1C8 FFE3                    jmp ebx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0001D1C0(C)   
|   
:0001D1CA 6852D10100              push 0001D152   
:0001D1CF 6848D10100              push 0001D148   
:0001D1D4 E841B30000              call 0002851A   
:0001D1D9 BF48D10100              mov edi, 0001D148   
:0001D1DE B937000000              mov ecx, 00000037   
:0001D1E3 B000                    mov al, 00   
:0001D1E5 F3                      repz   
:0001D1E6 AA                      stosb   
:0001D1E7 61                      popad   
:0001D1E8 C3                      ret   
  
看一下 0001D1C0 上面的几行代码，可能是将某些加密代码解密。在向后找 CALL，   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00029137(C)   
|   
:000292F0 8D7E70                  lea edi, dword ptr \[esi+70\]   
:000292F3 57                      push edi   
:000292F4 E815B3FFFF              call 0002460E   
:000292F9 57                      push edi   
:000292FA E86DB3FFFF              call 0002466C   
:000292FF 803F00                  cmp byte ptr \[edi\], 00   
:00029302 7470                    je 00029374   
:00029304 E895BCFFFF              call 00024F9E   
:00029309 6A06                    push 00000006   
:0002930B 33D2                    xor edx, edx   
:0002930D 59                      pop ecx   
:0002930E F7F1                    div ecx   
:00029310 85D2                    test edx, edx   
:00029312 7505                    jne 00029319   
          ^^^^   
          EB05   
:00029314 E8663EFFFF              call 0001D17F   
:00029319 .......   
  
用 HIEW 修改 00029312 处代码，执行 TRW2K，无注册提示框出现了。   
  
注：其它版本的 TRW2K 亦可。   
  
  
题外话：TRW2K 是很好的软件，虽然还有不完善的地方。对于其作者刘涛涛   
  
本人非常崇拜，此次未征得作者同意，还望他老人家见谅。另外对软件编写   
  
者做一个提醒，对于软件的编写过程中，任何疏忽都会遭成意想不到的后果。   
  
                                              CZS   
  
                                                    2000.3.3