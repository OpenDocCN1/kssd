
# [原创] `EFER` hook

最近在看HyperDbg这个项目，看到了一种基于VT的`syscall`挂钩方法，记录一下。`EFER`（Extended Feature Enable Register）是MSR寄存器的一种，相关功能可以在intel白皮书卷3A，第2.2.1节中找到。

![](upload/attach/202211/913941_ARTG6MBS83XC46C.png)

`EFER`寄存器的`SCE`位用来控制对`SYSCALL`/`SYSRET`的支持。

在卷2B，第4.3节可以找到`SYSCALL`指令的操作：

```
IF (CS.L ≠ 1 ) or (IA32_EFER.LMA ≠ 1) or (IA32_EFER.SCE ≠ 1)
(* Not in 64-Bit Mode or SYSCALL/SYSRET not enabled in IA32_EFER *)
THEN #UD; FI;
RCX ← RIP; (* Will contain address of next instruction *)
RIP ← IA32_LSTAR;
R11 ← RFLAGS;
RFLAGS ← RFLAGS AND NOT(IA32_FMASK);
CS.Selector ← IA32_STAR[47:32] AND FFFCH (* Operating system provides CS; RPL forced to 0 *)
(* Set rest of CS to a fixed value *)
CS.Base ← 0; (* Flat segment *)
CS.Limit ← FFFFFH; (* With 4-KByte granularity, implies a 4-GByte limit *)
CS.Type ← 11; (* Execute/read code, accessed *)
CS.S ← 1;
CS.DPL ← 0;
CS.P ← 1;
CS.L ← 1; (* Entry is to 64-bit mode *)
CS.D ← 0; (* Required if CS.L = 1 *)
CS.G ← 1; (* 4-KByte granularity *)
CPL ← 0;
SS.Selector ← IA32_STAR[47:32] + 8; (* SS just above CS *)
(* Set rest of SS to a fixed value *)
SS.Base ← 0; (* Flat segment *)
SS.Limit ← FFFFFH; (* With 4-KByte granularity, implies a 4-GByte limit *)
SS.Type ← 3; (* Read/write data, accessed *)
SS.S ← 1;
SS.DPL ← 0;
SS.P ← 1;
SS.B ← 1; (* 32-bit stack segment *)
SS.G ← 1; (* 4-KByte granularity *)
```

可以看出当`EFER`寄存器的`SCE`位为0时会产生`#UD`异常，如果我们手动将`SCE`清零，那么就会导致`#UD`异常，再通过设置VT中对`#UD`异常的响应事件就可以拦截`SYSCALL`。

用代码模拟`SYSCALL`：

```
_Use_decl_annotations_
BOOLEAN
SyscallHookEmulateSYSCALL(PGUEST_REGS Regs)
{
VMX_SEGMENT_SELECTOR Cs, Ss;
UINT32 InstructionLength;
UINT64 MsrValue;
UINT64 GuestRip;
UINT64 GuestRflags;
//
// Reading guest's RIP
//
__vmx_vmread(VMCS_GUEST_RIP, &GuestRip);
//
// Reading instruction length
//
__vmx_vmread(VMCS_VMEXIT_INSTRUCTION_LENGTH, &InstructionLength);
//
// Reading guest's Rflags
//
__vmx_vmread(VMCS_GUEST_RFLAGS, &GuestRflags);
//
// Save the address of the instruction following SYSCALL into RCX and then
// load RIP from IA32_LSTAR.
//
MsrValue = __readmsr(IA32_LSTAR);
Regs->rcx = GuestRip + InstructionLength;
GuestRip = MsrValue;
__vmx_vmwrite(VMCS_GUEST_RIP, GuestRip);
//
// Save RFLAGS into R11 and then mask RFLAGS using IA32_FMASK
//
MsrValue = __readmsr(IA32_FMASK);
Regs->r11 = GuestRflags;
GuestRflags &= ~(MsrValue | X86_FLAGS_RF);
__vmx_vmwrite(VMCS_GUEST_RFLAGS, GuestRflags);
//
// Load the CS and SS selectors with values derived from bits 47:32 of IA32_STAR
//
MsrValue = __readmsr(IA32_STAR);
Cs.Selector = (UINT16)((MsrValue >> 32) & ~3); // STAR[47:32] & ~RPL3
Cs.Base = 0; // flat segment
Cs.Limit = (UINT32)~0; // 4GB limit
Cs.Attributes.AsUInt = 0xA09B; // L+DB+P+S+DPL0+Code
SetGuestCs(&Cs);
Ss.Selector = (UINT16)(((MsrValue >> 32) & ~3) + 8); // STAR[47:32] + 8
Ss.Base = 0; // flat segment
Ss.Limit = (UINT32)~0; // 4GB limit
Ss.Attributes.AsUInt = 0xC093; // G+DB+P+S+DPL0+Data
SetGuestSs(&Ss);
return TRUE;
}
```

