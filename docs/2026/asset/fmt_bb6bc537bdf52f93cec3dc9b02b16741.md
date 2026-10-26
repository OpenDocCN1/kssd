
# [原创]MSN聊天记录查看器3.1注册分析

## 文章信息

- **文章标题**：MSN聊天记录查看器注册分析
- **文章作者**：bithaha
- **软件名称**：MSN聊天记录查看器
- **下载地址**：`2e3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6Y4L8Y4y4K6M7#2)9J5k6h3y4G2L8b7`.`.
- **加壳方式**：无壳
- **编写语言**：vc++
- **使用工具**：OD
- **软件介绍**：MSN聊天记录查看器3.1 本软件绿色软件，无需安装
- **作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

在华军找了个软柿子。

用`peid`一查没壳，心里好激动。

用`OD`载入，超级字符串搜索，有"注册成功"，直接来到如下代码处。在`00401CE0`处下断，输入注册码，来到下面：

```asm
00401CE0   .  6A FF         PUSH -1
00401CE2   .  68 48754200   PUSH msn聊天?00427548                      ;  SE 处理程序安装
00401CE7   .  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
00401CED   .  50            PUSH EAX
00401CEE   .  64:8925 00000>MOV DWORD PTR FS:[0],ESP
00401CF5   .  83EC 08       SUB ESP,8
00401CF8   .  56            PUSH ESI
00401CF9   .  8BF1          MOV ESI,ECX
00401CFB   .  6A 01         PUSH 1
00401CFD   .  E8 FFAA0100   CALL msn聊天?0041C801
00401D02   .  A1 8C464300   MOV EAX,DWORD PTR DS:[43468C]               ;
00401D07   .  894424 04     MOV DWORD PTR SS:[ESP+4],EAX
00401D0B   .  8D8E 1C010000 LEA ECX,DWORD PTR DS:[ESI+11C]
00401D11   .  C74424 14 000>MOV DWORD PTR SS:[ESP+14],0
00401D19   .  51            PUSH ECX
00401D1A   .  8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]
00401D1E   .  E8 E9C00100   CALL msn聊天?0041DE0C
00401D23   .  6A 08         PUSH 8
00401D25   .  68 28414300   PUSH msn聊天?00434128                      ;  1163659294813585
00401D2A   .  6A 08         PUSH 8
00401D2C   .  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]
00401D30   .  E8 2DC30100   CALL msn聊天?0041E062
00401D35   .  50            PUSH EAX
00401D36   .  E8 D5A00000   CALL msn聊天?0040BE10                       ;比较注册码是否为上面的黑名单
00401D3B   .  83C4 0C       ADD ESP,0C
00401D3E   .  85C0          TEST EAX,EAX
00401D40   .  75 10         JNZ SHORT msn聊天?00401D52
00401D42   .  50            PUSH EAX
00401D43   .  68 8C414300   PUSH msn聊天?0043418C                      ;  ngnsss
00401D48   .  68 68414300   PUSH msn聊天?00434168                      ;  这是个盗版的注册号，请注册正式版本
00401D4D   .  E9 96000000   JMP msn聊天?00401DE8
00401D52   >  6A 08         PUSH 8
00401D54   .  68 54414300   PUSH msn聊天?00434154                      ;  0386848021608060
00401D59   .  6A 08         PUSH 8
00401D5B   .  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]
00401D5F   .  E8 FEC20100   CALL msn聊天?0041E062
00401D64   .  50            PUSH EAX
00401D65   .  E8 A6A00000   CALL msn聊天?0040BE10                       ;又是比较黑名单
00401D6A   .  83C4 0C       ADD ESP,0C
00401D6D   .  85C0          TEST EAX,EAX
00401D6F   .  75 0D         JNZ SHORT msn聊天?00401D7E
00401D71   .  50            PUSH EAX
00401D72   .  68 8C414300   PUSH msn聊天?0043418C                      ;  ngnsss
00401D77   .  68 68414300   PUSH msn聊天?00434168                      ;  这是个盗版的注册号，请注册正式版本
00401D7C   .  EB 6A         JMP SHORT msn聊天?00401DE8
00401D7E   >  6A 08         PUSH 8
00401D80   .  68 14414300   PUSH msn聊天?00434114                      ;  1191759292915277
00401D85   .  6A 08         PUSH 8
00401D87   .  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]
00401D8B   .  E8 D2C20100   CALL msn聊天?0041E062
00401D90   .  50            PUSH EAX
00401D91   .  E8 7AA00000   CALL msn聊天?0040BE10                       ;还是比较黑名单
00401D96   .  83C4 0C       ADD ESP,0C
00401D99   .  85C0          TEST EAX,EAX
00401D9B   .  75 0D         JNZ SHORT msn聊天?00401DAA
00401D9D   .  50            PUSH EAX
00401D9E   .  68 8C414300   PUSH msn聊天?0043418C                      ;  ngnsss
00401DA3   .  68 68414300   PUSH msn聊天?00434168                      ;  这是个盗版的注册号，请注册正式版本
00401DA8   .  EB 3E         JMP SHORT msn聊天?00401DE8
00401DAA   >  51            PUSH ECX
00401DAB   .  8D5424 08     LEA EDX,DWORD PTR SS:[ESP+8]
00401DAF   .  8BCC          MOV ECX,ESP
00401DB1   .  896424 0C     MOV DWORD PTR SS:[ESP+C],ESP
00401DB5   .  52            PUSH EDX                                   ;注册码地址入栈
00401DB6   .  E8 D1BC0100   CALL msn聊天?0041DA8C                      ;
00401DBB   .  E8 B0FDFFFF   CALL msn聊天?00401B70                      ;这是关键比较函数
00401DC0   .  83C4 04       ADD ESP,4
00401DC3   .  85C0          TEST EAX,EAX
00401DC5   .  6A 00         PUSH 0
00401DC7   .  68 8C414300   PUSH msn聊天?0043418C                      ;  ngnsss
00401DCC      74 15         JE SHORT msn聊天?00401DE3                  ;关键跳,但是直接修改这里
                                                                         ;会在弹出注册成功以后又弹出
                                                                         ;注册失败
