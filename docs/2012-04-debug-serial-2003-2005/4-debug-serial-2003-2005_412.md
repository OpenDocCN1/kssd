
# 精装友情通讯录

> 原文：[`www.pediy.com/kssd/pediy06/pediy6261.htm`](https://www.pediy.com/kssd/pediy06/pediy6261.htm)

`精装友情通讯录`
`2003` 国庆版
`Delphi6.0` 无壳,明码比较

这个软件在进入的时候会要求输入密码,原是密码是 `laofuzi`,进入更改以后,可以使用更改以后的密码,但是这个密码依然可以使用!变成了万能密码了!!

升级以后改变了注册方式，要求输入注册名了，但是只能输入数字，有点不伦不类了……

反汇编,字符串参考 `"注册码正确，感谢你的注册！"` 得到如下代码段:

`/用户名:13`
`注册码:j636m5w124|f8160bd2`

```
:004E70FC 55                       push ebp
:004E70FD 8BEC                     mov ebp, esp
:004E70FF 81C454FFFFFF             add esp, FFFFFF54
:004E7105 53                       push ebx
:004E7106 33C9                     xor ecx, ecx
:004E7108 898D54FFFFFF             mov dword ptr [ebp+FFFFFF54], ecx
:004E710E 898D58FFFFFF             mov dword ptr [ebp+FFFFFF58], ecx
:004E7114 898D6CFFFFFF             mov dword ptr [ebp+FFFFFF6C], ecx
:004E711A 898D68FFFFFF             mov dword ptr [ebp+FFFFFF68], ecx
:004E7120 898D64FFFFFF             mov dword ptr [ebp+FFFFFF64], ecx
:004E7126 898D60FFFFFF             mov dword ptr [ebp+FFFFFF60], ecx
:004E712C 898D5CFFFFFF             mov dword ptr [ebp+FFFFFF5C], ecx
:004E7132 898D74FFFFFF             mov dword ptr [ebp+FFFFFF74], ecx
:004E7138 898D70FFFFFF             mov dword ptr [ebp+FFFFFF70], ecx
:004E713E 894DFC                   mov dword ptr [ebp-04], ecx
:004E7141 8BD8                     mov ebx, eax
:004E7143 33C0                     xor eax, eax
:004E7145 55                       push ebp
:004E7146 683E734E00               push 004E733E
:004E714B 64FF30                   push dword ptr fs:[eax]
:004E714E 648920                   mov dword ptr fs:[eax], esp
:004E7151 6880000000               push 00000080
:004E7156 8D857BFFFFFF             lea eax, dword ptr [ebp+FFFFFF7B]
:004E715C 50                       push eax

* Reference To: kernel32.GetSystemDirectoryA, Ord:0000h
                   |
:004E715D E88AFFF1FF               Call 004070EC
:004E7162 8D45FC                   lea eax, dword ptr [ebp-04]
:004E7165 8D957BFFFFFF             lea edx, dword ptr [ebp+FFFFFF7B]
:004E716B B981000000               mov ecx, 00000081
:004E7170 E8EFD8F1FF               call 00404A64
:004E7175 8D9574FFFFFF             lea edx, dword ptr [ebp+FFFFFF74]
:004E717B 8B83E4040000             mov eax, dword ptr [ebx+000004E4]
:004E7181 E80E4EF5FF               call 0043BF94------------------------------>取输入的注册码
:004E7186 83BD74FFFFFF00           cmp dword ptr [ebp+FFFFFF74], 00000000----->测试是否输入了注册码
:004E718D 741A                     je 004E71A9-------------------------------->不输入会有提示
:004E718F 8D9570FFFFFF             lea edx, dword ptr [ebp+FFFFFF70]
:004E7195 8B83EC040000             mov eax, dword ptr [ebx+000004EC]
:004E719B E8F44DF5FF               call 0043BF94------------------------------>取输入的用户名
:004E71A0 83BD70FFFFFF00           cmp dword ptr [ebp+FFFFFF70], 00000000----->比较是否输入了用户名
:004E71A7 750F                     jne 004E71B8------------------------------->输入则开始运算

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E718D(C)
|

* Possible StringData Ref from Code Obj ->"注册信息没有填写齐全"
                   |
:004E71A9 B854734E00               mov eax, 004E7354
:004E71AE E839E6F4FF               call 004357EC
:004E71B3 E935010000               jmp 004E72ED

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E71A7(C)
|
:004E71B8 8D956CFFFFFF             lea edx, dword ptr [ebp+FFFFFF6C]
:004E71BE 8B83E4040000             mov eax, dword ptr [ebx+000004E4]
:004E71C4 E8CB4DF5FF               call 0043BF94-----------------------------
:004E71C9 8B856CFFFFFF             mov eax, dword ptr [ebp+FFFFFF6C]--------->取得输入的注册码
:004E71CF 50                       push eax
:004E71D0 6874734E00               push 004E7374
:004E71D5 8D955CFFFFFF             lea edx, dword ptr [ebp+FFFFFF5C]
:004E71DB 8B83EC040000             mov eax, dword ptr [ebx+000004EC]
:004E71E1 E8AE4DF5FF               call 0043BF94----------------------------
:004E71E6 8B855CFFFFFF             mov eax, dword ptr [ebp+FFFFFF5C]-------->取得输入的用户名
:004E71EC E80F1FF2FF               call 00409100---------------------------->用户名转化为 16 进制
:004E71F1 8D9560FFFFFF             lea edx, dword ptr [ebp+FFFFFF60]-------->新地址
:004E71F7 E8F8BCFFFF               call 004E2EF4---------------------------->运算一
:004E71FC 8B8560FFFFFF             mov eax, dword ptr [ebp+FFFFFF60]-------->取得生成的数字串
:004E7202 E8F91EF2FF               call 00409100---------------------------->转化为 16 进制
:004E7207 8D9564FFFFFF             lea edx, dword ptr [ebp+FFFFFF64]-------->又一个新地址
:004E720D E8C2BDFFFF               call 004E2FD4---------------------------->运算二
:004E7212 FFB564FFFFFF             push dword ptr [ebp+FFFFFF64]------------>保存运算二的结果

* Possible StringData Ref from Code Obj ->"bd2"
                   |
:004E7218 6880734E00               push 004E7380
:004E721D 8D8568FFFFFF             lea eax, dword ptr [ebp+FFFFFF68]
:004E7223 BA03000000               mov edx, 00000003
:004E7228 E847D9F1FF               call 00404B74---------------------------->运算三
:004E722D 8B9568FFFFFF             mov edx, dword ptr [ebp+FFFFFF68]-------->这里是真注册码
:004E7233 58                       pop eax\---------------------------------->恢复输入的注册码
:004E7234 E8BFD9F1FF               call 00404BF8---------------------------->这里就是验证了
:004E7239 0F858A000000             jne 004E72C9----------------------------->这里跳到注册失败对话框

* Possible StringData Ref from Code Obj ->"注册码正确，感谢你的注册！"----->上面不跳则到正确
                   |
:004E723F B88C734E00               mov eax, 004E738C
:004E7244 E8A3E5F4FF               call 004357EC
:004E7249 A1C8BD4E00               mov eax, dword ptr [004EBDC8]
:004E724E 8B8018030000             mov eax, dword ptr [eax+00000318]
:004E7254 8B8008020000             mov eax, dword ptr [eax+00000208]
:004E725A 33D2                     xor edx, edx
:004E725C E84B8BFAFF               call 0048FDAC

* Possible StringData Ref from Code Obj ->"精装友情-软件已注册"
                   |
:004E7261 BAB0734E00               mov edx, 004E73B0
:004E7266 E89D8AFAFF               call 0048FD08
:004E726B 8D8558FFFFFF             lea eax, dword ptr [ebp+FFFFFF58]

* Possible StringData Ref from Code Obj ->"hdww1z.clp"
                   |
:004E7271 B9CC734E00               mov ecx, 004E73CC
:004E7276 8B55FC                   mov edx, dword ptr [ebp-04]
:004E7279 E882D8F1FF               call 00404B00
:004E727E 8B8D58FFFFFF             mov ecx, dword ptr [ebp+FFFFFF58]
:004E7284 B201                     mov dl, 01

* Possible StringData Ref from Code Obj ->"h}F"
                   |
:004E7286 A188734600               mov eax, dword ptr [00467388]
:004E728B E8A801F8FF               call 00467438

* Possible StringData Ref from Code Obj ->"326t5wefgs"
                   |
:004E7290 68E0734E00               push 004E73E0

* Possible StringData Ref from Code Obj ->"setet567"
                   |
:004E7295 B9F4734E00               mov ecx, 004E73F4

* Possible StringData Ref from Code Obj ->"sym"
                   |
:004E729A BA08744E00               mov edx, 004E7408
:004E729F 8B18                     mov ebx, dword ptr [eax]
:004E72A1 FF5304                   call [ebx+04]
:004E72A4 8D8554FFFFFF             lea eax, dword ptr [ebp+FFFFFF54]

* Possible StringData Ref from Code Obj ->"hdww1z.clp"
                   |
:004E72AA B9CC734E00               mov ecx, 004E73CC
```



```
:004E72AF 8B55FC           mov edx, dword ptr [ebp-04]

:004E72B2 E849D8F1FF       call 00404B00

:004E72B7 8B8554FFFFFF     mov eax, dword ptr [ebp+FFFFFF54]

:004E72BD BA02000000       mov edx, 00000002

:004E72C2 E88120F2FF       call 00409348

:004E72C7 EB24             jmp 004E72ED

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E7239(C)

* Possible StringData Ref from Code Obj -> "注册失败，请重新注册！"
                              |

:004E72C9 B814744E00       mov eax, 004E7414

:004E72CE E819E5F4FF       call 004357EC

:004E72D3 33D2             xor edx, edx

:004E72D5 8B83EC040000     mov eax, dword ptr [ebx+000004EC]

:004E72DB E8E44CF5FF       call 0043BFC4

:004E72E0 33D2             xor edx, edx

:004E72E2 8B83E4040000     mov eax, dword ptr [ebx+000004E4]

:004E72E8 E8D74CF5FF       call 0043BFC4

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004E71B3(U), :004E72C7(U)

:004E72ED 33C0             xor eax, eax

:004E72EF 5A               pop edx

:004E72F0 59               pop ecx

:004E72F1 59               pop ecx

:004E72F2 648910           mov dword ptr fs:[eax], edx

:004E72F5 6845734E00       push 004E7345

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004E7343(U)

:004E72FA 8D8554FFFFFF     lea eax, dword ptr [ebp+FFFFFF54]

:004E7300 BA02000000       mov edx, 00000002

:004E7305 E816D5F1FF       call 00404820

:004E730A 8D855CFFFFFF     lea eax, dword ptr [ebp+FFFFFF5C]

:004E7310 E8E7D4F1FF       call 004047FC

:004E7315 8D8560FFFFFF     lea eax, dword ptr [ebp+FFFFFF60]

:004E731B BA03000000       mov edx, 00000003

:004E7320 E8FBD4F1FF       call 00404820

:004E7325 8D856CFFFFFF     lea eax, dword ptr [ebp+FFFFFF6C]

:004E732B BA03000000       mov edx, 00000003

:004E7330 E8EBD4F1FF       call 00404820

:004E7335 8D45FC           lea eax, dword ptr [ebp-04]

:004E7338 E8BFD4F1FF       call 004047FC

:004E733D C3               ret

**************************************************************************

## 运算一

（这个运算步骤就是以前版本根据机器码生成注册码的那个过程了）

```
004E2EF4  /$  55            PUSH EBP
004E2EF5  |.  8BEC          MOV EBP,ESP
004E2EF7  |.  33C9          XOR ECX,ECX
004E2EF9  |.  51            PUSH ECX
004E2EFA  |.  51            PUSH ECX
004E2EFB  |.  51            PUSH ECX
004E2EFC  |.  51            PUSH ECX
004E2EFD  |.  53            PUSH EBX
004E2EFE  |.  56            PUSH ESI
004E2EFF  |.  8BF2          MOV ESI,EDX
004E2F01  |.  8BD8          MOV EBX,EAX-------------------------------->用户名的 16 进制(看到这里才明白为什么注册名只能输入数字......)
004E2F03  |.  33C0          XOR EAX,EAX
004E2F05  |.  55            PUSH EBP
004E2F06  |.  68 C42F4E00   PUSH JZYQ.004E2FC4
004E2F0B  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004E2F0E  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004E2F11  |.  81F3 F1250B00 XOR EBX,0B25F1----------------------------->关键参数
004E2F17  |.  8BC3          MOV EAX,EBX
004E2F19  |.  33D2          XOR EDX,EDX
004E2F1B  |.  52            PUSH EDX                        ; /Arg2 => 00000000
004E2F1C  |.  50            PUSH EAX                        ; |Arg1
004E2F1D  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]      ; |
004E2F20  |.  E8 A761F2FF   CALL JZYQ.004090CC               ; JZYQ.004090CC
004E2F25  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004E2F28  |.  0FB600        MOVZX EAX,BYTE PTR DS:[EAX]
004E2F2B  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E2F2E  |.  0FB652 01     MOVZX EDX,BYTE PTR DS:[EDX+1]
004E2F32  |.  03C2          ADD EAX,EDX
004E2F34  |.  B9 05000000   MOV ECX,5
004E2F39  |.  99            CDQ
004E2F3A  |.  F7F9          IDIV ECX
004E2F3C  |.  80C2 34       ADD DL,34
004E2F3F  |.  8855 F8       MOV BYTE PTR SS:[EBP-8],DL
004E2F42  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004E2F45  |.  0FB640 02     MOVZX EAX,BYTE PTR DS:[EAX+2]
004E2F49  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E2F4C  |.  0FB652 03     MOVZX EDX,BYTE PTR DS:[EDX+3]
004E2F50  |.  03C2          ADD EAX,EDX
004E2F52  |.  B9 05000000   MOV ECX,5
004E2F57  |.  99            CDQ
004E2F58  |.  F7F9          IDIV ECX
004E2F5A  |.  8BDA          MOV EBX,EDX
004E2F5C  |.  80C3 33       ADD BL,33
004E2F5F  |.  885D F9       MOV BYTE PTR SS:[EBP-7],BL
004E2F62  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
004E2F65  |.  8A55 F8       MOV DL,BYTE PTR SS:[EBP-8]
004E2F68  |.  E8 6F1AF2FF   CALL JZYQ.004049DC
004E2F6D  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
004E2F70  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004E2F73  |.  B9 1B000000   MOV ECX,1B
004E2F78  |.  E8 171EF2FF   CALL JZYQ.00404D94
004E2F7D  |.  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004E2F80  |.  8BD3          MOV EDX,EBX
004E2F82  |.  E8 551AF2FF   CALL JZYQ.004049DC
004E2F87  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
004E2F8A  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004E2F8D  |.  B9 19000000   MOV ECX,19
004E2F92  |.  E8 FD1DF2FF   CALL JZYQ.00404D94
004E2F97  |.  8BC6          MOV EAX,ESI
004E2F99  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E2F9C  |.  E8 AF18F2FF   CALL JZYQ.00404850
004E2FA1  |.  33C0          XOR EAX,EAX
004E2FA3  |.  5A            POP EDX
004E2FA4  |.  59            POP ECX
004E2FA5  |.  59            POP ECX
004E2FA6  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004E2FA9  |.  68 CB2F4E00   PUSH JZYQ.004E2FCB
004E2FAE  |>  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004E2FB1  |.  BA 02000000   MOV EDX,2
004E2FB6  |.  E8 6518F2FF   CALL JZYQ.00404820
004E2FBB  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004E2FBE  |.  E8 3918F2FF   CALL JZYQ.004047FC
004E2FC3  .   C3            RETN

