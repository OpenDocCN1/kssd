
```
65E9C860 ; int __cdecl sub_65E9C860(LPCSTR lpString,int file,int,int,int,int)
.text:65E9C860 sub_65E9C860    proc near               ; CODE XREF: IIERPCtl_Import+D1p
.text:65E9C860
.text:65E9C860 var_3C          = dword ptr -3Ch
.text:65E9C860 var_38          = dword ptr -38h
.text:65E9C860 var_34          = dword ptr -34h
.text:65E9C860 var_30          = dword ptr -30h
.text:65E9C860 var_2C          = dword ptr -2Ch
.text:65E9C860 var_28          = dword ptr -28h
.text:65E9C860 var_24          = dword ptr -24h
.text:65E9C860 var_20          = dword ptr -20h
.text:65E9C860 var_1C          = dword ptr -1Ch
.text:65E9C860 var_18          = dword ptr -18h
.text:65E9C860 var_10          = dword ptr -10h
.text:65E9C860 hWnd            = dword ptr -0Ch
.text:65E9C860 var_8           = dword ptr -8
.text:65E9C860 var_4           = dword ptr -4
.text:65E9C860 lpString        = dword ptr  8
.text:65E9C860 file            = dword ptr  0Ch
.text:65E9C860 arg_8           = dword ptr  10h
.text:65E9C860 arg_C           = dword ptr  14h
.text:65E9C860 arg_10          = dword ptr  18h
.text:65E9C860 arg_14          = dword ptr  1Ch
.text:65E9C860
.text:65E9C860                 push    ebp
.text:65E9C861                 mov     ebp, esp
.text:65E9C863                 sub     esp, 3Ch
.text:65E9C866                 lea     eax, [ebp+var_8]
.text:65E9C869                 push    esi
.text:65E9C86A                 xor     esi, esi
.text:65E9C86C                 push    eax             ; int
.text:65E9C86D                 push    1               ; hWnd
.text:65E9C86F                 mov     [ebp+var_8], esi
.text:65E9C872                 call    sub_65E9DE90		; 检测 realplayer 运行中吗，如果则将其起动
.text:65E9C872
								....
.text:65E9C8CE
.text:65E9C8D3
.text:65E9C8D3 loc_65E9C8D3:                           ; CODE XREF: sub_65E9C860+7Dj
.text:65E9C8D3                 mov     edx, [eax]
.text:65E9C8D5                 add     eax, 8
.text:65E9C8D8                 dec     ecx
.text:65E9C8D9                 lea     esi, [esi+edx+7]
.text:65E9C8DD                 jnz     short loc_65E9C8D3
.text:65E9C8DD
.text:65E9C8DF                 push    ebx
.text:65E9C8E0                 push    edi
.text:65E9C8E1                 lea     edi, [esi+0Bh]
.text:65E9C8E4                 push    edi
.text:65E9C8E5                 call    operator new(uint)
.text:65E9C8E5
.text:65E9C8EA                 mov     ecx, edi
.text:65E9C8EC                 mov     edx, eax
.text:65E9C8EE                 mov     ebx, ecx
.text:65E9C8F0                 xor     eax, eax
.text:65E9C8F2                 mov     edi, edx
.text:65E9C8F4                 add     esp, 4
.text:65E9C8F7                 shr     ecx, 2
.text:65E9C8FA                 rep stosd
.text:65E9C8FC                 mov     ecx, ebx
.text:65E9C8FE                 mov     [ebp+var_10], edx
.text:65E9C901                 and     ecx, 3
.text:65E9C904                 mov     [ebp+lpString], 5
.text:65E9C90B                 rep stosb
.text:65E9C90D                 mov     dword ptr [edx], 5
.text:65E9C913                 mov     [edx+4], esi
.text:65E9C916                 lea     eax, [edx+8]
.text:65E9C919                 lea     edx, [ebp+var_38]
.text:65E9C919
.text:65E9C91C
.text:65E9C91C loc_65E9C91C:                           ; CODE XREF: sub_65E9C860+E4j
.text:65E9C91C                 mov     ecx, [edx]
.text:65E9C91E                 mov     esi, [edx-4]
.text:65E9C921                 mov     [eax], ecx
.text:65E9C923                 mov     ebx, ecx
.text:65E9C925                 lea     edi, [eax+4]
.text:65E9C928                 add     edx, 8
.text:65E9C92B                 shr     ecx, 2
.text:65E9C92E                 rep movsd
.text:65E9C930                 mov     ecx, ebx
.text:65E9C932                 and     ecx, 3
.text:65E9C935                 rep movsb
.text:65E9C937                 mov     ecx, [eax]
.text:65E9C939                 lea     eax, [eax+ecx+4]
.text:65E9C93D                 mov     ecx, [ebp+lpString]
.text:65E9C940                 dec     ecx
.text:65E9C941                 mov     [ebp+lpString], ecx
.text:65E9C944                 jnz     short loc_65E9C91C
.text:65E9C944
.text:65E9C946                 mov     ebx, [ebp+var_10]
.text:65E9C949                 mov     edi, [ebp+hWnd]
.text:65E9C94C                 push    ebx             ; int
.text:65E9C94D                 push    edi             ; hWnd
.text:65E9C94E                 call    sub_65E9E060		;!!此处很重要了，这个进程将通过 WM_COPYDATA 向 realplayer 传送数据，包括 playlist
.text:65E9C94E
					...
.text:65E9C9A0                 mov     eax, esi
.text:65E9C9A2                 pop     edi
.text:65E9C9A3                 pop     ebx
.text:65E9C9A4                 pop     esi
.text:65E9C9A5                 mov     esp, ebp
.text:65E9C9A7                 pop     ebp
.text:65E9C9A8                 retn
.text:65E9C9A8
```

