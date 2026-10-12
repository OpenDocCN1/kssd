
# 聊天圣手 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum791.htm`](https://www.pediy.com/kssd/pediy03/forum791.htm)

该程序使用了 `PECompact v1.40-3` 挂防 `DEBUG` 插件，脱壳倒也容易。只是找注册码找了半天：

系统信息：`D96F>BPX040095`

* 用户名：`CHINA`
* 注册码：`VILCVKCHQ2269265`

```
4A2F74->D EAX (防 DEBUG)

0167:004A2F6B 90          NOP
0167:004A2F6C 55          PUSH EBP
0167:004A2F6D 8BEC        MOV EBP,ESP
0167:004A2F6F 53          PUSH EBX
0167:004A2F70 8B00        MOV EAX,[EAX]  <-正确的注册码
0167:004A2F72 8B12        MOV EDX,[EDX]  <-你输入的注册码
0167:004A2F74 E81339FFFF  CALL 0049688C  <-注册比较
0167:004A2F79 0F94C0      SETZ AL
0167:004A2F7C 83E001      AND EAX,BYTE +01
0167:004A2F7F 5B          POP EBX
0167:004A2F80 5D          POP EBP
0167:004A2F81 C3          RET
0167:004A2F82 90          NOP
0167:004A2F83 90          NOP
0167:004A2F84 55          PUSH EBP
```

## 暴力破解与找注册码方法

作者：`THK`

目的：仅是爱好！！

通过跟踪，注意到 `00401F5D` 是关键跳转：

```
:00401F0F FF4DE4           dec [ebp-1C]
:00401F12 8D45FC           lea eax, dword ptr [ebp-04]
:00401F15 BA02000000       mov edx, 00000002
:00401F1A E87D0F0A00       call 004A2E9C
:00401F1F 8B93F4020000     mov edx, dword ptr [ebx+000002F4]
:00401F25 8B83F0020000     mov eax, dword ptr [ebx+000002F0]
:00401F2B E8F00B0700       call 00472B20
:00401F30 8B8BE0020000     mov ecx, dword ptr [ebx+000002E0]
:00401F36 BA02000000       mov edx, 00000002
:00401F3B 8B81F0010000     mov eax, dword ptr [ecx+000001F0]
:00401F41 E82E130700       call 00473274
:00401F46 8B5338           mov edx, dword ptr [ebx+38]
:00401F49 81C2ACFEFFFF     add edx, FFFFFEAC
:00401F4F E8C0120700       call 00473214
:00401F54 53               push ebx
:00401F55 E8FA330000       call 00405354  //F8 追入
:00401F5A 59               pop ecx
:00401F5B 84C0             test al, al
:00401F5D 0F85AE000000     je 00402011     //关键跳转
:00401F63 C683F003000000   mov byte ptr [ebx+000003F0], 00
:00401F6A 66C745D81400     mov [ebp-28], 0014
:00401F70 33C0             xor eax, eax
:00401F72 8D55F8           lea edx, dword ptr [ebp-08]
:00401F75 8945F8           mov dword ptr [ebp-08], eax
:00401F78 8BC3             mov eax, ebx
:00401F7A FF45E4           inc [ebp-1C]
:00401F7D E842E70500       call 004606C4
:00401F82 8D4DF8           lea ecx, dword ptr [ebp-08]
:00401F85 51               push ecx
```

在 `00401F54` 处按 `F8` 来到这里：

```
:004057DB 8D4DA8           lea ecx, dword ptr [ebp-58]
:004057DE 51               push ecx
:004057DF E87BAD0900       call 004A055F
:004057E4 59               pop ecx
:004057E5 58               pop eax
:004057E6 EB78             jmp 00405860
:004057E8 66C745B87000     mov [ebp-48], 0070
:004057EE E8C2A60900       call 0049FEB5
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:00405779(U)`

```
:004057F3 8D55F8           lea edx, dword ptr [ebp-08]  //D EDX 你输入的注册码
:004057F6 8D45FC           lea eax, dword ptr [ebp-04]  //D EAX 正确的注册码
:004057F9 E86ED70900       call 004A2F6C
:004057FE 84C0             test al, al
:00405800 7430             je 00405832     //跳就死,将此处 NOP 暴力破解
:00405802 B001             mov al, 01
:00405804 BA02000000       mov edx, 00000002
:00405809 50               push eax
:0040580A 8D45F8           lea eax, dword ptr [ebp-08]
:0040580D FF4DC4           dec [ebp-3C]
:00405810 E887D60900       call 004A2E9C
:00405815 FF4DC4           dec [ebp-3C]
:00405818 8D45FC           lea eax, dword ptr [ebp-04]
:0040581B BA02000000       mov edx, 00000002
:00405820 E877D60900       call 004A2E9C
:00405825 58               pop eax
:00405826 8B55A8           mov edx, dword ptr [ebp-58]
:00405829 64891500000000   mov dword ptr fs:[00000000], edx
:00405830 EB2E             jmp 00405860
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  * `:00405800(C)`

```
:00405832 33C0             xor eax, eax
:00405834 BA02000000       mov edx, 00000002
:00405839 50               push eax
:0040583A 8D45F8           lea eax, dword ptr [ebp-08]
:0040583D FF4DC4           dec [ebp-3C]
:00405840 E857D60900       call 004A2E9C
:00405845 FF4DC4           dec [ebp-3C]
:00405848 8D45FC           lea eax, dword ptr [ebp-04]
:0040584B BA02000000       mov edx, 00000002
:00405850 E847D60900       call 004A2E9C
:00405855 58               pop eax
:00405856 8B55A8           mov edx, dword ptr [ebp-58]
:00405859 64891500000000   mov dword ptr fs:[00000000], edx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  * `:00405678(U), :004057E6(U), :00405830(U)`

```
:00405860 5F               pop edi
:00405861 5E               pop esi
:00405862 5B               pop ebx
:00405863 8BE5             mov esp, ebp
:00405865 5D               pop ebp
```

### 我的注册码

* 机器码：`D37F02SC580016`
* 用户名：`fyb`
* 注册码：`KTUJIZBOV7235816` `4dd466d0e1e0cc123afbe504da2462ca`