**************************************************************************

## 运算二

（这个过程和上面运算一很相似……）

```
004E2FD4  /$  55            PUSH EBP
004E2FD5  |.  8BEC          MOV EBP,ESP
004E2FD7  |.  33C9          XOR ECX,ECX
004E2FD9  |.  51            PUSH ECX
004E2FDA  |.  51            PUSH ECX
004E2FDB  |.  51            PUSH ECX
004E2FDC  |.  51            PUSH ECX
004E2FDD  |.  51            PUSH ECX
004E2FDE  |.  51            PUSH ECX
004E2FDF  |.  53            PUSH EBX
004E2FE0  |.  56            PUSH ESI
004E2FE1  |.  8BF2          MOV ESI,EDX
004E2FE3  |.  8BD8          MOV EBX,EAX--------------------------------->运算一的结果
004E2FE5  |.  33C0          XOR EAX,EAX
004E2FE7  |.  55            PUSH EBP
004E2FE8  |.  68 20314E00   PUSH JZYQ.004E3120
004E2FED  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004E2FF0  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004E2FF3  |.  81F3 8776FBDD XOR EBX,DDFB7687--------------------------->关键参数
004E2FF9  |.  8BC3          MOV EAX,EBX
004E2FFB  |.  33D2          XOR EDX,EDX
004E2FFD  |.  52            PUSH EDX                        ; /Arg2 => 00000000
004E2FFE  |.  50            PUSH EAX                        ; |Arg1
004E2FFF  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]      ; |
```



