
```
System __linkproc__ DoneExcept(void)

CODE:0042E4B8 loc_0_42E4B8:
    ; CODE XREF: ...
CODE:0042E4B8  mov eax, [ebp+ExpirationServer]
CODE:0042E4BB  call SetCheckedAlready
CODE:0042E4C0 loc_0_42E4C0:
    ; CODE XREF: ...
CODE:0042E4C0  xor eax, eax
CODE:0042E4C2  pop edx
CODE:0042E4C3  pop ecx
CODE:0042E4C4  pop ecx
CODE:0042E4C5  mov fs:[eax], edx
CODE:0042E4C8  push offset loc_0_42E4DD
CODE:0042E4CD loc_0_42E4CD:
    ; CODE XREF: ...
CODE:0042E4CD  lea eax, [ebp+var_C]
CODE:0042E4D0  call System __linkproc__ LStrClr(void *)
CODE:0042E4D5  retn
CODE:0042E4D6 ; ===========================================================================
CODE:0042E4D6 loc_0_42E4D6:
    ; DATA XREF: ...
CODE:0042E4D6  jmp loc_0_401178
CODE:0042E4DB ; ===========================================================================
CODE:0042E4DB  jmp short loc_0_42E4CD
CODE:0042E4DD ; ===========================================================================
CODE:0042E4DD loc_0_42E4DD:
    ; DATA XREF: ...
CODE:0042E4DD  pop edi
CODE:0042E4DE  pop esi
CODE:0042E4DF  pop ebx
CODE:0042E4E0  mov esp, ebp
CODE:0042E4E2  pop ebp
CODE:0042E4E3  retn
CODE:0042E4E3 sub_0_42E3DC endp
CODE:0042E4E3 ; ===========================================================================
CODE:0042E4E4 dword_0_42E4E4 dd 42000000h
    ; DATA XREF: ...
    ; 42000000 REAL4 = 32
```

**It is called here:**

```
CODE:0042E52A loc_0_42E52A:
    ; CODE XREF: ...
CODE:0042E52A  mov ebx, edx
CODE:0042E52C  mov esi, eax
CODE:0042E52E  xor edx, edx
CODE:0042E530  mov eax, esi
CODE:0042E532  call System::TObject::TObject(void)
CODE:0042E537  call Sysutils::Now(void)
CODE:0042E53C  fstp qword ptr [esi+20h] <--- Store now date and time!!!
CODE:0042E53F  wait
CODE:0042E540  mov eax, esi
CODE:0042E542  mov edx, [eax]
CODE:0042E544  call dword ptr [edx+8]
CODE:0042E547  mov eax, esi
CODE:0042E549  test bl, bl
CODE:0042E54B  jz short loc_0_42E55C
CODE:0042E54D  call System __linkproc__ AfterConstruction(System::TObject *)
CODE:0042E552  pop large dword ptr fs:0
CODE:0042E559  add esp, 0Ch
CODE:0042E55C loc_0_42E55C:
    ; CODE XREF: ...
CODE:0042E55C  mov eax, esi
CODE:0042E55E  pop esi
CODE:0042E55F  pop ebx
CODE:0042E560  retn
CODE:0042E560 sub_0_42E51C endp
```

From these two pieces code, we found three *IMPORTANT* variables:

- `CurrentDateTime`
- `ExpiredDateTime`
- `FileExpiredDateTime`

so, `BPM` these two address and you can find more check code:

