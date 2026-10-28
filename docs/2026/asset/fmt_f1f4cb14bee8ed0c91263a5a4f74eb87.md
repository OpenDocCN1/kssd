
```
00403A9B  |.  8B4E 18       mov ecx,dword ptr ds:[esi+0x18]           ; |
00403A9E  |.  8B52 0C       mov edx,dword ptr ds:[edx+0xC]            ; |.text8 的 VirtualAddress
00403AA1  |.  8B41 28       mov eax,dword ptr ds:[ecx+0x28]           ; |eax = 原oep(1788)
00403AA4  |.  8951 28       mov dword ptr ds:[ecx+0x28],edx           ; |修改ope 重新增加区块处执行
00403AA7  |.  8B0F          mov ecx,dword ptr ds:[edi]                ; |
00403AA9  |.  2B41 0C       sub eax,dword ptr ds:[ecx+0xC]            ; |
00403AAC  |.  2D 42010000   sub eax,0x142                             ; |
00403AB1  |.  A3 E6614000   mov dword ptr ds:[0x4061E6],eax           ; |
00403AB6  |.  FF33          push dword ptr ds:[ebx]                   ; |dest
00403AB8  |.  E8 6F0D0000   call <jmp.&MSVCRT.memcpy>                 ; \memcpy
00403ABD  |.  8B03          mov eax,dword ptr ds:[ebx]                ;  拷贝（357字节大小）地址406088作作为新区段
00403ABF  |.  6A 04         push 0x4                                  ; /n = 4
00403AC1  |.  68 F0614000   push Cracker.004061F0                     ; |src = Cracker.004061F0
00403AC6  |.  C680 64010000>mov byte ptr ds:[eax+0x164],0x1           ; |
00403ACD  |.  8B03          mov eax,dword ptr ds:[ebx]                ; |
00403ACF  |.  05 66010000   add eax,0x166                             ; |
00403AD4  |.  50            push eax                                  ; |dest
00403AD5  |.  E8 520D0000   call <jmp.&MSVCRT.memcpy>                 ; \memcpy
00403ADA  |.  8B03          mov eax,dword ptr ds:[ebx]                ;  拷贝CSO
00403ADC  |.  FF75 10       push [arg.3]                              ;  资源dll
00403ADF  |.  C680 65010000>mov byte ptr ds:[eax+0x165],0x78          ;  修改刚拷贝的第357字节为0x78
00403AE6  |.  8B03          mov eax,dword ptr ds:[ebx]
00403AE8  |.  05 6A010000   add eax,0x16A
00403AED  |.  50            push eax
00403AEE  |.  E8 B0DBFFFF   call Cracker.004016A3                     ;  拷贝C:\Program Files\Common Files\System\kd\*\*\*\*\*\*.dla路径
```

修改pe ,然后把修改后的pe 写回`dsound.dll.bat`

```
00403EDD  |.  6A 03         push 0x3
00403EDF  |.  57            push edi
00403EE0  |.  FF75 08       push [arg.1]                              ;  打开文件dsound.dll.bat
00403EE3  |.  FF55 F8       call [local.2]
00403EE6  |.  83F8 FF       cmp eax,-0x1
00403EE9  |.  8945 FC       mov [local.1],eax
00403EEC  |.  75 04         jnz XCracker.00403EF2
00403EEE  |.  6A 03         push 0x3
00403EF0  |.  EB 34         jmp XCracker.00403F26
00403EF2  |>  8BCE          mov ecx,esi
00403EF4  |.  E8 FDF9FFFF   call Cracker.004038F6
```

跟进这个call

修改PE