运算三(调整生成的注册码)

```
004E3002   |.   E8 C560F2FF   CALL JZYQ.004090CC                      ; JZYQ.004090CC
004E3007   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004E300A   |.   0FB600        MOVZX EAX,BYTE PTR DS:[EAX]
004E300D   |.   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E3010   |.   0FB652 01     MOVZX EDX,BYTE PTR DS:[EDX+1]
004E3014   |.   03C2          ADD EAX,EDX
004E3016   |.   B9 05000000   MOV ECX,5
004E301B   |.   99            CDQ
004E301C   |.   F7F9          IDIV ECX
004E301E   |.   80C2 66       ADD DL,66
004E3021   |.   8855 F8       MOV BYTE PTR SS:[EBP-8],DL
004E3024   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004E3027   |.   0FB640 02     MOVZX EAX,BYTE PTR DS:[EAX+2]
004E302B   |.   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E302E   |.   0FB652 03     MOVZX EDX,BYTE PTR DS:[EDX+3]
004E3032   |.   03C2          ADD EAX,EDX
004E3034   |.   B9 05000000   MOV ECX,5
004E3039   |.   99            CDQ
004E303A   |.   F7F9          IDIV ECX
004E303C   |.   80C2 75       ADD DL,75
004E303F   |.   8855 F9       MOV BYTE PTR SS:[EBP-7],DL
004E3042   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004E3045   |.   0FB640 04     MOVZX EAX,BYTE PTR DS:[EAX+4]
004E3049   |.   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E304C   |.   0FB652 05     MOVZX EDX,BYTE PTR DS:[EDX+5]
004E3050   |.   03C2          ADD EAX,EDX
004E3052   |.   B9 05000000   MOV ECX,5
004E3057   |.   99            CDQ
004E3058   |.   F7F9          IDIV ECX
004E305A   |.   80C2 7A       ADD DL,7A
004E305D   |.   8855 FA       MOV BYTE PTR SS:[EBP-6],DL
004E3060   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004E3063   |.   0FB640 06     MOVZX EAX,BYTE PTR DS:[EAX+6]
004E3067   |.   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E306A   |.   0FB652 07     MOVZX EDX,BYTE PTR DS:[EDX+7]
004E306E   |.   03C2          ADD EAX,EDX
004E3070   |.   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E3073   |.   0FB652 08     MOVZX EDX,BYTE PTR DS:[EDX+8]
004E3077   |.   03C2          ADD EAX,EDX
004E3079   |.   B9 05000000   MOV ECX,5
004E307E   |.   99            CDQ
004E307F   |.   F7F9          IDIV ECX
004E3081   |.   80C2 69       ADD DL,69
004E3084   |.   8855 FB       MOV BYTE PTR SS:[EBP-5],DL
004E3087   |.   8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
004E308A   |.   8A55 F8       MOV DL,BYTE PTR SS:[EBP-8]
004E308D   |.   E8 4A19F2FF   CALL JZYQ.004049DC
004E3092   |.   8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
004E3095   |.   8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004E3098   |.   B9 07000000   MOV ECX,7
004E309D   |.   E8 F21CF2FF   CALL JZYQ.00404D94
004E30A2   |.   8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004E30A5   |.   8A55 FB       MOV DL,BYTE PTR SS:[EBP-5]
004E30A8   |.   E8 2F19F2FF   CALL JZYQ.004049DC
004E30AD   |.   8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
004E30B0   |.   8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004E30B3   |.   B9 03000000   MOV ECX,3
004E30B8   |.   E8 D71CF2FF   CALL JZYQ.00404D94
004E30BD   |.   8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
004E30C0   |.   8A55 F9       MOV DL,BYTE PTR SS:[EBP-7]
004E30C3   |.   E8 1419F2FF   CALL JZYQ.004049DC
004E30C8   |.   8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]
004E30CB   |.   8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004E30CE   |.   B9 05000000   MOV ECX,5
004E30D3   |.   E8 BC1CF2FF   CALL JZYQ.00404D94
004E30D8   |.   8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
004E30DB   |.   8A55 FA       MOV DL,BYTE PTR SS:[EBP-6]
004E30DE   |.   E8 F918F2FF   CALL JZYQ.004049DC
004E30E3   |.   8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]
004E30E6   |.   8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004E30E9   |.   B9 09000000   MOV ECX,9
004E30EE   |.   E8 A11CF2FF   CALL JZYQ.00404D94
004E30F3   |.   8BC6          MOV EAX,ESI
004E30F5   |.   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004E30F8   |.   E8 5317F2FF   CALL JZYQ.00404850
004E30FD   |.   33C0          XOR EAX,EAX
004E30FF   |.   5A            POP EDX
004E3100   |.   59            POP ECX
004E3101   |.   59            POP ECX
004E3102   |.   64:8910       MOV DWORD PTR FS:[EAX],EDX
004E3105   |.   68 27314E00   PUSH JZYQ.004E3127
004E310A   |>   8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
004E310D   |.   BA 04000000   MOV EDX,4
004E3112   |.   E8 0917F2FF   CALL JZYQ.00404820
004E3117   |.   8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004E311A   |.   E8 DD16F2FF   CALL JZYQ.004047FC
004E311F   .   C3            RETN
```

