
`SDPI:004824A8 aDebuggerDetected db 'Debugger detected!',0`

`SDPI:004824BB`

`SDPI:004824BB ; ************* S U B R O U T I N E **********************`

`SDPI:004824BB`

`SDPI:004824BB ; 没多大作用检测,检测 UnhandleExceptxx`

`SDPI:004824BB ; API 的第一个字节是否下断了`

`SDPI:004824BB`

```
SDPI:004824BB CHK_UnHndexcptCC proc near                     ; CODE XREF: SDPI:0047DCE1 j
SDPI:004824BB             push    offset strUnhandledExcept ; relativeAddress
SDPI:004824C0             call    GetAbsAddress             ; 获取实际地址
SDPI:004824C5             push    eax
SDPI:004824C6             push    offset strkerdll          ; relativeAddress
SDPI:004824CB             call    GetAbsAddress             ; 获取实际地址
SDPI:004824D0             push    eax                       ; APIName
SDPI:004824D1             call    api_LoadLibraryA
SDPI:004824D6             push    eax                       ; hdll
SDPI:004824D7             call    API_GetProcAddr
SDPI:004824DC             test    eax, eax                  ; 判断 Unxxx 是否被下断了
SDPI:004824DE             jz      short locret_482510
SDPI:004824E0             cmp     byte ptr [eax], 0CCh
SDPI:004824E3             jnz     short locret_482510
SDPI:004824E5             call    Decode_47F088
SDPI:004824EA             push    0                         ; uType
SDPI:004824EC             push    offset strError           ; relativeAddress
SDPI:004824F1             call    GetAbsAddress             ; 获取实际地址
SDPI:004824F6             push    eax                       ; lpCaption
SDPI:004824F7             push    offset unk_4094A8         ; relativeAddress
SDPI:004824FC             call    GetAbsAddress             ; 获取实际地址
SDPI:00482501             push    eax                       ; lpText
SDPI:00482502             push    0                         ; hWnd
SDPI:00482504             call    MessageBoxA               ; MessageBoxA 函数，壳用的宏来处理，
SDPI:00482504                                                     ; 判断函数的前 5 位是否为 CC
SDPI:00482504                                                     ; 也就是判断有没有下 int3 断点
SDPI:00482509             push    9                         ; uExitCode
SDPI:0048250B             call    ExitProcess
SDPI:00482510

SDPI:00482510 locret_482510:                                 ; CODE XREF: CHK_UnHndexcptCC+23 j
SDPI:00482510                                                     ; CHK_UnHndexcptCC+28 j
SDPI:00482510             retn
SDPI:00482510 CHK_UnHndexcptCC endp ; sp = -4

SDPI:00482510

SDPI:00482510 ; -----------------------------------------------------------------------
SDPI:00482510 ; ----------------------------------------------------------------------------

SDPI:00482511 aZwqueryinformationp db 'ZwQueryInformationProcess',0
SDPI:0048252B aNtdll_dll        db 'ntdll.dll',0
```

`SDPI:00482535`

`SDPI:00482535 ; ************* S U B R O U T I N E **********************`

`SDPI:00482535`

`SDPI:00482535`

`SDPI:00482535 AntiDBG_482535  proc near                      ; CODE XREF: SDPI:0047ABC4 j`

`SDPI:00482535`

`SDPI:00482535 var_18          = dword ptr -18h`

`SDPI:00482535 var_4           = dword ptr -4`

`SDPI:00482535`