```
CODE:0042E3B8 ; ############### S U B R O U T I N E #######################################
CODE:0042E3B8 ; function IsExpired() : Boolean
CODE:0042E3B8 sub_0_42E3B8 proc near
    ; DATA XREF: ...
CODE:0042E3B8  push ebx
CODE:0042E3B9  mov ebx, eax
CODE:0042E3BB  mov eax, ebx
CODE:0042E3BD  mov edx, [eax]
CODE:0042E3BF  call dword ptr [edx+8]
CODE:0042E3C2  cmp byte ptr [ebx+8], 0
CODE:0042E3C6  jz short loc_0_42E3D7
CODE:0042E3C8  fld qword ptr [ebx+20h]
CODE:0042E3CB  fcomp qword ptr [ebx+10h] ; return (ExpiredDateTime > CurrentDateTime)
CODE:0042E3CE  fnstsw ax
CODE:0042E3D0  sahf
CODE:0042E3D1  ja short loc_0_42E3D7
CODE:0042E3D3  xor eax, eax
CODE:0042E3D5  pop ebx
CODE:0042E3D6  retn
CODE:0042E3D7 ; ===========================================================================
CODE:0042E3D7 loc_0_42E3D7:
    ; CODE XREF: ...
CODE:0042E3D7  mov al, 1
CODE:0042E3D9  pop ebx
CODE:0042E3DA  retn
CODE:0042E3DA sub_0_42E3B8 endp
```

You cann't simply patch this function to let me renturn *FALSE* directly because many `TExpirationServer` use `CurrentDateTime` and `ExpiredDateTime` extensively internal.

## 3. FileAge Checking

Lets now set breakpoint in `Sysutils::FileAge(System::AnsiString)`

### First time:

```
CODE:00446FA2  mov eax, [ebp+var_C]
CODE:00446FA5  call Sysutils::FileAge(System::AnsiString) ; FileAgeCheck2
CODE:00446FAA  mov esi, eax
CODE:00446FAC  call sub_0_42DF84
CODE:00446FB1  mov edx, [eax]
CODE:00446FB3  call dword ptr [edx+10h] ; GetExpiredDateTime
    ; Return FileExpiredDateTime
CODE:00446FB6  cmp esi, eax
CODE:00446FB8  jle short loc_0_446FDE
CODE:00446FBA  mov eax, [ebp+var_4]
CODE:00446FBD  add eax, 394h
CODE:00446FC2  call System __linkproc__ IntfClear(System::DelphiInterface<System::IInterface>&)
CODE:00446FC7  mov eax, [ebp+var_4]
CODE:00446FCA  add eax, 398h
CODE:00446FCF  call System __linkproc__ IntfClear(System::DelphiInterface<System::IInterface>&)
CODE:00446FD4  xor eax, eax
CODE:00446FD6  pop edx
CODE:00446FD7  pop ecx
CODE:00446FD8  pop ecx
CODE:00446FD9  mov fs:[eax], edx
CODE:00446FDC  jmp short loc_0_447038
CODE:00446FDE ; ===========================================================================
CODE:00446FDE loc_0_446FDE:
    ; CODE XREF: ...
CODE:00446FDE  mov eax, [ebp+var_4]
CODE:00446FE1  add eax, 394h
CODE:00446FE6  mov edx, ebx
CODE:00446FE8  call System __linkproc__ IntfCopy(System::DelphiInterface<System::IInterface>&,System::DelphiInterface<System::IInterface>)
CODE:00446FED  lea edx, [ebp+var_14]
CODE:00446FF0  mov eax, [ebp+var_4]
CODE:00446FF3  call sub_0_447D64
CODE:00446FF8  mov edx, [ebp+var_14]
CODE:00446FFB  mov eax, [ebp+var_4]
CODE:00446FFE  add eax, 398h
CODE:00447003  call System __linkproc__ IntfCopy(System::DelphiInterface<System::IInterface>&,System::DelphiInterface<System::IInterface>)
CODE:00447008  mov eax, [ebp+var_4]
CODE:0044700B  call sub_0_4470C0
```

### Second time:

```
CODE:00447580  movzx eax, word ptr [ebx+16h]
CODE:00447584  call sub_0_42F8B0 ; Read C:\Program Files\ModelMaker\Shared\Templates\Index.dat
CODE:00447589  test al, al
CODE:0044758B  jnz short loc_0_447592
CODE:0044758D  call Sysutils::Abort(void)
CODE:00447592 loc_0_447592:
    ; CODE XREF: ...
CODE:00447592  mov eax, [ebp+var_4]
CODE:00447595  mov
```