```
00404B74   $   53            PUSH EBX
00404B75   .   56            PUSH ESI
00404B76   .   57            PUSH EDI
00404B77   .   52            PUSH EDX
00404B78   .   50            PUSH EAX
00404B79   .   89D3          MOV EBX,EDX
00404B7B   .   31FF          XOR EDI,EDI
00404B7D   .   8B4C94 14     MOV ECX,DWORD PTR SS:[ESP+EDX*4+14]
00404B81   .   85C9          TEST ECX,ECX
00404B83   .   74 06         JE SHORT JZYQ.00404B8B
00404B85   .   3908          CMP DWORD PTR DS:[EAX],ECX
00404B87   .   75 02         JNZ SHORT JZYQ.00404B8B
00404B89   .   89C7          MOV EDI,EAX
00404B8B   >   31C0          XOR EAX,EAX
00404B8D   >   8B4C94 14     MOV ECX,DWORD PTR SS:[ESP+EDX*4+14]
00404B91   .   85C9          TEST ECX,ECX
00404B93   .   74 09         JE SHORT JZYQ.00404B9E
00404B95   .   0341 FC       ADD EAX,DWORD PTR DS:[ECX-4]
00404B98   .   39CF          CMP EDI,ECX
00404B9A   .   75 02         JNZ SHORT JZYQ.00404B9E
00404B9C   .   31FF          XOR EDI,EDI
00404B9E   >   4A            DEC EDX
00404B9F   .^   75 EC        JNZ SHORT JZYQ.00404B8D
00404BA1   .   85FF          TEST EDI,EDI
00404BA3   .   74 14         JE SHORT JZYQ.00404BB9
00404BA5   .   89C2          MOV EDX,EAX
00404BA7   .   89F8          MOV EAX,EDI
00404BA9   .   8B37          MOV ESI,DWORD PTR DS:[EDI]
00404BAB   .   8B76 FC       MOV ESI,DWORD PTR DS:[ESI-4]
00404BAE   .   E8 85020000   CALL JZYQ.00404E38
00404BB3   .   57            PUSH EDI
00404BB4   .   0337          ADD ESI,DWORD PTR DS:[EDI]
00404BB6   .   4B            DEC EBX
00404BB7   .   EB 08         JMP SHORT JZYQ.00404BC1
00404BB9   >   E8 02FDFFFF   CALL JZYQ.004048C0
00404BBE   .   50            PUSH EAX
00404BBF   .   89C6          MOV ESI,EAX
00404BC1   >   8B449C 18     MOV EAX,DWORD PTR SS:[ESP+EBX*4+18]
00404BC5   .   89F2          MOV EDX,ESI
00404BC7   .   85C0          TEST EAX,EAX
00404BC9   .   74 0A         JE SHORT JZYQ.00404BD5
00404BCB   .   8B48 FC       MOV ECX,DWORD PTR DS:[EAX-4]
00404BCE   .   01CE          ADD ESI,ECX
00404BD0   .   E8 1FDEFFFF   CALL JZYQ.004029F4
00404BD5   >   4B            DEC EBX
00404BD6   .^   75 E9        JNZ SHORT JZYQ.00404BC1
00404BD8   .   5A            POP EDX
00404BD9   .   58            POP EAX
00404BDA   .   85FF          TEST EDI,EDI
00404BDC   .   75 0C         JNZ SHORT JZYQ.00404BEA
00404BDE   .   85D2          TEST EDX,EDX
00404BE0   .   74 03         JE SHORT JZYQ.00404BE5
00404BE2   .   FF4A F8       DEC DWORD PTR DS:[EDX-8]
00404BE5   >   E8 66FCFFFF   CALL JZYQ.00404850
00404BEA   >   5A            POP EDX
00404BEB   .   5F            POP EDI
00404BEC   .   5E            POP ESI
00404BED   .   5B            POP EBX
00404BEE   .   58            POP EAX
00404BEF   .   8D2494        LEA ESP,DWORD PTR SS:[ESP+EDX*4]
00404BF2   .   FFE0          JMP EAX
00404BF4   .   C3            RETN
```

