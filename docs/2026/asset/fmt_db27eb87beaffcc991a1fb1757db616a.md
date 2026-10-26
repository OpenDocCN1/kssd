
```asm
  ebx, [eax+3BCh]

CODE:0044759B                mov
  eax, [ebp+var_4]

CODE:0044759E                inc
  dword ptr [eax+3B8h]

CODE:004475A4                mov
  eax, [ebp+var_4]

CODE:004475A7                cmp
  dword ptr [eax+3B8h], 1

CODE:004475AE                jnz
  loc_0_44777A

CODE:004475B4                xor
  edx, edx

CODE:004475B6                push
  ebp

CODE:004475B7                push
  offset loc_0_447727

CODE:004475BC                push
  dword ptr fs:[edx]

CODE:004475BF                mov
  fs:[edx], esp

CODE:004475C2                mov
  eax, [ebp+var_4]

CODE:004475C5                cmp
  dword ptr [eax+394h], 0

CODE:004475CC                jz
    short loc_0_447629

CODE:004475CE                lea
  edx, [ebp+var_18]

CODE:004475D1                mov
  eax, [ebp+var_4]

CODE:004475D4                mov
  eax, [eax+394h]

CODE:004475DA                mov
  ecx, [eax]

CODE:004475DC                call
  dword ptr [ecx+18h]

CODE:004475DF                mov
  eax, [ebp+var_18]

CODE:004475E2                lea
  edx, [ebp+var_14]

CODE:004475E5                mov
  ecx, [eax]

CODE:004475E7                call
  dword ptr [ecx+18h]

CODE:004475EA                mov
  eax, [ebp+var_14]

CODE:004475ED                call
  Sysutils::FileAge(System::AnsiString)

CODE:004475F2                mov
  edx, [ebp+var_C]

CODE:004475F5                add
  edx, 10h

CODE:004475F8                cmp
  eax, edx

CODE:004475FA                jl
    short loc_0_447629

CODE:004475FC                mov
  eax, 5

CODE:00447601                call
  sub_0_4040EC

CODE:00447606                mov
  eax, 6

CODE:0044760B                call
  sub_0_4040EC

CODE:00447610                mov
  eax, [ebp+var_4]

CODE:00447613                mov
  eax, [eax+3A0h]

CODE:00447619                test
  eax, eax

CODE:0044761B                jz
    short loc_0_447622

CODE:0044761D                call
  Forms::TCustomForm::Release(void)
```

## Third Time

```asm
CODE:00445237                jz
    short loc_0_445267

CODE:00445239                mov
  eax, [ebx]

CODE:0044523B                call
  Sysutils::FileAge(System::AnsiString)

CODE:00445240                mov
  edx, ds:off_0_4549D0 ; 20020417 <-- FIX ADDRESS

CODE:00445246                mov
  edx, [edx]

CODE:00445248                add
  edx, 2

CODE:0044524B                cmp
  eax, edx

CODE:0044524D                jge
  short loc_0_44526E

CODE:0044524F                lea
  edx, [ebp+var_18]

CODE:00445252                mov
  eax, [ebp+var_8]

CODE:00445255                call
  sub_0_444FA8

CODE:0044525A                mov
  edx, [ebp+var_18]

CODE:0044525D                mov
  eax, [ebp+var_4]

CODE:00445260                mov
  ecx, [eax]

CODE:00445262                call
  dword ptr [ecx+2Ch]

CODE:00445265                jmp
  short loc_0_44526E
```

I bpm `ds:off\_0\_4549D0` and found this:

```asm
CODE:004485DD                mov
  edx, ds:off_0_4549D0  <-- will change its content

CODE:004485E3                movzx
eax, word ptr [ebx+16h]

CODE:004485E7                call
  sub_0_42F8B0

CODE:004485EC                mov
  dl, 1

CODE:004485EE                mov
  eax, ds:`__vtable__'[Graphics::TBitmap]

CODE:004485F3                call
  Graphics::TBitmap::TBitmap(void)
```

### Go to `sub_0_42F8B0`

```asm
.  ////////////////////////////////////////////////////////////
.  // Read file data into a TStringList from
.  // C:\Program Files\ModelMaker\Shared\Templates\Index.dat
.  ///////////////////////////////////////////////////////////
.
.
.
CODE:0042F928                mov
  edx, [ebp+var_14]

CODE:0042F92B                lea
  ecx, [ebp+var_10] ; string to return

CODE:0042F92E                mov
  eax, [ebp+var_8]

CODE:0042F931                call
  Classes::TStrings::GetValue(System::AnsiString)