看下`SYSRET`的操作，可以看出也是`SCE`位置0时抛出`#UD`异常：

```
IF (CS.L ≠ 1 ) or (IA32_EFER.LMA ≠ 1) or (IA32_EFER.SCE ≠ 1)
(* Not in 64-Bit Mode or SYSCALL/SYSRET not enabled in IA32_EFER *)
THEN #UD; FI;
IF (CPL ≠ 0) OR (RCX is not canonical) THEN #GP(0); FI;
IF (operand size is 64-bit)
THEN (* Return to 64-Bit Mode *)
RIP ← RCX;
ELSE (* Return to Compatibility Mode *)
RIP ← ECX;
FI;
RFLAGS ← (R11 & 3C7FD7H) | 2; (* Clear RF, VM, reserved bits; set bit 2 *)
IF (operand size is 64-bit)
THEN CS.Selector ← IA32_STAR[63:48]+16;
ELSE CS.Selector ← IA32_STAR[63:48];
FI;
CS.Selector ← CS.Selector OR 3; (* RPL forced to 3 *)
(* Set rest of CS to a fixed value *)
CS.Base ← 0; (* Flat segment *)
CS.Limit ← FFFFFH; (* With 4-KByte granularity, implies a 4-GByte limit *)
CS.Type ← 11; (* Execute/read code, accessed *)
CS.S ← 1;
CS.DPL ← 3;
CS.P ← 1;
IF (operand size is 64-bit)
THEN (* Return to 64-Bit Mode *)
CS.L ← 1; (* 64-bit code segment *)
CS.D ← 0; (* Required if CS.L = 1 *)
ELSE (* Return to Compatibility Mode *)
CS.L ← 0; (* Compatibility mode *)
CS.D ← 1; (* 32-bit code segment *)
FI;
CS.G ← 1; (* 4-KByte granularity *)
CPL ← 3;
SS.Selector ← (IA32_STAR[63:48]+8) OR 3; (* RPL forced to 3 *)
(* Set rest of SS to a fixed value *)
SS.Base ← 0; (* Flat segment *)
SS.Limit ← FFFFFH; (* With 4-KByte granularity, implies a 4-GByte limit *)
SS.Type ← 3; (* Read/write data, accessed *)
SS.S ← 1;
SS.DPL ← 3;
SS.P ← 1;
SS.B ← 1; (* 32-bit stack segment*)
SS.G ← 1; (* 4-KByte granularity *)
```

用代码模拟：

```
_Use_decl_annotations_
BOOLEAN
SyscallHookEmulateSYSRET(PGUEST_REGS Regs)
{
VMX_SEGMENT_SELECTOR Cs, Ss;
UINT64 MsrValue;
UINT64 GuestRip;
UINT64 GuestRflags;
//
// Load RIP from RCX
//
GuestRip = Regs->rcx;
__vmx_vmwrite(VMCS_GUEST_RIP, GuestRip);
//
// Load RFLAGS from R11. Clear RF, VM, reserved bits
//
GuestRflags = (Regs->r11 & ~(X86_FLAGS_RF | X86_FLAGS_VM | X86_FLAGS_RESERVED_BITS)) | X86_FLAGS_FIXED;
__vmx_vmwrite(VMCS_GUEST_RFLAGS, GuestRflags);
//
// SYSRET loads the CS and SS selectors with values derived from bits 63:48 of IA32_STAR
//
MsrValue = __readmsr(IA32_STAR);
Cs.Selector = (UINT16)(((MsrValue >> 48) + 16) | 3); // (STAR[63:48]+16) | 3 (* RPL forced to 3 *)
Cs.Base = 0; // Flat segment
Cs.Limit = (UINT32)~0; // 4GB limit
Cs.Attributes.AsUInt = 0xA0FB; // L+DB+P+S+DPL3+Code
SetGuestCs(&Cs);
Ss.Selector = (UINT16)(((MsrValue >> 48) + 8) | 3); // (STAR[63:48]+8) | 3 (* RPL forced to 3 *)
Ss.Base = 0; // Flat segment
Ss.Limit = (UINT32)~0; // 4GB limit
Ss.Attributes.AsUInt = 0xC0F3; // G+DB+P+S+DPL3+Data
SetGuestSs(&Ss);
return TRUE;
}
```

