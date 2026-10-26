

```asm
.text:00450886 33 C0                     xor   eax, eax

.text:00450888 EB 55                     jmp   short loc_4508DF

.text:0045088A                         ; ---------------------------------------------------------------------------

.text:0045088A

.text:0045088A                         loc_45088A:                           ; CODE XREF: sub_4506B5+1C2j

.text:0045088A 8B 55 0C                  mov   edx, [ebp+arg_4]              ; 写注册表

.text:0045088D 52                        push  edx                           ; char *

.text:0045088E E8 FD EE 00 00            call  strlen

.text:00450893 83 C4 04                  add   esp, 4

.text:00450896 83 C0 01                  add   eax, 1

.text:00450899 50                        push  eax                           ; cbData

.text:0045089A 8B 45 0C                  mov   eax, [ebp+arg_4]

.text:0045089D 50                        push  eax                           ; lpData

.text:0045089E 6A 01                     push  1                             ; dwType

.text:004508A0 6A 00                     push  0                             ; Reserved

.text:004508A2 68 44 F9 4D 00            push  offset aCode                  ; "code"

.text:004508A7 8B 8D E8 FE FF FF         mov   ecx, [ebp+hKey]

.text:004508AD 51                        push  ecx                           ; hKey

.text:004508AE FF 15 18 00 4C 00         call  ds:RegSetValueExA

.text:004508B4 85 C0                     test  eax, eax

.text:004508B6 74 11                     jz    short loc_4508C9

.text:004508B8 8B 95 E8 FE FF FF         mov   edx, [ebp+hKey]

.text:004508BE 52                        push  edx                           ; hKey

.text:004508BF FF 15 00 00 4C 00         call  ds:RegCloseKey

.text:004508C5 33 C0                     xor   eax, eax

.text:004508C7 EB 16                     jmp   short loc_4508DF

.text:004508C9                         ; ---------------------------------------------------------------------------

.text:004508C9

.text:004508C9                         loc_4508C9:                           ; CODE XREF: sub_4506B5+201j

.text:004508C9 8B 85 E8 FE FF FF         mov   eax, [ebp+hKey]

.text:004508CF 50                        push  eax                           ; hKey

.text:004508D0 FF 15 00 00 4C 00         call  ds:RegCloseKey

.text:004508D6 B8 01 00 00 00            mov   eax, 1

.text:004508DB EB 02                     jmp   short loc_4508DF

.text:004508DD                         ; ---------------------------------------------------------------------------

.text:004508DD

.text:004508DD                         loc_4508DD:                           ; CODE XREF: sub_4506B5+13Dj

.text:004508DD 33 C0                     xor   eax, eax

.text:004508DF

.text:004508DF                         loc_4508DF:                           ; CODE XREF: sub_4506B5+191j

.text:004508DF                                                               ; sub_4506B5+1D3j ...

.text:004508DF 8B E5                     mov   esp, ebp

.text:004508E1 5D                        pop   ebp

.text:004508E2 C2 08 00                  retn  8

.text:004508E2                         sub_4506B5 endp
```

在上面的可知用户名如果超过16位，就只取前面的16位，注册码为32位，而且字符为0-F，不能为其它的字符，是作为一个整数输进去的。

从上面可以看出注册的算法为：

```
if F(name)=G(serial)
    注册成功;
```

因此重点是`G(serial)`的逆函数。

进入`.text:004507D9`处的`call sub_440E77`。

为了表达方便，把代码中处理注册码的分割为小段，不同的处理方式为一大段。

