

# 一网络验证脱机外挂的破解分析

> 原文：[`www.pediy.com/kssd/pediy11/119601.html`](https://www.pediy.com/kssd/pediy11/119601.html)

【文章标题】: 一网络验证脱机外挂的破解分析  
【文章作者】: 浮海观云  
【作者邮箱】: cc\_sz@163.com  
【作者主页】: http://www.money988.com/software/  
【作者 QQ 号】: 250341858  
【软件名称】: 热血江湖\*\*\*美服 1.68 版  
【软件大小】: <3M  
【下载地址】: 可能已经下载不到了  
【加壳方式】: Themida 1.8  
【保护方式】: 网络验证  
【编写语言】: VC++  
【使用工具】: OD,wpe  
【操作平台】: windows  
【软件介绍】: 一个很老的游戏的老外挂  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  Themida1.8 可以用高人们的脱壳脚本来处理，脱壳后虽然有地方被 VM 但是没关系，没有 VM 到我们需要的关键地方。  
  也可以直接用 OD 的内存挂接方法调试。  
    
  我最开始时没有找到脚本，先运行外挂程序，然后用 OD 内存挂接载入程序，断下后按 F9 让它运行，查看可执行模块－选择  
  外挂模块：  
  Executable modules, 项目 0  
   基址=00400000  
   大小=0046B000 (4632576.)  
   入口=00601014 hot168.<模块入口点>  
   名称=hot168  
   路径=在 d:\\hotcrazy\\hotcrazy.exe  
    
  在外挂模块上回车，来到程序空间：  
  00401000    8B4424 0C       MOV EAX,DWORD PTR SS:\[ESP+C\]  
  00401004    83EC 20         SUB ESP,20  
  00401007    8A88 04010000   MOV CL,BYTE PTR DS:\[EAX+104\]  
  0040100D    55              PUSH EBP  
  0040100E    8BA8 00010000   MOV EBP,DWORD PTR DS:\[EAX+100\]  
  00401014    57              PUSH EDI  
  00401015    8BFD            MOV EDI,EBP  
  00401017    C1E7 04         SHL EDI,4  
  0040101A    03F8            ADD EDI,EAX  
  0040101C    F6C1 02         TEST CL,2  
  0040101F    75 09           JNZ SHORT hot168.0040102A  
    
  在 OD 命令窗口下断:bpx send ,再下断:bpx recv，这样如果外挂连接验证时会断下来，我们试试看。现在填写好游戏的帐号  
  密码和外挂的帐号密码（可以是乱写的）然后登录...果然在 send 函数断下了！  
    
  0047CB70  |.  50            PUSH EAX                                         ; /Flags  
  0047CB71  |.  8B4D 10       MOV ECX,DWORD PTR SS:\[EBP+10\]                    ; |  
  0047CB74  |.  51            PUSH ECX                                         ; |DataSize  
  0047CB75  |.  8B55 0C       MOV EDX,DWORD PTR SS:\[EBP+C\]                     ; |  
  0047CB78  |.  52            PUSH EDX                                         ; |Data  
  0047CB79  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]                     ; |  
  0047CB7C  |.  50            PUSH EAX                                         ; |Socket  
  0047CB7D  |.  FF15 60955000 CALL DWORD PTR DS:\[<&ws2\_32.send>\]               ; \\send   ;断在这里  
    
  在 Data 中可以看到要发送到外挂验证服务器的数据,不过好像是 DES 加密过的，取消断点执行到返回。再 F9 时断在了 Recv  
    
  0047CAF0  |.  50            PUSH EAX                                         ; /Flags  
  0047CAF1  |.  8B4D 10       MOV ECX,DWORD PTR SS:\[EBP+10\]                    ; |  
  0047CAF4  |.  51            PUSH ECX                                         ; |BufSize  
  0047CAF5  |.  8B55 0C       MOV EDX,DWORD PTR SS:\[EBP+C\]                     ; |  
  0047CAF8  |.  52            PUSH EDX                                         ; |Buffer  
  0047CAF9  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]                     ; |  
  0047CAFC  |.  50            PUSH EAX                                         ; |Socket  
  0047CAFD  |.  FF15 5C955000 CALL DWORD PTR DS:\[<&ws2\_32.recv>\]               ; \\recv  
    
  Buffer 中可以看到从外挂服务器返回的加密数据，后面要跟踪解密过程，能搞清这个就可以做出自己的验证服务器了。不过  
  那应该不是件容易的事，能爆破或者改成本地验证也不错啊！  
    
  调试过程中找到一个关键字符串如下：  
  0045A677   > \\8B8B 08040000 MOV ECX,DWORD PTR DS:\[EBX+408\]  
  0045A67D   .  6A 00         PUSH 0  
  0045A67F   .  68 8C965100   PUSH 热血江湖.0051968C    ;非法使用!!!..  
  0045A684   .  6A 01         PUSH 1  
  0045A686   .  51            PUSH ECX  
  0045A687   .  E8 9483FEFF   CALL 热血江湖.00442A20  
    
  所以让上看是什么条件跳转到了这里，很容易地来到下面：  
  0045A4A1   .  8B93 08040000 MOV EDX,DWORD PTR DS:\[EBX+408\]  
  0045A4A7   .  8D4424 7C     LEA EAX,DWORD PTR SS:\[ESP+7C\]  
  0045A4AB   .  50            PUSH EAX  
  0045A4AC   .  8D4C24 28     LEA ECX,DWORD PTR SS:\[ESP+28\]  
  0045A4B0   .  51            PUSH ECX  
  0045A4B1   .  52            PUSH EDX  
  0045A4B2   .  E8 492B0200   CALL 热血江湖.0047D000  ;可以看出这个 CALL 很重要了！！！  
  0045A4B7   .  83C4 18       ADD ESP,18  
  0045A4BA   .  85C0          TEST EAX,EAX  
  0045A4BC   .  0F85 E7010000 JNZ 热血江湖.0045A6A9   ;跳走后连接服务器失败  
  0045A4C2   .  66:837C24 7A >CMP WORD PTR SS:\[ESP+7A\],6F  
  0045A4C8   .  0F8F 28010000 JG 热血江湖.0045A5F6    ;到“当前版本已过期“  
  0045A4CE   .  66:8B4424 7C  MOV AX,WORD PTR SS:\[ESP+7C\]  
  0045A4D3   .  66:3D 0900    CMP AX,9  
  0045A4D7   .  0F84 4B010000 JE 热血江湖.0045A628    ;这个跳到“征服者被非法修改”要重要下载最新版本  
  0045A4DD   .  66:3D 0300    CMP AX,3  
  0045A4E1   .  0F85 90010000 JNZ 热血江湖.0045A677   ;这个跳到“非法使用!!!..”  
    
  到了这里简单一点的爆破的话改一下的话 nop 掉 0045A4E1 就行了。但是你一定想知道 call 0047d000 里做了什么？这个很重要！  
  那么跟进它:  
  0047D000   $  55            PUSH EBP  
  0047D001   .  8BEC          MOV EBP,ESP  
  0047D003   .  6A FF         PUSH -1  
  0047D005   .  68 704E5000   PUSH 热血江湖.00504E70                               ;  SE 处理程序安装  
  0047D00A   .  64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]  
  0047D010   .  50            PUSH EAX  
  0047D011   .  64:8925 00000>MOV DWORD PTR FS:\[0\],ESP  
  0047D018   .  51            PUSH ECX  
  0047D019   .  B8 901C0000   MOV EAX,1C90  
    
  局部调用来自 0040DA9A, 0040E00B, 0040E61A, 0040EB78, 00455623, 0045A4B2  
    
  可以看到有 6 处调用了"0047D000"，所以上面说的简单 nop 是不行的，如果还是要认真分析，慢慢琢磨...  
  经过几次的反复调试，基本可以确定“0047D000“里就是去网络验证的关键调用！  
    
  0047D126   .  51            PUSH ECX                                         ; /Name  
  0047D127   .  FF15 8C955000 CALL DWORD PTR DS:\[<&ws2\_32.gethostbyname>\]      ; \\gethostbyname  
  。。。  
  0047D1A1   .  50            PUSH EAX  
  0047D1A2   .  FF15 88955000 CALL DWORD PTR DS:\[<&ws2\_32.inet\_ntoa>\]          ;  ws2\_32.inet\_ntoa  
    
  下面不久出了外挂验证服务器的 IP 地址  
  “0047CD90“是关键中的重点，要进入看看  
  0047D1FF   .  50            PUSH EAX  
  0047D200   .  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]  
  0047D203   .  8D95 6CE3FFFF LEA EDX,DWORD PTR SS:\[EBP-1C94\]  
  0047D209   .  52            PUSH EDX  
  0047D20A   .  50            PUSH EAX  
  0047D20B   .  E8 80FBFFFF   CALL 热血江湖.0047CD90                               ;  进入这里分析  
  >  
  0047CD90  /$  81EC 4C010000 SUB ESP,14C  
  0047CD96  |.  A1 1C915B00   MOV EAX,DWORD PTR DS:\[5B911C\]  
  0047CD9B  |.  898424 480100>MOV DWORD PTR SS:\[ESP+148\],EAX  
  0047CDA2  |.  8B8424 5C0100>MOV EAX,DWORD PTR SS:\[ESP+15C\]  
  0047CDA9  |.  50            PUSH EAX                                         ; /NetShort  
  0047CDAA  |.  66:C78424 1C0>MOV WORD PTR SS:\[ESP+11C\],2                      ; |  
  0047CDB4  |.  FF15 68955000 CALL DWORD PTR DS:\[<&ws2\_32.htons>\]              ; \\ntohs  
    
  ...  
  0047CDE5  |.  51            PUSH ECX                                         ; /pAddr  
  0047CDE6  |.  FF15 64955000 CALL DWORD PTR DS:\[<&ws2\_32.inet\_addr>\]          ; \\inet\_addr  
  0047CDEC  |.  6A 06         PUSH 6                                           ; /Protocol = IPPROTO\_TCP  
  0047CDEE  |.  6A 01         PUSH 1                                           ; |Type = SOCK\_STREAM  
  0047CDF0  |.  6A 02         PUSH 2                                           ; |Family = AF\_INET  
  0047CDF2  |.  898424 300100>MOV DWORD PTR SS:\[ESP+130\],EAX                   ; |  
  0047CDF9  |.  FF15 74955000 CALL DWORD PTR DS:\[<&ws2\_32.socket>\]             ; \\socket  
  0047CDFF  |.  8BB424 5C0100>MOV ESI,DWORD PTR SS:\[ESP+15C\]  
  0047CE06  |.  8B3D 90955000 MOV EDI,DWORD PTR DS:\[<&ws2\_32.setsockopt>\]      ;  ws2\_32.setsockopt  
  0047CE0C  |.  6A 04         PUSH 4                                           ; /DataSize = 4  
  0047CE0E  |.  8D5424 0C     LEA EDX,DWORD PTR SS:\[ESP+C\]                     ; |  
  0047CE12  |.  52            PUSH EDX                                         ; |Data  
  0047CE13  |.  68 06100000   PUSH 1006                                        ; |Option = SO\_RCVTIMEO  
  0047CE18  |.  68 FFFF0000   PUSH 0FFFF                                       ; |Level = SOL\_SOCKET  
  0047CE1D  |.  50            PUSH EAX                                         ; |Socket  
  0047CE1E  |.  8906          MOV DWORD PTR DS:\[ESI\],EAX                       ; |  
  0047CE20  |.  C74424 1C 204>MOV DWORD PTR SS:\[ESP+1C\],4E20                   ; |  
  0047CE28  |.  FFD7          CALL EDI                                         ; \\setsockopt  
    
  ...  
  0047CF01  |.  6A 10         PUSH 10                                          ; /AddrLen = 10 (16.)  
  0047CF03  |.  8D8C24 240100>LEA ECX,DWORD PTR SS:\[ESP+124\]                   ; |  
  0047CF0A  |.  51            PUSH ECX                                         ; |pSockAddr  
  0047CF0B  |.  52            PUSH EDX                                         ; |Socket  
  0047CF0C  |.  FF15 78955000 CALL DWORD PTR DS:\[<&ws2\_32.connect>\]            ; \\connect  
  0047CF12  |.  8B06          MOV EAX,DWORD PTR DS:\[ESI\]  
  0047CF14  |.  8D4C24 14     LEA ECX,DWORD PTR SS:\[ESP+14\]  
  0047CF18  |.  51            PUSH ECX                                         ; /pTimeout  
  0047CF19  |.  6A 00         PUSH 0                                           ; |Exceptfds = NULL  
  0047CF1B  |.  8D5424 24     LEA EDX,DWORD PTR SS:\[ESP+24\]                    ; |  
  0047CF1F  |.  52            PUSH EDX                                         ; |Writefds  
  0047CF20  |.  6A 00         PUSH 0                                           ; |Readfds = NULL  
  0047CF22  |.  6A 00         PUSH 0                                           ; |nfds = 0  
  0047CF24  |.  894424 34     MOV DWORD PTR SS:\[ESP+34\],EAX                    ; |  
  0047CF28  |.  C74424 30 010>MOV DWORD PTR SS:\[ESP+30\],1                      ; |  
  0047CF30  |.  C74424 28 0C0>MOV DWORD PTR SS:\[ESP+28\],0C                     ; |  
  0047CF38  |.  C74424 2C 000>MOV DWORD PTR SS:\[ESP+2C\],0                      ; |  
  0047CF40  |.  FF15 80955000 CALL DWORD PTR DS:\[<&ws2\_32.select>\]             ; \\select  
  。。。  
  0047CF18  |.  51            PUSH ECX                                         ; /pTimeout  
  0047CF19  |.  6A 00         PUSH 0                                           ; |Exceptfds = NULL  
  0047CF1B  |.  8D5424 24     LEA EDX,DWORD PTR SS:\[ESP+24\]                    ; |  
  0047CF1F  |.  52            PUSH EDX                                         ; |Writefds  
  0047CF20  |.  6A 00         PUSH 0                                           ; |Readfds = NULL  
  0047CF22  |.  6A 00         PUSH 0                                           ; |nfds = 0  
  0047CF24  |.  894424 34     MOV DWORD PTR SS:\[ESP+34\],EAX                    ; |  
  0047CF28  |.  C74424 30 010>MOV DWORD PTR SS:\[ESP+30\],1                      ; |  
  0047CF30  |.  C74424 28 0C0>MOV DWORD PTR SS:\[ESP+28\],0C                     ; |  
  0047CF38  |.  C74424 2C 000>MOV DWORD PTR SS:\[ESP+2C\],0                      ; |  
  0047CF40  |.  FF15 80955000 CALL DWORD PTR DS:\[<&ws2\_32.select>\]             ; \\select  
    
    
  ...  
  0047CF72  |.  50            PUSH EAX                                         ; /Socket  
  0047CF73  |.  FF15 94955000 CALL DWORD PTR DS:\[<&ws2\_32.closesocket>\]        ; \\closesocket  
    
  用到了 ws2\_32.dll,网络验证时需要这个进行 socket 编程，send，recv 就是来自这个 dll 里面  
    
  0047CF91  |> \\8B16          MOV EDX,DWORD PTR DS:\[ESI\]  
  0047CF93  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]  
  0047CF97  |.  51            PUSH ECX  
  0047CF98  |.  68 7E660480   PUSH 8004667E  
  0047CF9D  |.  52            PUSH EDX  
  0047CF9E  |.  C74424 1C 000>MOV DWORD PTR SS:\[ESP+1C\],0  
  0047CFA6  |.  FFD7          CALL EDI        ;注意这里里面有 vm 了，所以有隐私的部位应该在这里 a-ha-ha:)  
  0047CFA8  |.  83F8 FF       CMP EAX,-1  
  0047CFAB  |.  75 3A         JNZ SHORT 热血江湖.0047CFE7   ;EAX 正常返回 0,这里要跳走  
    
  0047CFE7  |> \\8B8C24 500100>MOV ECX,DWORD PTR SS:\[ESP+150\]  
  0047CFEE  |.  5F            POP EDI  
  0047CFEF  |.  33C0          XOR EAX,EAX  
  0047CFF1  |.  5E            POP ESI  
  0047CFF2  |.  E8 24C20000   CALL 热血江湖.0048921B  
  0047CFF7  |.  81C4 4C010000 ADD ESP,14C  
  0047CFFD  \\.  C3            RETN  
    
  说了这么多还没有看到怎么解决网络验证的办法呢？那好就干脆点吧。反正我们都喜欢看最关键的隐秘的地方  
  那就是我们断下 Recv 后不久可以到达下面：  
  。。。  
  上面有两处应该是 DES 解密函数,解密的 key 跟日期有关，解密后来到下面：  
  00496E62    8BBC24 2C080000        MOV EDI,DWORD PTR SS:\[ESP+82C\]                  ; 解密返回的验证数据  
  00496E69    B9 31000000            MOV ECX,31  
  00496E6E    8DB424 24040000 LEA ESI,DWORD PTR SS:\[ESP+424\]                   ; 替换成能通过验证的值，放在地址\[5231FA\]  
  00496E75    F3:A5                  REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ESI\]。  
    
  现在终于找到一个比较好的解密方案了，就是替换解密后的数据为能通过验证的数据。因为最开始，外挂是有免费试用的，所以  
  可以得到能通过验证的解密数据，我们只要替换一下就行了。  
    
  替换方法：  
  00496E6E    8DB424 24040000 LEA ESI,DWORD PTR SS:\[ESP+424\]  
  改成：  
  00496E6E    36:8D35 FA315200       LEA ESI,DWORD PTR SS:\[5231FA\]   ;内存\[5231FA\]是程序一大段全是 0 的，用来存放  
  要替换的数据  
    
  好，剩下的事就是写个 loader 在程序运行后，修改内存，添加自己的数据 over!  
  最后向论坛中的多产作者致敬！佩服你们的坚强意志，你们太不容易了，这活真累！  
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2010 年 08 月 31 日 23:58:27