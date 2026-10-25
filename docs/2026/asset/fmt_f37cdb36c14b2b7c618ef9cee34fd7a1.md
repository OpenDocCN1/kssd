
# Adult PDF Encrypt V2.1

强大的PDF文档加密工具，支持标准的40-bit或者128bit加密，可以给现有的pdf文档设置权限、增加用户和所有者等等。

源程序`Adult PDF Encrypt.exe`用`Aspack`加壳，脱壳后，`W32Dasm`反汇编

```
:00408D50 55                      push ebp
:00408D51 8BEC                    mov ebp, esp
:00408D53 81C4B8FEFFFF            add esp, FFFFFEB8
:00408D59 53                      push ebx
:00408D5A 8BD8                    mov ebx, eax
:00408D5C B89CA84D00              mov eax, 004DA89C
:00408D61 E8F2020C00              call 004C9058
:00408D66 8D8DB8FEFFFF            lea ecx, dword ptr [ebp+FFFFFEB8]
:00408D6C 8D95CCFEFFFF            lea edx, dword ptr [ebp+FFFFFECC]
:00408D72 8BC3                    mov eax, ebx
:00408D74 E8A3020000              call 0040901C                            关键Call进入
:00408D79 84C0                    test al, al
:00408D7B 0F84A7000000            je 00408E28                                    关键跳转
:00408D81 66C745DC0800            mov [ebp-24], 0008
:00408D87 8D95CCFEFFFF            lea edx, dword ptr [ebp+FFFFFECC]
:00408D8D 8D45FC                  lea eax, dword ptr [ebp-04]
:00408D90 E877C70C00              call 004D550C
:00408D95 FF45E8                  inc [ebp-18]
:00408D98 8B10                    mov edx, dword ptr [eax]
:00408D9A 8B8328050000            mov eax, dword ptr [ebx+00000528]
:00408DA0 E893790600              call 00470738
:00408DA5 FF4DE8                  dec [ebp-18]
:00408DA8 8D45FC                  lea eax, dword ptr [ebp-04]
:00408DAB BA02000000              mov edx, 00000002
:00408DB0 E8B7C80C00              call 004D566C
:00408DB5 66C745DC1400            mov [ebp-24], 0014
:00408DBB 8D95B8FEFFFF            lea edx, dword ptr [ebp+FFFFFEB8]
:00408DC1 8D45F8                  lea eax, dword ptr [ebp-08]
:00408DC4 E843C70C00              call 004D550C
:00408DC9 FF45E8                  inc [ebp-18]
:00408DCC 8B10                    mov edx, dword ptr [eax]
:00408DCE 8B8308050000            mov eax, dword ptr [ebx+00000508]
:00408DD4 E85F790600              call 00470738
:00408DD9 FF4DE8                  dec [ebp-18]
:00408DDC 8D45F8                  lea eax, dword ptr [ebp-08]
:00408DDF BA02000000              mov edx, 00000002
:00408DE4 E883C80C00              call 004D566C
:00408DE9 66C745DC2000            mov [ebp-24], 0020
* Possible StringData Ref from Data Obj ->"Registered version"   注册版本
                                  |
:00408DEF BAADA64D00              mov edx, 004DA6AD
:00408DF4 8D45F4                  lea eax, dword ptr [ebp-0C]
:00408DF7 E810C70C00              call 004D550C
:00408DFC FF45E8                  inc [ebp-18]
:00408DFF 8B10                    mov edx, dword ptr [eax]
:00408E01 8B8314050000            mov eax, dword ptr [ebx+00000514]
:00408E07 E82C790600              call 00470738
:00408E0C FF4DE8                  dec [ebp-18]
:00408E0F 8D45F4                  lea eax, dword ptr [ebp-0C]
:00408E12 BA02000000              mov edx, 00000002
:00408E17 E850C80C00              call 004D566C
:00408E1C C70540714D0010270000    mov dword ptr [004D7140], 00002710
:00408E26 EB3D                    jmp 00408E65
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408D7B(C)
|
:00408E28 66C745DC2C00            mov [ebp-24], 002C
* Possible StringData Ref from Data Obj ->"Trial version"    试用版本
                                  |
:00408E2E BAC0A64D00              mov edx, 004DA6C0
:00408E33 8D45F0                  lea eax, dword ptr [ebp-10]
:00408E36 E8D1C60C00              call 004D550C
:00408E3B FF45E8                  inc [ebp-18]
:00408E3E 8B10                    mov edx, dword ptr [eax]
```

