
# 汇编片段

```
77D196BD    6A 01           push    1
77D196BF    FF75 08         push    dword ptr [ebp+8]
77D196C2    E8 2AF2FFFF     call    77D188F1
77D196C7    5D              pop     ebp
77D196C8    C2 0400         retn    4
```

## 堆栈内容

```
0012FE50   0042D85E  /CALL 到 DispatchMessageA 来自 pwddlgmo.0042D858
0012FE54   0012FEDC  \pMsg = WM_LBUTTONUP hw = 120616 ("OK") Keys = 0 X = 55. Y = 17.
```

可以看到`DispatchMessageA`发送量`WM_LBUTTONUP`，句柄`hw = 120616 ("OK")`即`ok`按钮这个句柄值不固定，什么原因我就不说了。

## 消息结构体查看

有兴趣可以看看`msg`结构在内存中的情况怎么看呢

```
0012FE54   0012FEDC  \pMsg = WM_LBUTTONUP hw = 120616 ("OK") Keys = 0 X = 55. Y = 17.
```

`0012FEDC`即`msg`结构在内存中得首地址

`Dd 0012FEDC`

》》图7

![](https://bbs.kanxue.com/upload/attach/201004/271100_fy8mib34ucf4j65.jpg)

```
0012FEDC  00120616     .  --》hwnd===120616
0012FEE0  00000202    ..  --》message==202= WM_LBUTTONUP
0012FEE4  00000000  ....    --》wParam
0012FEE8  00110037  7. .  --》lParam
0012FEEC  01AEB9FD  ?  --》time
0012FEF0  000001E0  ?..    --》POINT
```

## 断点设置

4要返回代码

`Alt+m`对`00400000`到`00498000`下`F2`再按`F9`

断在`42c2f2`

`Jmp 42d8f0`

`0042D8F0` 即消息处理函数的

后面我就不多说了

## 消息处理函数

```
0042D8F0  /> \55            push    ebp                                                          ;  winmain
0042D8F1  |.  8BEC          mov     ebp, esp
0042D8F3  |.  83EC 6C       sub     esp, 6C
0042D8F6  |.  A1 10304900   mov     eax, dword ptr [493010]
0042D8FB  |.  33C5          xor     eax, ebp
0042D8FD  |.  8945 FC       mov     dword ptr [ebp-4], eax
0042D900  |.  53            push    ebx
0042D901  |.  56            push    esi
0042D902  |.  57            push    edi
0042D903  |.  C745 F8 FFFFF>mov     dword ptr [ebp-8], -1
0042D90A  |.  8B45 0C       mov     eax, dword ptr [ebp+C]
0042D90D  |.  8945 94       mov     dword ptr [ebp-6C], eax
0042D910  |.  817D 94 11010>cmp     dword ptr [ebp-6C], 111
0042D917  |.  74 05         je      short 0042D91E
0042D919  |.  E9 AE000000   jmp     0042D9CC
0042D91E  |>  8B45 10       mov     eax, dword ptr [ebp+10]
0042D921  |.  25 FFFF0000   and     eax, 0FFFF
0042D926  |.  0FB7C8        movzx   ecx, ax
0042D929  |.  894D F8       mov     dword ptr [ebp-8], ecx
0042D92C  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
0042D92F  |.  8945 94       mov     dword ptr [ebp-6C], eax
0042D932  |.  817D 94 8B270>cmp     dword ptr [ebp-6C], 278B
0042D939  |.  74 0E         je      short 0042D949
0042D93B  |.  817D 94 D8280>cmp     dword ptr [ebp-6C], 28D8
0042D942  |.  74 7E         je      short 0042D9C2
0042D944  |.  E9 81000000   jmp     0042D9CA
0042D949  |>  6A 20         push    20
0042D94B  |.  6A 00         push    0
0042D94D  |.  8D45 D8       lea     eax, dword ptr [ebp-28]
0042D950  |.  50            push    eax
0042D951  |.  E8 F5DBFFFF   call    0042B54B
0042D956  |.  83C4 0C       add     esp, 0C
0042D959  |.  6A 20         push    20                                                           ; /Count = 20 (32.)
0042D95B  |.  8D45 D8       lea     eax, dword ptr [ebp-28]                                      ; |
0042D95E  |.  50            push    eax                                                          ; |Buffer
0042D95F  |.  68 252A0000   push    2A25                                                         ; |/ControlID = 2A25 (10789.)
0042D964  |.  8B4D 08       mov     ecx, dword ptr [ebp+8]                                       ; ||
0042D967  |.  51            push    ecx                                                          ; ||hWnd
0042D968  |.  FF15 84744900 call    dword ptr [<&USER32.GetDlgItem>]                             ; |\GetDlgItem
0042D96E  |.  50            push    eax                                                          ; |hWnd
0042D96F  |.  FF15 88744900 call    dword ptr [<&USER32.GetWindowTextA>]                         ; \GetWindowTextA
0042D975  |.  68 EC3D4800   push    00483DEC                                                     ;  ASCII "123456"
0042D97A  |.  8D45 D8       lea     eax, dword ptr [ebp-28]
0042D97D  |.  50            push    eax
0042D97E  |.  E8 B7DDFFFF   call    0042B73A
0042D983  |.  83C4 08       add     esp, 8
0042D986  |.  85C0          test    eax, eax
0042D988  |.  74 20         je      short 0042D9AA
0042D98A  |.  6A 10         push    10                                                           ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
0042D98C  |.  68 E03D4800   push    00483DE0                                                     ; |Title = "Password"
0042D991  |.  68 C43D4800   push    00483DC4                                                     ; |Text = "Sorry! Wrong password."
0042D996  |.  8B45 08       mov     eax, dword ptr [ebp+8]                                       ; |
0042D999  |.  50            push    eax                                                          ; |hOwner
0042D99A  |.  FF15 8C744900 call    dword ptr [<&USER32.MessageBoxA>]                            ; \MessageBoxA
0042D9A0  |.  90            nop
0042D9A1  |.  00FF          add     bh, bh
0042D9A3  |.  15 90744900   adc     eax, <&USER32.PostQuitMessage>
0042D9A8  |.  EB 16         jmp     short 0042D9C0
0042D9AA  |>  6A 00         push    0                                                            ; /Style = MB_OK|MB_APPLMODAL
0042D9AC  |.  68 E03D4800   push    00483DE0                                                     ; |Title = "Password"
0042D9B1  |.  68 B03D4800   push    00483DB0                                                     ; |Text = "Right password."
0042D9B6  |.  8B45 08       mov     eax, dword ptr [ebp+8]                                       ; |
0042D9B9  |.  50            push    eax                                                          ; |hOwner
0042D9BA  |.  FF15 8C744900 call    dword ptr [<&USER32.MessageBoxA>]                            ; \MessageBoxA
0042D9C0  |>  EB 08         jmp     short 0042D9CA
0042D9C2  |>  6A 00         push    0                                                            ; /ExitCode = 0
0042D9C4  |.  FF15 90744900 call    dword ptr [<&USER32.PostQuitMessage>]                        ; \PostQuitMessage
0042D9CA  |>  EB 18         jmp     short 0042D9E4
0042D9CC  |>  8B45 14       mov     eax, dword ptr [ebp+14]
0042D9CF  |.  50            push    eax                                                          ; /lParam
0042D9D0  |.  8B4D 10       mov     ecx, dword ptr [ebp+10]                                      ; |
0042D9D3  |.  51            push    ecx                                                          ; |wParam
0042D9D4  |.  8B55 0C       mov     edx, dword ptr [ebp+C]                                       ; |
0042D9D7  |.  52            push    edx                                                          ; |Message
0042D9D8  |.  8B45 08       mov     eax, dword ptr [ebp+8]                                       ; |
0042D9DB  |.  50            push    eax                                                          ; |hWnd
0042D9DC  |.  FF15 94744900 call    dword ptr [<&USER32.DefWindowProcA>]                         ; \DefWindowProcA
0042D9E2  |.  EB 02         jmp     short 0042D9E6
0042D9E4  |>  33C0          xor     eax, eax
0042D9E6  |>  5F            pop     edi
0042D9E7  |.  5E            pop     esi
0042D9E8  |.  5B            pop     ebx
0042D9E9  |.  8B4D FC       mov     ecx, dword ptr [ebp-4]
0042D9EC  |.  33CD          xor     ecx, ebp
0042D9EE  |.  E8 C5D7FFFF   call    0042B1B8
0042D9F3  |.  8BE5          mov     esp, ebp
0042D9F5  |.  5D            pop     ebp
0042D9F6  \.  C2 1000       retn    10
```

## 文档

`doc`文档

[`dispatchMessageA log2.doc`](https://bbs.kanxue.com/attach-download-41208.htm)

---

# 欺骗消息过程

前面已经定位到`msg`的位置，只要我们在 `DispatchMessage`前重写`msg` 结构体，比如将`code`变为`WM_CLOSE`，

调整

```
tagMSG          struc ; (sizeof=0x1C)
00000000 hwnd            dd ?                    ; offset
00000004 message         dd ?
00000008 wParam          dd ?
0000000C lParam          dd ?
00000010 time            dd ?
```
