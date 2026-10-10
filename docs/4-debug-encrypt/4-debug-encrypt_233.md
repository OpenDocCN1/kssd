

# AHOO 通群发器

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-94.htm`](https://www.pediy.com/kssd/pediy08/pediy8-94.htm)

YAHOO 通群发器  
  
说明：因为接连碰到了 3 个软件，硬件跟踪发现都是同样的处理，但过程极其复杂，郁闷之余用查算法的工具查了下，DES，所以决定猛补。  
于是翻遍看雪精华，找到精华 5 的 2 篇文章，于是 step by step，终于有所体会。  
在此非常感谢 bbbsl 的文章－－变脸王，风雨无阻的文章－－公路坐标计算系统  1.0。  
  
1.定位注册按钮：  
Borland Delphi 6.0 - 7.0  
TForm\_Reg ： OnClick = btn\_ConfirmClick：004845F4;  
004845F4     55               push ebp  
  
2.流程：  
00484619     8D55 F8          lea edx,dword ptr ss:\[ebp-8\]  
0048461C     8B83 08030000    mov eax,dword ptr ds:\[ebx+308\]  
00484622     E8 7972FDFF      call AutoSend.0045B8A0                      ; 取得注册码  
00484627     8B45 F8          mov eax,dword ptr ss:\[ebp-8\]  
0048462A     8D4D FC          lea ecx,dword ptr ss:\[ebp-4\]  
0048462D     BA F0464800      mov edx,AutoSend.004846F0                   ; ASCII "45"  
00484632     E8 499DFFFF      call AutoSend.0047E380  
00484637     8D45 F4          lea eax,dword ptr ss:\[ebp-C\]  
0048463A     50               push eax  
0048463B     8D55 F0          lea edx,dword ptr ss:\[ebp-10\]  
0048463E     8B83 04030000    mov eax,dword ptr ds:\[ebx+304\]  
00484644     E8 5772FDFF      call AutoSend.0045B8A0                      ; 机器码  
00484649     8B45 F0          mov eax,dword ptr ss:\[ebp-10\]               ; (ASCII "1379193")  
0048464C     B9 64000000      mov ecx,64  
00484651     BA 03000000      mov edx,3  
00484656     E8 FDFFF7FF      call AutoSend.00404658                      ; 从第 3 位开始取  
0048465B     8B55 F4          mov edx,dword ptr ss:\[ebp-C\]                ;  (ASCII "79193")  
0048465E     8B45 FC          mov eax,dword ptr ss:\[ebp-4\]                ; ？？  
00484661     E8 DEFEF7FF      call AutoSend.00404544                      ; 比较  
00484666     75 31            jnz short AutoSend.00484699  
00484668     B8 FC464800      mov eax,AutoSend.004846FC                   ; ASCII "OK!"  
0048466D     E8 AEF2FFFF      call AutoSend.00483920  
00484672     68 08474800      push AutoSend.00484708                      ; ASCII "Main"  
00484677     8D55 EC          lea edx,dword ptr ss:\[ebp-14\]  
0048467A     8B83 08030000    mov eax,dword ptr ds:\[ebx+308\]  
00484680     E8 1B72FDFF      call AutoSend.0045B8A0  
00484685     8B55 EC          mov edx,dword ptr ss:\[ebp-14\]  
00484688     B9 18474800      mov ecx,AutoSend.00484718                   ; ASCII "option.ini"  
0048468D     B8 2C474800      mov eax,AutoSend.0048472C                   ; ASCII "SN"  
00484692     E8 8DE0FFFF      call AutoSend.00482724  
00484697     EB 1E            jmp short AutoSend.004846B7  
  
非常清晰，输入的注册码计算值＝机器码的后几位 79193。  
  
反复跟踪发现，如果注册码位数小于 16 位，那么 0048465E    mov eax,dword ptr ss:\[ebp-4\] 处的值都为 0。  
  
练码：8765432112345678  
cryptosearcher 查算法：DES+MD5  
  
3.算法之旅：00484632     E8 499DFFFF      call AutoSend.0047E380  
分两部分进行，一部分对 ASCII "45"下断获得密钥 key，一部分对注册码下断获得加解密结果。  
  
3.1 密钥 key：  
在 0048462D 处对 ASCII "45"下硬件访问断点跟踪。  
先是对字符串 45 的加长，得到 34 35 00 00 00 00 00 00，这样就有 64 位；  
然后转移了好几处，最后来到：  
0047D9A1     8B3498     mov esi,dword ptr ds:\[eax+ebx\*4\]  
0047D9A4     4B         dec ebx  
0047D9A5     56         push esi  
0047D9A6   ^ 79 F9      jns short AutoSend.0047D9A1  
堆栈：  
0012EC94   00003534  
0012EC98   00000000  
即 key：0012EC94  34 35 00 00 00 00 00 00   
  
3.1.1 往下是 DES 密钥的置换子密钥：PC-1  
0047D9C1     B8 08B54800      mov eax,AutoSend.0048B508          ; DES 密钥的置换子密钥的参数  
0047D9C6     8A18             mov bl,byte ptr ds:\[eax\]           ; 参数指针\*p  
0047D9C8     8BCB             mov ecx,ebx                        ; temp=\*p  
0047D9CA     80E1 07          and cl,7                           ; 以下操作为 bit 级  
0047D9CD     81E1 FF000000    and ecx,0FF  
0047D9D3     51               push ecx                           ; push temp&0x7  
0047D9D4     B9 07000000      mov ecx,7                          ; temp&0x7 和 temp mod 8 效果一样:)  
0047D9D9     5E               pop esi  
0047D9DA     2BCE             sub ecx,esi                        ; 7-temp&0x7  
0047D9DC     BE 01000000      mov esi,1  
0047D9E1     D3E6             shl esi,cl                         ; 需要将 0x00000001 向左移(7-temp&7)位  
0047D9E3     33C9             xor ecx,ecx  
0047D9E5     8ACB             mov cl,bl                          ; temp=\*p  
0047D9E7     C1E9 03          shr ecx,3                          ; 整除以 8  
0047D9EA     8B5D FC          mov ebx,dword ptr ss:\[ebp-4\]       ; key=34 35 00 00 00 00 00 00  
0047D9ED     0FB60C0B         movzx ecx,byte ptr ds:\[ebx+ecx\]    ; 逆取 key？得到第(\*p)bit 的值  
0047D9F1     23F1             and esi,ecx                        ; 用来判断第(\*p)bit 的值是否为 0  
0047D9F3     74 1D            je short AutoSend.0047DA12         ; 如果是 0 就不用操作了  
0047D9F5     8BCA             mov ecx,edx                        ; 不为 0 就将第 i 位置 1  
0047D9F7     83E1 07          and ecx,7  
0047D9FA     51               push ecx  
0047D9FB     B9 07000000      mov ecx,7  
0047DA00     5B               pop ebx  
0047DA01     2BCB             sub ecx,ebx  
0047DA03     B3 01            mov bl,1  
0047DA05     D2E3             shl bl,cl  
0047DA07     8BCA             mov ecx,edx  
0047DA09     C1E9 03          shr ecx,3  
0047DA0C     8B75 F8          mov esi,dword ptr ss:\[ebp-8\]  
0047DA0F     081C0E           or byte ptr ds:\[esi+ecx\],bl        ; 第 i 位置一  
0047DA12     42               inc edx  
0047DA13     40               inc eax  
0047DA14     83FA 38          cmp edx,38  
0047DA17   ^ 75 AD            jnz short AutoSend.0047D9C6  
  
AutoSend.0048B508 的参数为：  
0048B508  38 30 28 20 18 10 08 00  80( .  
0048B510  39 31 29 21 19 11 09 01  91)!.  
0048B518  3A 32 2A 22 1A 12 0A 02  :2\*".  
0048B520  3B 33 2B 23 3E 36 2E 26  ;3+#>6.&  
0048B528  1E 16 0E 06 3D 35 2D 25  =5-%  
0048B530  1D 15 0D 05 3C 34 2C 24  .<4,$  
0048B538  1C 14 0C 04 1B 13 0B 03  .  
将每个值＋1 就是 PC-1 的下标。  
  
说明：64 比特的密钥 K，经过 PC-1 后，生成 56 比特的串。其下标如表所示：   
PC-1   
57 49 41 33 25 17 09 01   
58 50 42 34 26 18 10 02   
59 51 43 35 27 19 11 03   
60 52 44 36 63 55 47 39   
31 23 15 07 62 54 46 38   
30 22 14 06 61 53 45 37   
29 21 13 05 28 20 12 04   
  
然后返回到：  
0047DB45     E8 46FEFFFF      call AutoSend.0047D990             ; PC-1  
  
3.1.2 往下，PC-2，获得 16 个 key：  
0047DBA4     BF 10000000      mov edi,10  
0047DBA9     BB B0B54800      mov ebx,AutoSend.0048B5B0          ; 左移位数的参数，Lsi  
0047DBAE     8B75 FC          mov esi,dword ptr ss:\[ebp-4\]  
0047DBB1     8D45 F1          lea eax,dword ptr ss:\[ebp-F\]       ; 把前 28bit 转一转:)  
0047DBB4     8A0B             mov cl,byte ptr ds:\[ebx\]           ; 转 turnit\[ebx\]下,一下一位  
0047DBB6     BA 03000000      mov edx,3  
0047DBBB     E8 00FFFFFF      call AutoSend.0047DAC0             ; 负责转圈的  
0047DBC0     8D45 ED          lea eax,dword ptr ss:\[ebp-13\]      ; 后 28 也进去转转  
0047DBC3     8A0B             mov cl,byte ptr ds:\[ebx\]  
0047DBC5     BA 03000000      mov edx,3  
0047DBCA     E8 F1FEFFFF      call AutoSend.0047DAC0             ; 负责转圈的  
0047DBCF     8A55 F1          mov dl,byte ptr ss:\[ebp-F\]  
0047DBD2     C1E2 04          shl edx,4  
0047DBD5     8A45 F2          mov al,byte ptr ss:\[ebp-E\]  
0047DBD8     33C9             xor ecx,ecx  
0047DBDA     8AC8             mov cl,al  
0047DBDC     C1E9 04          shr ecx,4  
0047DBDF     0AD1             or dl,cl  
0047DBE1     8855 E6          mov byte ptr ss:\[ebp-1A\],dl  
0047DBE4     C1E0 04          shl eax,4  
0047DBE7     33D2             xor edx,edx  
0047DBE9     8A55 F3          mov dl,byte ptr ss:\[ebp-D\]  
0047DBEC     C1EA 04          shr edx,4  
0047DBEF     0AC2             or al,dl  
0047DBF1     8845 E7          mov byte ptr ss:\[ebp-19\],al  
0047DBF4     8A55 F3          mov dl,byte ptr ss:\[ebp-D\]  
0047DBF7     C1E2 04          shl edx,4  
0047DBFA     8A45 F4          mov al,byte ptr ss:\[ebp-C\]  
0047DBFD     33C9             xor ecx,ecx  
0047DBFF     8AC8             mov cl,al  
0047DC01     C1E9 04          shr ecx,4  
0047DC04     0AD1             or dl,cl  
0047DC06     8855 E8          mov byte ptr ss:\[ebp-18\],dl  
0047DC09     C1E0 04          shl eax,4  
0047DC0C     0A45 ED          or al,byte ptr ss:\[ebp-13\]  
0047DC0F     8845 E9          mov byte ptr ss:\[ebp-17\],al  
0047DC12     8A45 EE          mov al,byte ptr ss:\[ebp-12\]  
0047DC15     8845 EA          mov byte ptr ss:\[ebp-16\],al  
0047DC18     8A45 EF          mov al,byte ptr ss:\[ebp-11\]  
0047DC1B     8845 EB          mov byte ptr ss:\[ebp-15\],al  
0047DC1E     8A45 F0          mov al,byte ptr ss:\[ebp-10\]  
0047DC21     8845 EC          mov byte ptr ss:\[ebp-14\],al  
0047DC24     6A 05            push 5  
0047DC26     8BCE             mov ecx,esi  
0047DC28     8D45 E6          lea eax,dword ptr ss:\[ebp-1A\]  
0047DC2B     BA 06000000      mov edx,6  
0047DC30     E8 F3FDFFFF      call AutoSend.0047DA28             ; 这里是从 56 取 48 的过程  
0047DC35     83C6 06          add esi,6                          ; 存放 16 个 key 的地址  
0047DC38     43               inc ebx  
0047DC39     4F               dec edi                            ; 要得到 16 个子密钥  
0047DC3A   ^ 0F85 71FFFFFF    jnz AutoSend.0047DBB1  
  
该比特串分为长度相等的比特串 C0 和 D0。然后 C0 和 D0 分别循环左移 Ls1 位，得到 C1 和 D1。C1 和 D1 合并起来生成 C1D1。C1D1 经过 PC-2 变换后即生成 48 比特的 K1。   
C1、D1 分别循环左移 LS2 位，再合并，经过 PC-2，生成子密钥 K2……依次类推直至生成子密钥 K16。   
注意：Lsi (I =1,2,….16)的数值是不同的。具体见下表：   
迭代顺序 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16   （LSi）  
左移位数 1 1 2 2 2 2 2 2 1  2  2  2  2  2  2  1   （左移位数的参数）  
  
得到的 16key（6 个字节×16）：  
0048CC10  00 08 00 04 00 10 00 20 00 80 00 50 00 04 04 41  .... .€.P.A  
0048CC20  82 00 40 00 00 10 04 08 00 80 20 08 10 04 80 00  ?@...€ €.  
0048CC30  02 00 60 A0 20 02 00 20 08 01 00 10 00 82 00 12  .\`?. ..?  
0048CC40  00 00 40 81 01 02 00 40 10 04 02 00 04 01 00 50  ..@?.@..P  
0048CC50  00 44 02 00 01 00 80 88 09 00 00 00 34 01 00 00  .D..€?...4..  
0048CC60  88 2A 00 20 10 00 00 00 49 02 10 08 00 04 09 00  ?. ...I...  
  
然后返回到：  
0047E1B7     E8 64F9FFFF      call AutoSend.0047DB20  
再往下就是注册码的处理了。  
  
3.2 跟踪注册码下断：  
取得注册码后下硬件访问断点跟踪，断下后返回：  
0047E3E8     E8 6B62F8FF      call AutoSend.00404658                      ; 顺取 2 位  
0047E3ED     8B45 F0          mov eax,dword ptr ss:\[ebp-10\]               ; 返回在此  
其实就是字符转换成字节，最后值：  
00D4B834  87 65 43 21 12 34 56 78  噀 C!4Vx  
然后转移 2 次：  
0047E1F9     8810             mov byte ptr ds:\[eax\],dl                    ; 断在此  
0012ED28  87 65 43 21 12 34 56 78  噀 C!4Vx  
0047DDA7     8811             mov byte ptr ds:\[ecx\],dl  
0012ED20  87 65 43 21 12 34 56 78  噀 C!4Vx  
  
3.2.1 HR 0012ED20：IP 置换  
0047D77D     0FB60C0E         movzx ecx,byte ptr ds:\[esi+ecx\]       ; 转换后的注册码   
0047D781     23F9             and edi,ecx                           ; 断在此  
...  
0047D79F     42               inc edx  
0047D7A0     40               inc eax  
0047D7A1     83FA 40          cmp edx,40                            ; 56，估计就是 IP 置换  
0047D7A4   ^ 75 B3            jnz short AutoSend.0047D759  
  
返回：  
0047DDB6     E8 81F9FFFF      call AutoSend.0047D73C           ; IP 置换  
0047DDBB     84DB             test bl,bl                       ; HR 012ED20 断下后返回处  
此时 ecx＝0F63F0C6，edx＝5580AA01  
  
此时可以推测程序是否是 IP 置换：  
输入注册码：8765432112345678，转换后：87 65 43 21 12 34 56 78  
10000111 01100101 01000011 00100001 00010010 00110100 01010110 01111000  
  3        11       19       27       35       43       51       59  
IP 置换下标：  
58 50 42 34 26 18 10 2   
60 52 44 36 28 20 12 4   
62 54 46 38 30 22 14 6   
64 56 48 40 32 24 16 8   
57 49 41 33 25 17 09 1   
59 51 43 35 27 19 11 3   
61 53 45 37 29 21 13 5   
63 55 47 39 31 23 15 7   
  
得到：  
11000110 11110000 01100011 00001111 00000001 10101010 10000000 01010101  
即：0012ECA0  C6 F0 63 0F 01 AA 80 55  起 c 獉 U  
  
至此可以肯定是 DES 加密的 IP 置换。往下思路就清晰多了。  
  
3.2.2 判断加密或解密：  
IP 置换的 call 返回后有一个判断：0047DDBB     test bl,bl         ; 返回处  
下面有 2 段一样的代码，估计一个是解密，一个是加密，程序给出的是 bl＝1，k 是从 16 开始取的。  
难道作者把解密当加密，把加密当解密进行变动？  
  
3.2.3 继续往下，将置换值分成 L,R 两部分，并解密：  
0047DE8D     B8 04000000      mov eax,4  
0047DE92     8B55 0C          mov edx,dword ptr ss:\[ebp+C\]  
0047DE95     8D75 F4          lea esi,dword ptr ss:\[ebp-C\]  
0047DE98     8A0A             mov cl,byte ptr ds:\[edx\]  
0047DE9A     880E             mov byte ptr ds:\[esi\],cl       // Li-1  
0047DE9C     46               inc esi  
0047DE9D     42               inc edx  
0047DE9E     48               dec eax  
0047DE9F   ^ 75 F7            jnz short AutoSend.0047DE98  
0047DEA1     B8 04000000      mov eax,4  
0047DEA6     8B55 0C          mov edx,dword ptr ss:\[ebp+C\]  
0047DEA9     83C2 04          add edx,4  
0047DEAC     8A0A             mov cl,byte ptr ds:\[edx\]  
0047DEAE     884A FC          mov byte ptr ds:\[edx-4\],cl     // Ri-1  
0047DEB1     42               inc edx  
0047DEB2     48               dec eax  
0047DEB3   ^ 75 F7            jnz short AutoSend.0047DEAC  
0047DEB5     6A 05            push 5  
0047DEB7     8D45 F0          lea eax,dword ptr ss:\[ebp-10\]  
0047DEBA     50               push eax  
0047DEBB     6A 03            push 3  
0047DEBD     8BCB             mov ecx,ebx  
0047DEBF     8B45 0C          mov eax,dword ptr ss:\[ebp+C\]  
0047DEC2     8B55 08          mov edx,dword ptr ss:\[ebp+8\]  
0047DEC5     E8 86FDFFFF      call AutoSend.0047DC50           // F 函数(Ri-1,Ki)  
0047DECA     B8 04000000      mov eax,4  
0047DECF     8D55 F4          lea edx,dword ptr ss:\[ebp-C\]  
0047DED2     8D75 F0          lea esi,dword ptr ss:\[ebp-10\]  
0047DED5     8B4D 0C          mov ecx,dword ptr ss:\[ebp+C\]  
0047DED8     83C1 04          add ecx,4  
0047DEDB     894D E8          mov dword ptr ss:\[ebp-18\],ecx  
0047DEDE     8A0A             mov cl,byte ptr ds:\[edx\]  
0047DEE0     320E             xor cl,byte ptr ds:\[esi\]           // Li-1 XOR Fi  
0047DEE2     8B7D E8          mov edi,dword ptr ss:\[ebp-18\]  
0047DEE5     880F             mov byte ptr ds:\[edi\],cl  
0047DEE7     FF45 E8          inc dword ptr ss:\[ebp-18\]  
0047DEEA     46               inc esi  
0047DEEB     42               inc edx  
0047DEEC     48               dec eax  
0047DEED   ^ 75 EF            jnz short AutoSend.0047DEDE  
0047DEEF     83EB 06          sub ebx,6  
0047DEF2     FF45 F8          inc dword ptr ss:\[ebp-8\]  
0047DEF5   ^ 75 96            jnz short AutoSend.0047DE8D  
  
解密的结果：  
0012ED20  61 BF 27 F2 E0 67 3A 83  a?蜞 g:  
  
3.2.4 用到的 f 函数：  
0047DEC5     E8 86FDFFFF      call AutoSend.0047DC50           // F 函数？  
它的功能是将 32 比特的输入再转化为 32 比特的输出  
  
0047DC50     55               push ebp  
0047DC51     8BEC             mov ebp,esp  
0047DC53     83C4 EC          add esp,-14  
0047DC56     53               push ebx  
0047DC57     56               push esi  
0047DC58     8B5D 10          mov ebx,dword ptr ss:\[ebp+10\]  
0047DC5B     85DB             test ebx,ebx  
0047DC5D     78 0A            js short AutoSend.0047DC69  
0047DC5F     C1EB 02          shr ebx,2  
0047DC62     8B3499           mov esi,dword ptr ds:\[ecx+ebx\*4\]     ; 参数之一－ki  
0047DC65     4B               dec ebx  
0047DC66     56               push esi  
0047DC67   ^ 79 F9            jns short AutoSend.0047DC62  
0047DC69     8BCC             mov ecx,esp  
0047DC6B     8BDA             mov ebx,edx  
0047DC6D     85DB             test ebx,ebx  
0047DC6F     78 0A            js short AutoSend.0047DC7B  
0047DC71     C1EB 02          shr ebx,2  
0047DC74     8B3498           mov esi,dword ptr ds:\[eax+ebx\*4\]     ; 参数之二－Ri-1  
0047DC77     4B               dec ebx  
0047DC78     56               push esi  
0047DC79   ^ 79 F9            jns short AutoSend.0047DC74  
0047DC7B     8BC4             mov eax,esp  
0047DC7D     894D FC          mov dword ptr ss:\[ebp-4\],ecx  
0047DC80     6A 05            push 5  
0047DC82     8D4D F6          lea ecx,dword ptr ss:\[ebp-A\]  
0047DC85     E8 BAFBFFFF      call AutoSend.0047D844               ; 变换 E，即膨胀过程，32 到 48  
0047DC8A     BB 06000000      mov ebx,6                            ; 6 个字节＝48bit  
0047DC8F     8B45 FC          mov eax,dword ptr ss:\[ebp-4\]         ; Ki  
0047DC92     8D55 F6          lea edx,dword ptr ss:\[ebp-A\]  
0047DC95     8A08             mov cl,byte ptr ds:\[eax\]             ; 顺取 ki  
0047DC97     300A             xor byte ptr ds:\[edx\],cl             ; 膨胀后与子密钥 ki 相异或  
0047DC99     42               inc edx  
0047DC9A     40               inc eax  
0047DC9B     4B               dec ebx  
0047DC9C   ^ 75 F7            jnz short AutoSend.0047DC95  
0047DC9E     8A45 F6          mov al,byte ptr ss:\[ebp-A\]             
...                                                                ; 修改 48bit 为 6bit×8，得到 8 字节  
0047DD15     33DB             xor ebx,ebx  
0047DD17     8D75 EE          lea esi,dword ptr ss:\[ebp-12\]  
0047DD1A     8BC3             mov eax,ebx  
0047DD1C     8A16             mov dl,byte ptr ds:\[esi\]  
0047DD1E     E8 31FCFFFF      call AutoSend.0047D954               ; s 盒查表  
0047DD23     8806             mov byte ptr ds:\[esi\],al             ; 查表值  
0047DD25     43               inc ebx  
0047DD26     46               inc esi  
0047DD27     83FB 08          cmp ebx,8                            ; 得到 8 字节＝32bit  
0047DD2A   ^ 75 EE            jnz short AutoSend.0047DD1A  
0047DD2C     BB 04000000      mov ebx,4                            ; 以下将 2 字节整合成 1 字节  
0047DD31     8D45 EE          lea eax,dword ptr ss:\[ebp-12\]  
0047DD34     8D55 F6          lea edx,dword ptr ss:\[ebp-A\]  
0047DD37     8A08             mov cl,byte ptr ds:\[eax\]  
0047DD39     C1E1 04          shl ecx,4  
0047DD3C     0A48 01          or cl,byte ptr ds:\[eax+1\]  
0047DD3F     880A             mov byte ptr ds:\[edx\],cl             ; 整合后保存，得到 4 个字节  
0047DD41     42               inc edx  
0047DD42     83C0 02          add eax,2  
0047DD45     4B               dec ebx  
0047DD46   ^ 75 EF            jnz short AutoSend.0047DD37  
0047DD48     8D45 F6          lea eax,dword ptr ss:\[ebp-A\]  
0047DD4B     BA 05000000      mov edx,5  
0047DD50     E8 7FFBFFFF      call AutoSend.0047D8D4               ; P 变换？输出 32bit  
0047DD55     BB 04000000      mov ebx,4  
0047DD5A     8D45 F6          lea eax,dword ptr ss:\[ebp-A\]  
0047DD5D     8B55 0C          mov edx,dword ptr ss:\[ebp+C\]  
0047DD60     8A08             mov cl,byte ptr ds:\[eax\]  
0047DD62     880A             mov byte ptr ds:\[edx\],cl             ; 转存  
0047DD64     42               inc edx  
0047DD65     40               inc eax  
0047DD66     4B               dec ebx  
0047DD67   ^ 75 F7            jnz short AutoSend.0047DD60  
0047DD69     8B75 E4          mov esi,dword ptr ss:\[ebp-1C\]  
0047DD6C     8B5D E8          mov ebx,dword ptr ss:\[ebp-18\]  
0047DD6F     8BE5             mov esp,ebp  
0047DD71     5D               pop ebp  
0047DD72     C2 0C00          retn 0C  
  
输入 Ri-1(32 比特)经过变换 E 后，膨胀为 48 比特。   
膨胀后的比特串分为 8 组，每组 6 比特。各组经过各自的 S 盒后，又变为 4 比特(具体过程见后)，合并后又成为 32 比特。该 32 比特经过 P 变换后，其下标列表如下：   
经过 P 变换后输出的比特串才是 32 比特的 f (Ri-1,Ki)。   
  
如果 message＝87 65 43 21 12 34 56 78  
f 函数的结果：  
3C DD C6 39；1B 4C 8C 73；68 C6 F7 F2；E7 17 77 3F；  
A5 FD DC 6E；8E 22 F1 C1；B3 CC 25 CC；D0 39 72 3B；  
60 FA 04 BE；D7 11 97 55；CC D2 4E 82；F1 70 A3 5A；  
6A 85 92 60；8B 7E 07 C8；A2 10 49 B9；6F 4A EC D6。  
  
3.2.5 对解密结果的 IP-1 置换：  
0047DF39     E8 82F8FFFF      call AutoSend.0047D7C0                      ; IP-1 置换？  
参数：0012ED20  61 BF 27 F2 E0 67 3A 83  a?蜞 g:  
结果：0012ED20  76 3F 34 18 19 FD E1 93  v?4  
  
逆置换的结果与取得的机器码部分比较，相等就注册成功。  
  
4.算法总结：  
作者很狡猾，提供的 key 只有 2 个字符，以 00 加长为 64bit＝34 35 00 00 00 00 00 00  
然后运算得到 16 个 key：  
k16：10 08 00 04 09 00；  k15：10 00 00 00 49 02   
k14：00 00 88 2A 00 20；  k13：09 00 00 00 34 01  
k12：02 00 01 00 80 88；  k11：04 01 00 50 00 44  
k10：00 40 10 04 02 00；  k09：00 00 40 81 01 02  
k08：00 10 00 82 00 12；  k07：20 02 00 20 08 01   
k06：80 00 02 00 60 A0；  k05：00 80 20 08 10 04  
k04：40 00 00 10 04 08；  k03：00 04 04 41 82 00   
k02：00 20 00 80 00 50；  k01：00 08 00 04 00 10  
  
注册码需要 16 位字符，转换成 8 个字节：87 65 43 21 12 34 56 78，  
DES 解密得到的值＝机器码第 3 位后的几位，那就注册成功。  
  
5.反推注册码：  
程序内置加密解密程序，这就好办，可以利用程序本身来加解密。  
  
先实践程序内置的加解密是否可用：  
练码：8765432112345678  
解密结果：76 3F 34 18 19 FD E1 93   
  
将 0047DDB6（IP 置换）前的参数 87 65 43 21 12 34 56 78 修改成 76 3F 34 18 19 FD E1 93，然后步过该 call，来到 0047DDBB，往下，修改跳转，使其进行加密（作者对输入的注册码进行转换后用到的是解密程序，因此反过来），一路往下，最后结果是：87 65 43 21 12 34 56 78，呵呵，有用。  
  
那好，修改为机器码后几位：79193，即  
37 39 31 39 33 00 00 00  
解密后得到：  
00D4EFB0  A8 51 CF EF 1D 17 0F 73  ≦巷 s  
好，修改 0047DDB6（IP 置换）的参数为 A8 51 CF EF 1D 17 0F 73，步过该 call，不修改跳转，继续，呵呵最后结果：37 39 31 39 33 00 00 00    
  
OK，成功！  
机器码：1379193  
注册码：A8 51 CF EF 1D 17 0F 73  
  
6.后记：  
程序虽然使用 DES 算法，但是它本身既提供了加密也提供了解密。这个很致命，因此你都不用自己写注册机，也不用其他工具解密，直接利用程序本身的加解密搞定注册码。 

放个 DES 的 delphi 源代码  
unit des;  
  
interface  
  
uses SysUtils,IniFiles;  
  
type  
  TKeyByte = array\[0..5\] of Byte;  
  TDesMode = (dmEncry, dmDecry);  
  
  function EncryStr(Str, Key: String): String;  
  function DecryStr(Str, Key: String): String;  
  function EncryStrHex(Str, Key: String): String;  
  function DecryStrHex(StrHex, Key: String): String;  
  
const  
  BitIP: array\[0..63\] of Byte =  
    (57, 49, 41, 33, 25, 17,  9,  1,  
     59, 51, 43, 35, 27, 19, 11,  3,  
     61, 53, 45, 37, 29, 21, 13,  5,  
     63, 55, 47, 39, 31, 23, 15,  7,  
     56, 48, 40, 32, 24, 16,  8,  0,  
     58, 50, 42, 34, 26, 18, 10,  2,  
     60, 52, 44, 36, 28, 20, 12,  4,  
     62, 54, 46, 38, 30, 22, 14,  6 );  
  
  BitCP: array\[0..63\] of Byte =  
    ( 39,  7, 47, 15, 55, 23, 63, 31,  
      38,  6, 46, 14, 54, 22, 62, 30,  
      37,  5, 45, 13, 53, 21, 61, 29,  
      36,  4, 44, 12, 52, 20, 60, 28,  
      35,  3, 43, 11, 51, 19, 59, 27,  
      34,  2, 42, 10, 50, 18, 58, 26,  
      33,  1, 41,  9, 49, 17, 57, 25,  
      32,  0, 40,  8, 48, 16, 56, 24 );  
  
  BitExp: array\[0..47\] of Integer =  
    ( 31, 0, 1, 2, 3, 4, 3, 4, 5, 6, 7, 8, 7, 8, 9,10,  
      11,12,11,12,13,14,15,16,15,16,17,18,19,20,19,20,  
      21,22,23,24,23,24,25,26,27,28,27,28,29,30,31,0  );  
  
  BitPM: array\[0..31\] of Byte =  
    ( 15, 6,19,20,28,11,27,16, 0,14,22,25, 4,17,30, 9,  
       1, 7,23,13,31,26, 2, 8,18,12,29, 5,21,10, 3,24 );  
  
  sBox: array\[0..7\] of array\[0..63\] of Byte =  
    ( ( 14,  4, 13,  1,  2, 15, 11,  8,  3, 10,  6, 12,  5,  9,  0,  7,  
         0, 15,  7,  4, 14,  2, 13,  1, 10,  6, 12, 11,  9,  5,  3,  8,  
         4,  1, 14,  8, 13,  6,  2, 11, 15, 12,  9,  7,  3, 10,  5,  0,  
        15, 12,  8,  2,  4,  9,  1,  7,  5, 11,  3, 14, 10,  0,  6, 13 ),  
  
      ( 15,  1,  8, 14,  6, 11,  3,  4,  9,  7,  2, 13, 12,  0,  5, 10,  
         3, 13,  4,  7, 15,  2,  8, 14, 12,  0,  1, 10,  6,  9, 11,  5,  
         0, 14,  7, 11, 10,  4, 13,  1,  5,  8, 12,  6,  9,  3,  2, 15,  
        13,  8, 10,  1,  3, 15,  4,  2, 11,  6,  7, 12,  0,  5, 14,  9 ),  
  
      ( 10,  0,  9, 14,  6,  3, 15,  5,  1, 13, 12,  7, 11,  4,  2,  8,  
        13,  7,  0,  9,  3,  4,  6, 10,  2,  8,  5, 14, 12, 11, 15,  1,  
        13,  6,  4,  9,  8, 15,  3,  0, 11,  1,  2, 12,  5, 10, 14,  7,  
         1, 10, 13,  0,  6,  9,  8,  7,  4, 15, 14,  3, 11,  5,  2, 12 ),  
  
      (  7, 13, 14,  3,  0,  6,  9, 10,  1,  2,  8,  5, 11, 12,  4, 15,  
        13,  8, 11,  5,  6, 15,  0,  3,  4,  7,  2, 12,  1, 10, 14,  9,  
        10,  6,  9,  0, 12, 11,  7, 13, 15,  1,  3, 14,  5,  2,  8,  4,  
         3, 15,  0,  6, 10,  1, 13,  8,  9,  4,  5, 11, 12,  7,  2, 14 ),  
  
      (  2, 12,  4,  1,  7, 10, 11,  6,  8,  5,  3, 15, 13,  0, 14,  9,  
        14, 11,  2, 12,  4,  7, 13,  1,  5,  0, 15, 10,  3,  9,  8,  6,  
         4,  2,  1, 11, 10, 13,  7,  8, 15,  9, 12,  5,  6,  3,  0, 14,  
        11,  8, 12,  7,  1, 14,  2, 13,  6, 15,  0,  9, 10,  4,  5,  3 ),  
  
      ( 12,  1, 10, 15,  9,  2,  6,  8,  0, 13,  3,  4, 14,  7,  5, 11,  
        10, 15,  4,  2,  7, 12,  9,  5,  6,  1, 13, 14,  0, 11,  3,  8,  
         9, 14, 15,  5,  2,  8, 12,  3,  7,  0,  4, 10,  1, 13, 11,  6,  
         4,  3,  2, 12,  9,  5, 15, 10, 11, 14,  1,  7,  6,  0,  8, 13 ),  
  
      (  4, 11,  2, 14, 15,  0,  8, 13,  3, 12,  9,  7,  5, 10,  6,  1,  
        13,  0, 11,  7,  4,  9,  1, 10, 14,  3,  5, 12,  2, 15,  8,  6,  
         1,  4, 11, 13, 12,  3,  7, 14, 10, 15,  6,  8,  0,  5,  9,  2,  
         6, 11, 13,  8,  1,  4, 10,  7,  9,  5,  0, 15, 14,  2,  3, 12 ),  
  
      ( 13,  2,  8,  4,  6, 15, 11,  1, 10,  9,  3, 14,  5,  0, 12,  7,  
         1, 15, 13,  8, 10,  3,  7,  4, 12,  5,  6, 11,  0, 14,  9,  2,  
         7, 11,  4,  1,  9, 12, 14,  2,  0,  6, 10, 13, 15,  3,  5,  8,  
         2,  1, 14,  7,  4, 10,  8, 13, 15, 12,  9,  0,  3,  5,  6, 11 ) );  
  
  BitPMC1: array\[0..55\] of Byte =  
    ( 56, 48, 40, 32, 24, 16,  8,  
       0, 57, 49, 41, 33, 25, 17,  
       9,  1, 58, 50, 42, 34, 26,  
      18, 10,  2, 59, 51, 43, 35,  
      62, 54, 46, 38, 30, 22, 14,  
       6, 61, 53, 45, 37, 29, 21,  
      13,  5, 60, 52, 44, 36, 28,  
      20, 12,  4, 27, 19, 11,  3 );  
  
  BitPMC2: array\[0..47\] of Byte =  
    ( 13, 16, 10, 23,  0,  4,  
       2, 27, 14,  5, 20,  9,  
      22, 18, 11,  3, 25,  7,  
      15,  6, 26, 19, 12,  1,  
      40, 51, 30, 36, 46, 54,  
      29, 39, 50, 44, 32, 47,  
      43, 48, 38, 55, 33, 52,  
      45, 41, 49, 35, 28, 31 );  
  
var  
  subKey: array\[0..15\] of TKeyByte;  
        
implementation  
  
procedure initPermutation(var inData: array of Byte);  
var  
  newData: array\[0..7\] of Byte;  
  i: Integer;  
begin  
  FillChar(newData, 8, 0);  
  for i := 0 to 63 do  
    if (inData\[BitIP\[i\] shr 3\] and (1 shl (7- (BitIP\[i\] and $07)))) <> 0 then  
      newData\[i shr 3\] := newData\[i shr 3\] or (1 shl (7-(i and $07)));  
  for i := 0 to 7 do inData\[i\] := newData\[i\];  
end;  
  
procedure conversePermutation(var inData: array of Byte);  
var  
  newData: array\[0..7\] of Byte;  
  i: Integer;  
begin  
  FillChar(newData, 8, 0);  
  for i := 0 to 63 do  
    if (inData\[BitCP\[i\] shr 3\] and (1 shl (7-(BitCP\[i\] and $07)))) <> 0 then  
      newData\[i shr 3\] := newData\[i shr 3\] or (1 shl (7-(i and $07)));  
  for i := 0 to 7 do inData\[i\] := newData\[i\];  
end;  
  
procedure expand(inData: array of Byte; var outData: array of Byte);  
var  
  i: Integer;  
begin  
  FillChar(outData, 6, 0);  
  for i := 0 to 47 do  
    if (inData\[BitExp\[i\] shr 3\] and (1 shl (7-(BitExp\[i\] and $07)))) <> 0 then  
      outData\[i shr 3\] := outData\[i shr 3\] or (1 shl (7-(i and $07)));  
end;  
  
procedure permutation(var inData: array of Byte);  
var  
  newData: array\[0..3\] of Byte;  
  i: Integer;  
begin  
  FillChar(newData, 4, 0);  
  for i := 0 to 31 do  
    if (inData\[BitPM\[i\] shr 3\] and (1 shl (7-(BitPM\[i\] and $07)))) <> 0 then  
      newData\[i shr 3\] := newData\[i shr 3\] or (1 shl (7-(i and $07)));  
  for i := 0 to 3 do inData\[i\] := newData\[i\];  
end;  
  
function si(s,inByte: Byte): Byte;  
var  
  c: Byte;  
begin  
  c := (inByte and $20) or ((inByte and $1e) shr 1) or  
    ((inByte and $01) shl 4);  
  Result := (sBox\[s\]\[c\] and $0f);  
end;  
  
procedure permutationChoose1(inData: array of Byte;  
  var outData: array of Byte);  
var  
  i: Integer;  
begin  
  FillChar(outData, 7, 0);  
  for i := 0 to 55 do  
    if (inData\[BitPMC1\[i\] shr 3\] and (1 shl (7-(BitPMC1\[i\] and $07)))) <> 0 then  
      outData\[i shr 3\] := outData\[i shr 3\] or (1 shl (7-(i and $07)));  
end;  
  
procedure permutationChoose2(inData: array of Byte;  
  var outData: array of Byte);  
var  
  i: Integer;  
begin  
  FillChar(outData, 6, 0);  
  for i := 0 to 47 do  
    if (inData\[BitPMC2\[i\] shr 3\] and (1 shl (7-(BitPMC2\[i\] and $07)))) <> 0 then  
      outData\[i shr 3\] := outData\[i shr 3\] or (1 shl (7-(i and $07)));  
end;  
  
procedure cycleMove(var inData: array of Byte; bitMove: Byte);  
var  
  i: Integer;  
begin  
  for i := 0 to bitMove - 1 do  
  begin  
    inData\[0\] := (inData\[0\] shl 1) or (inData\[1\] shr 7);  
    inData\[1\] := (inData\[1\] shl 1) or (inData\[2\] shr 7);  
    inData\[2\] := (inData\[2\] shl 1) or (inData\[3\] shr 7);  
    inData\[3\] := (inData\[3\] shl 1) or ((inData\[0\] and $10) shr 4);  
    inData\[0\] := (inData\[0\] and $0f);  
  end;  
end;  
  
procedure makeKey(inKey: array of Byte; var outKey: array of TKeyByte);  
const  
  bitDisplace: array\[0..15\] of Byte =  
    ( 1,1,2,2, 2,2,2,2, 1,2,2,2, 2,2,2,1 );  
var  
  outData56: array\[0..6\] of Byte;  
  key28l: array\[0..3\] of Byte;  
  key28r: array\[0..3\] of Byte;  
  key56o: array\[0..6\] of Byte;  
  i: Integer;  
begin  
  permutationChoose1(inKey, outData56);  
  
  key28l\[0\] := outData56\[0\] shr 4;  
  key28l\[1\] := (outData56\[0\] shl 4) or (outData56\[1\] shr 4);  
  key28l\[2\] := (outData56\[1\] shl 4) or (outData56\[2\] shr 4);  
  key28l\[3\] := (outData56\[2\] shl 4) or (outData56\[3\] shr 4);  
  key28r\[0\] := outData56\[3\] and $0f;  
  key28r\[1\] := outData56\[4\];  
  key28r\[2\] := outData56\[5\];  
  key28r\[3\] := outData56\[6\];  
  
  for i := 0 to 15 do  
  begin  
    cycleMove(key28l, bitDisplace\[i\]);  
    cycleMove(key28r, bitDisplace\[i\]);  
    key56o\[0\] := (key28l\[0\] shl 4) or (key28l\[1\] shr 4);  
    key56o\[1\] := (key28l\[1\] shl 4) or (key28l\[2\] shr 4);  
    key56o\[2\] := (key28l\[2\] shl 4) or (key28l\[3\] shr 4);  
    key56o\[3\] := (key28l\[3\] shl 4) or (key28r\[0\]);  
    key56o\[4\] := key28r\[1\];  
    key56o\[5\] := key28r\[2\];  
    key56o\[6\] := key28r\[3\];  
    permutationChoose2(key56o, outKey\[i\]);  
  end;  
end;  
  
procedure encry(inData, subKey: array of Byte;  
   var outData: array of Byte);  
var  
  outBuf: array\[0..5\] of Byte;  
  buf: array\[0..7\] of Byte;  
  i: Integer;  
begin  
  expand(inData, outBuf);  
  for i := 0 to 5 do outBuf\[i\] := outBuf\[i\] xor subKey\[i\];  
                                                // outBuf       xxxxxxxx xxxxxxxx xxxxxxxx xxxxxxxx xxxxxxxx xxxxxxxx  
  buf\[0\] := outBuf\[0\] shr 2;                                  //xxxxxx -> 2  
  buf\[1\] := ((outBuf\[0\] and $03) shl 4) or (outBuf\[1\] shr 4); // 4 <- xx xxxx -> 4  
  buf\[2\] := ((outBuf\[1\] and $0f) shl 2) or (outBuf\[2\] shr 6); //        2 <- xxxx xx -> 6  
  buf\[3\] := outBuf\[2\] and $3f;                                //                    xxxxxx  
  buf\[4\] := outBuf\[3\] shr 2;                                  //                           xxxxxx  
  buf\[5\] := ((outBuf\[3\] and $03) shl 4) or (outBuf\[4\] shr 4); //                                 xx xxxx  
  buf\[6\] := ((outBuf\[4\] and $0f) shl 2) or (outBuf\[5\] shr 6); //                                        xxxx xx  
  buf\[7\] := outBuf\[5\] and $3f;                                //                                               xxxxxx  
  for i := 0 to 7 do buf\[i\] := si(i, buf\[i\]);  
  for i := 0 to 3 do outBuf\[i\] := (buf\[i\*2\] shl 4) or buf\[i\*2+1\];  
  permutation(outBuf);  
  for i := 0 to 3 do outData\[i\] := outBuf\[i\];  
end;  
  
procedure desData(desMode: TDesMode;  
  inData: array of Byte; var outData: array of Byte);  
// inData, outData 都为 8Bytes，否则出错  
var  
  i, j: Integer;  
  temp, buf: array\[0..3\] of Byte;  
begin  
  for i := 0 to 7 do outData\[i\] := inData\[i\];  
  initPermutation(outData);  
  if desMode = dmEncry then  
  begin  
    for i := 0 to 15 do  
    begin  
      for j := 0 to 3 do temp\[j\] := outData\[j\];                 //temp = Ln  
      for j := 0 to 3 do outData\[j\] := outData\[j + 4\];          //Ln+1 = Rn  
      encry(outData, subKey\[i\], buf);                           //Rn ==Kn==> buf  
      for j := 0 to 3 do outData\[j + 4\] := temp\[j\] xor buf\[j\];  //Rn+1 = Ln^buf  
    end;  
  
    for j := 0 to 3 do temp\[j\] := outData\[j + 4\];  
    for j := 0 to 3 do outData\[j + 4\] := outData\[j\];  
    for j := 0 to 3 do outData\[j\] := temp\[j\];  
  end  
  else if desMode = dmDecry then  
  begin  
    for i := 15 downto 0 do  
    begin  
      for j := 0 to 3 do temp\[j\] := outData\[j\];  
      for j := 0 to 3 do outData\[j\] := outData\[j + 4\];  
      encry(outData, subKey\[i\], buf);  
      for j := 0 to 3 do outData\[j + 4\] := temp\[j\] xor buf\[j\];  
    end;  
    for j := 0 to 3 do temp\[j\] := outData\[j + 4\];  
    for j := 0 to 3 do outData\[j + 4\] := outData\[j\];  
    for j := 0 to 3 do outData\[j\] := temp\[j\];  
  end;  
  conversePermutation(outData);  
end;  
  
//////////////////////////////////////////////////////////////  
  
function EncryStr(Str, Key: String): String;  
var  
  StrByte, OutByte, KeyByte: array\[0..7\] of Byte;  
  StrResult: String;  
  I, J: Integer;  
begin  
  if (Length(Str) > 0) and (Ord(Str\[Length(Str)\]) = 0) then  
    raise Exception.Create('Error: the last char is NULL char.');  
  if Length(Key) < 8 then  
    while Length(Key) < 8 do Key := Key + Chr(0);  
  while Length(Str) mod 8 <> 0 do Str := Str + Chr(0);  
  
  for J := 0 to 7 do KeyByte\[J\] := Ord(Key\[J + 1\]);  
  makeKey(keyByte, subKey);  
  
  StrResult := '';  
  
  for I := 0 to Length(Str) div 8 - 1 do  
  begin  
    for J := 0 to 7 do  
      StrByte\[J\] := Ord(Str\[I \* 8 + J + 1\]);  
    desData(dmEncry, StrByte, OutByte);  
    for J := 0 to 7 do  
      StrResult := StrResult + Chr(OutByte\[J\]);  
  end;  
  
  Result := StrResult;  
end;  
  
function DecryStr(Str, Key: String): String;  
var  
  StrByte, OutByte, KeyByte: array\[0..7\] of Byte;  
  StrResult: String;  
  I, J: Integer;  
begin  
  if Length(Key) < 8 then  
    while Length(Key) < 8 do Key := Key + Chr(0);  
  
  for J := 0 to 7 do KeyByte\[J\] := Ord(Key\[J + 1\]);  
  makeKey(keyByte, subKey);  
  
  StrResult := '';  
  
  for I := 0 to Length(Str) div 8 - 1 do  
  begin  
    for J := 0 to 7 do StrByte\[J\] := Ord(Str\[I \* 8 + J + 1\]);  
    desData(dmDecry, StrByte, OutByte);  
    for J := 0 to 7 do  
      StrResult := StrResult + Chr(OutByte\[J\]);  
  end;  
  while (Length(StrResult) > 0) and  
    (Ord(StrResult\[Length(StrResult)\]) = 0) do  
    Delete(StrResult, Length(StrResult), 1);  
  Result := StrResult;  
end;  
  
///////////////////////////////////////////////////////////  
  
function EncryStrHex(Str, Key: String): String;  
var  
  StrResult, TempResult, Temp: String;  
  I: Integer;  
begin  
  TempResult := EncryStr(Str, Key);  
  StrResult := '';  
  for I := 0 to Length(TempResult) - 1 do  
  begin  
    Temp := Format('%x', \[Ord(TempResult\[I + 1\])\]);  
    if Length(Temp) = 1 then Temp := '0' + Temp;  
    StrResult := StrResult + Temp;  
  end;  
  Result := StrResult;  
end;  
  
function DecryStrHex(StrHex, Key: String): String;  
  function HexToInt(Hex: String): Integer;  
  var  
    I, Res: Integer;  
    ch: Char;  
  begin  
    Res := 0;  
    for I := 0 to Length(Hex) - 1 do  
    begin  
      ch := Hex\[I + 1\];  
      if (ch >= '0') and (ch <= '9') then  
        Res := Res \* 16 + Ord(ch) - Ord('0')  
      else if (ch >= 'A') and (ch <= 'F') then  
        Res := Res \* 16 + Ord(ch) - Ord('A') + 10  
      else if (ch >= 'a') and (ch <= 'f') then  
        Res := Res \* 16 + Ord(ch) - Ord('a') + 10  
      else raise  Exception.Create('Error: not a Hex String');  
    end;  
    Result := Res;  
  end;  
  
var  
  Str, Temp: String;  
  I: Integer;  
begin  
  Str := '';  
  for I := 0 to Length(StrHex) div 2 - 1 do  
  begin  
    Temp := Copy(StrHex, I \* 2 + 1, 2);  
    Str := Str + Chr(HexToInt(Temp));  
  end;  
  Result := DecryStr(Str, Key);  
  end;  
  end.