```
:65E9E060 ; int __cdecl sub_65E9E060(HWND hWnd,int)
.text:65E9E060 sub_65E9E060    proc near               ; CODE XREF: sub_65E9C4C0+10Ap
.text:65E9E060                                         ; sub_65E9C5F0+101p
.text:65E9E060                                         ; sub_65E9C720+2Ap
.text:65E9E060                                         ; sub_65E9C770+C3p
.text:65E9E060                                         ; sub_65E9C860+EEp
.text:65E9E060                                         ; sub_65E9C9B0+78p
.text:65E9E060
.text:65E9E060 lParam          = dword ptr -0Ch
.text:65E9E060 var_8           = dword ptr -8
.text:65E9E060 var_4           = dword ptr -4
.text:65E9E060 hWnd            = dword ptr  8
.text:65E9E060 arg_4           = dword ptr  0Ch
.text:65E9E060
.text:65E9E060                 push    ebp
.text:65E9E061                 mov     ebp, esp
.text:65E9E063                 sub     esp, 0Ch
.text:65E9E066                 xor     eax, eax
.text:65E9E068                 lea     edx, [ebp+lParam]
.text:65E9E06B                 mov     [ebp+var_8], eax
.text:65E9E06E                 push    edx             ; lParam
.text:65E9E06F                 mov     [ebp+var_4], eax
.text:65E9E072                 mov     eax, [ebp+arg_4]
.text:65E9E075                 mov     [ebp+var_4], eax
.text:65E9E078                 push    0               ; wParam
.text:65E9E07A                 mov     ecx, [eax+4]
.text:65E9E07D                 mov     eax, [ebp+hWnd]
.text:65E9E080                 add     ecx, 0Bh
.text:65E9E083                 push    WM_COPYDATA     ; Msg
.text:65E9E085                 push    eax             ; hWnd
.text:65E9E086                 mov     [ebp+lParam], 0
.text:65E9E08D                 mov     [ebp+var_8], ecx
.text:65E9E090                 call    ds:SendMessageA		; 发生消息给 realplayer, 将数据传给他
.text:65E9E096                 neg     eax
.text:65E9E098                 sbb     eax, eax
.text:65E9E09A                 neg     eax
.text:65E9E09C                 mov     esp, ebp
.text:65E9E09E                 pop     ebp
.text:65E9E09F                 retn
```

下面转入 `plugins\MPAMedia.dll`，这里会处理 `rpplugins\ierpplug.dll` 传过来的数据