> 中华灯谜
>
> 2003 国庆版
>
> Delphi6.0,无壳




## 反汇编分析

> 字符串参考："认证码及序列号错误！"，得如下结果：
>
> **机器码**：`84345`
>
> **注册码**：`w3336m5v593}f08283~`

```
:00507038 55                    push ebp
:00507039 8BEC                  mov ebp, esp
:0050703B 81C458FFFFFF          add esp, FFFFFF58
:00507041 53                    push ebx
:00507042 33C9                  xor ecx, ecx
:00507044 898D58FFFFFF          mov dword ptr [ebp+FFFFFF58], ecx
:0050704A 898D5CFFFFFF          mov dword ptr [ebp+FFFFFF5C], ecx
:00507050 898D74FFFFFF          mov dword ptr [ebp+FFFFFF74], ecx
:00507056 898D70FFFFFF          mov dword ptr [ebp+FFFFFF70], ecx
:0050705C 898D6CFFFFFF          mov dword ptr [ebp+FFFFFF6C], ecx
:00507062 898D68FFFFFF          mov dword ptr [ebp+FFFFFF68], ecx
:00507068 898D64FFFFFF          mov dword ptr [ebp+FFFFFF64], ecx
:0050706E 898D60FFFFFF          mov dword ptr [ebp+FFFFFF60], ecx
:00507074 894DFC                mov dword ptr [ebp-04], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00507010(C)
|

:00507077 8BD8                  mov ebx, eax
:00507079 33C0                  xor eax, eax
:0050707B 55                    push ebp
:0050707C 686E725000           push 0050726E
:00507081 64FF30               push dword ptr fs:[eax]
:00507084 648920               mov dword ptr fs:[eax], esp
:00507087 6880000000           push 00000080

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00507017(C)
|

:0050708C 8D857BFFFFFF          lea eax, dword ptr [ebp+FFFFFF7B]
:00507092 50                    push eax

* Reference To: kernel32.GetSystemDirectoryA, Ord:0000h
                          |

:00507093 E8FCFFEFFF           Call 00407094
:00507098 8D45FC               lea eax, dword ptr [ebp-04]
:0050709B 8D957BFFFFFF         lea edx, dword ptr [ebp+FFFFFF7B]
:005070A1 B981000000           mov ecx, 00000081
:005070A6 E881D9EFFF           call 00404A2C
:005070AB 8B8318030000         mov eax, dword ptr [ebx+00000318]
:005070B1 8B8008020000         mov eax, dword ptr [eax+00000208]
:005070B7 33D2                 xor edx, edx
:005070B9 E84677F6FF           call 0046E804
:005070BE 8B400C               mov eax, dword ptr [eax+0C]

* Possible StringData Ref from Code Obj -> "中华灯谜-软件已注册"
                          |

:005070C1 BA84725000           mov edx, 00507284
:005070C6 E8F5DAEFFF           call 00404BC0
:005070CB 750F                 jne 005070DC

* Possible StringData Ref from Code Obj -> "软件已注册"
                          |

:005070CD B8A0725000           mov eax, 005072A0
:005070D2 E82533F3FF           call 0043A3FC
:005070D7 E941010000           jmp 0050721D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005070CB(C)
|

:005070DC 8D9574FFFFFF          lea edx, dword ptr [ebp+FFFFFF74]
:005070E2 8B8350040000         mov eax, dword ptr [ebx+00000450]
:005070E8 E81BA1F3FF           call 00441208
:005070ED 83BD74FFFFFF00       cmp dword ptr [ebp+FFFFFF74], 00000000
:005070F4 0F840C010000         je 00507206
:005070FA 8D9570FFFFFF         lea edx, dword ptr [ebp+FFFFFF70]
:00507100 8B8350040000         mov eax, dword ptr [ebx+00000450]
:00507106 E8FDA0F3FF           call 00441208
:0050710B 8B8570FFFFFF         mov eax, dword ptr [ebp+FFFFFF70]
:00507111 50                    push eax

* Possible StringData Ref from Code Obj -> "w33"
                          |

:00507112 68B4725000           push 005072B4---------------------------------由此往下的分析见上面的文章
:00507117 8D9560FFFFFF         lea edx, dword ptr [ebp+FFFFFF60]
:0050711D 8B8348040000         mov eax, dword ptr [ebx+00000448]
:00507123 E8E0A0F3FF           call 00441208
:00507128 8B8560FFFFFF         mov eax, dword ptr [ebp+FFFFFF60]
:0050712E E89521F0FF           call 004092C8
:00507133 8D9564FFFFFF         lea edx, dword ptr [ebp+FFFFFF64]
:00507139 E80EDBFFFF           call 00504C4C
:0050713E 8B8564FFFFFF         mov eax, dword ptr [ebp+FFFFFF64]
:00507144 E87F21F0FF           call 004092C8
:00507149 8D9568FFFFFF         lea edx, dword ptr [ebp+FFFFFF68]
:0050714F E8D8DBFFFF           call 00504D2C
:00507154 FFB568FFFFFF         push dword ptr [ebp+FFFFFF68]
:0050715A 68C0725000           push 005072C0
:0050715F 8D856CFFFFFF         lea eax, dword ptr [ebp+FFFFFF6C]
:00507165 BA03000000           mov edx, 00000003
:0050716A E8CDD9EFFF           call 00404B3C
:0050716F 8B956CFFFFFF         mov edx, dword ptr [ebp+FFFFFF6C]--------------->真注册码
:00507175 58                   pop eax\----------------------------------------->输入的注册码
:00507176 E845DAEFFF           call 00404BC0----------------------------------->验证
:0050717B 0F8585000000         jne 00507206------------------------------------>跳到错误

* Possible StringData Ref from Code Obj -> "注册成功，谢谢您的注册！"
                          |

:00507181 B8CC725000           mov eax, 005072CC
:00507186 E87132F3FF           call 0043A3FC
:0050718B 8B8318030000         mov eax, dword ptr [ebx+00000318]
:00507191 8B8008020000         mov eax, dword ptr [eax+00000208]
:00507197 33D2                 xor edx, edx
:00507199 E86676F6FF           call 0046E804

* Possible StringData Ref from Code Obj -> "中华灯谜-软件已注册"
                          |

:0050719E BA84725000           mov edx, 00507284
:005071A3 E8B875F6FF           call 0046E760
:005071A8 8D855CFFFFFF         lea eax, dword ptr [ebp+FFFFFF5C]

* Possible StringData Ref from Code Obj -> "ilmCoIn_0_38.dll"
                          |

:005071AE B9F0725000           mov ecx, 005072F0
:005071B3 8B55FC               mov edx, dword ptr [ebp-04]
:005071B6 E80DD9EFFF           call 00404AC8
:005071BB 8B8D5CFFFFFF         mov ecx, dword ptr [ebp+FFFFFF5C]
:005071C1 B201                 mov dl, 01

* Possible StringData Ref from Code Obj -> "$XF"
                          |

:005071C3 A1444E4600           mov eax, dword ptr [00464E44]
:005071C8 E827DDF5FF           call 00464EF4

* Possible StringData Ref from Code Obj -> "326t5wefgs"
                          |

:005071CD 680C735000           push 0050730C

* Possible StringData Ref from Code Obj -> "setet567"
                          |

:005071D2 B920735000           mov ecx, 00507320

* Possible StringData Ref from Code Obj -> "sym"
                          |

:005071D7 BA34735000           mov edx, 00507334
:005071DC 8B18                 mov ebx, dword ptr [eax]
:005071DE FF5304               call [ebx+04]
:005071E1 8D8558FFFFFF         lea eax, dword ptr [ebp+FFFFFF58]

* Possible StringData Ref from Code Obj -> "ilmCoIn_0_38.dll"
                          |

:005071E7 B9F0725000           mov ecx, 005072F0
:005071EC 8B55FC               mov edx, dword ptr [ebp-04]
:005071EF E8D4D8EFFF           call 00404AC8
:005071F4 8B8558FFFFFF         mov eax, dword ptr [ebp+FFFFFF58]
```



