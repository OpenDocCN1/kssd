
```asm
  006D1399   .  64:8920        mov     dword ptr fs:[eax], esp  
  006D139C   .  A1 34EB7700    mov     eax, dword ptr [77EB34]  
  006D13A1   .  8B00           mov     eax, dword ptr [eax]  
  006D13A3   .  05 6C030000    add     eax, 36C  
  006D13A8   .  8B55 F8        mov     edx, dword ptr [ebp-8]  
  006D13AB   .  E8 C82AD3FF    call    00403E78                 ;  获取固定字符串  
  006D13B0   .  8D45 DC        lea     eax, dword ptr [ebp-24]  
  006D13B3   .  B9 8C156D00    mov     ecx, 006D158C            ;  spadger  
  006D13B8   .  8B55 F8        mov     edx, dword ptr [ebp-8]  
  006D13BB   .  E8 402DD3FF    call    00404100                 ;  用户名+固定字符串  
  006D13C0   .  8B55 DC        mov     edx, dword ptr [ebp-24]  
  006D13C3   .  8D45 E0        lea     eax, dword ptr [ebp-20]  
  006D13C6   .  E8 3932D3FF    call    00404604                 ;  验证是否组合成功(算法 CALL)  
  006D13CB   .  8B45 E0        mov     eax, dword ptr [ebp-20]   ;  UNICODE "KuNgBiMspadger"  
  006D13CE   .  8D55 E4        lea     edx, dword ptr [ebp-1C]  
  006D13D1   .  E8 2E56FFFF    call    006C6A04                 ;  算法结果 A=LCase(MD5(用户名+固定字符串))  
  006D13D6   .  8B45 E4        mov     eax, dword ptr [ebp-1C]   ;  UNICODE "a3e0585a9dabdd33d619dd0c19744feb"  
  006D13D9   .  8D55 E8        lea     edx, dword ptr [ebp-18]  
  006D13DC   .  E8 2356FFFF    call    006C6A04                 ;  算法结果 B=LCase(MD5(算法结果 A))  
  006D13E1   .  8B55 E8        mov     edx, dword ptr [ebp-18]   ;  ASCII "c74b4952dec5b0c61a77a2754beacd87"  
  006D13E4   .  8D45 F0        lea     eax, dword ptr [ebp-10]  
  006D13E7   .  E8 902CD3FF    call    0040407C                 ;  从算法结果 B 中开始重新组合注册码  
  006D13EC   .  8D45 D8        lea     eax, dword ptr [ebp-28]  
  006D13EF   .  50             push    eax  
  006D13F0   .  B9 05000000    mov     ecx, 5                   ;  取 5 位  
  006D13F5   .  33D2           xor     edx, edx                 ;  从首位取  
  006D13F7   .  8B45 F0        mov     eax, dword ptr [ebp-10]   ;  ASCII "c74b4952dec5b0c61a77a2754beacd87"  
  006D13FA   .  E8 BD2ED3FF    call    004042BC                 ;  取位作为★注册码 A 段★  
  006D13FF   .  FF75 D8        push    dword ptr [ebp-28]       ;  ASCII "c74b4"  
  006D1402   .  68 9C156D00    push    006D159C                 ;  取“-”并连接  
  006D1407   .  8D45 D4        lea     eax, dword ptr [ebp-2C]  
  006D140A   .  50             push    eax  
  006D140B   .  B9 05000000    mov     ecx, 5                   ;  取 5 位  
  006D1410   .  BA 05000000    mov     edx, 5                   ;  从第 5 位取  
  006D1415   .  8B45 F0        mov     eax, dword ptr [ebp-10]   ;  ASCII "c74b4952dec5b0c61a77a2754beacd87"  
  006D1418   .  E8 9F2ED3FF    call    004042BC                 ;  取位作为★注册码 B 段★  
  006D141D   .  FF75 D4        push    dword ptr [ebp-2C]       ;  ASCII "4952d"  
  006D1420   .  68 9C156D00    push    006D159C                 ;  取“-”并连接  
  006D1425   .  8D45 D0        lea     eax, dword ptr [ebp-30]  
  006D1428   .  50             push    eax  
  006D1429   .  B9 05000000    mov     ecx, 5                   ;  取 5 位  
  006D142E   .  BA 0A000000    mov     edx, 0A                  ;  从第 10 位取  
  006D1433   .  8B45 F0        mov     eax, dword ptr [ebp-10]   ;  ASCII "c74b4952dec5b0c61a77a2754beacd87"  
  006D1436   .  E8 812ED3FF    call    004042BC                 ;  取位作为★注册码 C 段★  
  006D143B   .  FF75 D0        push    dword ptr [ebp-30]       ;  ASCII "ec5b0"  
  006D143E   .  68 9C156D00    push    006D159C                 ;  取“-”并连接  
  006D1443   .  8D45 CC        lea     eax, dword ptr [ebp-34]  
  006D1446   .  50             push    eax  
  006D1447   .  B9 05000000    mov     ecx, 5                   ;  取 5 位  
  006D144C   .  BA 0F000000    mov     edx, 0F                  ;  从第 15 位取  
  006D1451   .  8B45 F0        mov     eax, dword ptr [ebp-10]   ;  ASCII "c74b4952dec5b0c61a77a2754beacd87"  
  006D1454   .  E8 632ED3FF    call    004042BC                 ;  取位作为★注册码 D 段★  
  006D1459   .  FF75 CC        push    dword ptr [ebp-34]       ;  ASCII "c61a7"  
  006D145C   .  8D45 EC        lea     eax, dword ptr [ebp-14]  
  006D145F   .  BA 07000000    mov     edx, 7  
  006D1464   .  E8 0B2DD3FF    call    00404174  
  006D1469   .  8D45 C4        lea     eax, dword ptr [ebp-3C]  
  006D146C   .  8B55 F4        mov     edx, dword ptr [ebp-C]  
  006D146F   .  E8 9031D3FF    call    00404604  
  006D1474   .  8B45 C4        mov     eax, dword ptr [ebp-3C]  
  006D1477   .  8D55 C8        lea     edx, dword ptr [ebp-38]  
  006D147A   .  E8 8555FFFF    call    006C6A04  
  006D147F   .  8B45 C8        mov     eax, dword ptr [ebp-38]  
  006D1482   .  50             push    eax  
  006D1483   .  8D45 BC        lea     eax, dword ptr [ebp-44]  
  006D1486   .  8B55 EC        mov     edx, dword ptr [ebp-14]  
  006D1489   .  E8 7631D3FF    call    00404604                 ;  重新组合后的注册码(真码)  
  006D148E   .  8B45 BC        mov     eax, dword ptr [ebp-44]   ;  UNICODE "c74b4-4952d-ec5b0-c61a7"  
  006D1491   .  8D55 C0        lea     edx, dword ptr [ebp-40]  
  006D1494   .  E8 6B55FFFF    call    006C6A04                 ;  LCase(MD5(真码))  
  006D1499   .  8B55 C0        mov     edx, dword ptr [ebp-40]   ;  UNICODE "b8eb02c9081b4b60a410bb9ebf41e1f2"  
  006D149C   .  58             pop     eax                     ;  UNICODE "c220732ec07b22455c4761dcc7f924f9"  
  006D149D   .  E8 3632D3FF    call    004046D8                 ;  验证不等则挂  
  006D14A2   .  75 3F          jnz     short 006D14E3           ;  跳则 GAME OVER  
  006D14A4   .  68 A8156D00    push    006D15A8                 ;  update info_shop set s_usetimes=88,s_regname="  
  006D14A9   .  FF75 F8        push    dword ptr [ebp-8]  
  006D14AC   .  68 E0156D00    push    006D15E0                 ;  ",s_regno="  
  006D14B1   .  FF75 F4        push    dword ptr [ebp-C]  
  006D14B4   .  68 F4156D00    push    006D15F4                 ;  "  
  006D14B9   .  8D45 B8        lea     eax, dword ptr [ebp-48]  
  006D14BC   .  BA 05000000    mov     edx, 5  
  006D14C1   .  E8 AE2CD3FF    call    00404174  
  006D14C6   .  8B55 B8        mov     edx, dword ptr [ebp-48]  
  006D14C9   .  A1 34EB7700    mov     eax, dword ptr [77EB34]  
  006D14CE   .  8B00           mov     eax, dword ptr [eax]  
  006D14D0   .  E8 23660800    call    00757AF8  
  006D14D5   .  8B45 FC        mov     eax, dword ptr [ebp-4]  
  006D14D8   .  C680 F4020000 >mov     byte ptr [eax+2F4], 1  
  006D14DF   .  B3 01          mov     bl, 1  
  006D14E1   .  EB 0C          jmp     short 006D14EF  
  006D14E3   >  8B45 FC        mov     eax, dword ptr [ebp-4]  
  006D14E6   .  C680 F4020000 >mov     byte ptr [eax+2F4], 0  
  006D14ED   .  33DB          xor     ebx, ebx  
  006D14EF   >  33C0          xor     eax, eax  
  006D14F1   .  5A             pop     edx  
  006D14F2   .  59             pop     ecx  
  006D14F3   .  59             pop     ecx  
  006D14F4   .  64:8910        mov     dword ptr fs:[eax], edx  
  006D14F7   .  EB 2F          jmp     short 006D1528  
  006D14F9   .^ E9 D220D3FF    jmp     004035D0  
  006D14FE   .  E8 8DBDD3FF    call    0040D290  
  006D1503   .  8B50 04        mov     edx, dword ptr [eax+4]  
  006D1506   .  A1 34EB7700    mov     eax, dword ptr [77EB34]  
  006D150B   .  8B00           mov     eax, dword ptr [eax]  
  006D150D   .  B9 00166D00    mov     ecx, 006D1600            ;  请输入用户名和密码，点击登录  
  006D1512   .  E8 89630800    call    007578A0  
```
