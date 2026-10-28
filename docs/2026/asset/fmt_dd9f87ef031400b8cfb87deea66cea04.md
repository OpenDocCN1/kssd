
```text
IF (CS.L ≠ 1) or (IA32_EFER.LMA ≠ 1) or (IA32_EFER.SCE ≠ 1) (* Not in 64-Bit Mode or SYSCALL/SYSRET not enabled in IA32_EFER *) THEN #UD; FI;
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

```cpp
_Use_decl_annotations_
BOOLEAN SyscallHookEmulateSYSRET(PGUEST_REGS Regs)
{
    VMX_SEGMENT_SELECTOR Cs, Ss;
    UINT64               MsrValue;
    UINT64               GuestRip;
    UINT64               GuestRflags;

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

手动清除SCE位并设置拦截：

```cpp
VOID SyscallHookConfigureEFER(BOOLEAN EnableEFERSyscallHook)
{
    IA32_EFER_REGISTER      MsrValue;
    IA32_VMX_BASIC_REGISTER VmxBasicMsr = {0};
    UINT32                  VmEntryControls = 0;
    UINT32                  VmExitControls = 0;

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

对#UD事件的响应：