```asm
.text:00440E77 55                        push  ebp                           ; 对注册码处理
.text:00440E77                                                               ; arg_0为注册码处理后的数据,共16个字节
.text:00440E77                                                               ; arg_4为结果存放地址
.text:00440E77                                                               ; 设注册码=s1s2s3s4
.text:00440E78 8B EC                     mov   ebp, esp
.text:00440E7A 83 EC 20                  sub   esp, 20h
.text:00440E7D 89 4D E0                  mov   [ebp+var_20], ecx             ; ecx这一个地址,指向一个table
.text:00440E80 8B 45 08                  mov   eax, [ebp+arg_0]
.text:00440E83 8B 08                     mov   ecx, [eax]
.text:00440E85 8B 55 E0                  mov   edx, [ebp+var_20]
.text:00440E88 03 8A 94 00 00 00         add   ecx, [edx+94h]
.text:00440E8E 89 4D EC                  mov   [ebp+var_14], ecx             ; 第一个双字:var_14,s1=s1+table[94h]
.text:00440E91 8B 45 08                  mov   eax, [ebp+arg_0]
.text:00440E94 8B 48 04                  mov   ecx, [eax+4]
.text:00440E97 8B 55 E0                  mov   edx, [ebp+var_20]
.text:00440E9A 03 8A 98 00 00 00         add   ecx, [edx+98h]
.text:00440EA0 89 4D F0                  mov   [ebp+var_10], ecx             ; 第二个双字:var_10,s2=s2+table[98h]
.text:00440EA3 8B 45 08                  mov   eax, [ebp+arg_0]
.text:00440EA6 8B 48 08                  mov   ecx, [eax+8]
.text:00440EA9 8B 55 E0                  mov   edx, [ebp+var_20]
.text:00440EAC 03 8A 9C 00 00 00         add   ecx, [edx+9Ch]
.text:00440EB2 89 4D F4                  mov   [ebp+var_C], ecx              ; 第三个双字:var_C,s3=s3+table[9Ch]
.text:00440EB5 8B 45 08                  mov   eax, [ebp+arg_0]
.text:00440EB8 8B 48 0C                  mov   ecx, [eax+0Ch]
.text:00440EBB 8B 55 E0                  mov   edx, [ebp+var_20]
.text:00440EBE 03 8A A0 00 00 00         add   ecx, [edx+0A0h]
.text:00440EC4 89 4D FC                  mov   [ebp+var_4], ecx              ; 第四个双字:var_4,s4=s4+table[0A0h]
===================================================================          ; _______________________
上面的的逆算法为:                                                            ;  上面为一段
                                                                               ; _______________________
              s1=s1-table[94h]
              s2=s2-table[98h]
              s3=s3-table[9Ch]
              s4=s4-table[0A0h]
===================================================================
.text:00440EC4
.text:00440EC4
.text:00440EC4
.text:00440EC4                                                               ;
.text:00440EC4                                                               ;
.text:00440EC7 8B 45 FC                  mov   eax, [ebp+var_4]              ; s4
.text:00440ECA C1 C8 08                  ror   eax, 8
.text:00440ECD 89 45 F8                  mov   [ebp+var_8], eax              ; var_8=tmp=(ror s4,8)
.text:00440ED0 8B 4D FC                  mov   ecx, [ebp+var_4]
.text:00440ED3 81 E1 FF 00 00 00         and   ecx, 0FFh
.text:00440ED9 8B 55 F4                  mov   edx, [ebp+var_C]
.text:00440EDC 33 14 8D BC ED 4D 00      xor   edx, table_1[ecx*4]
.text:00440EE3 89 55 F4                  mov   [ebp+var_C], edx              ; s3=s3^(table_1[(s4&0FFh)*4])
.text:00440EE6 8B 45 F8                  mov   eax, [ebp+var_8]
.text:00440EE9 25 FF 00 00 00            and   eax, 0FFh
.text:00440EEE 8B 4D F4                  mov   ecx, [ebp+var_C]
.text:00440EF1 03 0C 85 BC F1 4D 00      add   ecx, table_2[eax*4]
.text:00440EF8 89 4D F4                  mov   [ebp+var_C], ecx              ; s3=s3+table_2[(tmp&0FFh)*4]
.text:00440EFB 8B 55 FC                  mov   edx, [ebp+var_4]
.text:00440EFE C1 CA 10                  ror   edx, 10h
.text:00440F01 89 55 F8                  mov   [ebp+var_8], edx              ; var_8=tmp=(ror s4,10h)
.text:00440F04 8B 45 FC                  mov   eax, [ebp+var_4]
.text:00440F07 C1 C8 18                  ror   eax, 18h
.text:00440F0A 89 45 FC                  mov   [ebp+var_4], eax              ; s4=(ror s4,18h)
.text:00440F0D 8B 4D F8                  mov   ecx, [ebp+var_8]
.text:00440F10 81 E1 FF 00 00 00         and   ecx, 0FFh
```

