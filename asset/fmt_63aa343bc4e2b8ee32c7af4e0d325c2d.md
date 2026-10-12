
```
001B:004C868F   832000               AND      DWORD PTR [EAX],00  
001B:004C8692   6844724E00          PUSH     004E7244  
001B:004C8697   FF1520044E00        CALL     [KERNEL32!LoadLibraryA]  
001B:004C869D   8B3588034E00        MOV      ESI,[KERNEL32!FormatMessageA]  
001B:004C86A3   8945F4             MOV      [EBP-0C],EAX  
001B:004C86A6   85C0               TEST     EAX,EAX  
001B:004C86A8   BF00080000         MOV      EDI,00000800  
001B:004C86AD   7418               JZ       004C86C7  
001B:004C86AF   6A00               PUSH     00  
001B:004C86B1   8D4DFC             LEA      ECX,[EBP-04]  
001B:004C86B4   6A00               PUSH     00  
001B:004C86B6   51                 PUSH     ECX  
001B:004C86B7   57                 PUSH     EDI  
001B:004C86B8   FF7308             PUSH     DWORD PTR [EBX+08]  
001B:004C86BB   50                 PUSH     EAX  
001B:004C86BC   6800090000         PUSH     00000900  
001B:004C86C1   FFD6               CALL     ESI   //[KERNEL32!FormatMessageA]  
001B:004C86C3   85C0               TEST     EAX,EAX  
001B:004C86C5   7522               JNZ      004C86E9  
001B:004C86C7   33C0               XOR      EAX,EAX  
001B:004C86C9   8D4DFC             LEA      ECX,[EBP-04]  
001B:004C86CC   50                 PUSH     EAX  
001B:004C86CD   50                 PUSH     EAX  
001B:004C86CE   51                 PUSH     ECX  
001B:004C86CF   57                 PUSH     EDI  
001B:004C86D0   FF7308             PUSH     DWORD PTR [EBX+08]  
001B:004C86D3   50                 PUSH     EAX  
001B:004C86D4   6800110000         PUSH     00001100  
001B:004C86D9   FFD6               CALL     ESI   //[KERNEL32!FormatMessageA]  
001B:004C86DB   8BF0               MOV      ESI,EAX  
001B:004C86DD   85F6               TEST     ESI,ESI  
001B:004C86DF   7508               JNZ      004C86E9  
001B:004C86E1   8B4508             MOV      EAX,[EBP+08]  
001B:004C86E4   802000             AND      BYTE PTR [EAX],00  
001B:004C86E7   EB75               JMP      004C875E  
001B:004C86E9   817B08E32E0000     CMP      DWORD PTR [EBX+08],00002EE3  
001B:004C86F0   8B1D8C034E00        MOV      EBX,[KERNEL32!LocalFree]  
001B:004C86F6   754F               JNZ      004C8747  
001B:004C86F8   8B3534084E00        MOV      ESI,[WININET!InternetGetLastResponseInfoA  
001B:004C86FE   83651000           AND      DWORD PTR [EBP+10],00  
001B:004C8702   8D4510             LEA      EAX,[EBP+10]  
001B:004C8705   50                 PUSH     EAX  
001B:004C8706   8D45F8             LEA      EAX,[EBP-08]  
001B:004C8709   6A00               PUSH     00  
001B:004C870B   50                 PUSH     EAX  
001B:004C870C   FFD6               CALL     ESI  
001B:004C870E   85C0               TEST     EAX,EAX  
001B:004C8710   7544               JNZ      004C8756  
001B:004C8712   FF1558034E00        CALL     [ntdll!RtlGetLastWin32Error]  
001B:004C8718   83F87A             CMP      EAX,7A  
001B:004C871B   7539               JNZ      004C8756  
001B:004C871D   FF7510             PUSH     DWORD PTR [EBP+10]  
001B:004C8720   6A40               PUSH     40  
001B:004C8722   FF1550034E00        CALL     [KERNEL32!LocalAlloc]  
001B:004C8728   8BF8               MOV      EDI,EAX  
001B:004C872A   8D4510             LEA      EAX,[EBP+10]  
001B:004C872D   50                 PUSH     EAX  
001B:004C872E   8D45F8             LEA      EAX,[EBP-08]  
001B:004C8731   57                 PUSH     EDI  
001B:004C8732   50                 PUSH     EAX  
001B:004C8733   FFD6               CALL     ESI  
001B:004C8735   FF750C             PUSH     DWORD PTR [EBP+0C]  
001B:004C8738   57                 PUSH     EDI  
001B:004C8739   FF7508             PUSH     DWORD PTR [EBP+08]  
001B:004C873C   FF1584034E00        CALL     [KERNEL32!lstrcpyn]  
001B:004C8742   57                 PUSH     EDI  
001B:004C8743   FFD3               CALL     EBX  
001B:004C8745   EB0F               JMP      004C8756  
001B:004C8747   FF750C             PUSH     DWORD PTR [EBP+0C]  
001B:004C874A   FF75FC             PUSH     DWORD PTR [EBP-04]  
001B:004C874D   FF7508             PUSH     DWORD PTR [EBP+08]  
001B:004C8750   FF1584034E00        CALL     [KERNEL32!lstrcpyn]  
001B:004C8756   6A01               PUSH     01  
001B:004C8758   5E                 POP      ESI  
001B:004C8759   FF75FC             PUSH     DWORD PTR [EBP-04]  
001B:004C875C   FFD3               CALL     EBX  
001B:004C875E   FF75F4             PUSH     DWORD PTR [EBP-0C]  
001B:004C8761   FF15D4034E00        CALL     [KERNEL32!FreeLibrary]  
001B:004C8767   8BC6               MOV      EAX,ESI  
001B:004C8769   5F                 POP      EDI  
001B:004C876A   5E                 POP      ESI  
001B:004C876B   5B                 POP      EBX  
001B:004C876C   C9                 LEAVE  
001B:004C876D   C20C00             RET      000C  
```

