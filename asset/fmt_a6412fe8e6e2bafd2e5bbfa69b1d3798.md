
```
00435636  |.  6A 00         push    0                              ; |pOverlapped = NULL

00435638  |.  8D4D FC      lea     ecx, [local.1]                  ; |local.1 是数据大小

0043563B  |.  6A 00        push    0                              ; |Flags = 0

0043563D  |.  51          push    ecx                            ; |pBytesSent

0043563E  |.  8B48 10     mov     ecx, dword ptr [eax+10]         ; |

00435641  |.  8D55 F0     lea     edx, [local.4]                  ; |local.4  是发送的数据包

00435644  |.  6A 01       push    1                              ; |nBuffers = 1

00435646  |.  46          inc     esi                            ; |

00435647  |.  52          push    edx                            ; |pBuffers

00435648  |.  51          push    ecx                            ; |Socket

00435649  |.  8975 F0     mov     [local.4], esi                 ; |

0043564C  |.  FF15 50157A00 call    dword ptr [<&WS2_32.WSASend>]   ; \WSASend

00435652  |.  8D95 F0DBFFFF lea     edx, [local.2308]

00435658  |.  68 00200000  push    2000

0043565D  |.  52          push    edx

0043565E  |.  6A 03       push    3

00435660  |.  8BF0        mov     esi, eax

00435662  |.  E8 79D93100  call    <Client.没做什么,直接反回 0>          ;  记录发送了什么数据

00435667  |.  83FE FF     cmp     esi, -1

0043566A  |.  74 04       je      short Client.00435670

0043566C  |.  85F6        test    esi, esi

0043566E  |.  74 06       je      short Client.00435676

00435670  |>  FF15 54157A00 call    dword ptr [<&WS2_32.#111>]     ; [WSAGetLastError

00435676  |>  5F          pop     edi

00435677  |.  5E          pop     esi

00435678  |.  8BE5        mov     esp, ebp

0043567A  |.  5D          pop     ebp

0043567B  \.  C2 0800     ret     8
```
