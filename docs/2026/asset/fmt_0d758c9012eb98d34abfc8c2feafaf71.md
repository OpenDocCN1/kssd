
# 系统调用的学习

## sysente

`.text:004076F0` 处为 `_KiFastCallEntry` 过程。以下代码展示了该过程的完整汇编实现：

```
.text:004076F0 ; =============== S U B R O U T I N E =======================================
.text:004076F0
.text:004076F0
.text:004076F0 \_KiFastCallEntry proc near ; DATA XREF: \_KiTrap01+6F↓o
.text:004076F0 ; KiLoadFastSyscallMachineSpecificRegisters(x)+24↓o
.text:004076F0
.text:004076F0 var\_B = byte ptr -0Bh
.text:004076F0
.text:004076F0 ; FUNCTION CHUNK AT .text:004076C8 SIZE 00000023 BYTES
.text:004076F0 ; FUNCTION CHUNK AT .text:00407990 SIZE 00000014 BYTES
.text:004076F0
.text:004076F0 mov ecx, 23h
.text:004076F5 push 30h
.text:004076F7 pop fs
.text:004076F9 mov ds, ecx
.text:004076FB mov es, ecx
.text:004076FD mov ecx, ds:0FFDFF040h
.text:00407703 mov esp, [ecx+4]
.text:00407706 push 23h
.text:00407708 push edx
.text:00407709 pushf
.text:0040770A
.text:0040770A loc\_40770A: ; CODE XREF: \_KiFastCallEntry2+22↑j
.text:0040770A push 2
.text:0040770C add edx, 8
.text:0040770F popf
.text:00407710 or [esp+0Ch+var\_B], 2
.text:00407715 push 1Bh
.text:00407717 push dword ptr ds:0FFDF0304h
.text:0040771D push 0
.text:0040771F push ebp
.text:00407720 push ebx
.text:00407721 push esi
.text:00407722 push edi
.text:00407723 mov ebx, ds:0FFDFF01Ch
.text:00407729 push 3Bh
.text:0040772B mov esi, [ebx+124h]
.text:00407731 push dword ptr [ebx]
.text:00407733 mov dword ptr [ebx], 0FFFFFFFFh
.text:00407739 mov ebp, [esi+18h]
.text:0040773C push 1
.text:0040773E sub esp, 48h
.text:00407741 sub ebp, 29Ch
.text:00407747 mov byte ptr [esi+140h], 1
.text:0040774E cmp ebp, esp
.text:00407750 jnz loc\_4076C8
.text:00407756 and dword ptr [ebp+2Ch], 0
.text:0040775A test byte ptr [esi+2Ch], 0FFh
.text:0040775E mov [esi+134h], ebp
.text:00407764 jnz Dr\_FastCallDrSave
.text:0040776A
.text:0040776A loc\_40776A: ; CODE XREF: Dr\_FastCallDrSave+10↑j
.text:0040776A ; Dr\_FastCallDrSave+7C↑j
.text:0040776A mov ebx, [ebp+60h]
.text:0040776D mov edi, [ebp+68h]
.text:00407770 mov [ebp+0Ch], edx
.text:00407773 mov dword ptr [ebp+8], 0BADB0D00h
.text:0040777A mov [ebp+0], ebx
.text:0040777D mov [ebp+4], edi
.text:00407780 sti
.text:00407781
.text:00407781 loc\_407781: ; CODE XREF: \_KiBBTUnexpectedRange+18↑j
.text:00407781 ; \_KiSystemService+6E↑j
.text:00407781 mov edi, eax
.text:00407783 shr edi, 8
.text:00407786 and edi, 30h
.text:00407789 mov ecx, edi
.text:0040778B add edi, [esi+0E0h]
.text:00407791 mov ebx, eax
.text:00407793 and eax, 0FFFh
.text:00407798 cmp eax, [edi+8]
.text:0040779B jnb \_KiBBTUnexpectedRange
.text:004077A1 cmp ecx, 10h
.text:004077A4 jnz short loc\_4077C0
.text:004077A6 mov ecx, ds:0FFDFF018h
.text:004077AC xor ebx, ebx
.text:004077AE
.text:004077AE loc\_4077AE: ; DATA XREF: \_KiTrap0E+110↓o
.text:004077AE or ebx, [ecx+0F70h]
.text:004077B4 jz short loc\_4077C0
.text:004077B6 push edx
.text:004077B7 push eax
.text:004077B8 call ds:\_KeGdiFlushUserBatch
.text:004077BE pop eax
.text:004077BF pop edx
.text:004077C0
.text:004077C0 loc\_4077C0: ; CODE XREF: \_KiFastCallEntry+B4↑j
.text:004077C0 ; \_KiFastCallEntry+C4↑j
.text:004077C0 inc dword ptr ds:0FFDFF638h
.text:004077C6 mov esi, edx
.text:004077C8 mov ebx, [edi+0Ch]
.text:004077CB xor ecx, ecx
.text:004077CD mov cl, [eax+ebx]
.text:004077D0 mov edi, [edi]
.text:004077D2 mov ebx, [edi+eax\*4]
.text:004077D5 sub esp, ecx
.text:004077D7 shr ecx, 2
.text:004077DA mov edi, esp
.text:004077DC cmp esi, ds:\_MmUserProbeAddress
.text:004077E2 jnb loc\_407990
.text:004077E8
.text:004077E8 loc\_4077E8: ; CODE XREF: \_KiFastCallEntry+2A4↓j
.text:004077E8 ; DATA XREF: \_KiTrap0E+106↓o
.text:004077E8 rep movsd
.text:004077EA call ebx
.text:004077EC
.text:004077EC loc\_4077EC: ; CODE XREF: \_KiFastCallEntry+2AF↓j
.text:004077EC ; DATA XREF: \_KiTrap0E+126↓o ...
.text:004077EC mov esp, ebp
.text:004077EE
.text:004077EE loc\_4077EE: ; CODE XREF: \_KiBBTUnexpectedRange+38↑j
.text:004077EE ; \_KiBBTUnexpectedRange+43↑j
.text:004077EE mov ecx, ds:0FFDFF124h
.text:004077F4 mov edx, [ebp+3Ch]
.text:004077F7 mov [ecx+134h], edx
.text:004077F7 \_KiFastCallEntry endp
.text:004077F7
.text:004077FD
.text:004077FD ; =============== S U B R O U T I N E =======================================
```