> `:005071FA BA02000000` `mov edx, 00000002`

> `:005071FF E80C23F0FF` `call 00409510`

> `:00507204 EB17` `jmp 0050721D`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:005070F4(C), :0050717B(C)

> \* Possible StringData Ref from Code Obj -> `"认证码及序列号错误！"`

> `:00507206 B840735000` `mov eax, 00507340`

> `:0050720B E8EC31F3FF` `call 0043A3FC`

> `:00507210 33D2` `xor edx, edx`

> `:00507212 8B8350040000` `mov eax, dword ptr [ebx+00000450]`

> `:00507218 E81BA0F3FF` `call 00441238`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
> |:005070D7(U), :00507204(U)

> `:0050721D 33C0` `xor eax, eax`

> `:0050721F 5A` `pop edx`

> `:00507220 59` `pop ecx`

> `:00507221 59` `pop ecx`

> `:00507222 648910` `mov dword ptr fs:[eax], edx`

> `:00507225 6875725000` `push 00507275`

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00507273(U)

> `:0050722A 8D8558FFFFFF` `lea eax, dword ptr [ebp+FFFFFF58]`

> `:00507230 BA02000000` `mov edx, 00000002`

> `:00507235 E8AED5EFFF` `call 004047E8`

> `:0050723A 8D8560FFFFFF` `lea eax, dword ptr [ebp+FFFFFF60]`

> `:00507240 E87FD5EFFF` `call 004047C4`

> `:00507245 8D8564FFFFFF` `lea eax, dword ptr [ebp+FFFFFF64]`

> `:0050724B BA03000000` `mov edx, 00000003`

> `:00507250 E893D5EFFF` `call 004047E8`

> `:00507255 8D8570FFFFFF` `lea eax, dword ptr [ebp+FFFFFF70]`

> `:0050725B BA02000000` `mov edx, 00000002`

> `:00507260 E883D5EFFF` `call 004047E8`

> `:00507265 8D45FC` `lea eax, dword ptr [ebp-04]`

> `:00507268 E857D5EFFF` `call 004047C4`

> `:0050726D C3` `ret`

>
> `4dd466d0e1e0cc123afbe504da2462ca`
