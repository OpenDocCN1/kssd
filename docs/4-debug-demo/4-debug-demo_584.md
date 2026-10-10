

# 联创上网助手的简单分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-475.htm`](https://www.pediy.com/kssd/pediy07/pediy7-475.htm)

联创上网助手的简单分析  
  
苏州大学独塾湖校区上网方式用的是 802.1x 拨号协议，不能用普通路由器实现共享上网，拨号软件是联创通信的宽带上网助手，其中加了反代理检测，不许系统有双网卡，不许有多个 ip，也不许开代理服务器等违规产品，为了省上网费（1.2 元/小时，抢钱啊。。！但是网速很快，是 10M 的），我决定对它的程序开一开刀！  
  
PEID 查壳，无壳！！VC++7.0 写的。看来联创通信比较小看我们学生，这么大胆，居然不加壳！（其实最不能让我忍受的是它的密码加密算法实在太简单，这么简单的算法还不加壳。。。。）  
  
（一）破解反代理检测  
  
  
废话不多说了，用 OllyDBG 载入，bp MessageBoxA 下断点，运行，点连接，这是拨号成功，然后大概过了 1 秒，程序断在  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00406EDD   |.  E8 5E1C0000          call Dot1xCli.00408B40                 ;  这个 call 是下线
> 00406EE2   |.  8B0D 2C9D4100        mov ecx,dword ptr ds:\[419D2C\]
> 00406EE8   |.  6A 00                push 0                                 ; /Style = MB\_OK|MB\_APPLMODAL
> 00406EEA   |.  68 684D4100          push Dot1xCli.00414D68                 ; |Title = "反代理激活!"
> 00406EEF   |.  8D4424 0C            lea eax,dword ptr ss:\[esp+C\]           ; |
> 00406EF3   |.  50                   push eax                               ; |Text
> 00406EF4   |.  51                   push ecx                               ; |hOwner => NULL
> 00406EF5   |.  FF15 20424100        call near dword ptr ds:\[<&USER32.Messa>; \\MessageBoxA ★断在这里！！
> 00406EFB   |.  81C4 84030000        add esp,384
> 00406F01   |.  C3                   retn
> 
> * * *
> 
> 
> ```

  
  
这里出现反代理检测的对话框，我们向下运行，alt+F9,来到  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00407922    > \\8B8424 74030000      mov eax,dword ptr ss:\[esp+374\]    ;  Case 475 of switch 00406F8A
>  ★这个分支是反代理激活消息号是 475
> 00407929    .  8B8C24 70030000      mov ecx,dword ptr ss:\[esp+370\]
> 00407930    .  8B9424 68030000      mov edx,dword ptr ss:\[esp+368\]
> 00407937    .  50                   push eax
> 00407938    .  51                   push ecx
> 00407939    .  68 75040000          push 475
> 0040793E    .  52                   push edx
> 0040793F    .  E8 4CF4FFFF          call Dot1xCli.00406D90            ;  这个 call 就是处理代码
> 00407944    .  83C4 10              add esp,10                        ;  返回这里★
> 00407947    .  5F                   pop edi
> 00407948    .  5E                   pop esi
> 00407949    .  8BC5                 mov eax,ebp
> 0040794B    .  5D                   pop ebp
> 0040794C    .  81C4 58030000        add esp,358
> 00407952    .  C2 1000              retn 10                           ;  这个分支就是处理的过程，我们直接跳过就行了
> 
> * * *
> 
> 
> ```

  
去掉这个分支，就可以破解了，但我们这里去找找它怎么发现我们有违规产品的。记下消息号为 475，显然，程序用消息的方式来调用反代理激活。然后，点右键，选搜索，当前模块中的名称，然后选 SendMessageA，选出消息号为 475 的  
只有一个 SendMessageA 满足要求  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00405FCA   |> \\33FF                 xor edi,edi                       ;  这里判断是否有违规产品; Case 7C of switch 00405CDA
> 00405FCC   |.  E8 8FBCFFFF          call Dot1xCli.00401C60            ;  检测多网卡
> 00405FD1   |.  85C0                 test eax,eax
> 00405FD3   |.  75 05                jnz short Dot1xCli.00405FDA
> 00405FD5   |.  BF 01000000          mov edi,1                         ;  有多网卡 edi+1
> 00405FDA   |>  E8 51B7FFFF          call Dot1xCli.00401730            ;  检测多 ip 地址
> 00405FDF   |.  85C0                 test eax,eax
> 00405FE1   |.  75 03                jnz short Dot1xCli.00405FE6
> 00405FE3   |.  83CF 02              or edi,2                          ;  多个 ip 地址 edi+2
> 00405FE6   |>  833D C89D4100 14     cmp dword ptr ds:\[419DC8\],14      ;  检测 proxy 代理
> 00405FED   |.  7C 03                jl short Dot1xCli.00405FF2
> 00405FEF   |.  83CF 04              or edi,4                          ;  proxy 代理 edi+4
> 00405FF2   |>  833D CC9D4100 02     cmp dword ptr ds:\[419DCC\],2
> 00405FF9   |.  7C 03                jl short Dot1xCli.00405FFE
> 00405FFB   |.  83CF 05              or edi,5
> 00405FFE   |>  85FF                 test edi,edi                      ;  判断是否有违规产品
> 00406000   |.  C705 C89D4100 000000>mov dword ptr ds:\[419DC8\],0
> 0040600A   |.  C705 CC9D4100 000000>mov dword ptr ds:\[419DCC\],0
> 00406014   |.  0F84 CB010000        je Dot1xCli.004061E5              ;  没有，则不发消息，爆破点★改为 jmp，爆破
> 0040601A   |.  8BB424 1C010000      mov esi,dword ptr ss:\[esp+11C\]
> 00406021   |.  55                   push ebp
> 00406022   |.  8B2D EC414100        mov ebp,dword ptr ds:\[<&USER32.Ki>;  USER32.KillTimer
> 00406028   |.  68 F5010000          push 1F5                          ; /TimerID = 1F5 (501.)
> 0040602D   |.  56                   push esi                          ; |hWnd
> 0040602E   |.  FFD5                 call near ebp                     ; \\KillTimer
> 00406030   |.  6A 7C                push 7C                           ; /TimerID = 7C (124.)
> 00406032   |.  56                   push esi                          ; |hWnd
> 00406033   |.  FFD5                 call near ebp                     ; \\KillTimer
> 00406035   |.  57                   push edi                          ; /这里是哪种违规产品
> 00406036   |.  6A 00                push 0                            ; |wParam = 0
> 00406038   |.  68 75040000          push 475                          ; |检测到违规产品!!消息号为 475
> 0040603D   |.  56                   push esi                          ; |发给自己
> 0040603E   |.  FF15 E4414100        call near dword ptr ds:\[<&USER32.>; \\SendMessageA ★就这一处消息号为 475 的
> 
> * * *
> 
> 
> ```

  
  
其中检测多网卡的方法是遍历系统中可用的网卡树目，超过一个就 eax=1，这样，edi=edi+1，这样 edi 不为 0，即有违规品，然后发送 475 消息，出现反代理激活。检测多个 ip 也是一样，遍历系统中的所以 ip，超过一个 edi=edi+2，然后就发送 475 消息。检测 proxy 代理就有点不太看得懂了，不太懂网络编程，可能按数据包来算的吧。反正我的系统开了 proxy 代理也不出现反代理激活，还有多个 ip 也是，我不太懂一块网卡能有多个 ip？？有网络方面的高手请指点一下，我的机器只出现过第一个分支。不管了，爆破点太多了。  
  
这样，宽带上网助手就能容忍我们系统有双网卡了，有双网卡也就能共享上网了，方法不难，不是本文重点，就不提了。  
  
  
  
（二）保存密码的算法  
无意中看到 linkageclientconfig.ini 文件里有这么两行：  
LastUserName=F02143879  
Identify=2b1b4f3eb539abc0  
  
所以可以肯定是可逆的算法保存密码的，而且文件这么暴露。。。。  
od 中下断点 bp GetPrivateProfileStringA  
代码如下  
  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00402C56   |.  50                  push eax                             ; /IniFileName
> 00402C57   |.  68 96000000         push 96                              ; |BufSize = 96 (150.)
> 00402C5C   |.  83C1 18             add ecx,18                           ; |
> 00402C5F   |.  51                  push ecx                             ; |用户名存放处
> 00402C60   |.  68 A2474100         push Dot1xCli.004147A2               ; |Default = ""
> 00402C65   |.  68 8C474100         push Dot1xCli.0041478C               ; |Key = "LastUserName"
> 00402C6A   |.  68 34484100         push Dot1xCli.00414834               ; |Section = "General"
> 00402C6F   |.  FFD6                call near esi                        ; \\GetPrivateProfileStringA;读出用户名
> 00402C71   |.  33C0                xor eax,eax
> 00402C73   |.  8D5424 08           lea edx,dword ptr ss:\[esp+8\]
> 00402C77   |.  52                  push edx                             ; /IniFileName
> 00402C78   |.  68 2C010000         push 12C                             ; |BufSize = 12C (300.)
> 00402C7D   |.  B9 4B000000         mov ecx,4B                           ; |
> 00402C82   |.  8DBC24 D8000000     lea edi,dword ptr ss:\[esp+D8\]        ; |
> 00402C89   |.  F3:AB               rep stos dword ptr es:\[edi\]          ; |
> 00402C8B   |.  8D8424 D8000000     lea eax,dword ptr ss:\[esp+D8\]        ; |
> 00402C92   |.  50                  push eax                             ; |加密后的密码存放处
> 00402C93   |.  68 A2474100         push Dot1xCli.004147A2               ; |Default = ""
> 00402C98   |.  68 80474100         push Dot1xCli.00414780               ; |Key = "Identify"
> 00402C9D   |.  68 34484100         push Dot1xCli.00414834               ; |Section = "General"
> 00402CA2   |.  FFD6                call near esi                        ; \\GetPrivateProfileStringA
> 00402CA4   |.  A1 3C9D4100         mov eax,dword ptr ds:\[419D3C\]
> 00402CA9   |.  8B48 04             mov ecx,dword ptr ds:\[eax+4\]
> 00402CAC   |.  85C9                test ecx,ecx
> 00402CAE   |.  74 40               je short Dot1xCli.00402CF0
> 00402CB0   |.  8D50 18             lea edx,dword ptr ds:\[eax+18\]
> 00402CB3   |.  8BC2                mov eax,edx
> 00402CB5   |.  8D70 01             lea esi,dword ptr ds:\[eax+1\]
> 00402CB8   |>  8A08                /mov cl,byte ptr ds:\[eax\]
> 00402CBA   |.  40                  |inc eax
> 00402CBB   |.  84C9                |test cl,cl
> 00402CBD   |.^ 75 F9               \\jnz short Dot1xCli.00402CB8
> 00402CBF   |.  8D8C24 D0000000     lea ecx,dword ptr ss:\[esp+D0\]
> 00402CC6   |.  51                  push ecx
> 00402CC7   |.  2BC6                sub eax,esi
> 00402CC9   |.  50                  push eax
> 00402CCA   |.  52                  push edx
> 00402CCB   |.  E8 70F7FFFF         call Dot1xCli.00402440               ;  还原加密的密码★跟进
> 
> * * *
> 
> 
> ```

  
跟进后：解密代码  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> 00402440   /$  81EC A8000000       sub esp,0A8
> 00402446   |.  56                  push esi
> 00402447   |.  8BB424 B8000000     mov esi,dword ptr ss:\[esp+B8\]
> 0040244E   |.  8BC6                mov eax,esi
> 00402450   |.  57                  push edi
> 00402451   |.  8D50 01             lea edx,dword ptr ds:\[eax+1\]
> 00402454   |>  8A08                /mov cl,byte ptr ds:\[eax\]
> 00402456   |.  40                  |inc eax
> 00402457   |.  84C9                |test cl,cl
> 00402459   |.^ 75 F9               \\jnz short Dot1xCli.00402454
> 0040245B   |.  8B8C24 B4000000     mov ecx,dword ptr ss:\[esp+B4\]
> 00402462   |.  2BC2                sub eax,edx
> 00402464   |.  D1E8                shr eax,1
> 00402466   |.  8BF8                mov edi,eax
> 00402468   |.  8B8424 B8000000     mov eax,dword ptr ss:\[esp+B8\]
> 0040246F   |.  50                  push eax
> 00402470   |.  51                  push ecx
> 00402471   |.  8D5424 10           lea edx,dword ptr ss:\[esp+10\]
> 00402475   |.  52                  push edx
> 00402476   |.  E8 657D0000         call Dot1xCli.0040A1E0               ;  卡号 md5 存到 12fa70
> 0040247B   |.  83C4 0C             add esp,0C
> 0040247E   |.  33C9                xor ecx,ecx
> 00402480   |.  85FF                test edi,edi
> 00402482   |.  7E 43               jle short Dot1xCli.004024C7
> 00402484   |.  53                  push ebx
> 00402485   |>  8A044E              /mov al,byte ptr ds:\[esi+ecx\*2\]      ;  取第一个
> 00402488   |.  3C 39               |cmp al,39                           ;  比较是不是数字
> 0040248A   |.  0FBEC0              |movsx eax,al
> 0040248D   |.  7F 05               |jg short Dot1xCli.00402494
> 0040248F   |.  83E8 30             |sub eax,30                          ;  是则－30
> 00402492   |.  EB 03               |jmp short Dot1xCli.00402497
> 00402494   |>  83E8 57             |sub eax,57                          ;  不是－57
> 00402497   |>  8A544E 01           |mov dl,byte ptr ds:\[esi+ecx\*2+1\]    ;  取第二个
> 0040249B   |.  C0E0 04             |shl al,4                            ;  第一个移 4 位
> 0040249E   |.  80FA 39             |cmp dl,39                           ;  和上面一样处理，只是不移位
> 004024A1   |.  88440C 1C           |mov byte ptr ss:\[esp+ecx+1C\],al
> 004024A5   |.  0FBED2              |movsx edx,dl
> 004024A8   |.  7F 05               |jg short Dot1xCli.004024AF
> 004024AA   |.  83EA 30             |sub edx,30
> 004024AD   |.  EB 03               |jmp short Dot1xCli.004024B2
> 004024AF   |>  83EA 57             |sub edx,57
> 004024B2   |>  8BD9                |mov ebx,ecx
> 004024B4   |.  83E3 0F             |and ebx,0F
> 004024B7   |.  0AC2                |or al,dl                            ;  前面两个加起来
> 004024B9   |.  32441C 0C           |xor al,byte ptr ss:\[esp+ebx+C\]      ;  与卡号的 md5 值 xor(异或)
> 004024BD   |.  88440C 1C           |mov byte ptr ss:\[esp+ecx+1C\],al     ;  保存
> 004024C1   |.  41                  |inc ecx                             ;  指针后移
> 004024C2   |.  3BCF                |cmp ecx,edi                         ;  是否结束
> 004024C4   |.^ 7C BF               \\jl short Dot1xCli.00402485          ;  没结束继续
> 004024C6   |.  5B                  pop ebx
> 004024C7   |>  8D4424 18           lea eax,dword ptr ss:\[esp+18\]
> 004024CB   |.  C6440C 18 00        mov byte ptr ss:\[esp+ecx+18\],0       ;  结尾加'\\0'★这里就是密码
> 004024D0   |.  8BD6                mov edx,esi
> 004024D2   |.  8BC8                mov ecx,eax
> 004024D4   |.  5F                  pop edi
> 004024D5   |.  2BD1                sub edx,ecx
> 004024D7   |.  5E                  pop esi
> 004024D8   |>  8A08                /mov cl,byte ptr ds:\[eax\]
> 004024DA   |.  880C02              |mov byte ptr ds:\[edx+eax\],cl
> 004024DD   |.  40                  |inc eax
> 004024DE   |.  84C9                |test cl,cl
> 004024E0   |.^ 75 F6               \\jnz short Dot1xCli.004024D8
> 004024E2   |.  B8 01000000         mov eax,1
> 004024E7   |.  81C4 A8000000       add esp,0A8
> 004024ED   \\.  C3                  retn
> 
> * * *
> 
> 
> ```

  
密码每 2 位为一组，如果>39 就-57，否则-30.转换成数字，或字母。然后与卡号的 MD5 值异或，得到密码。安全起见，程序就不写了。  
如此简单的加密，而且不加壳，可见联创公司对我们上网用户的态度！  
  
BY 海风月影  
Greets to you all! 

 4dd466d0e1e0cc123afbe504da2462ca