
### CODE:00446FC7

```
CODE:00446FC7                 mov    eax, [ebp+var_4]
```

### CODE:00446FCA

```
CODE:00446FCA                 add    eax, 398h
```

### CODE:00446FCF

```
CODE:00446FCF                 call    System __linkproc__ IntfClear(System::DelphiInterface<System::IInterface> &)
```

### CODE:00446FD4

```
CODE:00446FD4                 xor    eax, eax
```

### CODE:00446FD6

```
CODE:00446FD6                 pop    edx
```

### CODE:00446FD7

```
CODE:00446FD7                 pop    ecx
```

### CODE:00446FD8

```
CODE:00446FD8                 pop    ecx
```

### CODE:00446FD9

```
CODE:00446FD9                 mov    fs:[eax], edx
```

### CODE:00446FDC

```
CODE:00446FDC                 jmp    short loc_0_447038
```

### CODE:00446FDE

```
CODE:00446FDE ; ===========================================================================
CODE:00446FDE
CODE:00446FDE loc_0_446FDE:                                    ; CODE XREF: ...
CODE:00446FDE                 mov    eax, [ebp+var_4]
```

### CODE:00446FE1

```
CODE:00446FE1                 add    eax, 394h
```

### CODE:00446FE6

```
CODE:00446FE6                 mov    edx, ebx
```

### CODE:00446FE8

```
CODE:00446FE8                 call    System __linkproc__ IntfCopy(System::DelphiInterface<System::IInterface> &,System::DelphiInterface<System::IInterface>)
```

### CODE:00446FED

```
CODE:00446FED                 lea    edx, [ebp+var_14]
```

### CODE:00446FF0

```
CODE:00446FF0                 mov    eax, [ebp+var_4]
```

### CODE:00446FF3

```
CODE:00446FF3                 call    sub_0_447D64
```

### CODE:00446FF8

```
CODE:00446FF8                 mov    edx, [ebp+var_14]
```

### CODE:00446FFB

```
CODE:00446FFB                 mov    eax, [ebp+var_4]
```

### CODE:00446FFE

```
CODE:00446FFE                 add    eax, 398h
```

### CODE:00447003

```
CODE:00447003                 call    System __linkproc__ IntfCopy(System::DelphiInterface<System::IInterface> &,System::DelphiInterface<System::IInterface>)
```

### CODE:00447008

```
CODE:00447008                 mov    eax, [ebp+var_4]
```

### CODE:0044700B

```
CODE:0044700B                 call    sub_0_4470C0
```

### Second time:

### CODE:00447580

```
CODE:00447580                 movzx  eax, word ptr [ebx+16h]
```

### CODE:00447584

```
CODE:00447584                 call    sub_0_42F8B0    ; Read C:\Program Files\ModelMaker\Shared\Templates\Index.dat
```

### CODE:00447589

```
CODE:00447589                 test    al, al
```

### CODE:0044758B

```
CODE:0044758B                 jnz    short loc_0_447592
```

### CODE:0044758D

```
CODE:0044758D                 call    Sysutils::Abort(void)
```

### CODE:00447592

```
CODE:00447592
CODE:00447592 loc_0_447592:                                    ; CODE XREF: ...
CODE:00447592                 mov    eax, [ebp+var_4]
```

### CODE:00447595

```
CODE:00447595                 mov    ebx, [eax+3BCh]
```

### CODE:0044759B

```
CODE:0044759B                 mov    eax, [ebp+var_4]
```

### CODE:0044759E

```
CODE:0044759E                 inc    dword ptr [eax+3B8h]
```

### CODE:004475A4

```
CODE:004475A4                 mov    eax, [ebp+var_4]
```

### CODE:004475A7

```
CODE:004475A7                 cmp    dword ptr [eax+3B8h], 1
```

### CODE:004475AE

```
CODE:004475AE                 jnz    loc_0_44777A
```

### CODE:004475B4

```
CODE:004475B4                 xor    edx, edx
```

### CODE:004475B6

```
CODE:004475B6                 push    ebp
```

### CODE:004475B7

```
CODE:004475B7                 push    offset loc_0_447727
```

### CODE:004475BC

```
CODE:004475BC                 push    dword ptr fs:[edx]
```

### CODE:004475BF

```
CODE:004475BF                 mov    fs:[edx], esp
```

### CODE:004475C2

```
CODE:004475C2                 mov    eax, [ebp+var_4]
```

### CODE:004475C5

```
CODE:004475C5                 cmp    dword ptr [eax+394h], 0
```

### CODE:004475CC

```
CODE:004475CC                 jz     short loc_0_447629
```

### CODE:004475CE

```
CODE:004475CE                 lea    edx, [ebp+var_18]
```

### CODE:004475D1

```
CODE:004475D1                 mov    eax, [ebp+var_4]
```

### CODE:004475D4

```
CODE:004475D4                 mov    eax, [eax+394h]
```

### CODE:004475DA

```
CODE:004475DA                 mov    ecx, [eax]
```

### CODE:004475DC

```
CODE:004475DC                 call    dword ptr [ecx+18h]
```

### CODE:004475DF

```
CODE:004475DF                 mov    eax, [ebp+var_18]
```

### CODE:004475E2

```
CODE:004475E2                 lea    edx, [ebp+var_14]
```

### CODE:004475E5

