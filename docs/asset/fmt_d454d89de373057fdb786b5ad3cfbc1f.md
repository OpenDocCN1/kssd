
这段汇编代码包含多个函数返回和跳转逻辑，主要涉及校验过程的返回路径。以下是对应地址的反汇编指令及注释：

```
0058466C    5B                      pop ebx                        ; ebx=FFFFFFFF
0058466D    8BE5                    mov esp,ebp
0058466F    5D                      pop ebp
00584670    C3                      retn                           ; 第二次校验正常，返回校验结果
```

返回到这里：

```
0044F798    FF93 20010000           call dword ptr ds:[ebx+120]
0044F79E    5B                      pop ebx                        ; 返回到这里，ebx=00B762AC
0044F79F    C3                      retn                           ; 继续返回校验结果
```

返回到这里：

```
0043E134    E8 FB150100             call dumped_.0044F734
0043E139    5B                      pop ebx                        ; 堆栈 [0012FDC4]=00B6D110 (00B6D110)，ebx=00B762AC
0043E13A    C3                      retn                           ; 继续返回校验结果
```

返回到这里：

```
00584CCE    E8 75EDE7FF             call dumped_.00403A48
00584CD3    33C0                    xor eax,eax                    ; 返回到这里进行异或，eax=0012FDA8
00584CD5    5A                      pop edx                        ; edx=00000000
00584CD6    59                      pop ecx                        ; ecx=00000000
00584CD7    59                      pop ecx                        ; ecx=00584CEE
00584CD8    64:8910                 mov dword ptr fs:[eax],edx
00584CDB    68 F54C5800             push dumped_.00584CF5
00584CE0    8D45 EC                 lea eax,dword ptr ss:[ebp-14]
00584CE3    BA 05000000             mov edx,5
00584CE8    E8 73F9E7FF             call dumped_.00404660
00584CED    C3                      retn
00584CEE   ^\E9 EDF2E7FF           jmp dumped_.00403FE0
00584CF3   ^ EB EB                 jmp short dumped_.00584CE0
00584CF5    5F                      pop edi                        ; edi=00470850
00584CF6    5E                      pop esi                        ; esi=0043E118
00584CF7    5B                      pop ebx                        ; ebx=00B6D110
00584CF8    8BE5                    mov esp,ebp
00584CFA    5D                      pop ebp
00584CFB    C3                      retn                           ; 继续返回校验结果
```

返回到这里：

```
0044376F    FF53 38                 call dword ptr ds:[ebx+38]
00443772    5B                      pop ebx                        ; 返回到这里，ebx=00BBC654
00443773    C3                      retn                           ; 继续返回校验结果
```

返回到这里：

```
00443658    33C0                    xor eax,eax                    ; 返回到这里进行异或清零，eax=00000000
0044365A    5A                      pop edx
0044365B    59                      pop ecx
0044365C    59                      pop ecx
0044365D    64:8910                 mov dword ptr fs:[eax],edx
00443660    EB 33                   jmp short dumped_.00443695
00443662   ^ E9 C506FCFF           jmp dumped_.00403D2C
00443667    A1 A48B5900             mov eax,dword ptr ds:[598BA4]
0044366C    8B00                    mov eax,dword ptr ds:[eax]
0044366E    8B55 FC                 mov edx,dword ptr ss:[ebp-4]
00443671    E8 92BC0200             call dumped_.0046F308
00443676    E8 190AFCFF             call dumped_.00404094
0044367B    EB 18                   jmp short dumped_.00443695
0044367D    8B43 08                 mov eax,dword ptr ds:[ebx+8]
00443680    50                      push eax
00443681    8B43 04                 mov eax,dword ptr ds:[ebx+4]
00443684    50                      push eax
00443685    56                      push esi
00443686    8B45 FC                 mov eax,dword ptr ss:[ebp-4]
00443689    8B40 34                 mov eax,dword ptr ds:[eax+34]
0044368C    50                      push eax
0044368D    E8 423EFCFF             call dumped_.004074D4                 ; jmp to user32.DefWindowProcA
00443692    8943 0C                 mov dword ptr ds:[ebx+C],eax
00443695    5F                      pop edi
00443696    5E                      pop esi
00443697    5B                      pop ebx
00443698    59                      pop ecx
00443699    5D                      pop ebp
0044369A    C3                      retn                           ; 继续返回校验结果
```

