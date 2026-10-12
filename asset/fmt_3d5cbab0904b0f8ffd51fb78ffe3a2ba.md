
```
0047F34C  push    ebp
0047F34D  mov     ebp, esp
0047F34F  add     esp, -23C
0047F355  push    ebx
0047F356  mov     ebx, 0047F494
0047F35B  mov     eax, dword ptr [481468]
0047F360  cmp     dword ptr [eax], 2              ;  判断操作系统
0047F363  jnz     short 0047F386
0047F365  push    0                               ; /hTemplateFile = NULL
0047F367  push    0                               ; |Attributes = 0
0047F369  push    3                               ; |Mode = OPEN_EXISTING
0047F36B  push    0                               ; |pSecurity = NULL
0047F36D  push    3                               ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0047F36F  push    C0000000                        ; |Access = GENERIC_READ|GENERIC_WRITE
0047F374  push    0047F498                        ; |FileName = "\\\\.\\PhysicalDrive0"
0047F379  call    <jmp.&kernel32.CreateFileA>     ; \\CreateFileA
0047F37E  mov     dword ptr [ebp-214], eax        ;  连接第 1 个物理硬盘
0047F384  jmp     short 0047F3A2
0047F386  push    0                               ; /hTemplateFile = NULL
0047F388  push    0                               ; |Attributes = 0
0047F38A  push    1                               ; |Mode = CREATE_NEW
0047F38C  push    0                               ; |pSecurity = NULL
0047F38E  push    0                               ; |ShareMode = 0
0047F390  push    0                               ; |Access = 0
0047F392  push    0047F4AC                        ; |FileName = "\\\\.\\SMARTVSD"
0047F397  call    <jmp.&kernel32.CreateFileA>     ; \\CreateFileA
0047F39C  mov     dword ptr [ebp-214], eax
0047F3A2  cmp     dword ptr [ebp-214], -1
0047F3A9  je      0047F48B
0047F3AF  xor     eax, eax
0047F3B1  push    ebp
0047F3B2  push    0047F463
0047F3B7  push    dword ptr fs:[eax]
0047F3BA  mov     dword ptr fs:[eax], esp
0047F3BD  lea     eax, dword ptr [ebp-239]
0047F3C3  xor     ecx, ecx
0047F3C5  mov     edx, 20                         ;  Clear InBuffer=20
0047F3CA  call    <FillChar>
0047F3CF  lea     eax, dword ptr [ebp-210]
0047F3D5  xor     ecx, ecx
0047F3D7  mov     edx, 210
0047F3DC  call    <FillChar>                      ;  Clear OutBuffer=210
0047F3E1  xor     eax, eax
0047F3E3  mov     dword ptr [ebp-218], eax
0047F3E9  mov     dword ptr [ebp-239], 200
0047F3F3  mov     byte ptr [ebp-234], 1
0047F3FA  mov     byte ptr [ebp-233], 1
0047F401  mov     byte ptr [ebp-230], 0A0
0047F408  mov     byte ptr [ebp-22F], 0EC
0047F40F  push    0                               ; /pOverlapped = NULL
0047F411  lea     eax, dword ptr [ebp-218]        ; |
0047F417  push    eax                              ; |pBytesReturned
0047F418  push    210                              ; |OutBufferSize = 210 (528.)
0047F41D  lea     eax, dword ptr [ebp-210]        ; |
0047F423  push    eax                              ; |OutBuffer
0047F424  push    20                               ; |InBufferSize = 20 (32.)
0047F426  lea     eax, dword ptr [ebp-239]        ; |
0047F42C  push    eax                              ; |InBuffer
0047F42D  push    7C088                            ; |IoControlCode = SMART_RCV_DRIVE_DATA
0047F432  mov     eax, dword ptr [ebp-214]        ; |
0047F438  push    eax                              ; |hDevice
0047F439  call    <jmp.&kernel32.DeviceIoContro>; \\DeviceIoControl
0047F43E  test    eax, eax                        ;  读取硬盘驱动数据
0047F440  jnz     short 0047F449
0047F442  call    0040399C
0047F447  jmp     short 0047F48B
0047F449  xor     eax, eax
0047F44B  pop     edx
0047F44C  pop     ecx
0047F44D  pop     ecx
0047F44E  mov     dword ptr fs:[eax], edx
0047F451  push    0047F46A
0047F456  mov     eax, dword ptr [ebp-214]
0047F45C  push    eax                              ; /hObject
0047F45D  call    <jmp.&kernel32.CloseHandle>     ; \\CloseHandle
```