00401DCE   .  68 48414300   PUSH msn聊天?00434148                      ;  注册成功
00401DD3   .  8BCE          MOV ECX,ESI
00401DD5   .  E8 46A00100   CALL msn聊天?0041BE20
00401DDA   .  8BCE          MOV ECX,ESI
00401DDC   .  E8 19CB0100   CALL msn聊天?0041E8FA
00401DE1   .  EB 0C         JMP SHORT msn聊天?00401DEF
00401DE3   >  68 3C414300   PUSH msn聊天?0043413C                      ; 注册号无效
00401DE8   >  8BCE          MOV ECX,ESI
00401DEA   .  E8 31A00100   CALL msn聊天?0041BE20
00401DEF   >  8D4C24 04     LEA ECX,DWORD PTR SS:[ESP+4]
00401DF3   .  C74424 14 FFF>MOV DWORD PTR SS:[ESP+14],-1
00401DFB   .  E8 17BF0100   CALL msn聊天?0041DD17
00401E00   .  8B4C24 0C     MOV ECX,DWORD PTR SS:[ESP+C]
00401E04   .  5E            POP ESI
00401E05   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX
00401E0C   .  83C4 14       ADD ESP,14
00401E0F   .  C3            RETN
```

跟进`00401DBB`关键比较函数如下：

```asm
00401B70  /$  6A FF         PUSH -1
00401B72  |.  68 28754200   PUSH msn聊天?00427528                      ;  SE 处理程序安装
00401B77  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
00401B7D  |.  50            PUSH EAX
00401B7E  |.  64:8925 00000>MOV DWORD PTR FS:[0],ESP
00401B85  |.  83EC 18       SUB ESP,18
00401B88  |.  53            PUSH EBX
00401B89  |.  6A 08         PUSH 8
00401B8B  |.  33DB          XOR EBX,EBX
00401B8D  |.  68 28414300   PUSH msn聊天?00434128                      ;  1163659294813585
00401B92  |.  6A 08         PUSH 8
00401B94  |.  8D4C24 38     LEA ECX,DWORD PTR SS:[ESP+38]
00401B98  |.  895C24 30     MOV DWORD PTR SS:[ESP+30],EBX
00401B9C  |.  E8 C1C40100   CALL msn聊天?0041E062                      ;和上面那个比较黑名单函数一样
00401BA1  |.  50            PUSH EAX
00401BA2  |.  E8 69A20000   CALL msn聊天?0040BE10
00401BA7  |.  83C4 0C       ADD ESP,0C
00401BAA  |.  85C0          TEST EAX,EAX
00401BAC  |.  0F84 06010000 JE msn聊天?00401CB8
00401BB2  |.  6A 08         PUSH 8
00401BB4  |.  68 14414300   PUSH msn聊天?00434114                      ;  1191759292915277
00401BB9  |.  6A 08         PUSH 8
00401BBB  |.  8D4C24 38     LEA ECX,DWORD PTR SS:[ESP+38]
00401BBF  |.  E8 9EC40100   CALL msn聊天?0041E062
00401BC4  |.  50            PUSH EAX
00401BC5  |.  E8 46A20000   CALL msn聊天?0040BE10                      ;仍然是比较黑名单
00401BCA  |.  83C4 0C       ADD ESP,0C
00401BCD  |.  85C0          TEST EAX,EAX
00401BCF  |.  0F84 E3000000 JE msn聊天?00401CB8
00401BD5  |.  8B4C24 2C     MOV ECX,DWORD PTR SS:[ESP+2C]
00401BD9  |.  33C0          XOR EAX,EAX
00401BDB  |.  894424 05     MOV DWORD PTR SS:[ESP+5],EAX
00401BDF  |.  885C24 04     MOV BYTE PTR SS:[ESP+4],BL
00401BE3  |.  66:894424 09  MOV WORD PTR SS:[ESP+9],AX
00401BE8  |.  884424 0B     MOV BYTE PTR SS:[ESP+B],AL
00401BEC  |.  8B41 F8       MOV EAX,DWORD PTR DS:[ECX-8]
00401BEF  |.  83F8 10       CMP EAX,10                                 ;比较注册码是否为16位(10h)
                                                                         ;没啥好看的,因为黑名单都是16位.作者真是善解人意.
00401BF2  |.  0F8C C0000000 JL msn聊天?00401CB8                        ;
00401BF8  |.  56            PUSH ESI
00401BF9  |.  68 04010000   PUSH 104
00401BFE  |.  8D4C24 34     LEA ECX,DWORD PTR SS:[ESP+34]
```
