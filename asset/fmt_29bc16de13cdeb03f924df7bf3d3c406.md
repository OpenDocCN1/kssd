
```
004CAC67    8B45 F8         mov eax,dword ptr ss:[ebp-8]          ; Serail4=Substr(Serail3,7,Length(Serail3)-6)

004CAC6A >   E8 D1E5F3FF    call Unpacked.00409240                ; ->SysUtils.Trim(AnsiString):AnsiString;overload;

004CAC6F    83BD CCFEFFFF 0 cmp dword ptr ss:[ebp-134],0

004CAC76    0F84 E2010000   je Unpacked.004CAE5E                  ; Serail4 不能为空

004CAC7C    8D95 C4FEFFFF   lea edx,dword ptr ss:[ebp-13C]

004CAC82    8B45 FC         mov eax,dword ptr ss:[ebp-4]

004CAC85 >   8B80 08030000  mov eax,dword ptr ds:[eax+308]        ; *TRegForm.FlatEdit1:TFlatEdit

004CAC8B >   E8 0C85F8FF    call Unpacked.0045319C                ; ->Controls.TControl.GetText(TControl):TCaption;

004CAC90    8B85 C4FEFFFF   mov eax,dword ptr ss:[ebp-13C]        ; 用户码

004CAC96    8D95 C8FEFFFF   lea edx,dword ptr ss:[ebp-138]

004CAC9C >   E8 9FE5F3FF    call Unpacked.00409240                ; ->SysUtils.Trim(AnsiString):AnsiString;overload;

004CACA1    8B95 C8FEFFFF   mov edx,dword ptr ss:[ebp-138]        ; 用户码

004CACA7    8B45 F8         mov eax,dword ptr ss:[ebp-8]          ; Serail4

004CACAA >   E8 E5A1F3FF    call Unpacked.00404E94                ; ->System.@LStrCmp;

004CACAF    0F85 A9010000   jnz Unpacked.004CAE5E                 ; Serail4<>用户码,则 Over!

004CACB5    B2 01           mov dl,1

004CACB7    A1 40A34300     mov eax,dword ptr ds:[43A340]

004CACBC >   E8 7FF7F6FF    call Unpacked.0043A440                ; ->Registry.TRegistry.Create(TRegistry;boolean);overload;

004CACC1    8945 F0         mov dword ptr ss:[ebp-10],eax

004CACC4    33C0            xor eax,eax

004CACC6    55              push ebp

004CACC7    68 FAAD4C00     push <Unpacked.->System.@HandleFinall>

004CACCC    64:FF30        push dword ptr fs:[eax]

004CACCF    64:8920        mov dword ptr fs:[eax],esp

004CACD2    BA 02000080     mov edx,80000002

004CACD7    8B45 F0         mov eax,dword ptr ss:[ebp-10]         ; 以下代码功能:在注册表中保存注册信息

004CACDA >   E8 01F8F6FF    call Unpacked.0043A4E0                ; ->Registry.TRegistry.SetRootKey(TRegistry;HKEY);

004CACDF    B1 01           mov cl,1

004CACE1    BA 30AF4C00     mov edx,Unpacked.004CAF30             ; ASCII "Software\\Microsoft\\Windows\\CurrentVersion\\Once"

004CACE6    8B45 F0         mov eax,dword ptr ss:[ebp-10]

004CACE9 >   E8 56F8F6FF    call Unpacked.0043A544                ; 

->Registry.TRegistry.OpenKey(TRegistry;AnsiString;Boolean):Boolean;

004CACEE    84C0            test al,al

004CACF0    0F84 EE000000   je Unpacked.004CADE4

004CACF6    B9 09000000     mov ecx,9

004CACFB    BA 68AF4C00     mov edx,Unpacked.004CAF68             ; ASCII "App"

004CAD00    8B45 F0         mov eax,dword ptr ss:[ebp-10]

004CAD03 >   E8 7CFAF6FF    call Unpacked.0043A784                ; 

->Registry.TRegistry.WriteInteger(TRegistry;AnsiString;Integer);

004CAD08    8D95 BCFEFFFF   lea edx,dword ptr ss:[ebp-144]

004CAD0E    8B45 FC         mov eax,dword ptr ss:[ebp-4]

004CAD11 >   8B80 08030000  mov eax,dword ptr ds:[eax+308]        ; *TRegForm.FlatEdit1:TFlatEdit

004CAD17 >   E8 8084F8FF    call Unpacked.0045319C                ; ->Controls.TControl.GetText(TControl):TCaption;

004CAD1C    8B85 BCFEFFFF   mov eax,dword ptr ss:[ebp-144]        ; 用户码

004CAD22    8D95 C0FEFFFF   lea edx,dword ptr ss:[ebp-140]

004CAD28 >   E8 13E5F3FF    call Unpacked.00409240                ; ->SysUtils.Trim(AnsiString):AnsiString;overload;

004CAD2D    8B8D C0FEFFFF   mov ecx,dword ptr ss:[ebp-140]

004CAD33    BA 74AF4C00     mov edx,Unpacked.004CAF74             ; ASCII "Guest"

004CAD38    8B45 F0         mov eax,dword ptr ss:[ebp-10]

004CAD3B >   E8 A0F9F6FF    call Unpacked.0043A6E0                ; 

->Registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);

004CAD40    8D95 B0FEFFFF   lea edx,dword ptr ss:[ebp-150]

004CAD46    8B45 FC         mov eax,dword ptr ss:[ebp-4]

004CAD49 >   8B80 14030000  mov eax,dword ptr ds:[eax+314]        ; *TRegForm.FlatEdit2:TFlatEdit

004CAD4F >   E8 4884F8FF    call Unpacked.0045319C                ; ->Controls.TControl.GetText(TControl):TCaption;

004CAD54    FFB5 B0FEFFFF   push dword ptr ss:[ebp-150]

004CAD5A    8D95 ACFEFFFF   lea edx,dword ptr ss:[ebp-154]

004CAD60    8B45 FC         mov eax,dword ptr ss:[ebp-4]

004CAD63 >   8B80 20030000  mov eax,dword ptr ds:[eax+320]        ; *TRegForm.FlatEdit5:TFlatEdit

004CAD69 >   E8 2E84F8FF    call Unpacked.0045319C                ; ->Controls.TControl.GetText(TControl):TCaption;

004CAD6E    FFB5 ACFEFFFF   push dword ptr ss:[ebp-154]

004CAD74    8D95 A8FEFFFF   lea edx,dword ptr ss:[ebp-158]

004CAD7A    8B45 FC         mov eax,dword ptr ss:[ebp-4]

004CAD7D >   8B80 18030000  mov eax,dword ptr ds:[eax+318]        ; *TRegForm.FlatEdit3:TFlatEdit

004CAD83 >   E8 1484F8FF    call Unpacked.0045319C                ; ->Controls.TControl.GetText(TControl):TCaption;

004CAD88    FFB5 A8FEFFFF   push dword ptr ss:[ebp-158]

004CAD8E    8D95 A4FEFFFF   lea edx,dword ptr ss:[ebp-15C]

004CAD94    8B45 FC         mov eax,dword ptr ss:[ebp-4]

004CAD97 >   8B80 1C030000  mov eax,dword ptr ds:[eax+31C]        ; *TRegForm.FlatEdit4:TFlatEdit

004CAD9D >   E8 FA83F8FF    call Unpacked.0045319C                ; ->Controls.TControl.GetText(TControl):TCaption;

004CADA2    FFB5 A4FEFFFF   push dword ptr ss:[ebp-15C]

004CADA8    8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]

004CADAE    BA 04000000     mov edx,4

004CADB3 >   E8 50A0F3FF    call Unpacked.00404E08                ; ->System.@LStrCatN;

004CADB8    8B85 B4FEFFFF   mov eax,dword ptr ss:[ebp-14C]

004CADBE    8D95 B8FEFFFF   lea edx,dword ptr ss:[ebp-148]

004CADC4 >   E8 77E4F3FF    call Unpacked.00409240                ; ->SysUtils.Trim(AnsiString):AnsiString;overload;

004CADC9    8B8D B8FEFFFF   mov ecx,dword ptr ss:[ebp-148]

004CADCF    BA 84AF4C00     mov edx,Unpacked.004CAF84             ; ASCII "ZBC"

004CADD4    8B45 F0         mov eax,dword ptr ss:[ebp-10]

004CADD7 >   E8 04F9F6FF    call Unpacked.0043A6E0                ; 

->Registry.TRegistry.WriteString(TRegistry;AnsiString;AnsiString);

004CADDC    8B45 F0         mov eax,dword ptr ss:[ebp-10]

004CADDF >   E8 CCF6F6FF    call Unpacked.0043A4B0                ; ->Registry.TRegistry.CloseKey(TRegistry);

004CADE4    33C0            xor eax,eax

004CADE6    5A              pop edx

004CADE7    59              pop ecx

004CADE8    59              pop ecx

004CADE9    64:8910        mov dword ptr fs:[eax],edx

004CADEC    68 01AE4C00     push Unpacked.004CAE01

004CADF1    8B45 F0         mov eax,dword ptr ss:[ebp-10]

004CADF4 >   E8 1F8EF3FF    call Unpacked.00403C18                ; ->System.TObject.Free(TObject);

004CADF9    C3              retn

004CADFA >^  E9 AD95F3FF    jmp Unpacked.004043AC                 ; ->System.@HandleFinally;

004CADFF   ^  EB F0          jmp short Unpacked.004CADF1

004CAE01    8D85 A0FEFFFF   lea eax,dword ptr ss:[ebp-160]

004CAE07    8D95 F0FEFFFF   lea edx,dword ptr ss:[ebp-110]

004CAE0D    B9 00010000     mov ecx,100

004CAE12 >   E8 E19EF3FF    call Unpacked.00404CF8                ; 

->System.@LStrFromArray(String;String;PAnsiChar;Integer);<+>

004CAE17    8B95 A0FEFFFF   mov edx,dword ptr ss:[ebp-160]

004CAE1D    8D45 F4         lea eax,dword ptr ss:[ebp-C]

004CAE20    B9 90AF4C00     mov ecx,Unpacked.004CAF90             ; ASCII "\\Winpt.inf"

004CAE25 >   E8 6A9FF3FF    call Unpacked.00404D94                ; ->System.@LStrCat3;

004CAE2A    8B45 F4         mov eax,dword ptr ss:[ebp-C]

004CAE2D >   E8 16EBF3FF    call Unpacked.00409948                ; ->SysUtils.FileExists(AnsiString):Boolean;

004CAE32    84C0            test al,al
```