```
0040390C  |> /FF70 38       /push dword ptr ds:[eax+0x38]             ;  内存对齐1000h
0040390F  |. |8B07          |mov eax,dword ptr ds:[edi]
00403911  |. |8BCE          |mov ecx,esi
00403913  |. |FF70 0C       |push dword ptr ds:[eax+0xC]              ;  区段RVA
00403916  |. |E8 C5FFFFFF   |call Cracker.004038E0
0040391B  |. |8B0F          |mov ecx,dword ptr ds:[edi]
0040391D  |. |8941 0C       |mov dword ptr ds:[ecx+0xC],eax
00403920  |. |8B46 18       |mov eax,dword ptr ds:[esi+0x18]
00403923  |. |8BCE          |mov ecx,esi
00403925  |. |FF70 38       |push dword ptr ds:[eax+0x38]
00403928  |. |8B07          |mov eax,dword ptr ds:[edi]
0040392A  |. |FF70 08       |push dword ptr ds:[eax+0x8]              ;  区段内存大小
0040392D  |. |E8 AEFFFFFF   |call Cracker.004038E0                    ;  对齐函数
00403932  |. |8B0F          |mov ecx,dword ptr ds:[edi]
00403934  |. |8941 08       |mov dword ptr ds:[ecx+0x8],eax
00403937  |. |8B46 18       |mov eax,dword ptr ds:[esi+0x18]
0040393A  |. |8BCE          |mov ecx,esi
0040393C  |. |FF70 3C       |push dword ptr ds:[eax+0x3C]             ;  200h
0040393F  |. |8B07          |mov eax,dword ptr ds:[edi]
00403941  |. |FF70 14       |push dword ptr ds:[eax+0x14]             ;  文件偏移
00403956  |.  8B07          |mov eax,dword ptr ds:[edi]
00403958  |.  FF70 10       |push dword ptr ds:[eax+0x10]             ;  文件大小
0040395B  |.  E8 80FFFFFF   |call Cracker.004038E0                    ;  对齐后大小
00403960  |.  8B0F          |mov ecx,dword ptr ds:[edi]
00403962  |.  43            |inc ebx
00403963  |.  83C7 04       |add edi,0x4
00403966  |.  8941 10       |mov dword ptr ds:[ecx+0x10],eax
00403969  |.  8B46 18       |mov eax,dword ptr ds:[esi+0x18]
0040396C  |.  0FB748 06     |movzx ecx,word ptr ds:[eax+0x6]
00403970  |.  3BD9          |cmp ebx,ecx
00403972  |.^ 7C 98         \jl XCracker.0040390C                     ;  将每个区段大小对齐
00403974  |.  5F            pop edi
00403975  |>  8B449E 18     mov eax,dword ptr ds:[esi+ebx\*4+0x18]     ;  。text8
00403979  |.  8B48 0C       mov ecx,dword ptr ds:[eax+0xC]            ;  VirtualAddress
0040397C  |.  0348 08       add ecx,dword ptr ds:[eax+0x8]            ;  ecx = sizeOfImage
0040397F  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]           ;  pe
00403982  |.  8948 50       mov dword ptr ds:[eax+0x50],ecx           ;  更该 SizeOfImage
00403985  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]
00403988  |.  89A8 C8000000 mov dword ptr ds:[eax+0xC8],ebp
0040398E  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]
00403991  |.  89A8 CC000000 mov dword ptr ds:[eax+0xCC],ebp
00403997  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]
0040399A  |.  89A8 D0000000 mov dword ptr ds:[eax+0xD0],ebp
004039A0  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]
004039A3  |.  89A8 D4000000 mov dword ptr ds:[eax+0xD4],ebp
004039A9  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]
004039AC  |.  89A8 D8000000 mov dword ptr ds:[eax+0xD8],ebp           ;  修改 目录表
```

可以发现这个病毒使用一个结构体存放PE信息

```
[esi+8] = IMAGE\_DOS\_HEADER (0x40)
[esi+10] = A8 //MS\_DOS 大小
[esi+C] = MS\_DOS
[esi+14] = IMAGE\_DOS\_Header大小 = 0x40
[esi+18] = Image\_Nt\_Header
[esi+1C] = 存放Image\_section\_header 转到这个地址可以发现
0012F3B8  18 3C 39 00 48 3C 39 00 78 3C 39 00 A8 3C 39 00  <9.H<9.x<9.?9.
0012F3C8  D8 3C 39 00 08 3D 39 00 38 3D 39 00 68 3D 39 00  ?9.=9.8=9.h=9.
0012F3D8  98 3D 39 00 C8 3D 39 00 F8 3D 39 00 28 3E 39 00  ?9.?9.?9.(>9.
0012F3E8  58 3E 39 00 88 3E 39 00 B8 3E 39 00 E8 3E 39 00  X>9.?9.?9.?9.
```

这个存放了各个区块头

上面这点发现了，，接下来就比较简单了

```
00403E7F  |.  8B3D 00504000 mov edi,dword ptr ds:[<&kernel32.LoadLibr>;  kernel32.LoadLibraryA
00403E85  |.  8BF1          mov esi,ecx
00403E87  |.  BB 10604000   mov ebx,Cracker.00406010                  ;  ASCII "Kernel32.dll"
00403E8C  |.  68 50604000   push Cracker.00406050                     ;  ASCII "CloseHandle"
00403E91  |.  8326 00       and dword ptr ds:[esi],0x0
00403E94  |.  53            push ebx                                  ; /FileName => "Kernel32.dll"
00403E95  |.  FFD7          call edi                                  ; \LoadLibraryA
00403E97  |.  50            push eax
00403E98  |.  E8 83DAFFFF   call Cracker.00401920
00403E9D  |.  59            pop ecx
00403E9E  |.  8945 F4       mov [local.3],eax
00403EA1  |.  59            pop ecx
00403EA2  |.  68 44604000   push Cracker.00406044                     ;  ASCII "CreateFileA"
00403EE0  |.  FF75 08       push [arg.1]                              ;  打开文件dsound.dll.bat
00403EE3  |.  FF55 F8       call [local.2]
00403EE6  |.  83F8 FF       cmp eax,-0x1
00403EE9  |.  8945 FC       mov [local.1],eax
00403EEC  |.  75 04         jnz XCracker.00403EF2
00403EEE  |.  6A 03         push 0x3
00403EF0  |.  EB 34         jmp XCracker.00403F26
00403EF2  |>  8BCE          mov ecx,esi
00403EF4  |.  E8 FDF9FFFF   call Cracker.004038F6
00403EF9  |.  8B46 18       mov eax,dword ptr ds:[esi+0x18]
00403EFC  |.  0FB740 06     movzx eax,word ptr ds:[eax+0x6]           ;  区段 = 5
00403F00  |.  8B4486 18     mov eax,dword ptr ds:[esi+eax\*4+0x18]     ;  .text8段
00403F04  |.  8B48 14       mov ecx,dword ptr ds:[eax+0x14]           ;  ecx = 指向文件偏移
00403F07  |.  8B40 10       mov eax,dword ptr ds:[eax+0x10]           ;  eax = 该区块文件大小
00403F0A  |.  03C1          add eax,ecx                               ;  eax = 文件大小
```
