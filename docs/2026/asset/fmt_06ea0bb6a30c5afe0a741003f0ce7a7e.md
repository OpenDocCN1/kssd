
# [原创]东方系统备份专家 脱壳，注册算法分析

[易语言] 东方系统备份专家 脱壳，注册算法分析

![](http://bbs.pediy.com/upload/2006/4/image/6.jpg_173.jpg)

```
00423633     68 A4529500          push 9552A4
00423638     E8 50BDFFFF          call 脱壳1111.0041F38D
0042363D     8945 F4              mov dword ptr ss:[ebp-C],eax
00423640     837D F4 01           cmp dword ptr ss:[ebp-C],1
00423644     0F85 07000000        jnz 脱壳1111.00423651
0042364A     B8 01000000          mov eax,1
0042364F     EB 02                jmp short 脱壳1111.00423653
00423651     33C0                 xor eax,eax
00423653     85C0                 test eax,eax
00423655     0F84 15010000        je <脱壳1111.文本型（基本数据类型）>
00423638     E8 50BDFFFF          call 脱壳1111.0041F38D
```

核心比较过程

跟入：

```
0041F38D     55                   push ebp
0041F38E     8BEC                 mov ebp,esp
0041F390     81EC 38000000        sub esp,38
0041F396     68 18000000          push 18
0041F39B <>  E8 296B0000          call 脱壳1111.00425EC9                               ; 从堆上分配内存空间（调用6号服务）
0041F3A0     83C4 04              add esp,4                                          ; 结构
0041F3A3     8945 FC              mov dword ptr ss:[ebp-4],eax
0041F3A6     8BF8                 mov edi,eax
0041F3A8 <>  BE 305E4000          mov esi,脱壳1111.00405E30                            ; （常量）
0041F3AD     AD                   lods dword ptr ds:[esi]
0041F3AE     AB                   stos dword ptr es:[edi]
0041F3AF     AD                   lods dword ptr ds:[esi]
0041F3B0     AB                   stos dword ptr es:[edi]
0041F3B1     33C0                 xor eax,eax
0041F3B3     B9 04000000          mov ecx,4
0041F3B8     F3:AB                rep stos dword ptr es:[edi]
0041F3BA     E8 AD020000          call 脱壳1111.0041F66C                               ; 得到变换后的机器吗
```

跟入。

```
0041F66C     55                   push ebp
0041F66D     8BEC                 mov ebp,esp
0041F66F     81EC 10000000        sub esp,10
0041F675     8B1D 78529500        mov ebx,dword ptr ds:[955278]
0041F67B     E8 97C6FFFF          call 脱壳1111.0041BD17
0041F680     B8 00000000          mov eax,0
0041F685     3BC1                 cmp eax,ecx
0041F687     7C 0D                jl short 脱壳1111.0041F696
0041F689     68 01000000          push 1
0041F68E <>  E8 42680000          call 脱壳1111.00425ED5                               ; 调用Runtime内部异常处理（调用0号服务）
0041F693     83C4 04              add esp,4
0041F696     C1E0 02              shl eax,2
0041F699     03D8                 add ebx,eax
0041F69B     895D FC              mov dword ptr ss:[ebp-4],ebx
0041F69E <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F6A3     6A 00                push 0
0041F6A5     68 55C22B15          push 152BC255
0041F6AA <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F6AF     6A 00                push 0
0041F6B1     FF35 7C529500        push dword ptr ds:[95527C]
0041F6B7     68 02000000          push 2
0041F6BC <>  BB CC000000          mov ebx,0CC                                        ; 位异或（系统核心支持库）
0041F6C1 <>  E8 09680000          call 脱壳1111.00425ECF                               ; 调用核心支持库命令（调用3号服务）
0041F6C6     83C4 1C              add esp,1C
0041F6C9 <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F6CE     6A 00                push 0
0041F6D0     50                   push eax
0041F6D1     68 01000000          push 1
0041F6D6 <>  BB D4010000          mov ebx,1D4                                        ; 取十六进制文本（系统核心支持库）
0041F6DB <>  E8 EF670000          call 脱壳1111.00425ECF                               ; 调用核心支持库命令（调用3号服务）
0041F6E0     83C4 10              add esp,10
0041F6E3     8945 F4              mov dword ptr ss:[ebp-C],eax
0041F6E6 <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F6EB     6A 00                push 0
0041F6ED     68 04000000          push 4
0041F6F2 <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F6F7     6A 00                push 0
0041F6F9     68 02000000          push 2
0041F6FE <>  68 04000080          push 80000004                                      ; 文本型（基本数据类型）
0041F703     6A 00                push 0
0041F705     8B45 F4              mov eax,dword ptr ss:[ebp-C]
0041F708     85C0                 test eax,eax
0041F70A     75 05                jnz short 脱壳1111.0041F711
0041F70C <>  B8 09564000          mov eax,脱壳1111.00405609                            ; （常量）
0041F711     50                   push eax
0041F712     68 03000000          push 3
0041F717 <>  BB 3C010000          mov ebx,13C                                        ; 取文本中间（系统核心支持库）
0041F71C <>  E8 AE670000          call 脱壳1111.00425ECF                               ; 调用核心支持库命令（调用3号服务）
0041F721     83C4 28              add esp,28
0041F724     8945 F0              mov dword ptr ss:[ebp-10],eax
0041F727     8B5D F4              mov ebx,dword ptr ss:[ebp-C]
0041F72A     85DB                 test ebx,ebx
0041F72C     74 09                je short 脱壳1111.0041F737
0041F72E     53                   push ebx
0041F72F <>  E8 8F670000          call 脱壳1111.00425EC3                               ; 销毁从堆上分配到的内存（调用8号服务）
0041F734     83C4 04              add esp,4
0041F737     8B45 F0              mov eax,dword ptr ss:[ebp-10]
0041F73A     50                   push eax
0041F73B     8B5D FC              mov ebx,dword ptr ss:[ebp-4]
0041F73E     8B1B                 mov ebx,dword ptr ds:[ebx]
0041F740     85DB                 test ebx,ebx
0041F742     74 09                je short 脱壳1111.0041F74D
0041F744     53                   push ebx
0041F745 <>  E8 79670000          call 脱壳1111.00425EC3                               ; 销毁从堆上分配到的内存（调用8号服务）
0041F74A     83C4 04              add esp,4
0041F74D     58                   pop eax
0041F74E     8B5D FC              mov ebx,dword ptr ss:[ebp-4]
0041F751     8903                 mov dword ptr ds:[ebx],eax
0041F753     8B1D 78529500        mov ebx,dword ptr ds:[955278]
0041F759     E8 B9C5FFFF          call 脱壳1111.0041BD17
0041F75E     B8 01000000          mov eax,1
0041F763     3BC1                 cmp eax,ecx
0041F765     7C 0D                jl short 脱壳1111.0041F774
0041F767     68 01000000          push 1
0041F76C <>  E8 64670000          call 脱壳1111.00425ED5                               ; 调用Runtime内部异常处理（调用0号服务）
0041F771     83C4 04              add esp,4
0041F774     C1E0 02              shl eax,2
0041F777     03D8                 add ebx,eax
0041F779     895D FC              mov dword ptr ss:[ebp-4],ebx
0041F77C <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F781     6A 00                push 0
0041F783     68 FDF9BA3A          push 3ABAF9FD
0041F788 <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F78D     6A 00                push 0
0041F78F     FF35 7C529500        push dword ptr ds:[95527C]
0041F795     68 02000000          push 2
0041F79A <>  BB CC000000          mov ebx,0CC                                        ; 位异或（系统核心支持库）
0041F79F <>  E8 2B670000          call 脱壳1111.00425ECF                               ; 调用核心支持库命令（调用3号服务）
0041F7A4     83C4 1C              add esp,1C
0041F7A7 <>  68 01030080          push 80000301                                      ; 整数型（基本数据类型）
0041F7AC     6A 00                push 0
0041F7AE     50                   push eax
0041F7AF     68 01000000          push 1
0041F7B4 <>  BB D4010000          mov ebx,1D4                                        ; 取十六进制文本（系统核心支持库）
0041F7B9 <>  E8 11670000          call 脱壳1111.00425ECF                               ; 调用核心支持库命令（调用3号服务）
0041F7BE     83C4 10              add esp,10
0041F7C1     8945 F4              mov dword ptr ss:[ebp-C],eax
```