* Referenced by a CALL at Address:

```
|:00408D74
|
:0040901C 53                      push ebx
:0040901D 56                      push esi
:0040901E 57                      push edi
:0040901F 81C400FCFFFF            add esp, FFFFFC00
:00409025 8BF9                    mov edi, ecx
:00409027 8BF2                    mov esi, edx
:00409029 8BD8                    mov ebx, eax
:0040902B 6800010000              push 00000100
:00409030 6A00                    push 00000000
:00409032 8D442408                lea eax, dword ptr [esp+08]
:00409036 50                      push eax
:00409037 E814FB0B00              call 004C8B50
:0040903C 83C40C                  add esp, 0000000C
:0040903F 6800010000              push 00000100
:00409044 6A00                    push 00000000
:00409046 8D942408010000          lea edx, dword ptr [esp+00000108]
:0040904D 52                      push edx
:0040904E E8FDFA0B00              call 004C8B50
:00409053 83C40C                  add esp, 0000000C
:00409056 6800010000              push 00000100
:0040905B 6A00                    push 00000000
:0040905D 8D8C2408020000          lea ecx, dword ptr [esp+00000208]
:00409064 51                      push ecx
:00409065 E8E6FA0B00              call 004C8B50
:0040906A 83C40C                  add esp, 0000000C
:0040906D 6800010000              push 00000100
:00409072 6A00                    push 00000000
:00409074 8D842408030000          lea eax, dword ptr [esp+00000308]
:0040907B 50                      push eax
:0040907C E8CFFA0B00              call 004C8B50
:00409081 83C40C                  add esp, 0000000C
:00409084 6800010000              push 00000100
:00409089 8D542404                lea edx, dword ptr [esp+04]
:0040908D 52                      push edx
* Reference To: KERNEL32.GetSystemDirectoryA, Ord:0000h             获取系统system目录
                                  |
:0040908E E8D7CE0C00              Call 004D5F6A
* Possible StringData Ref from Data Obj ->"adultreg.ini"                    注册文件名
                                  |
:00409093 6856A74D00              push 004DA756
:00409098 8D4C2404                lea ecx, dword ptr [esp+04]
:0040909C 51                      push ecx
:0040909D E852FB0B00              call 004C8BF4
:004090A2 83C408                  add esp, 00000008
:004090A5 54                      push esp
:004090A6 6800010000              push 00000100
:004090AB 8D842408010000          lea eax, dword ptr [esp+00000108]
:004090B2 50                      push eax
:004090B3 6A00                    push 00000000
* Possible StringData Ref from Data Obj ->"Mail"                                 Mail栏
                                  |
:004090B5 6878A74D00              push 004DA778
* Possible StringData Ref from Data Obj ->"PDFEncrypt\_Register"                     标题
                                  |
:004090BA 6864A74D00              push 004DA764
* Reference To: KERNEL32.GetPrivateProfileStringA, Ord:0000h
                                  |
:004090BF E876CE0C00              Call 004D5F3A
:004090C4 54                      push esp
:004090C5 6800010000              push 00000100
:004090CA 8D942408020000          lea edx, dword ptr [esp+00000208]
:004090D1 52                      push edx
:004090D2 6A00                    push 00000000
* Possible StringData Ref from Data Obj ->"Serial"                             Serial栏
                                  |
:004090D4 6891A74D00              push 004DA791
* Possible StringData Ref from Data Obj ->"PDFEncrypt\_Register"                     标题
                                  |
:004090D9 687DA74D00              push 004DA77D
```

对注册码进行校验

```
:00409158 56                      push esi
:00409159 8BF2                    mov esi, edx
:0040915B 85F6                    test esi, esi                            注册码是否为空
:0040915D 7504                    jne 00409163                                   不空跳转
:0040915F 33C0                    xor eax, eax
:00409161 5E                      pop esi
:00409162 C3                      ret
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040915D(C)
|
:00409163 803E00                  cmp byte ptr [esi], 00
:00409166 7504                    jne 0040916C
:00409168 33C0                    xor eax, eax
```
