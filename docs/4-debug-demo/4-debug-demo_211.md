

# NAG 去除

> 原文：[`www.pediy.com/kssd/pediy02/forum243.htm`](https://www.pediy.com/kssd/pediy02/forum243.htm)

 本人在使用 winRar２.71 时，过期，弹出 NaG.   
用 Exescope 打开 winrar.exe,找到 resource-->dialog-->REMINDER,发现   
是 NaG 窗口的内容，用 W32dasm8.93 打开 winrar.exe,查找 REMINDER，   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004011B2(C)   
|   
:00401506 833DC0CC460000          cmp dword ptr \[0046CCC0\], 00000000   
:0040150D 7546                    jne 00401555   
:0040150F 803D7430460000          cmp byte ptr \[00463074\], 00   
:00401516 753D                    jne 00401555   
:00401518 803DCC6C460000          cmp byte ptr \[00466CCC\], 00   
:0040151F 7534                    jne 00401555   
:00401521 A1A8F54600              mov eax, dword ptr \[0046F5A8\]   
:00401526 83F828                  cmp eax, 00000028   
:00401529 90                      nop   
:0040152A 90                      nop   
；抱歉，这里已修改了，但不难想象是一跳转。   
:0040152B 85C0                    test eax, eax   
:0040152D 7D26                    jge 00401555   
:0040152F C6057430460001          mov byte ptr \[00463074\], 01   
:00401536 6A00                    push 00000000   
:00401538 68A8BA4000              push 0040BAA8   
:0040153D 8B1524B94600            mov edx, dword ptr \[0046B924\]   
:00401543 52                      push edx   
  
\* Possible StringData Ref from Data Obj ->"REMINDER"   
                                  |   
:00401544 68E73B4600              push 00463BE7   
:00401549 8B0D20CC4600            mov ecx, dword ptr \[0046CC20\]   
:0040154F 51                      push ecx   
  
\* Reference To: USER32.DialogBoxParamA, Ord:0000h   
                                  |   
:00401550 E878090600              Call 00461ECD   
  
很快，它的 NaG 就被 Kill 掉了。   
  
  
　　　　　　　　　　　　　　CZS  2000.8.16   4dd466d0e1e0cc123afbe504da2462ca