```
SDPI:00482535             push    ecx
SDPI:00482536             call    Get_Version
SDPI:0048253B             cmp     eax, 80000000h
SDPI:00482540             jnb     short isWin9x_4825A5      ; 如果是 Win9x 则跳
SDPI:00482542             push    esi
SDPI:00482543             push    edi
SDPI:00482544             call    GetTickCount
SDPI:00482549             push    offset strZwQInfoProc     ; 获取 ZwQueryInformationProcess 的地址
SDPI:0048254E             mov     edi, eax
SDPI:00482550             call    GetAbsAddress             ; 获取实际地址
SDPI:00482555             push    eax
SDPI:00482556             push    offset strdllNTDLL        ; relativeAddress
SDPI:0048255B             call    GetAbsAddress             ; 获取实际地址
SDPI:00482560             push    eax                       ; APIName
SDPI:00482561             call    api_LoadLibraryA
SDPI:00482566             push    eax                       ; hdll
SDPI:00482567             call    API_GetProcAddr
SDPI:0048256C             mov     esi, eax
SDPI:0048256E             test    esi, esi                  ; 获取 ZwQueryInformationProcess 的地址
SDPI:0048256E                                                     ; 如果获取失败则直接跳去 IsDebuggerPresent
SDPI:0048256E                                                     ; 检测是否有 Ring3 级调试器
SDPI:00482570             jz      short CHECK_R3D
SDPI:00482572             push    0
SDPI:00482574             lea     eax, [esp+10h+var_4]
SDPI:00482578             push    4
SDPI:0048257A             push    eax
SDPI:0048257B             push    7
SDPI:0048257D             call    apiGetCurProcess
SDPI:00482582             push    eax
SDPI:00482583             call    esi                       ; 检测调试器是否存在
SDPI:00482585             test    eax, eax
SDPI:00482587             jnz     short CHECK_R3D
SDPI:00482589             mov     eax, [esp+20h+var_18]
SDPI:0048258D             test    eax, eax
SDPI:0048258F             jnz     short FoundDBG_48259F     ; 如果检测到调试器则 over 了
SDPI:00482591

SDPI:00482591 CHECK_R3D:                                 ; CODE XREF: AntiDBG_482535+3B j
SDPI:00482591                                                     ; AntiDBG_482535+52 j
SDPI:00482591             call    isDebuggerPresent
SDPI:00482596             test    eax, eax
SDPI:00482598             jnz     short FoundDBG_48259F
SDPI:0048259A             mov     edi, 80000000h          ; 没有检测到调试器则
SDPI:0048259A                                                     ; mov eax,80000000h
SDPI:0048259F

SDPI:0048259F FoundDBG_48259F:                            ; CODE XREF: AntiDBG_482535+5A j
SDPI:0048259F                                                     ; AntiDBG_482535+63 j
SDPI:0048259F             mov     eax, edi
SDPI:004825A1             pop     edi
SDPI:004825A2             pop     esi
SDPI:004825A3             pop     ecx
SDPI:004825A4             retn
SDPI:004825A5 ; ----------------------------------------------------------------------------
```

`SDPI:004825A5`

`體局駑#   @?`

`SDPI:004825AB ; ----------------------------------------------------------------------------`

`SDPI:004825AC aNtquerysysteminform db 'NtQuerySystemInformation',0`

`SDPI:004825C5 aNtice_sys       db 'ntice.sys',0`

`SDPI:004825CF aIceext_sys      db 'IceExt.sys',0`

`SDPI:004825DA`

`SDPI:004825DA ; ************* S U B R O U T I N E **********************`

`SDPI:004825DA`

`SDPI:004825DA ; 检测 RING 0 级调试器`

`SDPI:004825DA ; 只检测 ICE 和 ICEEXT`

`SDPI:004825DA ; 看来私下的秘密武器还是有`

`SDPI:004825DA ; 很大作用^\_^`

`SDPI:004825DA`

`SDPI:004825DA CHK_R0D_4825DA  proc near                      ; CODE XREF: SDPI:0047BCB5 j`

`SDPI:004825DA`

`SDPI:004825DA var_4A          = byte ptr -4Ah`

`SDPI:004825DA var_3A          = dword ptr -3Ah`

`SDPI:004825DA var_34          = dword ptr -34h`

`SDPI:004825DA var_28          = dword ptr -28h`

`SDPI:004825DA var_1C          = dword ptr -1Ch`

`SDPI:004825DA var_C           = dword ptr -0Ch`

`SDPI:004825DA @HMODEL         = dword ptr -4`

`SDPI:004825DA`

```
SDPI:004825DA             sub     esp, 0Ch
SDPI:004825DD             push    ebx
```
