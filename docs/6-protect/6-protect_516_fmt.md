
# 仙剑源码

> 原文：[`www.pediy.com/kssd/pediy06/pediy6489.htm`](https://www.pediy.com/kssd/pediy06/pediy6489.htm)

```asm
;DEBUG_T    =  TRUE   

@Loader_Start:   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 伪装为 DBPE 2.x -> Ding Boy   

  jmp $+2+20h   

  dd 400000h,400000h,0,0,0,0,0,0   

  pushfd   

  push  ebp   

  push  edi   

  push  esi   

  push  edx   

  push  ecx   

  push  ebx   

  pushfd   

  call  $+5   

  pop  ebp   

  db 81h, 0EDh ,0,0,0,0                      ; sub  ebp, 00000000   

  popfd   

  add  esp, 14h   

  pop  ebp   

  popfd   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

                                     _svkp1 0CDh   

  pushad   

                                     _svkp3 44h   

  ; 取起始地址   

  call  @Delta   

@Delta:   

  pop  ebp   

                                     _svkp2 0CCh   

  sub  ebp, offset @Delta   

                                     _svkp3 0E8h   

  push  ebp  ; 保存 ebp   

                                     _svkp1 0E9h   

  add  ebp, offset @Decrypt_Loader_Code   

                                     _svkp2 99h   

  push  ebp   

                                     _svkp2 33h   

  ret                            ; -> @Decrypt_Loader_Code   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; Banner :D   

  db  ' Sword-Protector (c) by LiNSoN[DFCG] release                      ',CR,LF   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

@Decoder:   

  ; 重点防护 & anti-disasmer   

                   _vxd   

  enter  0, 0   

                   _junk 0E9h   

  pushad   

                   _vxd   

  mov  eax, [ebp+10h]              ; eax -> pContext   

  assume eax : ptr CONTEXT   

                   _vxd   

  mov  ebx, [ebp+8]              ; ebx -> pExcept   

                   _mmx 0E9h   

  mov  ebx, [ebx]   

                   _trash 0E8h   

  .if  ebx == 0C000001Dh   

                   _vxd   

     ; mov     seg?, dx 产生的第一个异常   

     xor  ebx, ebx   

                   _vxd   

     and  [eax].iDr0, ebx   

                   _trash  0E8h   

     and  [eax].iDr1, ebx   

                   _mmx 0CDh   

     and  [eax].iDr2, ebx   

                   _trash  0E8h   

     and  [eax].iDr3, ebx   

                   _vxd   

     and  [eax].iDr6, 0FFFF0FF0h   

                   _trash  0E8h   

     and  [eax].iDr7, 0DC00h        ; 禁止掉所有断点   

     ; 修复 regEip 指针   

                   _jmper 0FFh   

     add  [eax].regEip, 2   

                   _vxd   

     jmp  @@ExceptProcessed   

  .elseif  ebx == 0C000001Eh   

                   _vxd   

    cld                   ; 正方向进行   

                   _jmper 0BEh   

    xchg  esi, [eax].regEsi   

                   _vxd   

    xor  esi, [eax].iDr0          ; 防止 DRx 断点, 若设置 DRx 会造成解码错误, 下同   

                   _vxd   

    xchg  edi, [eax].regEdi   

                   _jmper 0FFh   

    xor  edi, [eax].iDr1   

                   _junk 0E9h   

    xchg  ecx, [eax].regEcx   

                   _trash  0E8h                     

    xor  ecx, [eax].iDr2   

                   _jmper 0CCh   

    push  eax   

                   _jmper 0CCh   

    lodsb   

                   _trash  0E8h   

    VarDecryptBuff db VAR_PER_SIZE dup (0)  ; 解码   

                   _trash  0E8h   

    stosb   

                   _vxd   

    pop  eax   

                   _mmx 0CDh   

    xor  eax, [eax].iDr3   

                   _trash  0E8h   

    xchg  esi, [eax].regEsi   

                   _mmx 0E9h   

    xchg  edi, [eax].regEdi   

                   _mmx 0E8h   

    xchg  ecx, [eax].regEcx   

                   _vxd   

    dec  [eax].regEcx   

  .endif   

                   _vxd   

  .if  zero?   

                   _trash  0E8h   

    ; 恢复原始 SEH Handler   

    mov  ecx, [eax].regEsp   

                   _mmx 0CDh   

    push  [ecx]   

                   _jmper 0CCh   

    xor  ecx, ecx   

                   _jmper 0CDh   

    pop  fs:[ecx]   

                   _trash  0E8h   

    ; 卸载 SEH 链   

    add  [eax].regEsp, 8   

                   _mmx 0CDh   

    ; 修复异常   

    add  [eax].regEip, 4     

                   _vxd   

    ; 初始化 DRx   

    and  [eax].iDr0, ecx   

                   _trash  0E8h   

    and  [eax].iDr1, ecx   

                   _mmx 0CDh   

    and  [eax].iDr2, ecx   

                   _trash  0E8h   

    and  [eax].iDr3, ecx   

                   _vxd   

    and  [eax].iDr6, 0FFFF0FF0h   

                   _trash  0E8h   

    and  [eax].iDr7, 0DC00h        ; 禁止掉所有断点   

  .endif   

                   _mmx 0E8h   

  @@ExceptProcessed:   

  popad   

  assume eax : nothing   

                   _mmx 0E9h   

  xor  eax, eax   

                   _vxd   

  leave   

                   _trash  0E8h   

  ret   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

@Decrypt_Loader_Code:   

                   _jmper 0C7h   

  pop  ebp  ; 恢复 ebp   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 首层 Loader 代码解码   

                   _jmper 0CDh   

  lea  eax, @Decoder[ebp]   

                   _vxd   

  push  eax   

                   _trash 0E8h   

  xor  eax, eax   

                   _mmx 0E9h   

  push  fs:[eax]   

                   _junk 0E9h   

  mov  fs:[eax], esp   

  db    8Eh, 0FAh                 ;mov     seg?, dx 产生异常初始化 DRx 防止解码错误   

  mov  ecx, @Loader_Crypt_Size        ; 解码长度   

                   _vxd   

  lea  edi, @Loader_Crypt_Start[ebp]    ; 首地址   

                   _jmper 0CCh   

  mov  esi, edi   

                   _vxd   

  dd    0C8C70FF0h                  ; lock cmpxchg8b (e)ax 产生非法指令异常调用解码器, 解码完成会自行卸载   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

@Loader_Crypt_Start:   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 使用段选择子检测操作系统类别   

  mov  ax, ds   

  and  ax, 4   

  .if  zero?   

      inc  eax   

      mov  bNT[ebp], eax   

  .endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 取 Loader 校验和   

  lea  eax, @Loader_Start[ebp]   

  mov  ecx, LOADER_CRC_CHECK_SIZE   

  call  GetChecksum   

  mov  dwLoaderCRC[ebp], eax   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 检测 SoftICE   

  test  PROTECTION_FLAGS[ebp], CHECK_SI_FLAG   

  .if  !zero?   

    lea  esi, SEH[ebp]   

    assume  esi : ptr sSEH   

    lea  eax, SICheck1_SP[ebp]   

    mov  [esi].SaveEip, eax   

    assume  esi : nothing   

    mov   EDI, EBP   

    lea   eax, @SehHandler1[ebp]   

    xor  ebx, ebx   

    push  eax   

    push  fs:[ebx]   

    mov  fs:[ebx], esp   

    ; 0 - 未检测到 SoftICE   

    ; 1 - 检测到 SoftICE   

    mov  ebp, 04243484Bh   

    mov  ax, 04h   

                   _vxd   

    int  3     

   SICheck1_SP:   

    mov   EBP, EDI   

    ; 恢复 SEH 链   

    xor  ebx, ebx   

    pop  fs:[ebx]   

    add  esp, 4h   

    .if AL != 4   

                   _junk 0E9h   

      popad   

      ret   

    .endif   

  .endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 取外部函数地址   

  lea  eax, LoaderContinue1[ebp]   

  push  eax   

  ret   

  @DoGetProcAddr:   

    ; eax = 函数名称   

    ; esi = DLL 模块句柄
```



这是一个汇编代码片段，包含了软壳代码（shellcode）的逻辑，包括函数地址获取、调试器检测（SoftICE, NT Native API）以及防转储和校验和计算等。

```assembly
    push  eax   

    push  esi   

    call  _GetProcAddress[ebp]   

    cmp  byte ptr [eax], 0CCh   

    .if  zero?   

        add  esp, 24h   

    .endif   

    ret   

  LoaderContinue1:   

  ; 从 ImageImportDescriptor 中读取输入函数地址   

  mov  eax, dwImageBase[ebp]   

  add  eax, [eax+03Ch]   

  add  eax,080h   

  mov  ecx, [eax]                      ; ecx contains the VirtualAddress of the IT   

  add  ecx, dwImageBase[ebp]   

  add  ecx,16                          ; ecx points to the FirstThunk address of the IID   

  mov  eax,dword ptr [ecx]   

  add  eax, dwImageBase[ebp]   

  mov  ebx,dword ptr [eax]   

  mov  [ebp+_LoadLibrary],ebx   

  add  eax,4   

  mov  ebx,dword ptr [eax]   

  mov  _GetProcAddress[ebp],ebx   

  ; 取 Kernel.DLL 之函数   

  lea  eax,szKernel32[ebp]   

  push  eax   

  call  _LoadLibrary[ebp]   

  mov  esi, eax                ; esi -> Kernel 模块句柄   

  mov  dwKernelBase[ebp], eax   

  lea  edi, @DoGetProcAddr[ebp]   

  ;-> GetModuleHandle   

  lea  eax, [ebp+szGetModuleHandle]   

  call  edi   

  mov  [ebp+_GetModuleHandle],eax   


  ;-> VirtualProtect   

  lea  eax, [ebp+szVirtualProtect]   

  call  edi   

  mov  [ebp+_VirtualProtect],eax   


  ;-> GetModuleFileName   

  lea  eax, [ebp+szGetModuleFileName]   

  call  edi   

  mov  [ebp+_GetModuleFileName],eax   

  ;-> CreateFile   

  lea  eax, [ebp+szCreateFile]   

  call  edi   

  mov  [ebp+_CreateFile],eax   

  ;-> GlobalAlloc   

  lea  eax, [ebp+szGlobalAlloc]   

  call  edi   

  mov  [ebp+_GlobalAlloc],eax   

  ;-> GlobalFree   

  lea  eax, [ebp+szGlobalFree]   

  call  edi   

  mov  [ebp+_GlobalFree],eax   

  ;-> ReadFile   

  lea  eax, [ebp+szReadFile]   

  call  edi   

  mov  [ebp+_ReadFile],eax   

  ;-> WriteFile   

  lea  eax, [ebp+szWriteFile]   

  call  edi   

  mov  [ebp+_WriteFile],eax   

  ;-> GetFileSize   

  lea  eax, [ebp+szGetFileSize]   

  call  edi   

  mov  [ebp+_GetFileSize],eax   

  ;-> CloseHandle   

  lea  eax, [ebp+szCloseHandle]   

  call  edi   

  mov  _CloseHandle[ebp],eax   

  ;-> CreateEvent   

  lea  eax, [ebp+szCreateEvent]   

  call  edi   

  mov  [ebp+_CreateEvent],eax   

  ;-> GetCurrentProcessId   

  lea  eax,szGetCurrentProcessId[ebp]   

  call  edi   

  mov  [ebp+_GetCurrentProcessId], eax   

  ;-> GetCurrentProcess   

  lea  eax,szGetCurrentProcess[ebp]   

  call  edi   

  mov  [ebp+_GetCurrentProcess],  eax   

  ;-> GetStdHandle   

  lea  eax, szGetStdHandle[ebp]   

  call  edi   

  mov  _GetStdHandle[ebp],  eax   

  ;-> OpenProcess   

  lea  eax, szOpenProcess[ebp]   

  call  edi   

  mov  _OpenProcess[ebp],  eax   

  ;-> ReadProcessMemory   

  lea  eax, szReadProcessMemory[ebp]   

  call  edi   

  mov  _ReadProcessMemory[ebp],  eax   

  ;-> VirtualAlloc   

  lea  eax, szVirtualAlloc[ebp]   

  call  edi   

  mov  _VirtualAlloc[ebp],  eax   

  ;-> VirtualFree   

  lea  eax, szVirtualFree[ebp]   

  call  edi   

  mov  _VirtualFree[ebp],  eax   

  ;-> GetWindowsDirectory   

  lea  eax, szGetWindowsDirectory[ebp]   

  call  edi   

  mov  _GetWindowsDirectory[ebp],  eax   

  ;-> lstrcat   

  lea  eax, szlstrcat[ebp]   

  call  edi   

  mov  _lstrcat[ebp],  eax   

  ;-> lstrcmpi   

  lea  eax, szlstrcmpi[ebp]   

  call  edi   

  mov  _lstrcmpi[ebp],  eax   

  ; 取 NtDLL.DLL 之函数   

  lea  eax, szNTDLL[ebp]   

  push  eax   

  call  [_GetModuleHandle+ebp]   

  test  eax, eax   

  .if  !zero?   

    mov  esi, eax          ; esi -> NtDLL.DLL   

    ;-> ZwQueryInformationProcess   

    lea  eax, szZwQueryInformationProcess[ebp]   

    call  edi   

    mov  _ZwQueryInformationProcess[ebp],  eax   

  .endif     


  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 检测 SoftICE , 不支持 NT   

  test  PROTECTION_FLAGS[ebp], CHECK_SI_FLAG   

  .if  !zero?   

    lea   ESI, [EBP+SEH]   

    ASSUME ESI : PTR sSEH   

    lea   EAX, [EBP+OFFSET SICheck2_SP]   

    mov   [ESI].SaveEip, EAX   

    ASSUME ESI : NOTHING   

    xor   EBX, EBX   

    lea   EAX, [EBP+OFFSET @SehHandler2]   

    push  EAX   

    push  FS:[EBX]   

    mov   FS:[EBX], ESP   

    mov   EDI, EBP   

    mov   EAX, 4400h   

                          _jmper 0C7h   

    INT  68h   

  SICheck2_SP:   

    xor   EBX, EBX   

    POP  FS:[EBX]   

    add   ESP, 4   

    .if DI == 01297h || DI == 01277h || DI == 01330h   

                          _mmx 0FFh   

      popad   

                          _junk 0E8h   

      ret   

    .endif   

  .endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; NT's Native API 检测 Ring 3 级调试器   

  pushad   

  mov  edi, _ZwQueryInformationProcess[ebp]   

  test  edi, edi   

  .if  !zero?   

      push  0   

      push  0       


      push  4   

      lea  eax, [esp+8]   

      push  eax   

      push  7   

      call  _GetCurrentProcess[ebp]   

      push  eax   

      call  edi   

      pop  eax   

      test  eax, eax   

      .if  !zero?   

          add  esp, 20h*2   

          ret   

      .endif   

  .endif   

  popad   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   

  ; 检测父进程名 ( by kongfoo )   

  ;...   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 修改 ImageSize 防止进程转储   

  test  PROTECTION_FLAGS[ebp],ANTI_DUMP_FLAG   

  .if  !zero?   

      mov  eax, 30h   

      push  fs:[eax]   

      pop  eax   

        test     EAX, EAX   

        JS      fuapfdw_is9x          ; detected Win 9x   

   fuapfdw_isNT:   

        mov      EAX, [EAX+0Ch]   

        mov      EAX, [EAX+0Ch]   

        mov       dword ptr [EAX+20h], 1000h ; increase size variable   

        JMP     fuapfdw_finished   

   fuapfdw_is9x:   

        push     0   

        call     [ebp+_GetModuleHandle]   

        test     EDX, EDX   

        JNS     fuapfdw_finished      ; Most probably incompatible!!!   

        CMP      dword ptr [EDX+8], -1   

        JNE     fuapfdw_finished      ; Most probably incompatible!!!   

        mov      EDX, [EDX+4]          ; get address of internaly used   

                                       ; PE header   

        mov       dword ptr [EDX+50h], 1000h ; increase size variable   

   fuapfdw_finished:   

   .endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 取文件头写权限   

  mov  edi, dwImageBase[ebp]   

  add  edi, [edi+03Ch]   

  assume edi : ptr IMAGE_NT_HEADERS      ; edi -> 指向 PE Header   

  mov  esi, dwImageBase[ebp]   

  mov  ecx, [edi].OptionalHeader.SizeOfHeaders   

  assume edi : nothing   

  ; 修正页面访问权限   

  lea  eax, Buff[ebp]   

  push  eax   

  push  PAGE_READWRITE   

  push  ecx   

  push  dwImageBase[ebp]   

  call  _VirtualProtect[ebp]   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 计算校验和   

  test  PROTECTION_FLAGS[ebp],CHECK_HEADER_CRC   

  .if  !zero?   

    ; 取执行文件名   

    push  MAX_PATH   

    lea  edi, Buff[ebp]   

    push  edi              ; edi -> 文件名   

    push  0   

    call  _GetModuleFileName[ebp]   

    ; 映射文件   

    push  0   

    push  FILE_ATTRIBUTE_NORMAL   

    push  OPEN_EXISTING   

    push  NULL   

    push  FILE_SHARE_READ   

    push  GENERIC_READ   

    push  edi   

    call  _CreateFile[ebp]   

    inc  eax   

    ; INVALID_HANDLE_VALUE ?   

    jz    SkipChecksumCalc   

    dec  eax   

    mov  edi, eax            ; edi -> 文件句柄   

    push  NULL   

    push  edi   

    call  _GetFileSize[ebp]   

    sub  eax, CHECKSUM_SKIP_SIZE   

    xchg  eax, esi            ; esi -> 文件长度   

    push  esi   

    push  GMEM_FIXED+GMEM_ZEROINIT   

    call  _GlobalAlloc[ebp]   

    test  eax, eax   

    jz    SkipChecksumCalcAndCleanUp
```



```
  xchg  eax, ebx            ; ebx -> mem base 

  push  NULL   

  lea  eax, Buff\[ebp\]   

  push  eax   

  push  esi   

  push  ebx   

  push  edi   

  call  \_ReadFile\[ebp\]   

  ; 计算校验和   

  mov  eax,ebx   

  mov  ecx,esi   

  ; EBX -> hMem   

  ; EDI = hFile   

  pushad   

  call  GetChecksum   

  mov  dwCalcedCRC\[ebp\], eax   

  popad   

  ; 计算所得校验和将在初始化输入表后比较 >:-)   

  lea  eax, AfterCRCCalcContinue\[ebp\]   

  push  eax   

  ret   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 校验和计算函数   

  ; eax = file image base   

  ; ecx = 数据长度   

  GetChecksum:   

    mov  edi,eax            ; edi -> data pointer   

    xor  eax,eax            ; eax -> current bytes   

    xor  ebx,ebx            ; ebx -> current checksum   

    xor  edx,edx            ; edx -> Position (zero based)   

    CheckSumLoop:   

      mov  al, \[edi\]   

      mul edx   

      add  ebx,eax   

      inc edx   

      inc edi     

      loop  CheckSumLoop   

    xchg  eax,ebx            ; eax -> 校验和   

    ret   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  AfterCRCCalcContinue:   

  ; 打扫战场   

  push  ebx   

  call  \_GlobalFree\[ebp\]   

  xchg  esi,eax   

  SkipChecksumCalcAndCleanUp:   


  push  eax   

  push  edi   

  call  \_CloseHandle\[ebp\]   


  pop eax   

  SkipChecksumCalc:   

  .endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   

  ; 调用 IsDebuggerPresent 函数检测调试器   

  ifndef  DEBUG\_T   

    lea  EAX, \[EBP+OFFSET szIsDebuggerPresent\]   

    push  EAX   

    push  dwKernelBase\[ebp\]   

    call  \_GetProcAddress\[ebp\]   

    test    EAX, EAX    ; Windows 95 不支持此函数   

    .if  !zero?   

      call  EAX   

      OR   EAX, EAX   

      .IF  !ZERO?   

        popad   

        RET   

      .ENDIF   

    .ENDIF   

  endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 区段解码   

  test  PROTECTION\_FLAGS\[ebp\], NO\_CRYPT\_SECTION\_FLAG   

  .if  zero?   

    mov  eax, dwImageBase\[ebp\]   

    xor  ebx, ebx   

    inc  ebx   

    call  CryptPE   

  .endif   

  lea  eax, AfterDeCryptionContinue\[ebp\]   

  push  eax   

  ret   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 区段加/解密函数   

  ; eax: pointer to file memory   

  ; ebx: 0 - RawCrypt mode   

  ;      1 - VirtualCrypt mode   

  CryptPE:   


  mov  edi,eax   

  add  edi, \[edi+3Ch\]   

  assume edi : ptr IMAGE\_NT\_HEADERS      ; edi -> PE header   

  mov  esi,edi   

  add  esi,0F8h   

  assume esi : ptr IMAGE\_SECTION\_HEADER   ; esi -> Section header   

  xor  edx,edx   

  .REPEAT   

    ; -> 跳过特殊段名的段 !   

    .IF  dword ptr \[esi\].Name1 == ('crsr')   

      jmp @@LoopEnd   

    .ENDIF   

    .IF  dword ptr \[esi\].Name1 == ('rsr.')   

      jmp @@LoopEnd   

    .ENDIF   

    .IF  dword ptr \[esi\].Name1 == ('oler')   

      jmp @@LoopEnd   

    .ENDIF   

    .IF  dword ptr \[esi\].Name1 == ('ler.')   

      jmp @@LoopEnd   

    .ENDIF   

    .IF  dword ptr \[esi\].Name1 == ('ade.')   

      jmp @@LoopEnd   

    .ENDIF   

    .IF  dword ptr \[esi\].Name1 == 0   

      jmp @@LoopEnd   

    .ENDIF   

    ;-> skip also some other sections   

    .IF \[esi\].PointerToRawData == 0 || \[esi\].SizeOfRawData == 0   

      jmp @@LoopEnd   

    .ENDIF   

    SECTION\_NAME\_ENCRYPT\_NUM    =    19890813h   

    ; 用来加密段名的常数   

    xor  dword ptr \[esi\].Name1, SECTION\_NAME\_ENCRYPT\_NUM   

    xor  dword ptr \[esi+4\].Name1, SECTION\_NAME\_ENCRYPT\_NUM   

    ;-> en-/decrypt it   

    pushad   

    mov  ecx, \[esi\].SizeOfRawData   

    .IF ebx == 0        ; (ebx is a parameter)   

      mov  esi, \[esi\].PointerToRawData   

      add  ESI, EAX   

      call  EncryptSec   

    .ELSE   

      mov   esi, \[esi\].VirtualAddress   

      add   esi,eax   

      call  DecryptSec   

    .ENDIF   

    JMP SecDecryptContinue1   

  ; esi = CryptStart   

  ; ecx = CryptSize   

  DecryptSec:   

    mov  edi,esi   

  SecDecryptLoop:   

    lodsb   

    SecDecryptBuff db SEC\_PER\_SIZE dup (0)   

    stosb   

    loop  SecDecryptLoop   

  ret   

  SecDecryptContinue1:     


    popad   

    @@LoopEnd:     


    add  esi,SIZEOF IMAGE\_SECTION\_HEADER   

    inc  edx   

  .UNTIL dx == \[edi\].FileHeader.NumberOfSections   

  assume esi : nothing   

  assume edi : nothing   

  ret   

  AfterDeCryptionContinue:   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; SET Index Variable of TLS table to 0   

  ; 是否有 Tls 表   

  mov  edi, dwImageBase\[ebp\]   

  add  edi,  dword ptr \[edi+03Ch\]      ; edi -> 指向 PE Header   

  assume edi : ptr IMAGE\_NT\_HEADERS   

  mov  ebx, \[edi\].OptionalHeader.DataDirectory\[SIZEOF IMAGE\_DATA\_DIRECTORY \* 9\].VirtualAddress   

  assume edi : nothing   

  test  ebx,  ebx   

  .if  !zero?;  tls section   

    add  ebx, dwImageBase\[ebp\]      ; ebx -> pointer to tls table   

    assume ebx : ptr IMAGE\_TLS\_DIRECTORY32   

    mov  eax, \[ebx\].AddressOfIndex   

    mov  dword ptr \[eax\],0   

  .endif   

  assume ebx : nothing   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 比较 Loader 校验和   

  mov  eax, dwCalcedCRC\[ebp\]   

  test  eax, eax   

  .if  !zero?   

    cmp  dwOrgChecksum\[ebp\], eax   

    jnz  SkipInitIt   

  .endif   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 初始化输入表   

  ; 0 - 发生错误   

  ; 1 - 输入表成功初始化   

  lea  ESI, IIDInfo\[ebp\]                 ; ESI -> 指向当前 IID 数组   

  ASSUME ESI : PTR sItInfo   

  ; 准备函数重定向   

  test  PROTECTION\_FLAGS\[ebp\], API\_REDIRECT\_FLAG   

  .if  !zero?   

    pushad   

    lea  edi, Buff\[ebp\]   

    assume  edi : ptr sReThunkInfo   

    xor  ecx, ecx   

    .while  \[ESI\].FirstThunk   

      mov   EDX, \[ESI\].FirstThunk   

      add   EDX, dwImageBase\[ebp\]   

      .while  dword ptr \[edx\]   

        inc  ecx   

        add  edx, 4   

      .endw   

      add  esi, sizeof sItInfo   

    .endw   

    ; 为 IAT 分配内存   

    sub  edx, edx   

    mov  eax, sizeof sApiStub   


    mul  ecx   

    push  PAGE\_READWRITE      ; access type   

    push  MEM\_RESERVE or MEM\_COMMIT  ; flags   

    push  eax        ; memory size to allocate   

    push  7FF60000h   

    call  \_VirtualAlloc\[ebp\]    ; allocate memory   

    .IF !EAX    ; 异常退出   

      pop  eax   

      popad   

      ret   

    .ENDIF   

    mov  \[edi\].ApiStubMemAddr, eax   

    mov  \[edi\].pNextStub, eax   

    assume  edi : nothing   

    popad   

  .endif   

  ; start with the real routine   


  .while  \[esi\].FirstThunk != 0   

    ; 载入动态链接库   

    mov  ebx, \[esi\].DllNameRVA   

    add  ebx, dwImageBase\[ebp\]   

    ; decrypt dll string   

    mov  EAX,EBX   

    call  @EnDeCryptString   

    lea  EAX, InitITContinue1\[ebp\]    ; goto InitITContinue1   

    push  EAX   

    ret   

  ; eax = VA of target string   

  @EnDeCryptString:   

    push  ESI   

    push  EDI       


    mov  ESI,EAX   

    mov  EDI,EAX   

  @DllCryptLoop:   

    LODSB   

    ROR AL,4   

    STOSB   

    CMP BYTE PTR \[EDI\],0   

    JNZ @DllCryptLoop   

    POP EDI   

    POP ESI   

    ret   

  InitITContinue1:   

    push  ebx   

    call  \[ebp+\_LoadLibrary\]   

    test  eax,eax   

    jz SkipInitIt   

    ; zero dll name   

    push  eax                      ; save dll base   

    test  PROTECTION\_FLAGS\[ebp\],DESTROY\_IMPORT\_FLAG   

    .if  !zero?   

      mov  eax, ebx   

      lea  ebx, @KillString\[ebp\]   

      call  ebx   

    .endif   

    POP EBX                  ; EBX -> library handle   

    ; 处理 (Original-)FirstThunk 成员   

    mov  ecx, \[esi\].OrgFirstThunk   

    .if ecx == 0 
```



```
mov ecx, [esi].FirstThunk   

.endif   

add ecx, dwImageBase[ebp]      ; ecx -> pointer to current thunk   

mov edx, [esi].FirstThunk   

add edx, dwImageBase[ebp]      ; edx -> pointer to current thunk (always the non-original one)   

.WHILE dword ptr [ecx] != 0   

  test dword ptr [ecx], IMAGE_ORDINAL_FLAG32         ; is it an ordinal import ?   

.if zero?   

  ; 处理 name import   

  mov dword ptr eax, [ecx]   

  add eax,2   

  add eax, dwImageBase[ebp]                    ; eax points now to the Name of the Import   

  push eax   

  call @EnDeCryptString   

  pop eax   

  mov edi,eax                   ; 保存 API name 指针, 之后要清空   

  push edx   

  push ecx                                    ; save the Thunk pointers   

  push eax   

  push ebx   

  call _GetProcAddress[ebp]   

  test eax, eax   

  .if zero?   

    pop ecx   

    pop edx   

    jmp SkipInitIt   

  .endif   

  pop ecx   

  pop edx   

  ; 清零 API 名称   

  test PROTECTION_FLAGS[ebp], DESTROY_IMPORT_FLAG   

  .if !zero?   

    pushad   

    mov eax, edi   

    lea edi, @KillString[ebp]   

    call edi   

    popad   

  .endif         

  ; 填充 API 地址   

.else   

    ; 处理 ordinal import   

    push edx   

    push ecx                                    ; 保存 Thunk 指针   

    mov dword ptr eax, [ecx]   

    sub eax,080000000h   

    push eax   

    push ebx   

    call _GetProcAddress[ebp]   

    test eax,eax   

    jz    SkipInitIt   

    pop ecx   

    pop edx           

.endif   

  mov dword ptr [edx], eax   

  ; 加密输入表   

  ; eax = 当前函数地址   

   ; ebx = dll base   

   ; edx = non-org thunk pointer   

test PROTECTION_FLAGS[ebp], API_REDIRECT_FLAG   

                            _jmper  0CCh   

.if !zero?   

                                                                                     _junk 0e8h   

                             ;.IF [EBP+bNT]   

                                                                                     _junk 0E8h   

                              ; .IF EBX < 070000000h || EBX > 077FFFFFFh;      ; 非系统函数   

                                                                                     _junk 0e8h   

                                 ;   JMP SkipThunkRed   

                                 ; .ENDIF   

                             ;.ELSE   

                                                                                     _vxd   

                              ;  .IF EBX < 080000000h                ; 非系统函数   

                                                                                     _vxd   

                                 ;   JMP SkipThunkRed   

                                 ; .ENDIF   

                             ;.ENDIF   

  pushad   

  lea edi, Buff[ebp]   

  assume edi : ptr sReThunkInfo   

  mov esi, [edi].pNextStub   

  mov [edx], esi                ; 使 Thunk 指向 Stub Mem   

  pushad   

  xor ebx,  esi   

  ror ebx,  7                ; ebx == Key   

  xor eax, ebx   

  mov byte ptr [esi], 68h        ; push  NOT_REAL_ADDRESS   

  inc esi   

  mov [esi], eax   

  add esi, 4           

                    _vxd   

  mov word ptr [esi], 3481h        ; xor  dword ptr [esp], KEY   

  inc esi   

  inc esi   

  mov byte ptr [esi], 24h   

  inc esi   

  mov [esi], ebx   

                    _junk 0E9h   

  add esi, 4              

  mov byte ptr [esi], 0C3h          ; ret   

  popad   

      add [edi].pNextStub, sizeof sApiStub   

  assume edi : nothing   

  popad   

.endif   

add ecx,4   

add edx,4   

.ENDW   

add esi, sizeof sItInfo               ; 使 esi 指向下一个 IID   

.ENDW   

assume esi:nothing   

xor eax,eax   

inc eax   

SkipInitIt:   

dec eax   

.if !zero?   

popad   

ret   

.endif   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 把区段数改为天文数字防止转储   

test PROTECTION_FLAGS[ebp], ANTI_DUMP_FLAG   

.if !zero?   

mov eax, dwImageBase[ebp]   

add eax, [eax+3Ch]   

or [eax+06h], sp                    ; tELock 之代码   

.endif   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 初始化 OEP   

lea eax, [esp+_eax]   

push dwImageBase[ebp]   

pop [eax]   

mov ebx, dwOrgEntryPoint[ebp]   

add [eax], ebx   

ror dword ptr [eax], 7   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 清零 PE Header   

test PROTECTION_FLAGS[ebp], ERASE_HEADER_FLAG   

.if !zero?   

pushad   

mov eax, dwImageBase[ebp]   

add eax, [eax+03Ch]   

assume eax:ptr IMAGE_NT_HEADERS               ; -> PE Header   

mov ecx, [eax].OptionalHeader.SizeOfHeaders   

mov edi, dwImageBase[ebp]   

xor al, al   

assume eax:nothing   

@ZeroPEHeaderLoop:   

  stosb   

  loop @ZeroPEHeaderLoop   

popad   

.endif   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 检测调试器   

ifndef DEBUG_T   

pushad   

.if bNT[ebp]   

  push fs:[30h]   

  pop eax              ; eax -> PEB   

  movzx ebx, byte ptr [eax+2]   

  test bl, bl            ; 检测用户模式调试器   

  jz @@CheckDebuggerEnd   

  popad   

  popad   

  ret   

.else   

  push eax   

  sidt [esp-2]   

  pop ebx   

  add ebx,  18h   

  mov ecx,  [ebx+4]    ; int 3   

  mov cx,  [ebx]   

  mov edx,  [ebx+0Ch]  ; int 4   

  mov dx,  [ebx+8]   

  mov eax,  [ebx+14h]  ; int 5   

  mov ax,  [ebx+10h]          

  sub eax,  edx             

  sub edx,  ecx   

  sub eax,  edx   

  ; 系统初始化的 int 3 、int 4、int 5 之中断向量   

  ; 应在同一模块内，正常应有相同的高位字   

  ; 一般调试器修改 int 3，可能不改 int 4 & int 5   

  ; 检测 int 3 中断向量是否被改过   

  jz @@CheckDebuggerEnd   

  popad   

  popad   

  ret            

.endif   

@@CheckDebuggerEnd:   

popad   

endif   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 再次计算 Loader 校验和并比较   

lea eax, @Loader_Start[ebp]   

mov ecx, LOADER_CRC_CHECK_SIZE   

                  _vxd   

call GetChecksum   

                  _mmx 0C7h   

mov ebx, dwLoaderCRC[ebp]   

xor eax, ebx   

.if !zero?   

              _junk 02Ch   

    popad   

              _jmper 0E8h   

    ret   

.endif   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   

; 跳转层代码解码   

lea edi, @Loader_OEP_Jump_Code_Start[ebp]   

mov esi, edi   

mov ecx, @Loader_Crypt_OEP_Jump_Code_Size   

xor ebx, ebx   

 @OepJumpDecryptLoop:   

    lodsb   
```



```asm
    xor  al, OEP_JUMP_ENCRYPT_NUM   

    sub  al, bl   

    rol  al, 2   

    stosb   

    inc  ebx   

                       _jmper 0E9h   

    loop  @OepJumpDecryptLoop   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   

@Loader_OEP_Jump_Code_Start:   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━   

  ; 处理跳转表   

                       _junk 0EBh   

  test  PROTECTION_FLAGS[ebp], API_REDIRECT_FLAG   

  pushad   

                       _mmx 0E8h   

  jz  label_14_     


    mov  eax, [esp+20h+_eax]   

    rol  eax,  7   

    mov  esi,  eax   

    ; ESI == OEP   

    ; >>>>>>>> 搜索跳转表   

label_0_:  ;push  eax,ecx,edx,ebx,esp,ebp,esi,edi  ;1 字节   

  mov  ax,word  ptr  [esi]   

  cmp  al,50h   

  jb  label_1_   

  cmp  al,57h   

  ja  label_1_   

  inc  esi   

  jmp  label_0_   

label_1_:  ;push  xx  ;2 字节   

  cmp  al,6ah   

  jnz  label_2_   

  inc  esi   

  inc  esi   

  jmp  label_0_   

label_2_:  ;push  xxxxxxxx  ;5 字节   

  cmp  al,68h   

  jnz  label_3_   

  add  esi, 5   

  jmp  label_0_   

label_3_:  ;mov  eax,dword  ptr  [xxxxxxxx]  ;5 字节   

  cmp  al,0a1h   

  jnz  label_4_   

  add  esi, 5   

  jmp  label_0_   

label_4_:  ;sub  edx,edx    ;2 字节   

  cmp  ax,0d22bh   

  jnz  label_5_   

  inc  esi   

  inc  esi   

  jmp  label_0_   

label_5_:  ;push  [esp+xx]  ;4 字节   

  cmp  ax,74ffh   

  jnz  label_6_   

  add  esi, 4   

  jmp  label_0_   

label_6_:  ;mov  ebp,esp    ;2 字节   

  cmp  ax,0ec8bh   

  jnz  label_7_   

  inc  esi   

  inc  esi   

  jmp  label_0_   

label_7_:  ;call  xxxxxxx    ;5 字节   

  cmp  al,0e8h   

  jnz  label_8_   

  add  esi, 5   

  jmp  label_0_   

label_8_:   

  cmp  ax,0ff64h   

  jnz  label_9_   

  .if  byte  ptr  [esi+2]  ==  32h    ;push  dword  ptr  fs:[edx] ;3 字节   

    add  esi, 3   

  .elseif  byte  ptr  [esi+2]  ==  35h    ;push  dword  ptr  fs:[???] ;7 字节   

    add  esi, 7   

  .else   

    jmp  label_12_   

  .endif   

  jmp  label_0_   

label_9_:   

  cmp  ax,8964h   

  jnz  label_10_   

  .if  byte  ptr  [esi+2]  ==  22h    ;mov  dword  ptr  fs:[edx],esp ;3 字节   

    add  esi, 3   

  .elseif  byte  ptr  [esi+2]  ==  25h    ;mov  dword  ptr  fs:[???],esp ;7 字节   

    add  esi,  7   

  .else   

    jmp  label_12_   

  .endif   

  jmp  label_0_   

label_10_:        ;sub  esp,xx ; 3 字节   

  cmp  ax,0ec83h   

  jnz  label_11_   

  add  esi, 3   

  jmp  label_0_   

label_11_:   

  cmp  al,0A3h;  mov  dword  ptr  [xxxxxxxx],eax  5 字节   

  jnz  label_12_   

  add  esi, 5   

  jmp  label_0_   

label_12_:   

  cmp  al,0CCh;int  3 1byte   

  jnz  label_13_   

  inc  esi   

  jmp  label_0_   

label_13_:  ;跳转表！！！   

  cmp  ax, 25ffh   

  jnz  label_14_   

    .while  WORD  ptr  [esi]  ==  25ffh   

    mov  eax,  dword  ptr  [esi+2]   

    mov  BYTE  ptr  [esi],68h   

    inc  esi   

    mov  eax,  dword  ptr  [eax]   

    mov  [esi],eax   

    add  esi, 4   

    mov  byte  ptr  [esi],0c3h   

    inc  esi   

  .endw   

label_14_:; Game Over   

  popad   

  OepJumpCodeCont:   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ; 清零附加代码   

  xor  al, al   

  dec  al  ; al = 0FFh   

  lea  edi, @Loader_Start[ebp]   

  mov  ecx, (offset OepJumpCodeCont - offset @Loader_Start)   

  @LoaderZeroLoop:   

    stosb   

    loop  @LoaderZeroLoop   

  lea   EDI, [EBP+OFFSET @Loader_OEP_Jump_Code_End]   

  mov   ECX, (OFFSET @Loader_Crypt_End - OFFSET @Loader_OEP_Jump_Code_End)   

  @LoaderVarZeroLoop:   

    stosb   

    loop  @LoaderVarZeroLoop   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  ;                   飞 向 光 明 之 巅   

  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

  mov  eax, [esp+_eax]   

  mov  [esp-10h], eax   

  rol  dword  ptr  [esp-10h], 7   

  popad  ; 恢复起始寄存器   

      ; EAX - OEP   

                                        _jmper  0E9h   

  jmp  dword  ptr  [esp-30h]   

  ; 留言   

  db  'I am LiNSoN,you have got the OEP,Good job! QQ:150099182',0   

@Loader_OEP_Jump_Code_End:   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 清零 ASCII 字符串   

; EAX = ASCII 字符串首地址   

@KillString:   

  .while  byte  ptr  [eax]  !=  0   

    mov  byte  ptr  [eax],0   

    inc eax   

  .ENDW   

  ret   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

@SehHandler1 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD   

  push  EDI   

  mov   EAX,pContext   

  ASSUME EAX : PTR CONTEXT   

  mov   EDI, [EAX].regEdi   

  push  [EDI+SEH.SaveEip]   

  pop  [eax].regEip   

  mov   [eax].regEbp, EDI   

  mov   [EAX].regEax, 4     ; SI NOT detected !   

  mov   EAX,ExceptionContinueExecution   

  ASSUME EAX : NOTHING   

  POP  EDI   

  RET   

@SehHandler1 ENDP   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

@SehHandler2 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD   

  push  EDI   

  mov   EAX,pContext   

  ASSUME EAX : PTR CONTEXT   

  mov   EDI, [EAX].regEdi   

  push  [EDI+SEH.SaveEip]   

  pop  [eax].regEip   

  mov   [eax].regEbp, EDI   

  mov   [EAX].regEdi, 0     ; SI NOT detected !   

  mov   EAX,ExceptionContinueExecution   

  ASSUME EAX : NOTHING   

  POP  EDI   

  RET   

@SehHandler2 ENDP   

; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?   

; 结构定义   

sItInfo STRUCT   

  DllNameRVA        dd  ?   

  FirstThunk        dd  ?   

  OrgFirstThunk     dd  ?   

sItInfo ENDS   

sSEH STRUCT   

  OrgEsp            dd  ?   

  OrgEbp            dd  ?   

  SaveEip           dd  ?   

sSEH ENDS   

sReThunkInfo STRUCT   

  ApiStubMemAddr    DD  ?   

  pNextStub         DD  ?   

sReThunkInfo ENDS   

sApiStub  struct      ; 未使用此结构, 仅用来控制指针   

  PushOpcode      DB  ?;68h -> push  xxxxxxxx   

  PushDword       DD  ?;the address of api (xored)   

  XorOpcode       DB 3 dup(?);81h,34h,24h -> xor  dword  ptr  [esp],xxxxxxxx   

  XorKey          DD  ?;Key for xoring to get the true address   

  RetOpcode       DB  ?;C3h -> ret   

sApiStub  ends   

;----- LOADER VARIABLES -----   

dwImageBase           dd 0   

dwOrgEntryPoint       dd 0   

PROTECTION_FLAGS      dd 0   

dwCalcedCRC           dd 0   

dwLoaderCRC           dd 0   

bNT                   dd 0   

IIDInfo               db (SIZEOF sItInfo * MAX_IID_NUM) dup (0)   

SEH                   sSEH <0>   

_LoadLibrary          dd 0   

_GetProcAddress       dd 0   

szKernel32            db "Kernel32.dll",0   

dwKernelBase          dd 0   

szGetModuleHandle     db "GetModuleHandleA",0   

_GetModuleHandle      dd 0   

szVirtualProtect      db "VirtualProtect",0   

_VirtualProtect       dd 0   

szGetModuleFileName   db "GetModuleFileNameA",0   

_GetModuleFileName    dd 0   

szCreateFile          db "CreateFileA",0   

_CreateFile           dd 0   

szGlobalAlloc         db "GlobalAlloc",0   

_GlobalAlloc          dd 0   

szGlobalFree          db "GlobalFree",0   

_GlobalFree           dd 0   

szReadFile            db "ReadFile",0   

_ReadFile             dd 0   

szWriteFile       db  'WriteFile',0   

_WriteFile      dd  0   

szGetFileSize         db "GetFileSize",0   

_GetFileSize          dd 0   

szCloseHandle         db "CloseHandle",0   

_CloseHandle          dd 0   

szCreateEvent     db  "CreateEventA",0   

_CreateEvent    dd  0   

szIsDebuggerPresent   db "IsDebuggerPresent",0   

_IsDebuggerPresent  dd  0   

szGetCurrentProcessId  db  'GetCurrentProcessId', 0   

_GetCurrentProcessId   dd  0   

szGetCurrentProcess    db  'GetCurrentProcess', 0   

_GetCurrentProcess   dd  0   

szGetStdHandle     db  'GetStdHandle',0   

_GetStdHandle    dd  0   

szOpenProcess      db  'OpenProcess', 0   

_OpenProcess     dd  0
```



### 代码段示例

```
szReadProcessMemory    db  'ReadProcessMemory',0   

_ReadProcessMemory    dd  0   

szGetWindowsDirectory  db  'GetWindowsDirectoryA', 0   

_GetWindowsDirectory    dd  0   

szlstrcat          db  'lstrcatA', 0   

_lstrcat            dd  0   

szlstrcmpi          db  'lstrcmpiA', 0   

_lstrcmpi          dd  0   

szVirtualAlloc        db  'VirtualAlloc', 0   

_VirtualAlloc        dd  0   

szVirtualFree        db  'VirtualFree', 0   

_VirtualFree        dd  0   

szNTDLL            db 'NTDLL.DLL', 0   

dwNTDLLBase          dd  0   

szZwQueryInformationProcess  db 'ZwQueryInformationProcess', 0   

_ZwQueryInformationProcess    dd  0   

szExplorer          db  'Explorer.eXe', 0   

@Loader_Crypt_End:   

; 未被加密的变量:   

TlsBackupLabel:   

TlsBackup             IMAGE_TLS_DIRECTORY32 <0>   

ChecksumLabel:   

dwOrgChecksum           dd 0   

Buff                  db 0  ; 缓冲区，大小: 2000h(VS) - @Loader_Code_Size   

@Loader_End: 
```

### 建议与宏定义

建议用 `edit plus for masm` 观看。

顺送 `junkxx` 宏...呵呵

```
_junk  macro  prefix   
   IFDEF DEBUG_T   
;    db 4 dup(90h)   
   ELSE   
    jmp  $+5   
;    db  prefix   
;    db  NOT prefix+1   
    db  0C7h   
    db  084h   
    db  prefix   
   ENDIF   
   endm   
```

```
; 9 bytes macro   
_junk  macro  prefix   
   IFDEF DEBUG_T   
;    db 4 dup(90h)   
   ELSE   
    jmp  $+5   
;    db  prefix   
;    db  NOT prefix+1   
    db  0C7h   
    db  084h   
    db  prefix   
   ENDIF   
   endm   
```

*(Note: The original text had repeated macro definitions or similar structures. I will consolidate the unique macros found in the input.)*

```
_junk  macro  prefix   
   IFDEF DEBUG_T   
;    db 4 dup(90h)   
   ELSE   
    jmp  $+5   
;    db  prefix   
;    db  NOT prefix+1   
    db  0C7h   
    db  084h   
    db  prefix   
   ENDIF   
   endm   
```

```
; 9 bytes macro   
_trash  macro some_byte   
   IFDEF DEBUG_T   
;    db 10 dup(90h)   
   ELSE   
    call  $+6    ; you must step into(F8) if you want to continue   
    db  some_byte  ; tracing   
    add  esp,4    ; fix stack(dont change any! register)   
          ; add esp,4=3bytes instruction   
   ENDIF   
   endm   
```

```
_jmper  macro  shit   
   IFDEF DEBUG_T   
;    db 7 dup(90h)   
   ELSE   
    jl  $+5   
    jmp  $+5   
    db  shit   
    jz  $-3   
   ENDIF   
   endm   
```

```
_vxd  macro   
   IFDEF DEBUG_T   
;    db 5 dup(90h)   
   ELSE   
    jmp  $+4   
    db  0CDh,20h   
   ENDIF   
   endm   
```

```
_mmx  macro  shit   
   IFDEF DEBUG_T   
;    db 4 dup(90h)   
   ELSE   
    jmp  $+4   
    db  0Fh   
    db  shit   
   ENDIF   
   endm   
```

```
_svkp1  macro  junkcode   
   ;        call    loc_1   
   ;        _ONE_BYTE_JUNKCODE_   
   ;loc_1:        call    loc_2   
   ;        _TWO_BYTE_JUNKCODE_   
   ;loc_2:        add    dword ptr [esp], 0Bh   
   ;        add    dword ptr [esp+04h], 13h   
   ;        ret   
   ;        _ONE_BYTE_JUNKCODE_   
   ifndef  DEBUG_T   
    db  0E8h,01h,00h,00h,00h,junkcode,0E8h,02h,00h,00h,00h,junkcode,junkcode,83h,04h,24h,0Bh,83h,44h,24h,04h,13h,0C3h,junkcode   
   endif   
   endm   
```

```
_svkp2  macro  junkcode   
   ;        call    loc_1   
   ;        _TWO_BYTE_JUNKCODE_   
   ;loc_1:        add    dword ptr [esp]h, 08h   
   ;        ret   
   ;        _ONE_BYTE_JUNKCODE_   
   ifndef  DEBUG_T   
    db  0E8h,02h,00h,00h,00h,junkcode,junkcode,83h,04h,24h,08h,0C3h,junkcode   
   endif   
   endm   
```

```
_svkp3  macro  junkcode   
   ;        pushad   
   ;        call    loc_1   
   ;        _ONE_BYTE_JUNKCODE_   
   ;        jmp    loc_4   
   ;loc_1:        pop    eax   
   ;        jmp    loc_2   
   ;        _ONE_BYTE_JUNKCODE_   
   ;loc_2:        inc    eax   
   ;        jmp    loc_3   
   ;        _ONE_BYTE_JUNKCODE_   
   ;loc_3:        jmp    eax   
   ;loc_4:        popad   
   ifndef  DEBUG_T   
    db  60h,0E8h,03h,00h,00h,00h,junkcode,0EBh,0Ah,58h,0EBh,01h,junkcode,40h,0EBh,01h,junkcode,0FFh,0E0h,61h   
   endif   
   endm   
```

```
_dbpe1  macro  junkcode   

;      pushf   
;      push  0Ah   
;loc_1:    jnb    loc_3   
;      jmp    loc_2   
;          _TWO_BYTE_JUNKCODE_   
;loc_2:    call  loc_4   
;          _TWO_BYTE_JUNKCODE_   
;loc_3:    jnb    loc_2   
;          _TWO_BYTE_JUNKCODE_   
;loc_4:    add    esp,4   
;      jmp    loc_5   
;          _TWO_BYTE_JUNKCODE_   
;loc_5:    dec    dword ptr [esp]   
;      jno    loc_6   
;          _ONE_BYTE_JUNKCODE_   
;loc_6:    jns    loc_1   
;      jp    loc_7   
;          _ONE_BYTE_JUNKCODE_   
;loc_7:    add    esp,4   
;      popf   
;      jmp    loc_8   
;          _ONE_BYTE_JUNKCODE_   
;loc_8:    ......   
   ifndef  DEBUG_T   
    db    9Ch,6Ah,junkcode,73h,0Bh,0EBh,02h,junkcode,junkcode,0E8h,06h,00h,00h,00h,junkcode,junkcode,73h,0F7h,junkcode,junkcode,83h,0C4h,04h,0EBh,02h,junkcode,junkcode,0FFh,0Ch,24h,71h,01h,junkcode,79h,0E0h,7Ah,01h,junkcode,83h,0C4h,04h,9Dh,0EBh,01h,junkcode   
   endif   
   endm 
```

### `loaderstuff.inc`

```
;   
; pe header structure   
;   
  signature      equ  0  ; PE,0,0   
  cputype        equ  4  ; i386,i486 etc...   
  numObj         equ  6  ; count of sections   
  timestamp      equ  8   
  symboltable    equ  12   
  symbolnum      equ  16   
  NtHeaderSize   equ  20   
  DllFlags       equ  22  ; file is DLL or PE?   
  reserved3      equ  24   
  lmajor         equ  26  ; linker major   
  lminor         equ  27  ; linker minor   
  codesize       equ  28   
  initdatasize   equ  32   
  uninitdatasize equ  36   
  entrypointRVA  equ  40  ; starting address   
  baseofcode     equ  44   
  baseofdata     equ  48   
  imagebase      equ  52   
  objalign       equ  56  ; align between objects in file   
  filealign      equ  60  ; align between objects in memory   
  imagesize      equ  80   
  headersize     equ  84   
  checksum       equ  88  ; checksum   
  dllflags       equ  94   
  loaderflags    equ  112  ; not used at all   
  edatadir       equ  120   
  import         equ  128  ; import table RVA   
  importsize     equ  132  ; imports size   
  resource       equ  136   
  ressize        equ  140   
  reloc          equ  160  ; relocation table RVA   
  relocsize      equ  164  ; relocation table size   
  debug          equ  168   
  debugsize      equ  172   
  tls            equ  192   
  tlssize        equ  196   
  entryiat       equ  216   
  entryiatsize   equ  224   
  dimport        equ  224   
  dimportsize    equ  232   
```

```
;   
; object structure   
;   
  objname        equ  0  ; section name   2DWORD   
  objvsize       equ  8  ; virtual size   DWORD   
  objrva         equ  12  ; rva of section  DWORD   
  objpsize       equ  16  ; raw size   DWORD   
  objpoff        equ  20  ; raw offset   DWORD   
  objrle         equ  24  ; relocations   DWORD   
  objline        equ  28  ; linenumbers   DWORD   
  objrlenum      equ  32  ; no of relocations  WORD   
  objlinenum     equ  34  ; no of linenumbers  WORD   
  objflags       equ  36  ; object flags   DWORD   
  objlen         equ  40  ; total object length   
```

```
;   
; SetFilePointer stuff   
;   
  FILE_BEGIN     equ  0   
  FILE_CURRENT   equ  1   
  FILE_END       equ  2   
```

```
;   
; mapping stuff   
;   
  PAGE_READONLY    equ  2   
  PAGE_READWRITE   equ  4     
  PAGE_WRITECOPY   equ  8     
  SECTION_QUERY    equ  1   
  SECTION_MAP_WRITE  equ  2   
  SECTION_MAP_READ   equ  4   
  SECTION_MAP_EXECUTE  equ  8   
  SECTION_MAP_EXTEND_SIZE  equ  10   
  FILE_MAP_COPY    equ  SECTION_QUERY   
  FILE_MAP_WRITE   equ  SECTION_MAP_WRITE   
  FILE_MAP_READ    equ  SECTION_MAP_READ   
  MEM_COMMIT               equ     00001000h   
  MEM_RESERVE              equ     00002000h   
  MEM_DECOMMIT             equ     00004000h   
  MEM_RELEASE              equ     00008000h   
  MEM_FREE                 equ     00010000h   
  MEM_PRIVATE              equ     00020000h   
  MEM_MAPPED               equ     00040000h   
  MEM_TOP_DOWN             equ     00100000h   
  OF_READ        equ  0h  ;open file only for read   
  OF_WRITE       equ  1h  ;open file only for write   
  OF_READWRITE   equ  2h  ;open file only for read+write   
  OFN_HIDEREADONLY   equ  4   
```



## 常量定义

`OFN_OVERWRITEPROMPT` `equ` `2`  
`FILE_ATTRIBUTE_NORMAL` `equ` `080h`  
`OPEN_EXISTING` `equ` `3`  
`GENERIC_READ` `equ` `80000000h`  
`GENERIC_WRITE` `equ` `40000000h`  
`FILE_SHARE_READ` `equ` `0001h`  
`FILE_ATTRIBUTE_ARCHIVE` `equ` `00000020h`  
`CREATE_NEW` `equ` `1`  
`CREATE_ALWAYS` `equ` `2`  
`OPEN_EXISTING` `equ` `3`  
`OPEN_ALWAYS` `equ` `4`  
`TRUNCATE_EXISTING` `equ` `5`  
`CR` `equ` `0Dh`  
`LF` `equ` `0Ah`  
`NORMAL_PRIORITY_CLASS` `equ` `00000020h`  
`IDLE_PRIORITY_CLASS` `equ` `00000040h`  
`HIGH_PRIORITY_CLASS` `equ` `00000080h`  
`REALTIME_PRIORITY_CLASS` `equ` `00000100h`  
`ERROR_ALREADY_EXISTS` `equ` `183`  
`PC_WRITEABLE` `equ` `00020000H`  
`PC_USER` `equ` `00040000H`  
`PC_STATIC` `equ` `20000000H`  
`PC_LOCKED` `equ` `00000080H`  
`VMM` `equ` `00001h`  

### 标志位定义

`_cf` `equ` `0000000000000001b` ; carry flag  
`_pf` `equ` `0000000000000100b` ; parity flag  
`_af` `equ` `0000000000010000b` ; auxilary flag  
`_zf` `equ` `0000000001000000b` ; zero flag  
`_sf` `equ` `0000000010000000b` ; sign flag  
`_tf` `equ` `0000000100000000b` ; trap flag  
`_if` `equ` `0000001000000000b` ; interrupt flag  
`_df` `equ` `0000010000000000b` ; direction flag  
`_of` `equ` `0000100000000000b` ; overflow flag  

### 栈偏移定义

; pushad  
; usage:  
; pushad  
; ...  
; ...  
; mov [esp+_eax],eax  
; popad  

`_edi` `equ` `0`  
`_esi` `equ` `4`  
`_ebp` `equ` `8`  
`_esp` `equ` `12`  
`_ebx` `equ` `16`  
`_edx` `equ` `20`  
`_ecx` `equ` `24`  
`_eax` `equ` `28`  

## 宏与结构体定义

### 宏定义

`VxDCall` `macro` `Service`  
`db` `0CDh`  
`db` `020h`  
`dd` `Service`  
`endm`  

`VMMCall` `macro` `ServiceVMM`  
`db` `0CDh,20h`  
`dw` `ServiceVMM`  
`dw` `VMM`  
`endm`  

### 结构体定义

`MEMF` `struct`  
`mem_ptr` `dd` `?`  
`mem_handle` `dd` `?`  
`file_handle` `dd` `?`  
`file_size` `dd` `?`  
`MEMF` `ends`  

`MEMF_` `equ` `4*4`  

`CONSOLESCREENBUFFERINFO` `struct`  
`c_width` `dw` `?`  
`c_height` `dw` `?`  
`c_X` `dw` `?`  
`c_Y` `dw` `?`  
`c_attribs` `dw` `?`  
`c_smallrect` `dd` `2 dup(?)`  
`c_maxwidth` `dw` `?`  
`c_maxheight` `dw` `?`  
`CONSOLESCREENBUFFERINFO` `ends`  

`CONSOLESCREENBUFFERINFO_` `equ` `2+2+2+2+2+8+2+2`  

### 标准句柄定义

`STD_OUTPUT_HANDLE` `equ` `-11`  
`STD_INPUT_HANDLE` `equ` `-10`  
`STD_ERROR_HANDLE` `equ` `-12`  

### 其他宏定义

`callW` `macro` `api_func`  
`IFDEF` `USE_TASM`  
`extrn` `api_func:proc`  
`ENDIF`  
`call` `api_func`  
`endm`  

`pushb` `macro` `bytevalue`  
`db` `6Ah,bytevalue`  
`endm`  

`fcall` `macro` `popreg,junxx`  
`IFDEF` `DEBUG_T`  
`ELSE`  
`call` `$+6`  
`db` `junxx`  
`pop` `popreg`  
`ENDIF`  
`endm`  

`ud2_1` `macro`  
`db` `0Fh,0Bh` ; undocumented opcode to invoke exception,this one UD2  
`endm`  

`ud2_2` `macro`  
`db` `0Fh,0B9h` ; this is also UD2 but in disasembler it's showed as ??? unknown opcode  
`endm`  

`salc` `macro`  
`db` `D6h` ; set AL on carry flag  
`endm`  

`_sal` `macro` `byte` ; !real! sal eax instruction,undocumented,sal eax,16 is translated by asembler like tasm&masm as shl eax,16  
`db` `0C1h,0F0h,byte`  
`endm`  

---

## crypt.asm

### 常量计算

`@Loader_Code_Size` = `(offset @Loader_End - offset @Loader_Start)`  
`@Loader_Crypt_Size` = `(OFFSET @Loader_Crypt_End - OFFSET @Loader_Crypt_Start)`  
`@Loader_Crypt_OEP_Jump_Code_Size` = `(OFFSET @Loader_OEP_Jump_Code_End - OFFSET @Loader_OEP_Jump_Code_Start)`  
`CHECKSUM_SKIP_SIZE` = `5` ; (don't include the saved checksum itself in the checksum calculation)  
`TLS_BACKUP_ADDR` = `(offset TlsBackupLabel - offset @Loader_Start)`  
`CHECKSUM_ADDR` = `(OFFSET ChecksumLabel - OFFSET @Loader_Start)`  
`IT_SIZE` = `060h`  
`MAX_SECTION_NUM` = `20`  
`MAX_IID_NUM` = `30`  
`OEP_JUMP_ENCRYPT_NUM` = `('X')`  
`LOADER_CRC_CHECK_SIZE` = `(OFFSET @Loader_OEP_Jump_Code_Start - OFFSET @Loader_Start)`  
`VAR_PER_SIZE` = `030h`  
`SEC_PER_SIZE` = `030h`  

### 数据段

`.Const`  
`szDone` `db` `"文件加密成功 !",0`  
`szDoneCap` `db` `":)",0`  
`szFileErr` `db` `"无法访问文件 :D",0`  
`szNoPEErr` `db` `"无效的 PE 文件 :(",0`  
`szNoMemErr` `db` `"内存不足 -_-;",0`  
`szFsizeErr` `db` `"奇怪的文件 !",0`  
`szNoRoom4SectionErr` `db` `"空间不足以加入新区段 :(",0`  
`szSecNumErr` `db` `"区段太多 !",0`  
`szIIDErr` `db` `"Too much ImageImportDescriptors !",0`  
`ALIGN_CORRECTION` `dd` `01000h` ; this big value is e.g. needed for WATCOM compiled files  
`DEPACKER_SECTION_NAME` `dd` `0`  
`szKernel` `db` `"KeRnEl32.dLl",0`  
`szLoadLibrary` `db` `"LoadLibraryA",0`  
`szGetProcAddress` `db` `"GetProcAddress",0`  

`.Data`  
`pMap` `dd` `0`  
`dwBytesRead` `dd` `0`  
`dwBytesWritten` `dd` `0`  
`pMem` `dd` `0`  
`dwFsize` `dd` `0`  
`dwOutPutSize` `dd` `0`  
`dwNewFileEnd` `dd` `0`  
`dwNTHeaderAddr` `dd` `0`  
`dwSectionNum` `dd` `0`  
`dwNewSectionRO` `dd` `0`  
`dwOrgITRVA` `dd` `0`  
`hFile` `dd` `0`  

### 代码段

`.Code`

`Protect` `PROC` `szFname : LPSTR, hDlg : HWND, dwProtFlags : DWORD`

`assume` `fs : nothing`  
`CALL` `InitRandom`  
;----- MAP THE FILE -----  
`invoke` `CreateFile, szFname, GENERIC_WRITE + GENERIC_READ, FILE_SHARE_WRITE + FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0`  
`cmp` `eax, INVALID_HANDLE_VALUE`  
`jz` `FileErr`  
`mov` `hFile, eax`  
`invoke` `GetFileSize, hFile, 0`  
`.IF` `eax == 0`  
`push` `hFile`  
`call` `CloseHandle`  
`jmp` `FsizeErr`  
`.ENDIF`  
`mov` `dwFsize, eax`  
`mov` `eax, dwFsize`  
`add` `eax, IT_SIZE`  
`add` `eax, @Loader_Code_Size`  
`add` `eax, ALIGN_CORRECTION`  
`mov` `dwOutPutSize, eax`  
`push` `eax`  
`push` `GMEM_FIXED + GMEM_ZEROINIT`  
`call` `GlobalAlloc`  
`.IF` `eax == NULL`  
`push` `hFile`  
`call` `CloseHandle`  
`jmp` `MemErr`  
`.ENDIF`  
`mov` `pMem, eax`  
`invoke` `ReadFile, hFile, pMem, dwFsize, offset dwBytesRead, NULL`  
; ----- check the PE Signature and get some needed values -----  
`mov` `edi, pMem`  
`.IF` `word ptr [edi] != 'ZM'`  
`push` `pMem`  
`call` `GlobalFree`  
`push` `hFile`  
`call` `CloseHandle`  
`jmp` `PEErr`  
`.ENDIF`  
`add` `edi, [edi+3Ch]`  
`.IF` `word ptr [edi] != 'EP'`  
`push` `pMem`  
`call` `GlobalFree`  
`push` `hFile`  
`call` `CloseHandle`  
`jmp` `PEErr`  
`.ENDIF`  
; 备份文件  
`pushad`  
`invoke` `IsDlgButtonChecked, hDlg, IDC_BACKUP`  
`.IF` `eax == BST_CHECKED`  
`push` `offset cFname` ; find file name length  
`call` `lstrlen`  
`xchg` `eax, ecx`  
`push` `ecx`  
`push` `ecx` ; save used regs  
`push` `edx`  
`push` `PAGE_READWRITE` ; access type  
`push` `MEM_RESERVE or MEM_COMMIT` ; flags  
`push` `ecx` ; size  
`push` `0`  
`call` `VirtualAlloc`  
`pop` `edx`  
`pop` `ecx`  
`add` `eax, 5`  
; allocate memory page  
`xchg` `eax, edi` ; edi --> memory  
`mov` `esi, offset cFname` ; original file name  
`pop` `ecx` ; pop size  
`push` `ecx` ; save size  
`push` `edi` ; save pointer to memory  
`push` `0` ; fail if exist  
`push` `edi` ; new file name  
`push` `esi` ; file name  
`push` `edi` ; pointer to backup file name ptr  
`rep` `movsb` ; copy original file name  
`mov` `eax, 'KAB.'` ; add .BAK suffix  
`stosd` ; store it  
`call` `DeleteFile` ; delete backup file if exists  



```asm
  call  CopyFile      ; make backup   

  pop  eax        ; pop memory ptr   

  pop  ecx        ; pop memory size   

  push  MEM\_DECOMMIT   

  push  ecx        ; memory size   

  push  eax        ; memory pointer   

  call  VirtualFree   

  .endif   

  popad   

  mov dwNTHeaderAddr,edi   

  assume edi : ptr IMAGE\_NT\_HEADERS   

  push [edi].OptionalHeader.DataDirectory[SIZEOF IMAGE\_DATA\_DIRECTORY].VirtualAddress   

  pop dwOrgITRVA   

  push word ptr [edi].FileHeader.NumberOfSections   

  pop word ptr dwSectionNum   

  .IF dwSectionNum > MAX\_SECTION\_NUM   

     JMP SecNumErr   

  .ENDIF   

  push [edi].OptionalHeader.AddressOfEntryPoint   

  pop dwOrgEntryPoint   

  push [edi].OptionalHeader.ImageBase   

  pop dwImageBase   

  ;----- 删除 Bound Import & IAT DIRECTORIES -----   

  XOR  EAX, EAX   

  MOV  ECX, 4   

  LEA  EDI, [EDI].OptionalHeader.DataDirectory[11 \*SIZEOF IMAGE\_DATA\_DIRECTORY].VirtualAddress   

  assume edi : nothing   

   DirDelLoop:   

        STOSD   

        LOOP DirDelLoop   

  ;----- ENCRYPT DLL/API NAMES & SAVE IT & DESTROY IID's -----   

  PUSH dwOrgITRVA   

  PUSH pMem   

  CALL RVA2Offset   

  PUSH EAX   

  PUSH pMem   

  CALL ProcessOrgIT   

  OR   EAX, EAX   

  .IF  ZERO?   

     PUSH pMem   

     CALL GlobalFree   

     PUSH hFile   

     CALL CloseHandle   

     JMP  IIDErr   

  .ENDIF   

  ;----- ADD THE PACKER SECTION -----   

  push pMem   

  call AddSection   

  .IF eax == 0   

     push pMem   

     call GlobalFree   

     push hFile   

     call CloseHandle   

     jmp NoRoom4SectionErr   

  .ENDIF   

  ;----- CREATE PACKER IMPORT TABLE -----   

  xchg eax,esi          ; esi -> pointer to Section Header of the new section   

  assume esi : ptr IMAGE\_SECTION\_HEADER   

  mov eax,[esi].PointerToRawData   

  MOV  dwNewSectionRO, EAX   

  add eax,pMem   

  push [esi].VirtualAddress   

  push eax   

  call AssembleIT   

  ;---- REPLACE TLS TABLE -----   

  push [esi].VirtualAddress   

  push pMem   

  call ProcessTlsTable   

  ;------ 加密块 -----   

  pushad   

  ; 生成 PER   

  PUSH SEC\_PER\_SIZE   

  PUSH OFFSET SecDecryptBuff   

  PUSH OFFSET SecEncryptBuff   

  CALL MakePER   

  ; 加密数据!   

  test  dwProtFlags, NO\_CRYPT\_SECTION\_FLAG   

  .if  zero?   

      mov eax, pMem   

      xor  ebx, ebx       


      call CryptPE   

  .endif   

  popad   

  ; ----- UPDATE PE HEADER -----   

  mov edi,dwNTHeaderAddr   

  assume edi : ptr IMAGE\_NT\_HEADERS    ; edi -> pointer to PE header   

  ; ImportTable RVA ...   

  push [esi].VirtualAddress   

  pop [edi].OptionalHeader.DataDirectory[SIZEOF IMAGE\_DATA\_DIRECTORY].VirtualAddress   

  ; EntryPoint...   

  mov eax,[esi].VirtualAddress   

  add eax,IT\_SIZE   

  mov [edi].OptionalHeader.AddressOfEntryPoint,eax   

  ; SizeOfImage ...   

  mov eax,[esi].VirtualAddress   

  add eax,[esi].Misc.VirtualSize   

  mov [edi].OptionalHeader.SizeOfImage,eax   

  add eax,[esi].Misc.VirtualSize   

  ; 连接器版本号   

  mov [edi].OptionalHeader.MajorLinkerVersion,88;主版本号   

  mov [edi].OptionalHeader.MinorLinkerVersion,88;次版本号   

  ; save protection flags...   

  push dwProtFlags   

  pop PROTECTION\_FLAGS   

  assume esi : nothing   

  assume edi : nothing   

  ; ----- CALCULATE THE NEW EOF -----   

  mov eax,dwNewSectionRO   

  add eax,IT\_SIZE   

  add eax, @Loader\_Code\_Size   

  mov dwNewFileEnd,eax   

  ; ----- COPY LOADER CODE TO FILE MEMORY & DO CHECKSUM STUFF ------   

  mov edi,dwNewSectionRO   

  add edi,IT\_SIZE   

  add edi,pMem   

  mov esi,offset @Loader\_Start   

  mov ecx,@Loader\_Code\_Size   

  rep movsb   

  ;----- ENCRYPT OEP JUMP CODE -----   

  MOV  EDI, pMem   

  ADD  EDI, dwNewSectionRO   

  ADD  EDI, IT\_SIZE   

  ADD  EDI, (OFFSET @Loader\_OEP\_Jump\_Code\_Start - OFFSET @Loader\_Start)   

  MOV  ESI, EDI   

  MOV  ECX, @Loader\_Crypt\_OEP\_Jump\_Code\_Size   

  XOR  EBX, EBX   

   OepJumpEncryptLoop:   

        LODSB   

     ROR  AL, 2   

     ADD  AL, BL   

     XOR  AL, OEP\_JUMP\_ENCRYPT\_NUM   

     STOSB   

     INC EBX   

     LOOP OepJumpEncryptLoop   

  ;----- ENCRYPT LOADER -----   

  ; generate PER   

  PUSH VAR\_PER\_SIZE   

  MOV  EAX, pMem   

  ADD  EAX, dwNewSectionRO   

  ADD  EAX, IT\_SIZE   

  ADD  EAX, (OFFSET VarDecryptBuff - OFFSET @Loader\_Start)   

  PUSH EAX   

  PUSH OFFSET VarEncryptBuff   

  CALL MakePER   

  ; encryption !   

  MOV  EDI, pMem   

  ADD  EDI, dwNewSectionRO   

  ADD  EDI, IT\_SIZE   

  ADD  EDI, (OFFSET @Loader\_Crypt\_Start - OFFSET @Loader\_Start)   

  MOV  ECX, @Loader\_Crypt\_Size   

  MOV  ESI, EDI   

   @@VarEncryptionLoop:   

        LODSB   

  VarEncryptBuff DB VAR\_PER\_SIZE DUP (0)   

        STOSB   

        LOOP @@VarEncryptionLoop   

  ;----- CALCULATE CHECKSUM -----   

  mov eax,pMem   

  mov ecx,dwNewFileEnd   

  sub ecx,CHECKSUM\_SKIP\_SIZE   

  call GetChecksum   

  mov dwOrgChecksum,eax   

  ;----- PASTE CHECKSUM ------   

  MOV EAX, pMem   

  ADD EAX, IT\_SIZE   

  ADD EAX, dwNewSectionRO   

  ADD EAX, CHECKSUM\_ADDR   

  MOV EDX, dwOrgChecksum   

  MOV DWORD PTR [EAX], EDX   

  ; ----- WRITE FILE MEMORY TO DISK -----   

  invoke SetFilePointer,hFile,0,NULL,FILE\_BEGIN   

  invoke WriteFile,hFile,pMem,dwOutPutSize,offset dwBytesWritten,NULL   

  ; ------ FORCE CALCULATED FILE SIZE ------   

  invoke SetFilePointer,hFile,dwNewFileEnd,NULL,FILE\_BEGIN   

  invoke SetEndOfFile,hFile   

  invoke MessageBox,hDlg,offset szDone,offset szDoneCap,MB\_ICONINFORMATION   

  ; ----- CLEAN UP -----   

  push pMem   

  call GlobalFree   

  push hFile   

  call CloseHandle   

@@Exit:   

  ret   

;----- ERROR MESSAGES -----   

MemErr:   

  mov eax,offset szNoMemErr   

  jmp ShowErr   

PEErr:   

  mov eax,offset szNoPEErr   

  jmp ShowErr   

FileErr:   

  mov eax,offset szFileErr   

  jmp ShowErr   

NoRoom4SectionErr:   

  mov eax,offset szNoRoom4SectionErr   

  jmp ShowErr   

FsizeErr:   

  mov eax,offset szFsizeErr   

  jmp ShowErr   

SecNumErr:   

  mov eax,offset szSecNumErr   

  jmp ShowErr   

IIDErr:   

  MOV  EAX, OFFSET szIIDErr   

  JMP  ShowErr   

ShowErr:   

  invoke MessageBox,hDlg,eax,offset szErr,MB\_ICONERROR   

  jmp @@Exit   

Protect ENDP   

;--------- functions -----------------   

; esi = CryptStart   

; ecx = CryptSize   

EncryptSec:   

  mov edi,esi   

SecEncryptLoop:   

  LODSB   

  SecEncryptBuff DB SEC\_PER\_SIZE DUP (0)   

  STOSB   

  LOOP SecEncryptLoop   

  RET   

; return values:   

; 0 - no room for a new section   

; 1 - file already encrypted   

; else: returns a pointer to the IMAGE\_SECTION\_HEADER struct of the new section   

AddSection PROC USES edi esi ebx ecx edx, pMem\_ : LPVOID   

  LOCAL dwSecNum   : DWORD   

  mov edi,pMem\_   

  add DWORD PTR edi,[edi+03Ch]   

  assume edi : ptr IMAGE\_NT\_HEADERS      ; edi -> pointer to PE header   

  ; check whether there's room for a new section   

  xor eax,eax   

  mov ax,[edi].FileHeader.NumberOfSections   

  mov dwSecNum,eax   

  mov ecx,SIZEOF IMAGE\_SECTION\_HEADER   

  imul eax,eax                             ; eax contains the size of the whole section header   

  add eax,SIZEOF IMAGE\_SECTION\_HEADER     ; add the size being needed for our new section   

  mov ecx,edi          ; ecx -> address of PE Header   

  sub ecx,pMem\_        ; sub ecx Map Base   

  add ecx,eax          ; ecx + calculated header sizes   

  add ecx,0F8h         ; add the size of the PE header   

  .IF ecx > [edi].OptionalHeader.SizeOfHeaders   

     xor eax,eax   

     jmp @@ExitProc\_AS   

  .ENDIF   

  ; create a new section   

  mov esi,edi   

  add esi,0F8h   

  assume esi : ptr IMAGE\_SECTION\_HEADER  ; esi -> pointer to section headers   

  ; go to the last section
```



```asm
mov edx,dwSecNum   
sub edx,1   
.REPEAT   
   ; force the writeable flag   
   mov eax,[esi].Characteristics   
   or eax,080000000h   
   mov [esi].Characteristics,eax   
   add esi,SIZEOF IMAGE_SECTION_HEADER   
   dec edx   
.UNTIL edx == 0   
; start to build the new section   
mov edx,esi   
add edx,SIZEOF IMAGE_SECTION_HEADER    ; edx -> pointer to the new section   
assume edx : ptr IMAGE_SECTION_HEADER   
; VirtualAddress...   
mov eax,[esi].VirtualAddress   
add eax,[esi].Misc.VirtualSize   
push 01000h   
push eax   
call PEAlign   
mov [edx].VirtualAddress,eax   
; VirtualSize..   
mov [edx].Misc.VirtualSize,02000h   
; RawSize..   
mov eax,IT_SIZE   
add eax,@Loader_Code_Size   
mov [edx].SizeOfRawData,eax   
; Section name   
lea  eax,[edx].Name1   
push DEPACKER_SECTION_NAME   
pop  [eax]   
MOV  DWORD PTR [EAX+4],0   
; Characteristics   
mov [edx].Characteristics,0E00000E0h   
; RawOffset   
mov eax,[esi].PointerToRawData   
add eax,[esi].SizeOfRawData   
push 0200h   
push eax   
call PEAlign   
mov [edx].PointerToRawData,eax     
mov eax,edx          ; eax -> will be returned   
; update the PE header   
inc [edi].FileHeader.NumberOfSections   
assume edx : nothing   
assume esi : nothing   
assume edi : nothing   
@@ExitProc_AS:   
ret   
AddSection ENDP   
```

```asm
AssembleIT PROC USES ebx ecx edx esi edi, pAddress4IT : LPVOID, dwNewSectionVA : DWORD   
mov esi,pAddress4IT          ; esi -> base of the new IT     
; Zero the memory for the new IT   
mov eax,pAddress4IT   
mov ecx,IT_SIZE   
  ZeroMem:   
mov byte ptr [eax],0   
inc eax   
loop ZeroMem   
; build a new,nice ImportTable :)   
mov ebx,esi   
mov eax,SIZEOF IMAGE_IMPORT_DESCRIPTOR   
xor edx,edx   
mov ecx,2   
mul ecx   
add ebx,eax           ; make ebx point after the terminating IID   
assume esi:ptr IMAGE_IMPORT_DESCRIPTOR   
mov eax,ebx                             ; process the IID Name   
sub eax,esi   
add eax,dwNewSectionVA   
mov [esi].Name1,eax   
push esi   
mov esi,offset szKernel   
mov edi,ebx   
.REPEAT   
   lodsb   
   stosb   
.UNTIL byte ptr [esi] == 0   
pop esi   
mov ebx,edi   
inc ebx   
mov eax,ebx                             ; process the FirstThunk pointers   
sub eax,esi   
add eax,dwNewSectionVA   
mov [esi].FirstThunk,eax                 
mov edx,ebx   
add edx,10   
mov eax,edx   
sub eax,esi   
add eax,dwNewSectionVA   
mov [ebx],eax   
add edx,2   
push esi   
mov esi,offset szLoadLibrary   
mov edi,edx   
.REPEAT   
   lodsb   
   stosb   
.UNTIL byte ptr [esi] == 0   
pop esi   
mov edx,edi   
add ebx,4   
mov eax,edx   
sub eax,esi   
add eax,dwNewSectionVA   
mov [ebx],eax   
add edx,2   
mov esi,offset szGetProcAddress   
mov edi,edx   
.REPEAT   
   lodsb   
   stosb   
.UNTIL byte ptr [esi] == 0   
assume esi : nothing   
ret   
AssembleIT ENDP   
```

```asm
ProcessTlsTable PROC USES edi ebx esi ecx, pFileMem : LPVOID, CryptSectionVA : DWORD   
LOCAL  pTlsDirAddr : LPVOID   
; check whether there's a tls table   
mov edi,pFileMem   
add edi,[edi+03Ch]       ; edi -> pointer to PE header   
assume edi : ptr IMAGE_NT_HEADERS   
lea ebx,[edi].OptionalHeader.DataDirectory[SIZEOF IMAGE_DATA_DIRECTORY * 9].VirtualAddress   
mov pTlsDirAddr,ebx   
mov ebx,[ebx]   
assume edi : nothing   
cmp ebx,0        ; no tls section   
jz ExitTlsFixProc   
; get a RAW pointer to the tls table   
push ebx   
push pFileMem   
call RVA2Offset   
cmp eax,0   
jz ExitTlsFixProc   
mov esi,pFileMem   
add esi,eax      ; esi -> pointer to tls tables   
; copy the whole TLS table into the loader data part   
mov edi,offset TlsBackup   
mov ecx,sizeof IMAGE_TLS_DIRECTORY32   
rep movsb   
; fix the TLS DIRECTORY VA   
mov eax,CryptSectionVA   
add eax,IT_SIZE   
add eax,TLS_BACKUP_ADDR   
mov esi,pTlsDirAddr   
mov [esi],eax   
ExitTlsFixProc:   
ret   
ProcessTlsTable ENDP   
```

```asm
; This function encrypts the dll name strings, saves the ImageImportDescriptors to the loader data   
; and destroys them.   
; return values:   
; 1 - success   
; 0 - too much IID's !   
ProcessOrgIT PROC USES edi esi edx, pFileImage : LPVOID, pITBaseRO : LPVOID   
LOCAL dwIIDNum : DWORD   
; clear the IIDInfo array   
XOR EAX,EAX   
MOV EDI, OFFSET IIDInfo   
MOV ECX, SIZEOF IIDInfo   
  ClearArrayLoop:   
      STOSB   
      LOOP ClearArrayLoop   
; get a random number   
INVOKE GetTickCount   
XOR EAX, ("yoda")   
MOV EDX,EAX       ; EDX -> stupid number :)   
; start   
MOV  dwIIDNum, 0   
MOV  EDI,pITBaseRO   
ADD  EDI,pFileImage   
ASSUME EDI : PTR IMAGE_IMPORT_DESCRIPTOR      ; EDI -> IID   
MOV ESI,OFFSET IIDInfo   
ASSUME ESI : PTR sItInfo      ; ESI -> Loder IT data array   
.WHILE [EDI].Name1   
   ; too much IID's ?   
   INC  dwIIDNum   
   .IF dwIIDNum == (MAX_IID_NUM)   
      XOR  EAX, EAX   
      JMP  POIT_Exit   
   .ENDIF   
   ; save IID Infos   
   M2M <[EDI].Name1>, <[ESI].DllNameRVA>   
   M2M <[EDI].OriginalFirstThunk>, <[ESI].OrgFirstThunk>   
   M2M <[EDI].FirstThunk>, <[ESI].FirstThunk>   
   ;-> get dll pointer   
   PUSH  [EDI].Name1   
   PUSH  pFileImage   
   CALL  RVA2Offset   
   ADD   EAX, pFileImage   
   ;-> crypt string   
   CALL @EnDeCryptString   
     ;--- CRYPT API name strings ---   
     PUSH ESI   
     MOV  ESI, [EDI].OriginalFirstThunk   
     .IF !ESI   
       MOV ESI, [EDI].FirstThunk   
     .ENDIF   
     PUSH ESI   
     PUSH pFileImage   
     CALL RVA2Offset   
     MOV  ESI, EAX   
     ADD  ESI, pFileImage   
     .WHILE DWORD PTR [ESI]  ; ESI -> Thunk 指针   
       MOV  EAX, [ESI]   
       ; is it an Ordinal Import ?   
     TEST EAX,IMAGE_ORDINAL_FLAG32   
     JNZ  SkipApiString   
       PUSH EAX   
       PUSH pFileImage   
       CALL RVA2Offset   
       test  eax, eax   
       jz    SkipApiString   
       add  eax, pFileImage   
   inc  eax   
   inc  eax       ; 跳过 Hint   
       call  @EnDeCryptString   
 SkipApiString:   
    
       add  esi, 4   
     .ENDW   
     POP ESI   
     ; destroy Original IID   
     MOV [EDI].Name1, EDX   
     MOV [EDI].OriginalFirstThunk, EDX   
     MOV [EDI].FirstThunk, EDX   
     MOV [EDI].TimeDateStamp, EDX   
     MOV [EDI].ForwarderChain, EDX   
   ; EDI -> 指向下一个 IID   
    
   ADD EDI,SIZEOF IMAGE_IMPORT_DESCRIPTOR   
   ADD ESI,SIZEOF sItInfo   
.ENDW   
ASSUME ESI : NOTHING   
ASSUME EDI : NOTHING   
XOR  EAX, EAX   
INC  EAX   
 POIT_Exit:   
RET   
ProcessOrgIT ENDP   
```

```asm
; returns aligned value   
PEAlign PROC USES ecx edx, dwTarNum : DWORD, dwAlignTo : DWORD   
mov ecx,dwAlignTo   
mov eax,dwTarNum   
xor edx,edx   
div ecx   
cmp edx,0   
jz AlreadyAligned   
inc eax   
  AlreadyAligned:   
 mul ecx   
ret   
PEAlign ENDP   
```

```asm
; calulates the Offset from a RVA   
; Base    - base of the MMF   
; dwITRVA - the RVA to calculate   
; returns 0 if an error occurred else the calculated Offset will be returned   
RVA2Offset PROC USES ebx ecx edx, Base : DWORD,dwITRVA : DWORD   
; get the pointer to the NT header   
mov eax,Base   
add eax,[eax+03Ch]   
invoke ImageRvaToSection,eax,Base,dwITRVA   
test eax,eax   
```



`@@ExitProc`

`xchg eax,ebx`

`assume ebx : ptr IMAGE_SECTION_HEADER`

`mov eax,dwITRVA`

`sub eax,[ebx].VirtualAddress`

`add eax,[ebx].PointerToRawData`

`assume ebx : nothing`

`@@ExitProc:`

`ret`

`RVA2Offset ENDP`

`include loaderstuff.inc`

`include LOADER.ASM`

`* * *`

`.686P`

`.MMX`

`.Model Flat,StdCall`

`Option CaseMap:None`

`include masm32includekernel32.inc`

`include masm32includeuser32.inc`

`include masm32includecomdlg32.inc`

`include masm32includeshell32.inc`

`include masm32includeimagehlp.inc`

`includelib masm32libkernel32.lib`

`includelib masm32libuser32.lib`

`includelib masm32libcomdlg32.lib`

`includelib masm32libshell32.lib`

`includelib masm32libimagehlp.lib`

`include masm32includewindows.inc`

`include resource.inc`

`include MACROS.ASM`

`include JUNKXX.INC`

`.CONST`

`CHECK_SI_FLAG equ 1`

`ERASE_HEADER_FLAG equ 2`

`DESTROY_IMPORT_FLAG equ 4`

`CHECK_HEADER_CRC equ 8`

`ANTI_DUMP_FLAG equ 16`

`API_REDIRECT_FLAG equ 32`

`NO_CRYPT_SECTION_FLAG equ 64`

`szFilter db "可执行文件",0,"*.eXe",0,"所有文件",0,"*.*",0,0`

`szCurDir db ".",0`

`szNoFile db "未选择文件 !",0`

`szErr db "错误",0`

`.Data`

`hInst dd 0`

`hDLG dd 0`

`ofn OPENFILENAME <>`

`cFname db MAX_PATH dup (0)`

`szCaption db '『 仙剑 - 宿命 』',0`

`szAboutInfo db ' 仙剑保护器 - linson/forgot',CR,LF,`

`' 致意:',CR,LF,`

`' y0da hying fly fxyang shinegood kongfoo jeff jwh51 goodmorning ',CR,LF,`

`' ...... and you ',0`

`.Code`

`include CryptStuff.ASM`

`include PER.ASM`

```asm
DlgProc Proc hDlg:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
  Local hDrop:HANDLE
  pushad
  .IF uMsg == WM_INITDIALOG
    invoke LoadIcon,hInst,IDI_ICON
    invoke SendMessage,hDlg,WM_SETICON,TRUE,eax
    push hDlg
    pop hDLG
    ; 是否已经运行?
    invoke FindWindow, NULL, offset szCaption
    test eax, eax
    jz runchOK
    invoke FlashWindow, eax, 1; 闪动窗口
    invoke SendMessage,hDlg,WM_CLOSE,NULL,NULL; 退出
runchkOK:
    ; 设置窗口标题
    invoke SetWindowText, hDlg, offset szCaption
    ; 设置默认复选框
    invoke CheckDlgButton,hDlg, IDC_SICHECK , TRUE
    invoke CheckDlgButton,hDlg,IDC_CHECKHEADERCRC, TRUE
    invoke CheckDlgButton,hDlg,IDC_DESTROYIMPORT, TRUE
    invoke CheckDlgButton,hDlg,IDC_ANTIDUMP, TRUE
    invoke CheckDlgButton,hDlg,IDC_APIREDIRECT, TRUE
    invoke DragAcceptFiles,hDlg,TRUE
  .ELSEIF uMsg == WM_DROPFILES
    push wParam
    pop hDrop
    invoke DragQueryFile,hDrop,0,offset cFname,sizeof cFname
    invoke DragFinish,hDrop
    invoke SetDlgItemText,hDlg,IDC_TARGETFILE,offset cFname
  .ELSEIF uMsg == WM_COMMAND
    mov eax,wParam
    .IF ax == IDC_CLOSE
      invoke SendMessage,hDlg,WM_CLOSE,NULL,NULL
    .ELSEIF eax == IDC_ABOUT
      invoke MessageBox, hDlg, offset szAboutInfo, offset szCaption, 64
    .ELSEIF ax == IDC_CHOOSEFILE
      ; get a file path
      mov ofn.lStructSize,SIZEOF ofn
      mov ofn.lpstrFilter,offset szFilter
      push hDlg
      pop ofn.hwndOwner
      mov ofn.lpstrFile, offset cFname
      mov ofn.nMaxFile,SIZEOF cFname
      mov ofn.lpstrInitialDir,offset szCurDir
      mov ofn.Flags, OFN_FILEMUSTEXIST or OFN_PATHMUSTEXIST or OFN_LONGNAMES or OFN_HIDEREADONLY
      push offset ofn
      call GetOpenFileName
      test eax,eax
      jz @@ExitDlgProc
      invoke SetDlgItemText,hDlg,IDC_TARGETFILE,offset cFname
    .ELSEIF ax == IDC_CRYPT
      ; ----- was a file selected ? -----
      mov eax,offset cFname
      .IF byte ptr [eax] == 0
        invoke MessageBox,hDlg,offset szNoFile,offset szErr,MB_ICONERROR
        jmp @@ExitDlgProc
      .ENDIF
      ; ---- 生成保护选项 -----
      xor edi,edi
      invoke IsDlgButtonChecked,hDlg,IDC_SICHECK
      .IF eax == BST_CHECKED
        or edi,CHECK_SI_FLAG
      .ENDIF
      invoke IsDlgButtonChecked,hDlg,IDC_ERASEPEHEADER
      .IF eax == BST_CHECKED
        or edi,ERASE_HEADER_FLAG
      .ENDIF
      invoke IsDlgButtonChecked,hDlg,IDC_DESTROYIMPORT
      .IF eax == BST_CHECKED
        or edi,DESTROY_IMPORT_FLAG
      .ENDIF
      invoke IsDlgButtonChecked,hDlg,IDC_CHECKHEADERCRC
      .IF eax == BST_CHECKED
        or edi,CHECK_HEADER_CRC
      .ENDIF
      invoke IsDlgButtonChecked,hDlg,IDC_ANTIDUMP
      .IF eax == BST_CHECKED
        or edi,ANTI_DUMP_FLAG
      .ENDIF
      INVOKE IsDlgButtonChecked,hDlg,IDC_APIREDIRECT
      .IF EAX == BST_CHECKED
        OR EDI, API_REDIRECT_FLAG
      .ENDIF
      INVOKE IsDlgButtonChecked,hDlg,IDC_NOCRYPTSECTION
      .IF EAX == BST_CHECKED
        OR EDI, NO_CRYPT_SECTION_FLAG
      .ENDIF
      push edi
      push hDlg
      push offset cFname
      call Protect
    .ENDIF
  .ELSEIF uMsg == WM_CLOSE
    invoke EndDialog,hDlg,0
  .ENDIF
  popad
@@ExitDlgProc:
  xor eax,eax
  ret
DlgProc endp

Main:
  invoke GetModuleHandle, NULL
  mov hInst, eax
  invoke DialogBoxParam, eax, IDD_MAINDLG, 0, offset DlgProc, 0
  invoke ExitProcess, 0

End Main
```

`* * *`

`; -> Polymorphic En-/Decryption routine generator for per byte encryption <-`

`; by yoda`

`;---- STRUCTs ----`

```asm
sPERTable STRUCT
  dwSize      DD ?
  dwEncrypt   DD ?
  dwDecrypt   DD ?
  RandNumType DD ?
sPERTable ENDS

; RandNumType:
; 0 - no random num needed
; 1 - 3th byte must be a random number
; 2 - 2nd byte must be a random number

;----- EQUs -----
PERItems          EQU 14

;----- CONST ----
.CONST
; all opcodes are in reverse order
PERTable          DD 1
          DD 090h        ; NOP
          DD 090h        ; NOP
          DD 0
          DD 1
          DD 0F9h        ; STC
          DD 0F9h        ; STC
          DD 0
          DD 1
          DD 0F8h        ; CLC
          DD 0F8h        ; CLC
          DD 0
          DD 2
          DD 0C0FEh      ; INC  AL
          DD 0C8FEh      ; DEC  AL
          DD 0
          DD 2
          DD 00004       ; ADD AL, 0
          DD 0002Ch      ; SUB AL, 0
          DD 2
          DD 2
          DD 0002Ch      ; SUB AL, 0
          DD 00004       ; ADD AL, 0
          DD 2
          DD 2
          DD 0C102h      ; ADD AL, CL
          DD 0C12Ah      ; SUB AL, CL
          DD 0
          DD 2
          DD 0C12Ah      ; SUB AL, CL
          DD 0C102h      ; ADD AL, CL
          DD 0
          DD 2
          DD 00034h      ; XOR AL, 0
          DD 00034h      ; XOR AL, 0
          DD 2
          DD 3
          DD 000C8C0h    ; ROR  AL, 0
          DD 000C0C0h    ; ROL  AL, 0
          DD 1
          DD 3
          DD 000C0C0h    ; ROL  AL, 0
          DD 000C8C0h    ; ROR  AL, 0
          DD 1
          DD 3
          DD 0E801EBh    ; Self modifing
          DD 0E801EBh    ; Self modifing
          DD 0
          DD 3
          DD 0E901EBh    ; Self modifing
          DD 0E901EBh    ; Self modifing
          DD 0
```



```asm
DD 3   
DD 0C201EBh        ; Self modifying   

DD 0C201EBh        ; Self modifying   

DD 0       

.DATA   

dwRandVal           DD 0   

.CODE   

; srand should only called one time !!!   

InitRandom PROC   

  ; manage the random generator   

  CALL GetTickCount   

  PUSH EAX   

  CALL srand   

  RET   

InitRandom ENDP   

MakePER PROC pEncryptBuff : LPVOID, pDecryptBuff : LPVOID, dwSize : DWORD   

  LOCAL dwCurRandNum : DWORD   

  ; prepare some things   

  MOV  EDI, pEncryptBuff          ; EDI -> EncryptBuffer   

  MOV  ESI, pDecryptBuff          ; ESI -> DecryptBuffer   

  ADD  ESI, dwSize          ; ESI will be filled from down to top   

  ; generate !   

  .REPEAT        


     ; get a random PER Item   

     PUSH PERItems   

     CALL rand   

     MOV  EBX, SIZEOF sPERTable   

     XOR  EDX, EDX   

     MUL  EBX   

     ADD  EAX, OFFSET PERTABLE   

     XCHG EAX, EDX             ; EDX -> PER Table Item   

     ASSUME EDX : PTR sPERTable   

     ; is this item too big   

     MOV  EBX, [EDX].dwSize   

     CMP  EBX, dwSize   

     JG   Retry   

     ;---- past the Opcode ----   

     ;-> encryption buffer   

     MOV  ECX, [EDX].dwSize   

     MOV  EAX, [EDX].dwEncrypt      

     MOV  ECX, [EDX].dwSize   

     .WHILE ECX != 0   

          MOV  BYTE PTR [EDI], AL   

          ADD  EDI, 1   

          ROR  EAX, 8   

          DEC  ECX   

     .ENDW   

     ; generate the random num   

     MOV  EAX, [EDX].RandNumType   

     .IF  EAX == 1 || EAX == 2   

          MOV  EBX, EDI   

          SUB  EBX, 1   

          PUSH 0F8h   

          CALL rand   

          INC  EAX        ; avoid 0 !   

          MOV  dwCurRandNum, EAX   

          MOV  BYTE PTR [EBX], AL   

     .ENDIF     


     ; update variables/pointers   

     MOV  EAX, [EDX].dwSize   

     SUB  dwSize, EAX   

     ;-> decryption buffer   

     MOV  ECX, [EDX].dwSize   

     MOV  EAX, [EDX].dwDecrypt      

     SUB  ECX, 1   

     .WHILE ECX != 0   

           ROR  EAX, 8   

           DEC  ECX   

     .ENDW   

     MOV  ECX, [EDX].dwSize   

     .WHILE ECX != 0   

          SUB  ESI, 1   

          MOV  BYTE PTR [ESI], AL   

          ROL  EAX, 8   

          DEC  ECX   

     .ENDW   

     ; generate the random num   

     MOV  EAX, [EDX].RandNumType   

     .IF  EAX == 1   

          MOV  EBX, ESI   

          ADD  EBX, 2   

          MOV  EAX, dwCurRandNum   

          MOV  BYTE PTR [EBX], AL   

     .ELSEIF EAX == 2   

          MOV  EBX, ESI   

          ADD  EBX, 1   

          MOV  EAX, dwCurRandNum   

          MOV  BYTE PTR [EBX], AL   


     .ENDIF     


     ASSUME EDX : NOTHING   

       Retry:   

  .UNTIL dwSize == 0   


  RET   

MakePER ENDP   

rand PROC USES edx ebx, dwRange : DWORD   

  MOV  EAX, dwRandVal   

  ; save new random number   

  ADD  EAX, 0567h   

  ROL  EAX, 1   

  MOV  dwRandVal, EAX   

  ; get new random number   

  XOR  EDX, EDX   

  MOV  ECX, 32   

    BitLoop:   

      SHR   EAX, 1   

      .IF   CARRY?   

          XOR  EAX, 013245769h   

      .ENDIF   

  LOOP BitLoop   

  ; force dwRange   

  XOR  EDX, EDX   

  MOV  EBX, dwRange   

  DIV  EBX   

  XCHG EAX, EDX   

  RET   

rand ENDP   

srand PROC dwRandNum : DWORD   

  MOV  EAX, dwRandNum   

  MOV  dwRandVal, EAX   

  RET   

srand ENDP
```

`4dd466d0e1e0cc123afbe504da2462ca`
