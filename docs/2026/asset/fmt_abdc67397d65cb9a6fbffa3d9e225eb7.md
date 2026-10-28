
```text
.text:00402220     ; int __cdecl sub_402220(HWND hWnd)

.text:00402220     sub_402220     proc near               ; CODE XREF: ChildWndProc+4Dp

.text:00402220

.text:00402220     MultiByteStr    = byte ptr -24Ch

.text:00402220     Str             = byte ptr -148h

.text:00402220     var_147         = byte ptr -147h

.text:00402220     hLibModule      = dword ptr -44h

.text:00402220     nCount          = dword ptr -40h

.text:00402220     var_3C          = dword ptr -3Ch

.text:00402220     lpWideCharStr   = dword ptr -38h

.text:00402220     pvarg           = VARIANTARG ptr -34h

.text:00402220     p               = dword ptr -24h

.text:00402220     This            = dword ptr -20h

.text:00402220     nALL            = dword ptr -1Ch

.text:00402220     dwResult        = dword ptr -18h

.text:00402220     var_14          = dword ptr -14h

.text:00402220     ppvObject       = dword ptr -10h

.text:00402220     var_C           = dword ptr -0Ch

.text:00402220     var_4           = dword ptr -4

.text:00402220     pdisp           = dword ptr  8

.text:00402220

.text:00402220 000                 push    ebp

.text:00402221 004                 mov     ebp, esp

.text:00402223 004                 push    0FFFFFFFFh

.text:00402225 008                 push    offset SEH_402220

.text:0040222A 00C                 mov     eax, large fs:0

.text:00402230 00C                 push    eax

.text:00402231 010                 mov     large fs:0, esp

.text:00402238 010                 sub     esp, 240h

.text:0040223E 250                 push    ebx

.text:0040223F 254                 push    esi

.text:00402240 258                 push    edi

.text:00402241 25C                 push    0               ; pvReserved

.text:00402243 260                 call    ds:CoInitialize

.text:00402249 25C                 push    offset aD_0     ; "?

.text:0040224E 260                 call    ds:LoadLibraryA

.text:00402254 25C                 test    eax, eax

.text:00402256 25C                 mov     [ebp+hLibModule], eax

.text:00402259 25C                 jnz     short loc_402271

.text:0040225B 25C                 mov     eax, 1

.text:00402260 25C                 mov     ecx, [ebp+var_C]

.text:00402263 25C                 mov     large fs:0, ecx

.text:0040226A 25C                 pop     edi

.text:0040226B 258                 pop     esi

.text:0040226C 254                 pop     ebx

.text:0040226D 250                 mov     esp, ebp

.text:0040226F 004                 pop     ebp

.text:00402270 000                 retn

.text:00402271     ; ---------------------------------------------------------------------------

.text:00402271

.text:00402271     loc_402271:                             ; CODE XREF: sub_402220+39j

.text:00402271 25C                 push    offset String   ; "?

.text:00402276 260                 call    ds:RegisterWindowMessageA

.text:0040227C 25C                 mov     ecx, eax

.text:0040227E 25C                 push    bx

.text:00402280 25E                 push    dx

.text:00402282 260                 push    eax

.text:00402283 264                 cmp     dh, 43h

.text:00402286 264                 dec     bl

.text:00402288 264                 dec     bl

.text:0040228A 264                 or      bl, dh

.text:0040228C 264                 cmp     bl, 68h

.text:0040228F 264                 add     bl, 3Ch

.text:00402292 264                 dec     dl

.text:00402294 264                 dec     bh

.text:00402296 264                 cmp     eax, 6E32h

.text:0040229B 264                 pop     eax

.text:0040229C 260                 pop     dx

.text:0040229E 25E                 pop     bx

.text:004022A0 25C                 lea     eax, [ebp+dwResult]

.text:004022A3 25C                 push    eax             ; lpdwResult

.text:004022A4 260                 push    3E8h            ; uTimeout

.text:004022A9 264                 push    2               ; fuFlags

.text:004022AB 268                 push    0               ; lParam

.text:004022AD 26C                 push    0               ; wParam

.text:004022AF 270                 push    ecx             ; Msg

.text:004022B0 274                 mov     ecx, [ebp+pdisp]

.text:004022B3 274                 push    ecx             ; hWnd

.text:004022B4 278                 call    ds:SendMessageTimeoutA

.text:004022BA 25C                 push    bx

.text:004022BC 25E                 push    si

.text:004022BE 260                 push    edi

.text:004022BF 264                 or      edi, 3F3Ch

.text:004022C5 264                 and     si, cx

.text:004022C8 264                 pop     edi

.text:004022C9 260                 pop     si

.text:004022CB 25E                 pop     bx

.text:004022CD 25C                 mov     edx, [ebp+hLibModule]

.text:004022D0 25C                 push    offset a7r      ; "?R?

.text:004022D5 260                 push    edx             ; hModule

.text:004022D6 264                 call    ds:GetProcAddress

.text:004022DC 25C                 xor     ebx, ebx

.text:004022DE 25C                 cmp     eax, ebx

.text:004022E0 25C                 jnz     short loc_4022F8

.text:004022E2 25C                 mov     eax, 1

.text:004022E7 25C                 mov     ecx, [ebp+var_C]

.text:004022EA 25C                 mov     large fs:0, ecx

.text:004022F1 25C                 pop     edi

.text:004022F2 258                 pop     esi

.text:004022F3 254                 pop     ebx

.text:004022F4 250                 mov     esp, ebp

.text:004022F6 004                 pop     ebp

.text:004022F7 000                 retn

.text:004022F8     ; ---------------------------------------------------------------------------

.text:004022F8

.text:004022F8     loc_4022F8:                             ; CODE XREF: sub_402220+C0j

.text:004022F8 25C                 mov     [ebp+This], ebx

.text:004022FB 25C                 mov     edx, [ebp+dwResult]

.text:004022FE 25C                 lea     ecx, [ebp+This]

.text:00402301 25C                 push    ecx

.text:00402302 260                 push    ebx

.text:00402303 264                 push    offset CLSID_IHTMLDocument2

.text:00402308 268                 push    edx

.text:00402309 26C                 mov     [ebp+var_4], ebx

.text:0040230C 26C                 call    eax

.text:0040230E 25C                 mov     esi, eax

.text:00402310 25C                 mov     eax, [ebp+This]

.text:00402313 25C                 cmp     esi, ebx

.text:00402315 25C                 jl      loc_40269D

.text:0040231B 25C                 cmp     eax, ebx

.text:0040231D 25C                 jz      loc_40269D

.text:00402323 25C                 mov     [ebp+p], ebx

.text:00402326 25C                 mov     ecx, [eax]

.text:00402328 25C                 lea     edx, [ebp+p]

.text:0040232B 25C                 push    edx             ; p

.text:0040232C 260                 push    eax             ; This

.text:0040232D 264                 mov     byte ptr [ebp+var_4], 1

.text:00402331 264                 call    [ecx+IHTMLDocument2Vtbl.get_all]

.text:00402334 25C                 mov     esi, eax

.text:00402336 25C                 mov     eax, [ebp+p]

.text:00402339 25C                 cmp     esi, ebx

.text:0040233B 25C                 jl      loc_402694

.text:00402341 25C                 cmp     eax, ebx

.text:00402343 25C                 jz      loc_402694

.text:00402349 25C                 lea     edx, [ebp+nALL]

.text:0040234C 25C                 mov     [ebp+nALL], ebx

.text:0040234F 25C                 mov     ecx, [eax]

.text:00402351 25C                 push    edx             ; p

.text:00402352 260                 push    eax             ; This

.text:00402353 264                 call    [ecx+IHTMLDocument2Vtbl.get_body]

.text:00402356 25C                 mov     eax, [ebp+nALL]

.text:00402359 25C                 mov     [ebp+nCount], ebx

.text:0040235C 25C                 cmp     eax, ebx

.text:0040235E 25C                 jle     loc_40266E

.text:00402364

.text:00402364     loc_402364:                             ; CODE XREF: sub_402220+376j

.text:00402364 25C                 call    ds:rand

.text:0040236A 25C                 cdq

.text:0040236B 25C                 idiv    [ebp+nALL]

.text:0040236E 25C                 mov     word ptr [ebp+pvarg.anonymous_0], 3

.text:00402374 25C                 mov     eax, edx

.text:00402376 25C                 mov     dword ptr [ebp+pvarg.anonymous_0+8], eax

.text:00402379 25C                 mov     [ebp+pdisp], ebx

.text:0040237C 25C                 mov     [ebp+ppvObject], ebx

.text:0040237F 25C                 lea     edx, [ebp+pdisp]

.text:00402382 25C                 mov     esi, dword ptr [ebp+pvarg.anonymous_0+4]

.text:00402385 25C                 push    edx             ; pdisp

.text:00402386 260                 mov     edx, dword ptr [ebp+pvarg.anonymous_0]

.text:00402389 260                 sub     esp, 10h

.text:0040238C 270                 mov     ecx, [ebp+p]

.text:0040238F 270                 mov     ebx, esp

.text:00402391 270                 sub     esp, 10h

.text:00402394 280                 mov     edi, [ecx]

.text:00402396 280                 mov     byte ptr [ebp+var_4], 4

.text:0040239A 280                 mov     [ebx], edx

.text:0040239C 280                 mov     [ebx+4], esi

.text:0040239F 280                 mov     esi, dword ptr [ebp+pvarg.anonymous_0+0Ch]

.text:004023A2 280                 mov     [ebx+8], eax

.text:004023A5 280                 mov     [ebx+0Ch], esi

.text:004023A8 280                 mov     ebx, esp

.text:004023AA 280                 push    ecx             ; This

.text:004023AB 284                 mov     [ebx], edx

.text:004023AD 284                 mov     edx, dword ptr [ebp+pvarg.anonymous_0+4]

.text:004023B0 284                 mov     [ebx+4], edx

.text:004023B3 284                 mov     [ebx+8], eax

.text:004023B6 284                 mov     [ebx+0Ch], esi

.text:004023B9 284                 call    [edi+IHTMLElementCollectionVtbl.item]

.text:004023BC 25C                 mov     esi, eax

.text:004023BE 25C                 mov     eax, [ebp+pdisp]

.text:004023C1 25C                 xor     ebx, ebx

.text:004023C3 25C                 cmp     esi, ebx

.text:004023C5 25C                 jl      loc_402642

.text:004023CB 25C                 cmp     eax, ebx

.text:004023CD 25C                 jz      loc_402642

.text:004023D3 25C                 mov     ecx, [eax]

.text:004023D5 25C                 lea     edx, [ebp+ppvObject]

.text:004023D8 25C                 push    edx             ; ppvObject

.text:004023D9 260                 push    offset CLSID_IHTMLElement ; riid

.text:004023DE 264                 push    eax             ; This

.text:004023DF 268                 call    [ecx+IDispatchVtbl.QueryInterface]

.text:004023E1 25C                 mov     esi, eax

.text:004023E3 25C                 mov     eax, [ebp+ppvObject]

.text:004023E6 25C                 cmp     esi, ebx

.text:004023E8 25C                 jl      loc_40261D

.text:004023EE 25C                 cmp     eax, ebx

.text:004023F0 25C                 jz      loc_40261D

.text:004023F6 25C                 mov     ecx, [eax]

.text:004023F8 25C                 lea     edx, [ebp+lpWideCharStr]

.text:004023FB 25C                 push    edx             ; p

.text:004023FC 260                 push    eax             ; This

.text:004023FD 264                 call    [ecx+IHTMLDocument2Vtbl.get_forms]

.text:00402400 25C                 mov     esi, eax

.text:00402402 25C                 cmp     esi, ebx

.text:00402404 25C                 jl      loc_4025E4

.text:0040240A 25C                 mov     eax, [ebp+lpWideCharStr]

.text:0040240D 25C                 cmp     eax, ebx

.text:0040240F 25C                 jnz     short loc_40241E

.text:00402411 25C                 mov     eax, [ebp+ppvObject]

.text:00402414 25C                 mov     byte ptr [ebp+var_4], 3

.text:00402418 25C                 cmp     eax, ebx

.text:0040241A 25C                 jz      short loc_402467

.text:0040241C 25C                 jmp     short loc_402461

.text:0040241E     ; ---------------------------------------------------------------------------

.text:0040241E

.text:0040241E     loc_40241E:                             ; CODE XREF: sub_402220+1EFj

.text:0040241E 25C                 lea     ecx, [ebp+MultiByteStr]

.text:00402424 25C                 push    104h            ; cbMultiByte

.text:00402429 260                 push    ecx             ; lpMultiByteStr

.text:0040242A 264                 push    eax             ; lpWideCharStr

.text:0040242B 268                 call    W2A

.text:00402430 268                 mov     edx, [ebp+lpWideCharStr]

.text:00402433 268                 add     esp, 0Ch

.text:00402436 25C                 push    edx             ; bstrString

.text:00402437 260                 call    ds:SysFreeString

.text:0040243D 25C                 lea     eax, [ebp+MultiByteStr]

.text:00402443 25C                 push    offset Str2     ; "僂"

.text:00402448 260                 push    eax             ; Str1

.text:00402449 264                 call    ds:_stricmp

.text:0040244F 264                 add     esp, 8

.text:00402452 25C                 test    eax, eax

.text:00402454 25C                 jz      short loc_402481

.text:00402456 25C                 mov     eax, [ebp+ppvObject]

.text:00402459 25C                 mov     byte ptr [ebp+var_4], 3

.text:0040245D 25C                 cmp     eax, ebx

.text:0040245F 25C                 jz      short loc_402467

.text:00402461

.text:00402461     loc_402461:                             ; CODE XREF: sub_402220+1FCj

.text:00402461 25C                 mov     ecx, [eax]

.text:00402463 25C                 push    eax             ; This

.text:00402464 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:00402467

.text:00402467     loc_402467:                             ; CODE XREF: sub_402220+1FAj

.text:00402467                                             ; sub_402220+23Fj

.text:00402467 25C                 mov     eax, [ebp+pdisp]

.text:0040246A 25C                 mov     byte ptr [ebp+var_4], 2

.text:0040246E 25C                 cmp     eax, ebx

.text:00402470 25C                 jz      short loc_402478

.text:00402472 25C                 mov     edx, [eax]

.text:00402474 25C                 push    eax             ; This

.text:00402475 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:00402478

.text:00402478     loc_402478:                             ; CODE XREF: sub_402220+250j

.text:00402478 25C                 lea     eax, [ebp+pvarg]

.text:0040247B 25C                 push    eax

.text:0040247C 260                 jmp     loc_402584

.text:00402481     ; ---------------------------------------------------------------------------

.text:00402481

.text:00402481     loc_402481:                             ; CODE XREF: sub_402220+234j

.text:00402481 25C                 mov     [ebp+var_14], ebx

.text:00402484 25C                 mov     eax, [ebp+pdisp]

.text:00402487 25C                 lea     edx, [ebp+var_14]

.text:0040248A 25C                 push    edx             ; ppvObject

.text:0040248B 260                 push    offset CLSID_IHTMLAnchorElement ; riid

.text:00402490 264                 mov     ecx, [eax]

.text:00402492 264                 push    eax             ; This

.text:00402493 268                 mov     byte ptr [ebp+var_4], 5

.text:00402497 268                 call    [ecx+IDispatchVtbl.QueryInterface]

.text:00402499 25C                 mov     esi, eax

.text:0040249B 25C                 mov     eax, [ebp+var_14]

.text:0040249E 25C                 cmp     esi, ebx

.text:004024A0 25C                 jl      loc_4025F1

.text:004024A6 25C                 cmp     eax, ebx

.text:004024A8 25C                 jz      loc_4025F1

.text:004024AE 25C                 mov     ecx, [eax]

.text:004024B0 25C                 lea     edx, [ebp+var_3C]

.text:004024B3 25C                 push    edx             ; p

.text:004024B4 260                 push    eax             ; This

.text:004024B5 264                 call    [ecx+IHTMLAnchorElementVtbl.get_href]

.text:004024B8 25C                 mov     esi, eax

.text:004024BA 25C                 cmp     esi, ebx

.text:004024BC 25C                 jl      loc_4025A1

.text:004024C2 25C                 mov     edx, [ebp+var_3C]

.text:004024C5 25C                 cmp     edx, ebx

.text:004024C7 25C                 jnz     short loc_4024FC

.text:004024C9 25C                 mov     eax, [ebp+var_14]

.text:004024CC 25C                 mov     byte ptr [ebp+var_4], 4

.text:004024D0 25C                 cmp     eax, ebx

.text:004024D2 25C                 jz      short loc_4024DA

.text:004024D4 25C                 mov     ecx, [eax]

.text:004024D6 25C                 push    eax             ; This

.text:004024D7 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:004024DA

.text:004024DA     loc_4024DA:                             ; CODE XREF: sub_402220+2B2j

.text:004024DA 25C                 mov     eax, [ebp+ppvObject]

.text:004024DD 25C                 mov     byte ptr [ebp+var_4], 3

.text:004024E1 25C                 cmp     eax, ebx

.text:004024E3 25C                 jz      short loc_4024EB

.text:004024E5 25C                 mov     edx, [eax]

.text:004024E7 25C                 push    eax             ; This

.text:004024E8 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:004024EB

.text:004024EB     loc_4024EB:                             ; CODE XREF: sub_402220+2C3j

.text:004024EB 25C                 mov     eax, [ebp+pdisp]

.text:004024EE 25C                 mov     byte ptr [ebp+var_4], 2

.text:004024F2 25C                 cmp     eax, ebx

.text:004024F4 25C                 jz      loc_402580

.text:004024FA 25C                 jmp     short loc_40257A

.text:004024FC     ; ---------------------------------------------------------------------------

.text:004024FC

.text:004024FC     loc_4024FC:                             ; CODE XREF: sub_402220+2A7j

.text:004024FC 25C                 mov     ecx, 40h

.text:00402501 25C                 xor     eax, eax

.text:00402503 25C                 lea     edi, [ebp+var_147]

.text:00402509 25C                 mov     [ebp+Str], bl

.text:0040250F 25C                 rep stosd

.text:00402511 25C                 stosw

.text:00402513 25C                 stosb

.text:00402514 25C                 lea     eax, [ebp+Str]

.text:0040251A 25C                 push    104h            ; cbMultiByte

.text:0040251F 260                 push    eax             ; lpMultiByteStr

.text:00402520 264                 push    edx             ; lpWideCharStr

.text:00402521 268                 call    W2A

.text:00402526 268                 mov     ecx, [ebp+var_3C]

.text:00402529 268                 add     esp, 0Ch

.text:0040252C 25C                 push    ecx             ; bstrString

.text:0040252D 260                 call    ds:SysFreeString

.text:00402533 25C                 lea     edx, [ebp+Str]

.text:00402539 25C                 push    edx             ; Str

.text:0040253A 260                 call    sub_402140

.text:0040253F 260                 add     esp, 4

.text:00402542 25C                 test    eax, eax

.text:00402544 25C                 jz      short loc_4025BF

.text:00402546 25C                 push    ebx             ; dwMilliseconds

.text:00402547 260                 call    ds:Sleep

.text:0040254D 25C                 mov     eax, [ebp+var_14]

.text:00402550 25C                 mov     byte ptr [ebp+var_4], 4

.text:00402554 25C                 cmp     eax, ebx

.text:00402556 25C                 jz      short loc_40255E

.text:00402558 25C                 mov     ecx, [eax]

.text:0040255A 25C                 push    eax             ; This

.text:0040255B 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:0040255E

.text:0040255E     loc_40255E:                             ; CODE XREF: sub_402220+336j

.text:0040255E 25C                 mov     eax, [ebp+ppvObject]

.text:00402561 25C                 mov     byte ptr [ebp+var_4], 3

.text:00402565 25C                 cmp     eax, ebx

.text:00402567 25C                 jz      short loc_40256F

.text:00402569 25C                 mov     edx, [eax]

.text:0040256B 25C                 push    eax             ; This

.text:0040256C 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:0040256F

.text:0040256F     loc_40256F:                             ; CODE XREF: sub_402220+347j

.text:0040256F 25C                 mov     eax, [ebp+pdisp]

.text:00402572 25C                 mov     byte ptr [ebp+var_4], 2

.text:00402576 25C                 cmp     eax, ebx

.text:00402578 25C                 jz      short loc_402580

.text:0040257A

.text:0040257A     loc_40257A:                             ; CODE XREF: sub_402220+2DAj

.text:0040257A 25C                 mov     ecx, [eax]

.text:0040257C 25C                 push    eax             ; This

.text:0040257D 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:00402580

.text:00402580     loc_402580:                             ; CODE XREF: sub_402220+2D4j

.text:00402580                                             ; sub_402220+358j

.text:00402580 25C                 lea     edx, [ebp+pvarg]

.text:00402583 25C                 push    edx             ; pvarg

.text:00402584

.text:00402584     loc_402584:                             ; CODE XREF: sub_402220+25Cj

.text:00402584 260                 call    ds:VariantClear

.text:0040258A 25C                 mov     eax, [ebp+nCount]

.text:0040258D 25C                 mov     ecx, [ebp+nALL]

.text:00402590 25C                 inc     eax

.text:00402591 25C                 cmp     eax, ecx

.text:00402593 25C                 mov     [ebp+nCount], eax

.text:00402596 25C                 jl      loc_402364

.text:0040259C 25C                 jmp     loc_40266E

.text:004025A1     ; ---------------------------------------------------------------------------

.text:004025A1

.text:004025A1     loc_4025A1:                             ; CODE XREF: sub_402220+29Cj

.text:004025A1 25C                 mov     eax, [ebp+var_14]

.text:004025A4 25C                 mov     byte ptr [ebp+var_4], 4

.text:004025A8 25C                 cmp     eax, ebx

.text:004025AA 25C                 jz      short loc_4025B2

.text:004025AC 25C                 mov     ecx, [eax]

.text:004025AE 25C                 push    eax             ; This

.text:004025AF 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:004025B2

.text:004025B2     loc_4025B2:                             ; CODE XREF: sub_402220+38Aj

.text:004025B2 25C                 mov     eax, [ebp+ppvObject]

.text:004025B5 25C                 mov     byte ptr [ebp+var_4], 3

.text:004025B9 25C                 cmp     eax, ebx

.text:004025BB 25C                 jz      short loc_402610

.text:004025BD 25C                 jmp     short loc_40260A

.text:004025BF     ; ---------------------------------------------------------------------------

.text:004025BF

.text:004025BF     loc_4025BF:                             ; CODE XREF: sub_402220+324j

.text:004025BF 25C                 push    0Ah             ; dwMilliseconds

.text:004025C1 260                 call    ds:Sleep

.text:004025C7 25C                 mov     eax, [ebp+ppvObject]

.text:004025CA 25C                 push    eax             ; This

.text:004025CB 260                 mov     ecx, [eax]

.text:004025CD 260                 call    [ecx+IHTMLElementVtbl.click]

.text:004025D3 25C                 mov     eax, [ebp+var_14]

.text:004025D6 25C                 mov     byte ptr [ebp+var_4], 4

.text:004025DA 25C                 cmp     eax, ebx

.text:004025DC 25C                 jz      short loc_4025E4

.text:004025DE 25C                 mov     edx, [eax]

.text:004025E0 25C                 push    eax             ; This

.text:004025E1 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:004025E4

.text:004025E4     loc_4025E4:                             ; CODE XREF: sub_402220+1E4j

.text:004025E4                                             ; sub_402220+3BCj

.text:004025E4 25C                 mov     eax, [ebp+ppvObject]

.text:004025E7 25C                 mov     byte ptr [ebp+var_4], 3

.text:004025EB 25C                 cmp     eax, ebx

.text:004025ED 25C                 jz      short loc_40262B

.text:004025EF 25C                 jmp     short loc_402625

.text:004025F1     ; ---------------------------------------------------------------------------

.text:004025F1

.text:004025F1     loc_4025F1:                             ; CODE XREF: sub_402220+280j

.text:004025F1                                             ; sub_402220+288j

.text:004025F1 25C                 cmp     eax, ebx

.text:004025F3 25C                 mov     byte ptr [ebp+var_4], 4

.text:004025F7 25C                 jz      short loc_4025FF

.text:004025F9 25C                 mov     ecx, [eax]

.text:004025FB 25C                 push    eax             ; This

.text:004025FC 260                 call    [ecx+IDispatchVtbl.Release]

.text:004025FF

.text:004025FF     loc_4025FF:                             ; CODE XREF: sub_402220+3D7j

.text:004025FF 25C                 mov     eax, [ebp+ppvObject]

.text:00402602 25C                 mov     byte ptr [ebp+var_4], 3

.text:00402606 25C                 cmp     eax, ebx

.text:00402608 25C                 jz      short loc_402610

.text:0040260A

.text:0040260A     loc_40260A:                             ; CODE XREF: sub_402220+39Dj

.text:0040260A 25C                 mov     edx, [eax]

.text:0040260C 25C                 push    eax             ; This

.text:0040260D 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:00402610

.text:00402610     loc_402610:                             ; CODE XREF: sub_402220+39Bj

.text:00402610                                             ; sub_402220+3E8j

.text:00402610 25C                 mov     eax, [ebp+pdisp]

.text:00402613 25C                 mov     byte ptr [ebp+var_4], 2

.text:00402617 25C                 cmp     eax, ebx

.text:00402619 25C                 jz      short loc_402664

.text:0040261B 25C                 jmp     short loc_40265E

.text:0040261D     ; ---------------------------------------------------------------------------

.text:0040261D

.text:0040261D     loc_40261D:                             ; CODE XREF: sub_402220+1C8j

.text:0040261D                                             ; sub_402220+1D0j

.text:0040261D 25C                 cmp     eax, ebx

.text:0040261F 25C                 mov     byte ptr [ebp+var_4], 3

.text:00402623 25C                 jz      short loc_40262B

.text:00402625

.text:00402625     loc_402625:                             ; CODE XREF: sub_402220+3CFj

.text:00402625 25C                 mov     ecx, [eax]

.text:00402627 25C                 push    eax             ; This

.text:00402628 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:0040262B

.text:0040262B     loc_40262B:                             ; CODE XREF: sub_402220+3CDj

.text:0040262B                                             ; sub_402220+403j

.text:0040262B 25C                 mov     eax, [ebp+pdisp]

.text:0040262E 25C                 mov     byte ptr [ebp+var_4], 2

.text:00402632 25C                 cmp     eax, ebx

.text:00402634 25C                 jz      short loc_40263C

.text:00402636 25C                 mov     edx, [eax]

.text:00402638 25C                 push    eax             ; This

.text:00402639 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:0040263C

.text:0040263C     loc_40263C:                             ; CODE XREF: sub_402220+414j

.text:0040263C 25C                 lea     eax, [ebp+pvarg]

.text:0040263F 25C                 push    eax

.text:00402640 260                 jmp     short loc_402668

.text:00402642     ; ---------------------------------------------------------------------------

.text:00402642

.text:00402642     loc_402642:                             ; CODE XREF: sub_402220+1A5j

.text:00402642                                             ; sub_402220+1ADj

.text:00402642 25C                 mov     ecx, [ebp+ppvObject]

.text:00402645 25C                 mov     byte ptr [ebp+var_4], 3

.text:00402649 25C                 cmp     ecx, ebx

.text:0040264B 25C                 jz      short loc_402656

.text:0040264D 25C                 mov     edx, [ecx]

.text:0040264F 25C                 push    ecx             ; This

.text:00402650 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:00402653 25C                 mov     eax, [ebp+pdisp]

.text:00402656

.text:00402656     loc_402656:                             ; CODE XREF: sub_402220+42Bj

.text:00402656 25C                 cmp     eax, ebx

.text:00402658 25C                 mov     byte ptr [ebp+var_4], 2

.text:0040265C 25C                 jz      short loc_402664

.text:0040265E

.text:0040265E     loc_40265E:                             ; CODE XREF: sub_402220+3FBj

.text:0040265E 25C                 mov     ecx, [eax]

.text:00402660 25C                 push    eax             ; This

.text:00402661 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:00402664

.text:00402664     loc_402664:                             ; CODE XREF: sub_402220+3F9j

.text:00402664                                             ; sub_402220+43Cj

.text:00402664 25C                 lea     edx, [ebp+pvarg]

.text:00402667 25C                 push    edx             ; pvarg

.text:00402668

.text:00402668     loc_402668:                             ; CODE XREF: sub_402220+420j

.text:00402668 260                 call    ds:VariantClear

.text:0040266E

.text:0040266E     loc_40266E:                             ; CODE XREF: sub_402220+13Ej

.text:0040266E                                             ; sub_402220+37Cj

.text:0040266E 25C                 mov     eax, [ebp+p]

.text:00402671 25C                 mov     byte ptr [ebp+var_4], bl

.text:00402674 25C                 cmp     eax, ebx

.text:00402676 25C                 jz      short loc_40267E

.text:00402678

.text:00402678     loc_402678:                             ; CODE XREF: sub_402220+47Bj

.text:00402678 25C                 mov     ecx, [eax]

.text:0040267A 25C                 push    eax             ; This

.text:0040267B 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:0040267E

.text:0040267E     loc_40267E:                             ; CODE XREF: sub_402220+456j

.text:0040267E                                             ; sub_402220+479j

.text:0040267E 25C                 mov     eax, [ebp+This]

.text:00402681 25C                 mov     [ebp+var_4], 0FFFFFFFFh

.text:00402688 25C                 cmp     eax, ebx

.text:0040268A 25C                 jz      short loc_4026AE

.text:0040268C 25C                 mov     edx, [eax]

.text:0040268E 25C                 push    eax             ; This

.text:0040268F 260                 call    [edx+IHTMLDocument2Vtbl.Release]

.text:00402692 25C                 jmp     short loc_4026AE

.text:00402694     ; ---------------------------------------------------------------------------

.text:00402694

.text:00402694     loc_402694:                             ; CODE XREF: sub_402220+11Bj

.text:00402694                                             ; sub_402220+123j

.text:00402694 25C                 cmp     eax, ebx

.text:00402696 25C                 mov     byte ptr [ebp+var_4], bl

.text:00402699 25C                 jz      short loc_40267E

.text:0040269B 25C                 jmp     short loc_402678

.text:0040269D     ; ---------------------------------------------------------------------------

.text:0040269D

.text:0040269D     loc_40269D:                             ; CODE XREF: sub_402220+F5j

.text:0040269D                                             ; sub_402220+FDj

.text:0040269D 25C                 cmp     eax, ebx

.text:0040269F 25C                 mov     [ebp+var_4], 0FFFFFFFFh

.text:004026A6 25C                 jz      short loc_4026AE

.text:004026A8 25C                 mov     ecx, [eax]

.text:004026AA 25C                 push    eax             ; This

.text:004026AB 260                 call    [ecx+IHTMLDocument2Vtbl.Release]

.text:004026AE

.text:004026AE     loc_4026AE:                             ; CODE XREF: sub_402220+46Aj

.text:004026AE                                             ; sub_402220+472j ...

.text:004026AE 25C                 mov     edx, [ebp+hLibModule]

.text:004026B1 25C                 push    edx             ; hLibModule

.text:004026B2 260                 call    ds:FreeLibrary

.text:004026B8 25C                 call    ds:CoUninitialize

.text:004026BE 25C                 mov     ecx, [ebp+var_C]

.text:004026C1 25C                 mov     eax, esi

.text:004026C3 25C                 pop     edi

.text:004026C4 258                 pop     esi

.text:004026C5 254                 mov     large fs:0, ecx

.text:004026CC 254                 pop     ebx

.text:004026CD 250                 mov     esp, ebp

.text:004026CF 004                 pop     ebp

.text:004026D0 000                 retn

.text:004026D0     sub_402220     endp
```
