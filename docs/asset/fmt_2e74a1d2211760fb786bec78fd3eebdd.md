
## 汇编代码分析

以下是 `.text` 段中关于注册表操作的汇编代码：

```asm
.text:00450854 83 C4 04                 add   esp, 4
.text:00450857 83 C0 01                 add   eax, 1
.text:0045085A 50                       push  eax                       ; cbData
.text:0045085B 8B 55 08                 mov   edx, [ebp+lpData]
.text:0045085E 52                       push  edx                       ; lpData
.text:0045085F 6A 01                    push  1                         ; dwType
.text:00450861 6A 00                    push  0                         ; Reserved
.text:00450863 68 3C F9 4D 00           push  offset ValueName          ; "name"
.text:00450868 8B 85 E8 FE FF FF        mov   eax, [ebp+hKey]
.text:0045086E 50                       push  eax                       ; hKey
.text:0045086F FF 15 18 00 4C 00        call  ds:RegSetValueExA
.text:00450875 85 C0                    test  eax, eax
.text:00450877 74 11                    jz    short loc_45088A
.text:00450879 8B 8D E8 FE FF FF        mov   ecx, [ebp+hKey]
.text:0045087F 51                       push  ecx                       ; hKey
.text:00450880 FF 15 00 00 4C 00        call  ds:RegCloseKey
.text:00450886 33 C0                    xor   eax, eax
.text:00450888 EB 55                    jmp   short loc_4508DF

.text:0045088A
.text:0045088A
loc_45088A: ; CODE XREF: sub_4506B5+1C2j
.text:0045088A 8B 55 0C                 mov   edx, [ebp+arg_4]          ; 写注册表
.text:0045088D 52                       push  edx                       ; char *
.text:0045088E E8 FD EE 00 00           call  strlen
.text:00450893 83 C4 04                 add   esp, 4
.text:00450896 83 C0 01                 add   eax, 1
.text:00450899 50                       push  eax                       ; cbData
.text:0045089A 8B 45 0C                 mov   eax, [ebp+arg_4]
.text:0045089D 50                       push  eax                       ; lpData
.text:0045089E 6A 01                    push  1                         ; dwType
.text:004508A0 6A 00                    push  0                         ; Reserved
.text:004508A2 68 44 F9 4D 00           push  offset aCode              ; "code"
.text:004508A7 8B 8D E8 FE FF FF        mov   ecx, [ebp+hKey]
.text:004508AD 51                       push  ecx                       ; hKey
.text:004508AE FF 15 18 00 4C 00        call  ds:RegSetValueExA
.text:004508B4 85 C0                    test  eax, eax
.text:004508B6 74 11                    jz    short loc_4508C9
.text:004508B8 8B 95 E8 FE FF FF        mov   edx, [ebp+hKey]
.text:004508BE 52                       push  edx                       ; hKey
.text:004508BF FF 15 00 00 4C 00        call  ds:RegCloseKey
.text:004508C5 33 C0                    xor   eax, eax
.text:004508C7 EB 16                    jmp   short loc_4508DF

.text:004508C9
loc_4508C9: ; CODE XREF: sub_4506B5+201j
.text:004508C9 8B 85 E8 FE FF FF        mov   eax, [ebp+hKey]
.text:004508CF 50                       push  eax                       ; hKey
.text:004508D0 FF 15 00 00 4C 00        call  ds:RegCloseKey
.text:004508D6 B8 01 00 00 00           mov   eax, 1
.text:004508DB EB 02                    jmp   short loc_4508DF

.text:004508DD
loc_4508DD: ; CODE XREF: sub_4506B5+13Dj
.text:004508DD 33 C0                    xor   eax, eax

.text:004508DF
loc_4508DF: ; CODE XREF: sub_4506B5+191j
            ; sub_4506B5+1D3j ...
.text:004508DF 8B E5                    mov   esp, ebp
.text:004508E1 5D                       pop   ebp
.text:004508E2 C2 08 00                 retn  8
.text:004508E2 sub_4506B5 endp
```

### 逻辑说明

在上面的代码中可知，用户名如果超过 16 位，就只取前面的 16 位，注册码为 32 位，而且字符为 0-F，不能为其它的字符，是作为一个整数输进去的。

从上面可以看出注册的算法为：

```
if F(name)=G(serial)
      注册成功;
```

因此重点是 `G(serial)` 的逆函数。

进入 `.text:004507D9` 处的 `call sub_440E77`。

为了表达方便，把代码中处理注册码的分割为小段，不同的处理方式为一大段。

### 注册码处理代码

```asm
.text:00440E77 55                       push  ebp                       ; 对注册码处理
                                        ; arg_0 为注册码处理后的数据，共 16 个字节
                                        ; arg_4 为结果存放地址
                                        ; 设注册码=s1s2s3s4
.text:00440E78 8B EC                    mov   ebp, esp
.text:00440E7A 83 EC 20                 sub   esp, 20h
.text:00440E7D 89 4D E0                 mov   [ebp+var_20], ecx         ; ecx 这一个地址，指向一个 table
.text:00440E80 8B 45 08                 mov   eax, [ebp+arg_0]
.text:00440E83 8B 08                    mov   ecx, [eax]
.text:00440E85 8B 55 E0                 mov   edx, [ebp+var_20]
.text:00440E88 03 8A 94 00 00 00        add   ecx, [edx+94h]
.text:00440E8E 89 4D EC                 mov   [ebp+var_14], ecx         ; 第一个双字:var_14,s1=s1+table[94h]
.text:00440E91 8B 45 08                 mov   eax, [ebp+arg_0]
.text:00440E94 8B 48 04                 mov   ecx, [eax+4]
.text:00440E97 8B 55 E0                 mov   edx, [ebp+var_20]
.text:00440E9A 03 8A 98 00 00 00        add   ecx, [edx+98h]
.text:00440EA0 89 4D F0                 mov   [ebp+var_10], ecx         ; 第二个双字:var_10,s2=s2+table[98h]
.text:00440EA3 8B 45 08                 mov   eax, [ebp+arg_0]
.text:00440EA6 8B 48 08                 mov   ecx, [eax+8]
.text:00440EA9 8B 55 E0                 mov   edx, [ebp+var_20]
.text:00440EAC 03 8A 9C 00 00 00        add   ecx, [edx+9Ch]
.text:00440EB2 89 4D F4                 mov   [ebp+var_C], ecx          ; 第三个双字:var_C,s3=s3+table[9Ch]
.text:00440EB5 8B 45 08                 mov   eax, [ebp+arg_0]
.text:00440EB8 8B 48 0C                 mov   ecx, [eax+0Ch]
.text:00440EBB 8B 55 E0                 mov   edx, [ebp+var_20]
.text:00440EBE 03 8A A0 00 00 00        add   ecx, [edx+0A0h]
.text:00440EC4 89 4D FC                 mov   [ebp+var_4], ecx          ; 第四个双字:var_4,s4=s4+table[0A0h]
```

### 逆算法

上面的逆算法为：

```
s1=s1-table[94h]
s2=s2-table[98h]
s3=s3-table[9Ch]
s4=s4-table[0A0h]
```
