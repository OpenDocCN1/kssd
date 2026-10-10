

# 热血江湖 2.0 数据发包函数分析

> 原文：[`www.pediy.com/kssd/pediy10/62894.html`](https://www.pediy.com/kssd/pediy10/62894.html)

热血江湖刚刚更新到 2.0 了,今天分析了一下数据发包函数.更多的慢慢分析.  
因为这些函数包里没有做太多游戏影响游戏安全的东西,此函数里也没做什么事,所以我把分析结果发布出来,提大家参考一下.  
如有错误之处,请指出.\[2008-04-11\]  
  
  
  
数据包大小:61  
HEX 数据:  
AA 55 37 00 01 11 03 CC 2D 0D 42 49 AC BA 5F 48 FA 7D 92 A1 4F CB E4 F4 62 04 BB 8B 9E AC 46 59 89 2C DE 08 10 DC EA A7 85 22 42 88 9A 5F 98 BF CE C2 4B 4B 7E 77 FF 39   
  
FC 0A 00 55 AA  
ASCII 数据:  
.U7.....-.BI..\_H.}..O...b.....FY.,......."B..\_....KK~w.9...U.  
  
  
004354F0  /$  55            push    ebp  
004354F1  |.  8BEC          mov     ebp, esp  
004354F3  |.  B8 10240000   mov     eax, 2410                                   ;  EAX=0x2410  
004354F8  |.  E8 63873000   call    Client.0073DC60  
004354FD  |.  8B41 10       mov     eax, dword ptr \[ecx+10\]  
00435500  |.  56            push    esi  
00435501  |.  83F8 FF       cmp     eax, -1  
00435504  |.  57            push    edi  
00435505  |.  894D F8       mov     \[local.2\], ecx  
00435508  |.  0F84 68010000 je      Client.00435676                             ;  eax=\[ecx+0x10\]=-1 OVER  
0043550E  |.  8079 14 01    cmp     byte ptr \[ecx+14\], 1  
00435512  |.  0F85 5E010000 jnz     Client.00435676                             ;  \[ecx+0x14\]=1 OVER  
00435518  |.  8B75 08       mov     esi, \[arg.1\]                                ;  esi=arg.1  
0043551B  |.  66:8B06       mov     ax, word ptr \[esi\]  
0043551E  |.  66:3D 409C    cmp     ax, 9C40                                    ;  0x9C40  
00435522  |.  72 06         jb      short Client.0043552A  
00435524  |.  66:3D 50C3    cmp     ax, 0C350                                   ;  0x0C350  
00435528  |.  76 09         jbe     short Client.00435533                       ;  <=0xC350  
0043552A  |>  66:A1 0067520>mov     ax, word ptr \[4526700\]                      ;  如果前两位<0x9C40,ax=\[0x4526700\]  
00435530  |.  66:8906       mov     word ptr \[esi\], ax                          ;  也就是说 0x9C40<x<0x0C350 时，x 不变  
00435533  |>  53            push    ebx  
00435534  |.  8B5D 0C       mov     ebx, \[arg.2\]                                ;  ebx=arg.2  
00435537  |.  B9 00080000   mov     ecx, 800  
0043553C  |.  33C0          xor     eax, eax                                    ;  eax=0  
0043553E  |.  8DBD F0DBFFFF lea     edi, \[local.2308\]                           ;  edi=local.2308  
00435544  |.  C745 FC 00000>mov     \[local.1\], 0                                ;  local.1=0  
0043554B  |.  F3:AB         rep     stos dword ptr es:\[edi\]                     ;  copy 0x800/4 DWORD; 用０填  
0043554D  |.  0FBF4E 02     movsx   ecx, word ptr \[esi+2\]                       ;  扩展 copy  
00435551  |.  8D43 09       lea     eax, dword ptr \[ebx+9\]  
00435554  |.  C685 F0DBFFFF>mov     byte ptr \[ebp-2410\], 0AA                    ;  发送包头，0xAA,0x55  
0043555B  |.  8885 F2DBFFFF mov     byte ptr \[ebp-240E\], al  
00435561  |.  88A5 F3DBFFFF mov     byte ptr \[ebp-240D\], ah                     ;  0xAA,0x55,al,ah  
00435567  |.  A1 18675204   mov     eax, dword ptr \[4526718\]  
0043556C  |.  C685 F1DBFFFF>mov     byte ptr \[ebp-240F\], 55  
00435573  |.  50            push    eax                                         ;  eax=\[0x4526718\]  
00435574  |.  51            push    ecx                                         ;  ecx=command=\[arg.1+0x2\]  
00435575  |.  68 B8B78600   push    Client.0086B7B8                             ;  ASCII "Send() command:%X, encry:%d"  
0043557A  |.  8885 F4DBFFFF mov     byte ptr \[ebp-240C\], al  
00435580  |.  E8 FB0F0100   call    <Client.直接返回,无返回值>                          ;  应该是记录发送什么指令,游戏调试时用的  
00435585  |.  8BCB          mov     ecx, ebx                                    ;  ecx=arg.2  
00435587  |.  8DBD F5DBFFFF lea     edi, dword ptr \[ebp-240B\]                   ;  edi=\[addr+0x4\]  
0043558D  |.  8BD1          mov     edx, ecx                                    ;  edx=ecx=arg.2   临时把数据包大小存入 edx  
0043558F  |.  33C0          xor     eax, eax                                    ;  eax=0  
00435591  |.  C1E9 02       shr     ecx, 2                                      ;  ecx>>=2   ecx/2/2  
00435594  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword ptr \[esi\]    ;  edi<=esi, esi=arg.1,  在这里猜想,arg.1 是发送数据,arg.2 是数据长度  
00435596  |.  8BCA          mov     ecx, edx                                    ;  ecx=edx=arg.2 恢复数据包大小到 ecx  
00435598  |.  83C4 0C       add     esp, 0C                                     ;  不知道做什么<=============  
0043559B  |.  83E1 03       and     ecx, 3                                      ;  ecx &= 3  
0043559E  |.  F3:A4         rep     movs byte ptr es:\[edi\], byte ptr \[esi\]      ;  又来....这是做什么啊.不是 copy 过了嘛,, 注意,这里是 byte,上面是 dword  
004355A0  |.  8D73 05       lea     esi, dword ptr \[ebx+5\]                      ;  esi=&\[arg.2+0x5\]  
004355A3  |.  8D8D F0DBFFFF lea     ecx, \[local.2308\]                           ;  ecx=&local.2308  
004355A9  |.  68 00200000   push    2000  
004355AE  |.  51            push    ecx  
004355AF  |.  898435 F0DBFF>mov     dword ptr \[ebp+esi-2410\], eax               ;  \[&数据包-0x0A\] = 0  
004355B6  |.  6A 02         push    2  
004355B8  |.  898435 F4DBFF>mov     dword ptr \[ebp+esi-240C\], eax               ;  \[&数据包-0x06\] = 0  
004355BF  |.  83C6 08       add     esi, 8  
004355C2  |.  C68435 F0DBFF>mov     byte ptr \[ebp+esi-2410\], 55  
004355CA  |.  46            inc     esi  
004355CB  |.  C68435 F0DBFF>mov     byte ptr \[ebp+esi-2410\], 0AA                ;  0x55,0xAA 结尾  
004355D3  |.  E8 08DA3100   call    <Client.没做什么,直接反回 0>                         ;  应该是记录发送什么数据,游戏调试时用的  
004355D8  |.  85C0          test    eax, eax                                    ;  CALL(0x2,&local.2308,0x2000);  
004355DA  |.  5B            pop     ebx  
004355DB  |.  74 4B         je      short Client.00435628                       ;  记录失败就报错.这里始终会跳.  
004355DD  |.  8B55 08       mov     edx, \[arg.1\]  
004355E0  |.  B9 00010000   mov     ecx, 100  
004355E5  |.  33C0          xor     eax, eax  
004355E7  |.  8DBD F0FBFFFF lea     edi, \[local.260\]  
004355ED  |.  F3:AB         rep     stos dword ptr es:\[edi\]  
004355EF  |.  0FBF42 02     movsx   eax, word ptr \[edx+2\]  
004355F3  |.  50            push    eax  
004355F4  |.  8D8D F0FBFFFF lea     ecx, \[local.260\]  
004355FA  |.  68 98B78600   push    Client.0086B798                             ;  ASCII "\_17Encryption Error: command %d"  
004355FF  |.  51            push    ecx  
00435600  |.  E8 F57E3000   call    Client.0073D4FA  
00435605  |.  83C4 0C       add     esp, 0C  
00435608  |.  8D95 F0FBFFFF lea     edx, \[local.260\]  
0043560E  |.  6A 00         push    0                                           ; /Style = MB\_OK|MB\_APPLMODAL  
00435610  |.  6A 00         push    0                                           ; |Title = NULL  
00435612  |.  52            push    edx                                         ; |Text  
00435613  |.  FF15 CC137A00 call    dword ptr \[<&USER32.GetActiveWindow>\]       ; |\[GetActiveWindow  
00435619  |.  50            push    eax                                         ; |hOwner  
0043561A  |.  FF15 C8137A00 call    dword ptr \[<&USER32.MessageBoxA>\]           ; \\MessageBoxA  
00435620  |.  5F            pop     edi  
00435621  |.  5E            pop     esi  
00435622  |.  8BE5          mov     esp, ebp  
00435624  |.  5D            pop     ebp  
00435625  |.  C2 0800       retn    8  
00435628  |>  8D85 F0DBFFFF lea     eax, \[local.2308\]  
0043562E  |.  6A 00         push    0                                           ; /Callback = NULL  
00435630  |.  8945 F4       mov     \[local.3\], eax                              ; |  
00435633  |.  8B45 F8       mov     eax, \[local.2\]                              ; |  
00435636  |.  6A 00         push    0                                           ; |pOverlapped = NULL  
00435638  |.  8D4D FC       lea     ecx, \[local.1\]                              ; |local.1 是数据大小  
0043563B  |.  6A 00         push    0                                           ; |Flags = 0  
0043563D  |.  51            push    ecx                                         ; |pBytesSent  
0043563E  |.  8B48 10       mov     ecx, dword ptr \[eax+10\]                     ; |  
00435641  |.  8D55 F0       lea     edx, \[local.4\]                              ; |local.4  是发送的数据包  
00435644  |.  6A 01         push    1                                           ; |nBuffers = 1  
00435646  |.  46            inc     esi                                         ; |  
00435647  |.  52            push    edx                                         ; |pBuffers  
00435648  |.  51            push    ecx                                         ; |Socket  
00435649  |.  8975 F0       mov     \[local.4\], esi                              ; |  
0043564C  |.  FF15 50157A00 call    dword ptr \[<&WS2\_32.WSASend>\]               ; \\WSASend  
00435652  |.  8D95 F0DBFFFF lea     edx, \[local.2308\]  
00435658  |.  68 00200000   push    2000  
0043565D  |.  52            push    edx  
0043565E  |.  6A 03         push    3  
00435660  |.  8BF0          mov     esi, eax  
00435662  |.  E8 79D93100   call    <Client.没做什么,直接反回 0>                         ;  记录发送了什么数据  
00435667  |.  83FE FF       cmp     esi, -1  
0043566A  |.  74 04         je      short Client.00435670  
0043566C  |.  85F6          test    esi, esi  
0043566E  |.  74 06         je      short Client.00435676  
00435670  |>  FF15 54157A00 call    dword ptr \[<&WS2\_32.#111>\]                  ; \[WSAGetLastError  
00435676  |>  5F            pop     edi  
00435677  |.  5E            pop     esi  
00435678  |.  8BE5          mov     esp, ebp  
0043567A  |.  5D            pop     ebp  
0043567B  \\.  C2 0800       retn    8