手动清除`SCE`位并设置拦截：

```
VOID
SyscallHookConfigureEFER(BOOLEAN EnableEFERSyscallHook)
{
IA32_EFER_REGISTER MsrValue;
IA32_VMX_BASIC_REGISTER VmxBasicMsr = {0};
UINT32 VmEntryControls = 0;
UINT32 VmExitControls = 0;
//
// Reading IA32_VMX_BASIC_MSR
//
VmxBasicMsr.AsUInt = __readmsr(IA32_VMX_BASIC);
//
// Read previous VM-Entry and VM-Exit controls
//
__vmx_vmread(VMCS_CTRL_VMENTRY_CONTROLS, &VmEntryControls);
__vmx_vmread(VMCS_CTRL_PRIMARY_VMEXIT_CONTROLS, &VmExitControls);
MsrValue.AsUInt = __readmsr(IA32_EFER);
if (EnableEFERSyscallHook)
{
MsrValue.SyscallEnable = FALSE;
//
// Set VM-Entry controls to load EFER
//
__vmx_vmwrite(VMCS_CTRL_VMENTRY_CONTROLS, HvAdjustControls(VmEntryControls | VM_ENTRY_LOAD_IA32_EFER, VmxBasicMsr.VmxControls ? IA32_VMX_TRUE_ENTRY_CTLS : IA32_VMX_ENTRY_CTLS));
//
// Set VM-Exit controls to save EFER
//
__vmx_vmwrite(VMCS_CTRL_PRIMARY_VMEXIT_CONTROLS, HvAdjustControls(VmExitControls | VM_EXIT_SAVE_IA32_EFER, VmxBasicMsr.VmxControls ? IA32_VMX_TRUE_EXIT_CTLS : IA32_VMX_EXIT_CTLS));
//
// Set the GUEST EFER to use this value as the EFER
//
__vmx_vmwrite(VMCS_GUEST_EFER, MsrValue.AsUInt);
//
// also, we have to set exception bitmap to cause vm-exit on #UDs
//
HvSetExceptionBitmap(EXCEPTION_VECTOR_UNDEFINED_OPCODE);
}
else
{
MsrValue.SyscallEnable = TRUE;
//
// Set VM-Entry controls to load EFER
//
__vmx_vmwrite(VMCS_CTRL_VMENTRY_CONTROLS, HvAdjustControls(VmEntryControls & ~VM_ENTRY_LOAD_IA32_EFER, VmxBasicMsr.VmxControls ? IA32_VMX_TRUE_ENTRY_CTLS : IA32_VMX_ENTRY_CTLS));
//
// Set VM-Exit controls to save EFER
//
__vmx_vmwrite(VMCS_CTRL_PRIMARY_VMEXIT_CONTROLS, HvAdjustControls(VmExitControls & ~VM_EXIT_SAVE_IA32_EFER, VmxBasicMsr.VmxControls ? IA32_VMX_TRUE_EXIT_CTLS : IA32_VMX_EXIT_CTLS));
//
// Set the GUEST EFER to use this value as the EFER
//
__vmx_vmwrite(VMCS_GUEST_EFER, MsrValue.AsUInt);
//
// Because we're not save or load EFER on vm-exits so
// we have to set it manually
//
__writemsr(IA32_EFER, MsrValue.AsUInt);
//
// unset the exception to not cause vm-exit on #UDs
//
ProtectedHvRemoveUndefinedInstructionForDisablingSyscallSysretCommands();
}
}
```

对`#UD`事件的响应：