-------------------------------------------------

=============================================================

### 五、从消息框“试用客户每次只能发送 10 条信息”入手

跟踪破解功能限制；

`G` `MessageBoxA`

```
001B:004BA309  FF1510054E00        CALL     [USER32!MessageBoxA]  

............  

001B:004BA33B  C20C00             RET      000C  //ret to 004BA360  
001B:004BA3AB  E88EFFFFFF         CALL     004BA33E  

............  

001B:004BA33E  ......（多处调用）  
001B:004BA35A  FF928C000000       CALL     [EDX+0000008C]  //调用 004BA257 MessageBoxA（见下面 ★★★★）显示消息框  
001B:004BA373  C3   RET          //RET TO 00418E05 见下面  
```

对消息框的调用来源于 `00418B50` 的过程，用修正了 `JAVASCRIPT` 版本启动，全部选择“化工类”192 条信息，再在 `00418B50` 设断，不登录，按发送按钮，选择试用按钮，立即中断，分析发现该过程存在发送条数的判断，然后循环发送。

-----------------------------------------

```
001B:00418B50  6AFF                PUSH     FF  
001B:00418B52  68C6564D00         PUSH     004D56C6  
001B:00418B57  64A100000000       MOV      EAX,FS:[00000000]  
001B:00418B5D  50                 PUSH     EAX  
001B:00418B5E  64892500000000     MOV      FS:[00000000],ESP  
001B:00418B65  81EC4C010000       SUB      ESP,0000014C  
001B:00418B6B  53                 PUSH     EBX  
001B:00418B6C  56                 PUSH     ESI  
001B:00418B6D  8BF1               MOV      ESI,ECX  
001B:00418B6F  33DB               XOR      EBX,EBX  
001B:00418B71  891D28A05100       MOV      [0051A028],EBX  
001B:00418B77  891D746B5100       MOV      [00516B74],EBX  
001B:00418B7D  8B4E04             MOV      ECX,[ESI+04]  
001B:00418B80  89742408           MOV      [ESP+08],ESI  
001B:00418B84  39591C             CMP      [ECX+1C],EBX  
001B:00418B87  7505               JNZ      00418B8E  
001B:00418B89  E86201FFFF         CALL     00408CF0  
001B:00418B8E  8B4E04             MOV      ECX,[ESI+04]  
001B:00418B91  E8EA0FFFFF         CALL     00409B80  //取选择条目数量,eax=000000C0，即十进制 192  
001B:00418B96  3BC3               CMP      EAX,EBX  
001B:00418B98  89442410           MOV      [ESP+10],EAX  
001B:00418B9C  0F8E8A020000       JLE      00418E2C  
001B:00418BA2  55                 PUSH     EBP  
001B:00418BA3  57                 PUSH     EDI  
001B:00418BA4  53                 PUSH     EBX  
001B:00418BA5  8D8C249C000000     LEA      ECX,[ESP+0000009C]  
001B:00418BAC  E8DFA40000         CALL     00423090  
001B:00418BB1  53                 PUSH     EBX  
001B:00418BB2  8D8C249C000000     LEA      ECX,[ESP+0000009C]  
001B:00418BB9  899C2468010000     MOV      [ESP+00000168],EBX  
001B:00418BC0  E86BA60000         CALL     00423230  //显示发布进度条  
001B:00418BC5  53                 PUSH     EBX  
001B:00418BC6  8D8C249C000000     LEA      ECX,[ESP+0000009C]  
001B:00418BCD  E80EA70000         CALL     004232E0  
001B:00418BD2  8B3D58074E00       MOV      EDI,[USER32!PeekMessageA]  
```
