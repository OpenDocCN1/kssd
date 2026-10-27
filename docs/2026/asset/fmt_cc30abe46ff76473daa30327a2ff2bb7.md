
# Public my Pal loader source...:D

**标 题:** Public my Pal loader source...:D
**发信人:** linson
**时 间:** 2004年4月03日 05:28
**详细信息:**

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
  db 81h, 0EDh ,0,0,0,0                        ; sub  ebp, 00000000
  popfd
  add  esp, 14h
  pop  ebp
  popfd
  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?

                                              \_svkp1 0CDh
  pushad
                                              \_svkp3 44h
  ; 取起始地址
  call  @Delta
@Delta:
  pop  ebp
                                              \_svkp2 0CCh
  sub  ebp, offset @Delta
                                              \_svkp3 0E8h
  push  ebp  ; 保存 ebp
                                              \_svkp1 0E9h
  add  ebp, offset @Decrypt_Loader_Code
                                              \_svkp2 99h
  push  ebp
                                              \_svkp2 33h
  ret                                ; -> @Decrypt_Loader_Code
  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?
  ; Banner :D
  db  ' Sword-Protector (c) by LiNSoN[DFCG] release                       ',CR,LF
  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?
@Decoder:
  ; 重点防护 & anti-disasmer
                            \_vxd
  enter  0, 0
                            \_junk 0E9h
  pushad
                            \_vxd
  mov  eax, [ebp+10h]              ; eax -> pContext
  assume eax : ptr CONTEXT
                            \_vxd
  mov  ebx, [ebp+8]              ; ebx -> pExcept
                            \_mmx 0E9h
  mov  ebx, [ebx]
                            \_trash 0E8h
  .if  ebx == 0C000001Dh
                            \_vxd
      ; mov     seg?, dx 产生的第一个异常
      xor  ebx, ebx
                            \_vxd
      and  [eax].iDr0, ebx
                            \_trash  0E8h
      and  [eax].iDr1, ebx
                            \_mmx 0CDh
      and  [eax].iDr2, ebx
                            \_trash  0E8h
      and  [eax].iDr3, ebx
                            \_vxd
      and  [eax].iDr6, 0FFFF0FF0h
                            \_trash  0E8h
      and  [eax].iDr7, 0DC00h        ; 禁止掉所有断点
      ; 修复 regEip 指针
                            \_jmper 0FFh
      add  [eax].regEip, 2
                            \_vxd
      jmp  @@ExceptProcessed
  .elseif  ebx == 0C000001Eh
                            \_vxd
    cld                      ; 正方向进行
                            \_jmper 0BEh
    xchg  esi, [eax].regEsi
                            \_vxd
    xor  esi, [eax].iDr0          ; 防止 DRx 断点, 若设置 DRx 会造成解码错误, 下同
                            \_vxd
    xchg  edi, [eax].regEdi
                            \_jmper 0FFh
    xor  edi, [eax].iDr1
                            \_junk 0E9h
    xchg  ecx, [eax].regEcx
                            \_trash  0E8h
    xor  ecx, [eax].iDr2
                            \_jmper 0CCh
    push  eax
                            \_jmper 0CCh
    lodsb
                            \_trash  0E8h
    VarDecryptBuff db VAR_PER_SIZE dup (0)  ; 解码
                            \_trash  0E8h
    stosb
                            \_vxd
    pop  eax
                            \_mmx 0CDh
    xor  eax, [eax].iDr3
                            \_trash  0E8h
    xchg  esi, [eax].regEsi
                            \_mmx 0E9h
    xchg  edi, [eax].regEdi
                            \_mmx 0E8h
    xchg  ecx, [eax].regEcx
                            \_vxd
    dec  [eax].regEcx
  .endif
                            \_vxd
  .if  zero?
                            \_trash  0E8h
    ; 恢复原始 SEH Handler
    mov  ecx, [eax].regEsp
                            \_mmx 0CDh
    push  [ecx]
                            \_jmper 0CCh
    xor  ecx, ecx
                            \_jmper 0CDh
    pop  fs:[ecx]
                            \_trash  0E8h
    ; 卸载 SEH 链
    add  [eax].regEsp, 8
                            \_mmx 0CDh
    ; 修复异常
    add  [eax].regEip, 4
                            \_vxd
    ; 初始化 DRx
    and  [eax].iDr0, ecx
                            \_trash  0E8h
    and  [eax].iDr1, ecx
                            \_mmx 0CDh
    and  [eax].iDr2, ecx
                            \_trash  0E8h
    and  [eax].iDr3, ecx
                            \_vxd
    and  [eax].iDr6, 0FFFF0FF0h
                            \_trash  0E8h
    and  [eax].iDr7, 0DC00h        ; 禁止掉所有断点
  .endif
                            \_mmx 0E8h
  @@ExceptProcessed:
  popad
  assume eax : nothing
                            \_mmx 0E9h
  xor  eax, eax
                            \_vxd
  leave
                            \_trash  0E8h
  ret
  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?
@Decrypt_Loader_Code:
                            \_jmper 0C7h
  pop  ebp  ; 恢复 ebp
  ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━?
  ; 首层 Loader 代码解码
                            \_jmper 0CDh
  lea  eax, @Decoder[ebp]
                            \_vxd
  push  eax
                            \_trash 0E8h
  xor  eax, eax
                            \_mmx 0E9h
  push  fs:[eax]
                            \_junk 0E9h
  mov  fs:[eax], esp
  db    8Eh, 0FAh                ;mov     seg?, dx 产生异常初始化 DRx 防止解码错误
  mov  ecx, @Loader_Crypt_Size        ; 解码长度
                            \_vxd
  lea  edi, @Loader_Crypt_Start[ebp]    ; 首地址
                            \_jmper 0CCh
  mov  esi, edi
                            \_vxd
  dd    0C8C70FF0h                ; lock cmpxchg8b (e)ax 产生非法指令异常调用解码器, 解码完成会自行卸载
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
                            \_vxd
    int  3
   SICheck1_SP:
    mov   EBP, EDI
    ; 恢复 SEH 链
    xor  ebx, ebx
    pop  fs:[ebx]
    add  esp, 4h
    .if AL != 4
                            \_junk 0E9h
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
    ; esi = DLL模块句柄
    push  eax
    push  esi
    call  \_GetProcAddress[ebp]
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
  mov  ecx, [eax]                            ; ecx contains the VirtualAddress of the IT
  add  ecx, dwImageBase[ebp]
  add  ecx,16                               ; ecx points to the FirstThunk address of the IID
  mov  eax,dword ptr [ecx]
  add  eax, dwImageBase[ebp]
  mov  ebx,dword ptr [eax]
  mov  [ebp+\_LoadLibrary],ebx
  add  eax,4
  mov  ebx,dword ptr [eax]
  mov  \_GetProcAddress[ebp],ebx
  ; 取 Kernel.DLL 之函数
  lea  eax,szKernel32[ebp]
  push  eax
  call  \_LoadLibrary[ebp]
  mov  esi, eax                ; esi -> Kernel 模块句柄
```