CODE:0042F936                mov
  eax, [ebp+var_10] ; store EXPIRED DATE TIME

CODE:0042F939                or
    edx, 0FFFFFFFFh

CODE:0042F93C                call
  Sysutils::StrToIntDef(System::AnsiString,int)

CODE:0042F941                mov
  [esi], eax        ; CRACK here, put a right value
to [esi]

CODE:0042F943                cmp
  dword ptr [esi], 0

CODE:0042F946                setnl
[ebp+var_1]

CODE:0042F94A
```

This file is encrypted and here is the decryped memory dump:

```plaintext
0023:03661044 43 75 73 74 6F 6D 3D 31-36 34 0D 0A 54 65 6D 70  Custom=164..Temp
0023:03661054 6C 61 74 65 73 3D 34 34-0D 0A 31 34 39 3D 37 34  lates=44..149=74
0023:03661064 37 38 37 37 39 33 30 0D-0A 57 69 7A 61 72 64 73  7877930..Wizards
0023:03661074 3D 37 35 32 0D 0A 50 61-74 74 65 72 6E 73 3D 31  =752..Patterns=1
0023:03661084 30 32 34 0D 0A 00 00 00-00 67 0F 01 64 67 0F 01  024......g..dg..
```

There are 5 lines and you can clearly see `149=747877930` that is written by Installer program.

## CONCLUSION

You have seen the all protection code I found, they can be classified
of two catagories: 1. `TExpirationServer` 2. `Index.dat`

In first protection, the CORE security code is read expiration date
from the HIDDEN key of windows registry. In second part, another
expiration date is read from encrypt text file, `Index.dat`. After
the two expiration date are got, then the program will compare current
date and the file date that you are editting with them when you use
every function of it, such as Add Field, Add Class, etc. If expiration
is detected some object will be release and the program will raise
exception becuase AV error.

## THE ISSUES THAT I HAVEN'T GOT CLEAR

1. The encrypt/decrypt algorithm which used to process `Index.dat`

2. Why you are always expired once you're dectected expired? It must
   store some other information on my machine. I must delete security
   registry and uninstall, then re-install again.

3. Maybe there are other checkpoint I have not find :( I hope there is
   NONE.

4. Installer will check date too and protection code is in `MMXInstall.dll`.
   I have no time to make a \*Ripped\* version. So, if you want to install,
   you must set date to a \*Right\* date. Sorry for inconvenient.

## CRACK .......... 2009/01/31 -> 3A3F0000

### CODE 1.1: `TExpirationServer` Read Registry

```asm
.0042DFB0: 52
        push        edx

.0042DFB1: BA00003F3A
  mov        edx,03A3F0000

.0042DFB6: 8911
      mov        [ecx],edx

.0042DFB8: 33C0
      xor        eax,eax

.0042DFBA: B001
      mov        al,001

.0042DFBC: 5A
        pop        edx

.0042DFBD: C3
        retn
```

### CODE 1.2: `TExpirationServer` `GetExpiredDateTime`

```asm
.0042E46F: B301
      mov        bl,001

.0042E471: 90
        nop

.0042E472: 8B45FC
    mov        eax,[ebp][-04]

.0042E475: 885808
    mov        [eax][08],bl
```

### CODE 2: `Index.dat`

```asm
.0042F936: B800003F3A
  mov        eax,03A3F0000

.0042F93B: EB04
      jmps      .00042F941

.0042F93D: 90
        nop

.0042F93E: 90
        nop

.0042F93F: 90
        nop

.0042F940: 90
        nop

.0042F941: 8906
      mov        [esi],eax

.0042F943: 833E00
    cmp        d,[esi],000
```

```plaintext
> =====================================================================
>
>
> >fc MIDEXD6.dll MIDEXD6.old /b
> Comparing files MIDEXD6.dll and MIDEXD6.OLD
> 0002D3B0: 52 55
> 0002D3B1: BA 8B
> 0002D3B2: 00 EC
> 0002D3B3: 00 6A
> 0002D3B4: 3F 00
> 0002D3B5: 3A 6A
> 0002D3B6: 89 00
> 0002D3B7: 11 6A
> 0002D3B8: 33 00
> 0002D3B9: C0 6A
> 0002D3BA: B0 00
> 0002D3BB: 01 6A
> 0002D3BC: 5A 00
> 0002D3BD: C3 6A
> 0002D3BE: 5A 00
> 0002D3BF: C3 6A
> 0002D86F: B3 0F
> 0002D870: 01 96
```