```
CODE:004475E5                 mov    ecx, [eax]
```

### CODE:004475E7

```
CODE:004475E7                 call    dword ptr [ecx+18h]
```

### CODE:004475EA

```
CODE:004475EA                 mov    eax, [ebp+var_14]
```

### CODE:004475ED

```
CODE:004475ED                 call    Sysutils::FileAge(System::AnsiString)
```

### CODE:004475F2

```
CODE:004475F2                 mov    edx, [ebp+var_C]
```

### CODE:004475F5

```
CODE:004475F5                 add    edx, 10h
```

### CODE:004475F8

```
CODE:004475F8                 cmp    eax, edx
```

### CODE:004475FA

```
CODE:004475FA                 jl     short loc_0_447629
```

### CODE:004475FC

```
CODE:004475FC                 mov    eax, 5
```

### CODE:00447601

```
CODE:00447601                 call    sub_0_4040EC
```

### CODE:00447606

```
CODE:00447606                 mov    eax, 6
```

### CODE:0044760B

```
CODE:0044760B                 call    sub_0_4040EC
```

### CODE:00447610

```
CODE:00447610                 mov    eax, [ebp+var_4]
```

### CODE:00447613

```
CODE:00447613                 mov    eax, [eax+3A0h]
```

### CODE:00447619

```
CODE:00447619                 test    eax, eax
```

### CODE:0044761B

```
CODE:0044761B                 jz     short loc_0_447622
```

### CODE:0044761D

```
CODE:0044761D                 call    Forms::TCustomForm::Release(void)
```

### Third Time:

### CODE:00445237

```
CODE:00445237                 jz     short loc_0_445267
```

### CODE:00445239

```
CODE:00445239                 mov    eax, [ebx]
```

### CODE:0044523B

```
CODE:0044523B                 call    Sysutils::FileAge(System::AnsiString)
```

### CODE:00445240

```
CODE:00445240                 mov    edx, ds:off_0_4549D0 ; 20020417 <-- FIX ADDRESS
CODE:00445246                 mov    edx, [edx]
CODE:00445248                 add    edx, 2
CODE:0044524B                 cmp    eax, edx
CODE:0044524D                 jge    short loc_0_44526E
CODE:0044524F                 lea    edx, [ebp+var_18]
CODE:00445252                 mov    eax, [ebp+var_8]
CODE:00445255                 call    sub_0_444FA8
CODE:0044525A                 mov    edx, [ebp+var_18]
CODE:0044525D                 mov    eax, [ebp+var_4]
CODE:00445260                 mov    ecx, [eax]
CODE:00445262                 call    dword ptr [ecx+2Ch]
CODE:00445265                 jmp    short loc_0_44526E
```

### I bpm ds:off_0_4549D0 and found this:

### CODE:004485DD

```
CODE:004485DD                 mov    edx, ds:off_0_4549D0  <-- will change its content
CODE:004485E3                 movzx  eax, word ptr [ebx+16h]
CODE:004485E7                 call    sub_0_42F8B0
CODE:004485EC                 mov    dl, 1
CODE:004485EE                 mov    eax, ds:`__vtable__'[Graphics::TBitmap]
CODE:004485F3                 call    Graphics::TBitmap::TBitmap(void)
```

### Go to sub_0_42F8B0 :

```
.  ////////////////////////////////////////////////////////////
.  // Read file data into a TStringList from
.  // C:\Program Files\ModelMaker\Shared\Templates\Index.dat
.  ///////////////////////////////////////////////////////////
.
.
```

### CODE:0042F928

```
CODE:0042F928                 mov    edx, [ebp+var_14]
CODE:0042F92B                 lea    ecx, [ebp+var_10] ; string to return
CODE:0042F92E                 mov    eax, [ebp+var_8]
CODE:0042F931                 call    Classes::TStrings::GetValue(System::AnsiString)
CODE:0042F936                 mov    eax, [ebp+var_10] ; store EXPIRED DATE TIME
CODE:0042F939                 or      edx, 0FFFFFFFFh
CODE:0042F93C                 call    Sysutils::StrToIntDef(System::AnsiString,int)
CODE:0042F941                 mov    [esi], eax        ; CRACK here, put a right value to [esi]
CODE:0042F943                 cmp    dword ptr [esi], 0
CODE:0042F946                 setnl  [ebp+var_1]
CODE:0042F94A
```

### This file is encrypted and here is the decryped memory dump:

```
0023:03661044 43 75 73 74 6F 6D 3D 31-36 34 0D 0A 54 65 6D 70   Custom=164..Temp
0023:03661054 6C 61 74 65 73 3D 34 34-0D 0A 31 34 39 3D 37 34   lates=44..149=74
0023:03661064 37 38 37 37 39 33 30 0D-0A 57 69 7A 61 72 64 73    7877930..Wizards
0023:03661074 3D 37 35 32 0D 0A 50 61-74 74 65 72 6E 73 3D 31    =752..Patterns=1
0023:03661084 30 32 34 0D 0A 00 00 00-00 67 0F 01 64 67 0F 01    024......g..dg..
```

There are 5 lines and you can clearly see `"149=747877930"` that is written by Installer program.

### CONCLUSION

You have seen the all protection code I found, they can be classified of two catagories: 1. `TExpirationServer` 2. `Index.dat`

In first protection, the CORE security code is read expiration date