获取硬盘数据后会先用`FixFileName`修复一下字符串，再`Trim`去除空格，然后进行第 3 个`CALL`进行处理。

第三个`call 0047F4BC`跟进后只是简单的对硬盘序列号累加，结果记为`sum`，代码也不贴了。

第四个`call 0047F684`作用是用`sum`对一幅图片进行解密：

代码:

```
0047F684  push    ebp
0047F685  mov     ebp, esp
0047F687  add     esp, -10
0047F68A  push    ebx
0047F68B  push    esi
0047F68C  push    edi
0047F68D  mov     ebx, edx
0047F68F  mov     dword ptr [ebp-4], eax
0047F692  xor     eax, eax
0047F694  push    ebp
0047F695  push    0047F6CE
0047F69A  push    dword ptr fs:[eax]
0047F69D  mov     dword ptr fs:[eax], esp
0047F6A0  mov     edx, 1
0047F6A5  mov     eax, dword ptr [ebp-4]
0047F6A8  call    <GetScanline>                   ;  获取图片第 1 线数据
0047F6AD  push    eax
0047F6AE  xor     edx, edx
0047F6B0  mov     eax, dword ptr [ebp-4]
0047F6B3  call    <GetScanline>                   ;  获取图片第 0 线数据
0047F6B8  pop     edx
0047F6B9  xchg    eax, edx
0047F6BA  sub     eax, edx
0047F6BC  cdq
0047F6BD  xor     eax, edx
0047F6BF  sub     eax, edx
0047F6C1  mov     dword ptr [ebp-8], eax          ;  相减得到每一线数据长度
0047F6C4  xor     eax, eax
0047F6C6  pop     edx
0047F6C7  pop     ecx
0047F6C8  pop     ecx
0047F6C9  mov     dword ptr fs:[eax], edx
0047F6CC  jmp     short 0047F6EE
……异常处理
0047F6EE  mov     eax, dword ptr [48140C]
0047F6F3  mov     dword ptr [eax], ebx            ;  保存累加结果 sum
0047F6F5  mov     eax, dword ptr [ebp-4]
0047F6F8  mov     edx, dword ptr [eax]
0047F6FA  call    dword ptr [edx+20]              ;  图片高度
0047F6FD  dec     eax
0047F6FE  test    eax, eax
0047F700  jl      short 0047F73E
0047F702  inc     eax
0047F703  mov     dword ptr [ebp-10], eax
0047F706  mov     dword ptr [ebp-C], 0            ;  i=0
0047F70D  mov     edx, dword ptr [ebp-C]          ;  ----循环开始
0047F710  mov     eax, dword ptr [ebp-4]
0047F713  call    <GetScanline>                   ;  获取图片每一线数据
0047F718  mov     edi, eax
0047F71A  mov     ebx, dword ptr [ebp-8]          ;  取每一线数据长度
0047F71D  dec     ebx
0047F71E  test    ebx, ebx
0047F720  jl      short 0047F736
0047F722  inc     ebx
0047F723  xor     esi, esi
0047F725  mov     eax, 100
0047F72A  call    00402B84                        ;  sum=sum*8088405+1,al=sum 高 8 位
0047F72F  xor     byte ptr [edi+esi], al          ;  对数据进行 XOR 解密
0047F732  inc     esi
0047F733  dec     ebx
0047F734  jnz     short 0047F725                  ;  对每一线的每一字节解密
0047F736  inc     dword ptr [ebp-C]               ;  i=i+1
0047F739  dec     dword ptr [ebp-10]              ;  取下一线
0047F73C  jnz     short 0047F70D                  ;  循环结束----
0047F73E  pop     edi
0047F73F  pop     esi
0047F740  pop     ebx
0047F741  mov     esp, ebp
0047F743  pop     ebp
0047F744  retn
```

可以看出对图片数据的每一字节都进行了`XOR`加密了。

最后一个`call 0047F758`就是比较图片内容了：

代码:
