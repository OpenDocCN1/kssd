
# 浅析 QQ 炫舞 6 开挂

> 原文：[`www.pediy.com/kssd/pediy11/116205.html`](https://www.pediy.com/kssd/pediy11/116205.html)

偶去年在网上看到有一些“大牛”代练 QQ 炫舞，还上传了视频，个人感觉挺酷的，他用的是六开挂，售价还不菲（200RMB），老婆一狠心就买了下来（ 哎...也太鄙视她老公了![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")）![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

真是倒霉透了，只要一在我的电脑上运行就蓝哦....![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")估计是释放了驱动，而驱动导致系统崩溃，二话不说，`PEiD` 查壳..`UPX` 的（真是命好...）（`VC` 写的），`OD` 载入 ESP 定律脱掉、修复，为了安全起见还是在虚拟机下运行，![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")蓝蓝蓝...![](img/7eab1bc95cbb1be9f15eb01bf012328f.jpg "狂怒")，回到主系统，载入程序，下 `CreateServiceA` 断点，断下，释放出名为 `QQSafeRun.sys` 的驱动，复制，用 `W32Dasm` 反汇编，分析此驱动是挂钩 `SSDT`，Hook 了 `NtCreateMutant` 内核函数，直接返回 0（靠...真够绝的！），大家都知道 NT 之后系统 `SSDT` 为只读属性，要改写就得设置成可写的，当然方法有多种，它用的是 `CR0` 欺骗，反汇编代码如下：

```
................
................
cli
mov eax,cr0
xor eax,10000h
mov cr0,eax
................
一些操作
................
mov eax,cr0
xor eax,10000h
mov cr0,eax
sti
...................
...................
```

这种方法在单核 PC 上还好，而偶的是四核的，难怪狂蓝![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")，偶是学硬件开发的，深深的知道四个 CPU 挂在一个总线上将意味着什么~~~

网上一些网友也曾讨论过类似问题的就绝方案，这似乎已经有点老掉牙了，可是出于偶职业的原因还是拿出来研究研究...调用原子操作函数似乎是个很好的办法，但是似乎也存在不妥之处，虽然 `DPC` 是最好的选择，但偶更喜欢用 `MDL`+“原子操作”解决，于是通过分析反汇编代码功能，用 windows 汇编语言（只会这个![](img/cda69129882b40377a87144c782680e1.jpg "微笑")）写了个驱动，这个驱动同样挂钩 `SSDT`，hook 了 `NtCreateMutant` 内核函数，在多核 PC 下运行稳定，没有再出现蓝屏...![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")，这个驱动在 `XP+sp2/sp3` 下调试通过，偶将 `MDL` 操作 `SSDT` 的函数写成了通用函数，以便以后调用......

生成的驱动可以用驱动加载软件加载，实践证明：只要您的电脑配置够猛可以尽情的多开 n 个 QQ 炫舞....。

偶是学底层程序和硬件开发的，所以嘛...写不出好好的 Ring3 程序....有待努力！！！！

驱动代码如下：

编译环境：`RadASM`

编程语言：windows 汇编语言

```asm
;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●
;●                             SafeHook_Func.asm                              ●
;●                                                                           ●
;● 作者:CoCol   QQ:1174968967                                               ●
;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

;--------------------------------------------------------------------------------------
;函数:GetSerTablInf
;参数说明:pSerTablBase:服务表基址变量、pTablSize:服务表大小变量
;功能:得到系统服务表信息
;--------------------------------------------------------------------------------------
GetSerTablInf proc pSerTablBase:dword,pTablSize:dword
  pushad
  mov eax,KeServiceDescriptorTable;获取 KeServiceDescriptorTable 地址指针
  mov esi,[eax]
  mov eax,[esi];获取 KeServiceDescriptorTable->ServiceTableBase
  mov edi,pSerTablBase
  mov [edi],eax
  add esi,8h
  mov eax,[esi]
  sal eax,2h
  mov edi,pTablSize
  mov [edi],eax
  popad
  ret
GetSerTablInf endp

;--------------------------------------------------------------------------------------
;函数:LockMdl
;参数说明:ObAddr:源地址、Codesize:尺寸、pMdl:MDL 指针、pTab:新表指针
;功能:生成物理映像页
;--------------------------------------------------------------------------------------
LockMdl proc ObAddr:dword,Codesize:dword,pMdl:dword,pTab:dword
  local Status:NTSTATUS
  pushad
  mov Status,STATUS_SUCCESS
  invoke MmCreateMdl,NULL,ObAddr,Codesize
  .if sdword ptr eax<0
    mov Status,STATUS_UNSUCCESSFUL;创建 MDL 不成功返回 STATUS_UNSUCCESSFUL
    jmp Exit
  .endif
  mov edi,pMdl
  mov [edi],eax
  mov ebx,eax
  assume ebx: ptr MDL
  invoke MmBuildMdlForNonPagedPool,ebx
  or [ebx].MdlFlags,81H
  invoke MmMapLockedPages,ebx,KernelMode
  mov edi,pTab
  mov[edi],eax
  Exit:
  popad
  mov eax,Status
  ret
LockMdl endp

;--------------------------------------------------------------------------------------
;函数:UnLockMdl
;参数说明:dwMdl:MDL 地址、dwTab:新表地址
;功能:释放生成的物理映像页
;--------------------------------------------------------------------------------------
UnLockMdl proc dwMdl:dword,dwTab:dword
  pushad
  invoke MmUnmapLockedPages,dwTab,dwMdl
  invoke IoFreeMdl,dwMdl
  popad
  ret
UnLockMdl endp

;--------------------------------------------------------------------------------------
;函数:KeInterlockedExchange
;参数说明:@plTarget:目标地址指针、@lValue:源地址
;功能:原子操作函数
;--------------------------------------------------------------------------------------
KeInterlockedExchange proc @plTarget:dword,@lValue:dword
  pushad
      mov ecx,@plTarget
      mov edx,@lValue
      mov eax,dword ptr[ecx]
      @@:lock cmpxchg dword ptr[ecx],edx
      jne @B
      popad
      ret
KeInterlockedExchange endp

;--------------------------------------------------------------------------------------
;函数:HookSSDT
;参数说明:@pRelKeFunc:原函数地址、@pAddrToRelFunc:原函数地址指针、@pNewFunc:自定义函数指针
;         @swFuncName:导出函数的函数名 UNICODE 字符串指针、Id:未导出函数 ID 号
;功能:Hook 指定内核函数          【MapServiceDescriptorTable:自定义全局变量】
;--------------------------------------------------------------------------------------
HookSSDT proc @pRelKeFunc:dword,@pAddrToRelFunc:dword,@pNewFunc:dword,@swFuncName:dword,Id:dword
  local @@Irql:dword
  pushad
  mov esi,MapServiceDescriptorTable;获取 KeServiceDescriptorTable->ServiceTableBase 物理映像地址
  .if Id==NULL
    invoke MmGetSystemRoutineAddress,@swFuncName
    inc eax
    movzx ecx,byte ptr[eax];获取服务号
  .else
    mov ecx,Id
  .endif
  sal ecx,2h
  add esi,ecx
  mov edi,@pAddrToRelFunc
  mov [edi],esi
  mov edi,dword ptr[esi];(edi)=欲 Hook 内核函数的地址
  push esi
  mov esi,@pRelKeFunc
  mov [esi],edi
        invoke KeRaiseIrqlToDpcLevel;●提升中断优先级
        mov @@Irql,eax
        pop edi
        mov esi,@pNewFunc
        invoke KeInterlockedExchange,edi,esi
        invoke KeLowerIrql,@@Irql;●恢复中断优先级
        popad
  ret
HookSSDT endp

;--------------------------------------------------------------------------------------
;函数:UnHookSSDT
;参数说明:@pRelKeFunc:原函数地址、@AddrToRelFunc:原函数地址变量
;功能:恢复被 Hook 的内核函数
;--------------------------------------------------------------------------------------
UnHookSSDT proc @pRelKeFunc:dword,@AddrToRelFunc:dword
  local @@Irql:dword
  pushad
        invoke KeRaiseIrqlToDpcLevel
        mov @@Irql,eax
        mov edi,@AddrToRelFunc
        mov esi,@pRelKeFunc
        invoke KeInterlockedExchange,edi,esi
        invoke KeLowerIrql,@@Irql
        popad
  ret
UnHookSSDT endp

;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●
;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●
;●                             Demo.inc                                         ●
;●                                                                           ●
;● 作者:CoCol   QQ:1174968967                                               ●
;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

;==============================Include=================================================
include   w2k\ntstatus.inc
include   w2k\ntddk.inc
include   w2k\hal.inc
include   w2k\ntoskrnl.inc
include   w2k\ntdef.inc
include   Macros\Strings.mac

;==============================IncludeLib==============================================
includelib   w2k\ntoskrnl.lib
includelib   w2k\hal.lib

;===============================Typedef================================================
_NTCREATEMUTANT typedef proto:dword,:dword,:dword,:dword
NTCREATEMUTANT typedef ptr _NTCREATEMUTANT

;======================================================================================
;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●
;●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●
;●                         SafeHookSSDT.Asm                               ●
;●                                                                           ●
```



## 486p; 保护模式

```asm
.model flat,stdcall
option casemap:none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include Demo.inc
;--------------------------------------------------------------------

.data
RealNtCreateMutant NTCREATEMUTANT ?
DescriptorAddrToMutant dd ?
MapServiceDescriptorTable dd ?
MewMDL dd ?
SerTablBase dd ?
TablSize dd ?

.code
include SafeHook\_Func.asm

HookNtCreateMutant proc MutantHandle:dword, DesiredAccess:dword, ObjectAttributes:dword, InitialOwner:dword
  pushad
  invoke RealNtCreateMutant, MutantHandle, DesiredAccess, ObjectAttributes, InitialOwner
  popad
  xor eax, eax
  ret
HookNtCreateMutant endp

DriverUnload proc pDriverObject:PDRIVER\_OBJECT
  pushad
  ;----------------------------------------------------------------------
  invoke UnHookSSDT, RealNtCreateMutant, DescriptorAddrToMutant
  invoke UnLockMdl, MewMDL, MapServiceDescriptorTable
  ;----------------------------------------------------------------------
  popad
  ret
DriverUnload endp

DriverEntry proc pDriverObject:PDRIVER\_OBJECT, pusRegistryPath:PUNICODE\_STRING ; 入口函数
  local Status:dword
  pushad
  mov Status, STATUS\_SUCCESS
  ;----------------------------------------------------------------------
  invoke GetSerTablInf, offset SerTablBase, offset TablSize
  invoke LockMdl, SerTablBase, TablSize, offset MewMDL, offset MapServiceDescriptorTable
  .if eax == STATUS\_UNSUCCESSFUL
    mov Status, eax
    jmp Exit
  .endif
  invoke HookSSDT, offset RealNtCreateMutant, offset DescriptorAddrToMutant, offset HookNtCreateMutant, NULL, 2bH
  ;----------------------------------------------------------------------
  mov esi, pDriverObject
  assume esi: ptr DRIVER\_OBJECT
  mov [esi].DriverUnload, offset DriverUnload ; 设置卸载例程
  assume esi: nothing
  Exit:
  popad
  mov eax, Status
  ret
DriverEntry endp

end DriverEntry
```