sysenter 指令进 0 环后调用 `KiFastCallEntry`。

### KiFastCallEntry 函数如何初始化 _KTRAP_FRAME

```
.text:004076F0\_KiFastCallEntry: ; DATA XREF: KiLoadFastSyscallMachineSpecificRegisters(x)+24o
.text:004076F0 ; \_KiTrap01+72o
.text:004076F0 mov ecx, 23h ;让ecx存储023h(三环描述符)
.text:004076F5 push 30h
.text:004076F7 pop fs ; fs = 0x30, ffc093df`f0000001, 0环数据段
.text:004076F9 mov ds, ecx ; ds = 0x23, 00cff300`0000ffff, 3环数据段
.text:004076FB mov es, ecx ; es = 0x23（es辅助ds）
.text:004076FD mov ecx, ds:0FFDFF040h;将TSS地址存储在ecx中 fs:[40h]
.text:00407703 mov esp, [ecx+4] ; esp = \_KPCR.TSS.Esp0，切换到0环堆栈
.text:00407706 push 23h ; \_KTRAP\_FRAME.HardwareSegSs = 0x23,保存旧ss
.text:00407708 push edx ; \_KTRAP\_FRAME.HardwareEsp = edx, edx是3环栈顶，参数指针
.text:00407709 pushf ; \_KTRAP\_FRAME.EFlags = EFlags，保存3环eflag寄存器
.text:0040770A
.text:0040770A loc\_46655A: ; CODE XREF: \_KiSystemService+96j
.text:0040770A push 2 ;将0环eflag寄存器存储在栈中
.text:0040770C add edx, 8 ; edx 指向3环API参数
.text:0040770F popf ; EFlags = 0x02，即清空0环所有标志位
.text:0040770F ; 此时 esp 指向 EFlags
.text:0040770F or [esp+0A4h+var\_A3], 2 ; \_KTRAP\_FRAME.EFlags 即3环 EFlags 的 IF = 1
.text:00407715 push 1Bh ; \_KTRAP\_FRAME.SegCs = 0x1B, 3环代码段
.text:00407717 push dword ptr ds:0FFDF0304h ; \_KTRAP\_FRAME.Eip = \_KUSER\_SHARED\_DATA.SystemCallReturn
.text:0040771D push 0 ; \_KTRAP\_FRAME.ErrCode = 0
.text:0040771F push ebp ; \_KTRAP\_FRAME.Ebp = ebp
.text:00407720 push ebx ; \_KTRAP\_FRAME.Ebx = ebx
.text:00407721 push esi ; \_KTRAP\_FRAME.Esi = esi
.text:00407722 push edi ; \_KTRAP\_FRAME.Edi = edi
.text:00407723 mov ebx, ds:0FFDFF01Ch ; ebx = \_KPCR.SelfPcr，即 ebx 指向 \_KPCR
.text:00407729 push 3Bh ; \_KTRAP\_FRAME.SegFs = 0x3B
.text:0040772B mov esi, [ebx+124h] ; esi = \_KPCR.\_KPRCB.CurrentThread
.text:00407731 push dword ptr [ebx] ; \_KTRAP\_FRAME.ExceptionList = \_KPCR.NtTib.ExceptionList
.text:00407733 mov dword ptr [ebx], 0FFFFFFFFh ; \_KPCR.NtTib.ExceptionList = -1
.text:00407739 mov ebp, [esi+18h] ; ebp = \_KPCR.\_KPRCB.CurrentThread.InitialStack
.text:0040773C push 1 ; \_KTRAP\_FRAME.PreviousPreviousMode = 1，表示从3环来
.text:0040773E sub esp, 48h ; esp 指向 \_KTRAP\_FRAME
.text:00407741 sub ebp, 29Ch
.text:00407747 mov byte ptr [esi+140h], 1 ; CurrentThread.PreviousMode = 1，表示从3环调用来
.text:0040774E cmp ebp, esp
.text:00407750 jnz short loc\_46653C ; 如果 ebp != esp，跳转到异常处理
.text:00407750 ; 正常情况下，esp，ebp 均指向 \_KTRAP\_FRAME
.text:00407756 and dword ptr [ebp+2Ch], 0 ; \_KTRAP\_FRAME.Dr7 = 0
.text:0040775A test byte ptr [esi+2Ch], 0FFh
.text:0040775E mov [esi+134h], ebp ; CurrentThread.TrapFrame = ebp，即指向当前 \_KTRAP\_FRAME
.text:00407764 jnz Dr\_FastCallDrSave ; 如果 DebugActive == 1（被调试），那么跳转到 Dr\_FastCallDrSave
.text:00407764 ; Dr\_FastCallDrSave 的功能是保存调试寄存器
.text:0040776A
.text:0040776A loc\_4665B6: ; CODE XREF: Dr\_FastCallDrSave+10j
.text:0040776A ; Dr\_FastCallDrSave+7Cj
.text:0040776A mov ebx, [ebp+60h]
.text:0040776D mov edi, [ebp+68h]
.text:00407770 mov [ebp+0Ch], edx ; \_KTRAP\_FRAME.DbgArgPointer = edx, 保存3环参数指针
.text:00407773 mov dword ptr [ebp+8], 0BADB0D00h
.text:0040777A mov [ebp+0], ebx ; \_KTRAP\_FRAME.DbgEbp = \_KTRAP\_FRAME.Ebp
.text:00040777D mov [ebp+4], edi ; \_KTRAP\_FRAME.DbgEip = \_KTRAP\_FRAME.Eip
.text:00407780 sti
```

### 小结：

原来的寄存器存储到了 `_Trap_Frame` 结构体里，3 环 API 参数指针通过 `EDX` 传给 0 环。

双机调试配置参看我的另一篇文章 [[原创]内核学习-双机调试环境搭建-软件逆向-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-268519.htm)，基于 xp sp3 系统。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202107/921642_KZQ25B2F9JP37TP.jpg)

分析 `ReadProcessMemory` 了解系统调用。

## 一：分析 ReadProcessMemory

**ida 打开 `kernel32.dll`**（`ReadProcessMemory` 函数在这个 dll 里）

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202107/921642_S3HSG2AAZAXEN7Q.jpg)

参数压栈，call `NTReadVirtualMemory`，跳转至 `loc_7C802204`。

`loc_7C802204` 调用了 `sub_7C8093FD`。

![在这里插入图片描述](https://bbs.kanxue.com/upload/attach/202107/921642_WJ8S28UXPEUJ2W9.jpg)

`sub_7C8093FD` 内部调用了另一个函数 `RtlNtStatusToDosError`。
