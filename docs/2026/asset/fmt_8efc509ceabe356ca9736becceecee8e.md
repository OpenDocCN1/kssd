
```assembly
00401626  |.  8850 08       MOV BYTE PTR DS:[EAX+8],DL               ; |
00401629  |.  6A 00         PUSH 0                                   ; |Address = NULL
0040162B  |.  892D 28DC4000 MOV DWORD PTR DS:[40DC28],EBP            ; |
00401631  |.  C746 3C 00020>MOV DWORD PTR DS:[ESI+3C],200            ; |
00401638  |.  FF15 28A04000 CALL DWORD PTR DS:[<&KERNEL32.VirtualAll>; \VirtualAlloc
0040163E  |.  8B1D F8DB4000 MOV EBX,DWORD PTR DS:[40DBF8]            ;  申请了文件大小的空间
00401644  |.  8B4B 28       MOV ECX,DWORD PTR DS:[EBX+28]            ;  OEP
00401647  |.  8BE8          MOV EBP,EAX                              ;  新空间的指针 010C0000
00401649  |.  A1 24DC4000   MOV EAX,DWORD PTR DS:[40DC24]            ;  连接好的PE文件 010B0000
0040164E  |.  8D7C01 09     LEA EDI,DWORD PTR DS:[ECX+EAX+9]
00401652  |.  B9 0A000000   MOV ECX,0A
00401657  |.  BE 64B34000   MOV ESI,Fengyue'.0040B364
0040165C  |.  F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>;  cOPY 了一个壳的标识
0040165E  |.  66:A5         MOVS WORD PTR ES:[EDI],WORD PTR DS:[ESI]
00401660  |.  8B53 54       MOV EDX,DWORD PTR DS:[EBX+54]
00401663  |.  52            PUSH EDX
00401664  |.  50            PUSH EAX
00401665  |.  55            PUSH EBP
00401666  |.  892D 00DC4000 MOV DWORD PTR DS:[40DC00],EBP
0040166C  |.  E8 6F510000   CALL Fengyue'.004067E0                   ;  先Copy了PE头 Fengyue'sDll-Game v0.22 by 海風月影[CUG]
00401671  |.  8B73 54       MOV ESI,DWORD PTR DS:[EBX+54]
00401674  |.  83C4 0C       ADD ESP,0C
00401677  |.  66:837B 06 00 CMP WORD PTR DS:[EBX+6],0
0040167C  |.  8935 28DC4000 MOV DWORD PTR DS:[40DC28],ESI
00401682  |.  C74424 10 000>MOV DWORD PTR SS:[ESP+10],0
0040168A  |.  76 40         JBE SHORT Fengyue'.004016CC
0040168C  |.  8B3D 18DC4000 MOV EDI,DWORD PTR DS:[40DC18]
00401692  |.  83C7 10       ADD EDI,10
00401695  |>  8B4F FC       /MOV ECX,DWORD PTR DS:[EDI-4]
00401698  |.  8B07          |MOV EAX,DWORD PTR DS:[EDI]
0040169A  |.  030D 24DC4000 |ADD ECX,DWORD PTR DS:[40DC24]
004016A0  |.  50            |PUSH EAX
004016A1  |.  51            |PUSH ECX
004016A2  |.  8D142E        |LEA EDX,DWORD PTR DS:[ESI+EBP]
004016A5  |.  52            |PUSH EDX
004016A6  |.  E8 35510000   |CALL Fengyue'.004067E0
004016AB  |.  8B4424 1C     |MOV EAX,DWORD PTR SS:[ESP+1C]
004016AF  |.  0FB74B 06     |MOVZX ECX,WORD PTR DS:[EBX+6]
004016B3  |.  0337          |ADD ESI,DWORD PTR DS:[EDI]
004016B5  |.  83C0 01       |ADD EAX,1
004016B8  |.  83C4 0C       |ADD ESP,0C
004016BB  |.  83C7 28       |ADD EDI,28
004016BE  |.  3BC1          |CMP EAX,ECX
004016C0  |.  894424 10     |MOV DWORD PTR SS:[ESP+10],EAX
004016C4  |.^ 7C CF         \JL SHORT Fengyue'.00401695              ;  这里就是Copy 5个内存区段的数据格式 保存成 磁盘文件格式
004016C6  |.  8935 28DC4000 MOV DWORD PTR DS:[40DC28],ESI
004016CC  |>  5F            POP EDI
004016CD  |.  5E            POP ESI
004016CE  |.  5D            POP EBP
004016CF  |.  5B            POP EBX
004016D0  |.  83C4 08       ADD ESP,8
004016D3  \.  C3            RETN
```

<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

高手飘过，如对处理PE的流程感兴趣，不要取看文章，随便加载一个PE文件，然后查看PE头各数据的信息，然去直接动态跟踪该流程。百看不如一练，写东西比直接调试麻烦的多~~

分析的有点乱，有出错之处请以自己的调试为准。先读取硬盘的磁盘文件，再根据PE头各区段`VirtualAddress`、`VirtualAddress`和`VirtualSize`来将磁盘文件映射到内存，处理映射到内存的PE文件，包括重定位处理、添加区段等操作。处理结束后再根据PE头区段的`PointerToRawData`、`VirtualSize`和`SizeOfRawData`来存储为磁盘文件格式并保存。

通过动态调试把壳处理PE文件：处理PE头（`OEP`等）、节表、添加区段、增加壳代码并重定位输入表、将PE文件映射内存处理后并保存为磁盘文件的过程有了一个更清晰的认识。文中有些E文写的可能不是很对，大家动态跟踪的时候自己修正一下吧。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