返回到这里：

```
00426448    FF11                    call dword ptr ds:[ecx]
0042644A    83C4 0C                 add esp,0C                     ; 返回到这里
0042644D    58                      pop eax                        ; eax 清零，eax=00000000
0042644E    5D                      pop ebp
0042644F    C2 1000                 retn 10                        ; 继续返回校验结果
```

返回到这里：

```
77D37AD7    817C24 04 CDABBADC      cmp dword ptr ss:[esp+4],DCBAABCD      ; 返回到这里，堆栈 ss:[0012FE58]=DCBAABCD
77D37ADF    74 11                   je short user32.77D37AF2
77D37AE1    813C24 CDABBADC         cmp dword ptr ss:[esp],DCBAABCD
77D37AE8    75 05                   jnz short user32.77D37AEF
77D37AEA    83EC 04                 sub esp,4
77D37AED    EB 03                   jmp short user32.77D37AF2
77D37AEF    83C4 10                 add esp,10
77D37AF2    83C4 08                 add esp,8
77D37AF5    5B                      pop ebx
77D37AF6    5F                      pop edi
77D37AF7    5E                      pop esi
77D37AF8    5D                      pop ebp
77D37AF9    C2 1400                 retn 14                        ; 继续返回校验结果
```

返回到这里：

```
77D3CCD4    8945 E4                 mov dword ptr ss:[ebp-1C],eax          ; 返回到这里
77D3CCD7   ^ EB B0                  jmp short user32.77D3CC89              ; 向上跳转
```

向上跳转到这里：

```
77D3CC89    834D FC FF              or dword ptr ss:[ebp-4],FFFFFFFF
77D3CC8D    E8 49000000             call user32.77D3CCDB
77D3CC92    8B45 E4                 mov eax,dword ptr ss:[ebp-1C]
77D3CC95    E8 B7070200             call user32.77D5D451
77D3CC9A    C2 2000                 retn 20
```

返回到这里：

```
77D14455    8BC8                    mov ecx,eax                    ; 返回到这里
77D14457    A1 585ED677             mov eax,dword ptr ds:[77D65E58]
77D1445C    F640 02 04              test byte ptr ds:[eax+2],4
77D14460   ^ 75 AF                  jnz short user32.77D14411              ; 向下跳转
```

向下跳转到这里：

```
77D14411    33D2                    xor edx,edx
77D14413    3955 E4                 cmp dword ptr ss:[ebp-1C],edx
77D14416    74 4A                   je short user32.77D14462               ; 向下跳转
```

向下跳转到这里：

```
77D14416   /74 4A                   je short user32.77D14462               ; 向下跳转
77D14418   |64:A1 18000000          mov eax,dword ptr fs:[18]
77D1441E   |3990 40070000           cmp dword ptr ds:[eax+740],edx
77D14424   |74 3C                   je short user32.77D14462
77D14426   |64:A1 18000000          mov eax,dword ptr fs:[18]
```

......(代码太多以次省略一部分)

```
0046F047    E8 C084F9FF             call dumped_.0040750C                  ; jmp to user32.DispatchMessageA
0046F04C    EB 07                   jmp short dumped_.0046F055             ; 最终返回到这里，说名在第 2 次校验时，作者很下了一点功夫滴~~
0046F04E    C686 9C000000 01         mov byte ptr ds:[esi+9C],1
0046F055    8BC3                    mov eax,ebx
0046F057    5A                      pop edx
0046F058    5F                      pop edi
0046F059    5E                      pop esi
0046F05A    5B                      pop ebx
0046F05B    C3                      retn                           ; 为返回程序做最后准备
```

返回到这里：

```
0046F07E    E8 41FFFFFF             call dumped_.0046EFC4
0046F083    84C0                    test al,al                     ; 返回到这里，al=01
0046F085    75 09                   jnz short dumped_.0046F090
0046F087    8BD4                    mov edx,esp
0046F089    8BC3                    mov eax,ebx
0046F08B    E8 98080000             call dumped_.0046F928
0046F090    83C4 1C                 add esp,1C
0046F093    5B                      pop ebx
0046F094    C3                      retn                           ; 为返回程序做最后准备
```

返回到这里：(★)
