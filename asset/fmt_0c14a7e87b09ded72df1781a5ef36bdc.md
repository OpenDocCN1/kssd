
# Assembly Disassembly and Analysis

## Main Registration Logic

*   Reference to control `e3` : N.A.

```
00532DE3   8B8304030000           mov     eax, [ebx+$0304]

* Reference to: Controls.TControl.GetText(TControl):TCaption;
|
00532DE9   E812C6F3FF             call    0046F400
00532DEE   FF75C8                 push    dword ptr [ebp-$38]
00532DF1   8D55C4                 lea     edx, [ebp-$3C]

* Reference to control `e4` : N.A.
|
00532DF4   8B830C030000           mov     eax, [ebx+$030C]

* Reference to: Controls.TControl.GetText(TControl):TCaption;
|
00532DFA   E801C6F3FF             call    0046F400
00532DFF   FF75C4                 push    dword ptr [ebp-$3C]
00532E02   8D45D4                 lea     eax, [ebp-$2C]
00532E05   BA04000000             mov     edx, $00000004

* Reference to: System.@LStrCatN;
|
00532E0A   E8FD19EDFF             call    0040480C
00532E0F   8B4DD4                 mov     ecx, [ebp-$2C]

* Possible String Reference to: 'regkey'
|
00532E12   BAE02E5300             mov     edx, $00532EE0
00532E17   8BC6                   mov     eax, esi

* Reference to: Registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);
|
00532E19   E816D5F0FF             call    00440334        //save key to registry
00532E1E   8D55C0                 lea     edx, [ebp-$40]

* Reference to control `Edit1` : N.A.
|
00532E21   8B832C030000           mov     eax, [ebx+$032C]

* Reference to: Controls.TControl.GetText(TControl):TCaption;
|
00532E27   E8D4C5F3FF             call    0046F400
00532E2C   8B4DC0                 mov     ecx, [ebp-$40]

* Possible String Reference to: 'regto'
|
00532E2F   BAF02E5300             mov     edx, $00532EF0
00532E34   8BC6                   mov     eax, esi

* Reference to: Registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);
|
00532E36   E8F9D4F0FF             call    00440334      //save username to registry
00532E3B   8BC6                   mov     eax, esi

* Reference to: System.TObject.Free(TObject);
|
00532E3D   E83608EDFF             call    00403678
00532E42   A1C4155900             mov     eax, dword ptr [$005915C4]
00532E47   8B00                   mov     eax, [eax]
00532E49   B201                   mov     dl, $01
00532E4B   8B08                   mov     ecx, [eax]
00532E4D   FF5164                 call    dword ptr [ecx+$64]

* Possible String Reference to: 'Thank you for registration! Please 
|                       reload program now.'
|
00532E50   B8002F5300             mov     eax, $00532F00

* Reference to: Dialogs.ShowMessage(AnsiString);
|
00532E55   E84E42F0FF             call    004370A8      //registration success notification
00532E5A   8BC3                   mov     eax, ebx

* Reference to: Forms.TCustomForm.Close(TCustomForm);
|
00532E5C   E85B9DF5FF             call    0048CBBC
00532E61   EB0A                   jmp     00532E6D

* Possible String Reference to: 'Sorry, this registration key is not
|                       valid.'
|
00532E63   B8402F5300             mov     eax, $00532F40

* Reference to: Dialogs.ShowMessage(AnsiString);
|
00532E68   E83B42F0FF             call    004370A8    //if jump here, show validation failure
00532E6D   33C0                   xor     eax, eax
00532E6F   5A                     pop     edx
00532E70   59                     pop     ecx
00532E71   59                     pop     ecx
00532E72   648910                 mov     fs:[eax], edx

***** FINALLY
|
* Possible String Reference to: '^\[嬪\]?
|
00532E75   688F2E5300             push    $00532E8F
00532E7A   8D45C0                 lea     eax, [ebp-$40]
00532E7D   BA10000000             mov     edx, $00000010

* Reference to: System.@LStrArrayClr(void;void;Integer);
|
00532E82   E82916EDFF             call    004044B0
00532E87   C3                     ret

* Reference to: System.@HandleFinally;
|
00532E88   E97F0FEDFF             jmp     00403E0C
00532E8D   EBEB                   jmp     00532E7A

***** END
|
00532E8F   5E                     pop     esi
00532E90   5B                     pop     ebx
00532E91   8BE5                   mov     esp, ebp
00532E93   5D                     pop     ebp
00532E94   C3                     ret

=======================================================================================
       Registration code length required not less than 20 characters.
       Continue to follow the above validation CALL to see how the registration code is validated. :)
=======================================================================================       

00588ED0   55                     push    ebp
00588ED1   8BEC                   mov     ebp, esp
00588ED3   83C4F8                 add     esp, -$08
00588ED6   53                     push    ebx
00588ED7   33C9                   xor     ecx, ecx
00588ED9   894DF8                 mov     [ebp-$08], ecx
00588EDC   8955FC                 mov     [ebp-$04], edx
00588EDF   8B45FC                 mov     eax, [ebp-$04]

* Reference to: System.@LStrAddRef(void;void):Pointer;
|
00588EE2   E855BAE7FF             call    0040493C       //this is the registration code passed above
00588EE7   33C0                   xor     eax, eax       //just like this in delphi, you'll get used to it
00588EE9   55                     push    ebp
00588EEA   6800955800             push    $00589500

***** TRY
|
00588EEF   64FF30                 push    dword ptr fs:[eax]
00588EF2   648920                 mov     fs:[eax], esp
00588EF5   8B45FC                 mov     eax, [ebp-$04]
00588EF8   8A00                   mov     al, byte ptr [eax] //take the first character of the registration code
00588EFA   3C30                   cmp     al, $30
00588EFC   724E                   jb      00588F4C
00588EFE   3C39                   cmp     al, $39
00588F00   770F                   jnbe    00588F11
00588F02   8D45F8                 lea     eax, [ebp-$08]
00588F05   BA18955800             mov     edx, $00589518

* Reference to: System.@LStrCat;
|
00588F0A   E845B8E7FF             call    00404754          //valid between '0'-'9'
00588F0F   EB3B                   jmp     00588F4C
00588F11   3C41                   cmp     al, $41
00588F13   7237                   jb      00588F4C
00588F15   3C5A                   cmp     al, $5A
00588F17   770F                   jnbe    00588F28
00588F19   8D45F8                 lea     eax, [ebp-$08]
00588F1C   BA18955800             mov     edx, $00589518

* Reference to: System.@LStrCat;
|
00588F21   E82EB8E7FF             call    00404754          //valid between 'A'-'Z'
00588F26   EB24                   jmp     00588F4C
00588F28   3C61                   cmp     al, $61
00588F2A   7220                   jb      00588F4C
00588F2C   3C7A                   cmp     al, $7A
00588F2E   770F                   jnbe    00588F3F
00588F30   8D45F8                 lea     eax, [ebp-$08]
00588F33   BA18955800             mov     edx, $00589518

* Reference to: System.@LStrCat;
|
00588F38   E817B8E7FF             call    00404754          //valid between 'a'-'z'
00588F3D   EB0D                   jmp     00588F4C
00588F3F   8D45F8                 lea     eax, [ebp-$08]
00588F42   BA24955800             mov     edx, $00589524   //here is '2'

* Reference to: System.@LStrCat;
|
00588F47   E808B8E7FF             call    00404754
00588F4C   8B45FC                 mov     eax, [ebp-$04]
00588F4F   8A4001                 mov     al, byte ptr [eax+$01] //take the second character of the registration code
00588F52   3C30                   cmp     al, $30
00588F54   7220                   jb      00588F76
00588F56   3C39                   cmp     al, $39
00588F58   770F                   jnbe    00588F69
00588F5A   8D45F8                 lea     eax, [ebp-$08]
00588F5D   BA18955800             mov     edx, $00589518   //here is '1'

* Reference to: System.@LStrCat;
|
```
