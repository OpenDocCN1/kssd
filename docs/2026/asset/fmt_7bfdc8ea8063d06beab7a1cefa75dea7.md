
```
.text:00450776 66 89 45 DD               mov   [ebp-23h], ax
.text:0045077A C7 45 B4 00 00 00 00      mov   [ebp+var_4C], 0
.text:00450781 EB 09                     jmp   short loc_45078C
.text:00450783                         ; ---------------------------------------------------------------------------
.text:00450783
.text:00450783                         loc_450783:                           ; CODE XREF: sub_4506B5+115j
.text:00450783 8B 4D B4                  mov   ecx, [ebp+var_4C]
.text:00450786 83 C1 01                  add   ecx, 1
.text:00450789 89 4D B4                  mov   [ebp+var_4C], ecx             ; var_4C为计数器
.text:0045078C
.text:0045078C                         loc_45078C:                           ; CODE XREF: sub_4506B5+CCj
.text:0045078C 83 7D B4 10               cmp   [ebp+var_4C], 10h
.text:00450790 7D 3A                     jge   short loc_4507CC
.text:00450792 6A 02                     push  2                             ; size_t,复制为2个字节
.text:00450792                                                               ; 循环16次,因此注册码要32个字节
.text:00450794 8B 55 B4                  mov   edx, [ebp+var_4C]
.text:00450797 8B 45 0C                  mov   eax, [ebp+arg_4]              ; arg_4指向注册码
.text:0045079A 8D 0C 50                  lea   ecx, [eax+edx*2]              ; src
.text:0045079D 51                        push  ecx                           ; void *
.text:0045079E 8D 55 DC                  lea   edx, [ebp+var_24]             ; dest
.text:004507A1 52                        push  edx                           ; void *
.text:004507A2 E8 F5 EF 00 00            call  memcpy                        ; 复制
.text:004507A7 83 C4 0C                  add   esp, 0Ch
.text:004507AA 8D 45 C8                  lea   eax, [ebp+var_38]
.text:004507AD 50                        push  eax                           ; 存放地址
.text:004507AE 68 00 F9 4D 00            push  offset asc_4DF900             ; 以16进制数输入%x
.text:004507B3 8D 4D DC                  lea   ecx, [ebp+var_24]
.text:004507B6 51                        push  ecx                           ; char *
.text:004507B7 FF 15 94 0C 4C 00         call  ds:sscanf
.text:004507BD 83 C4 0C                  add   esp, 0Ch
.text:004507C0 8B 55 B4                  mov   edx, [ebp+var_4C]
.text:004507C3 8A 45 C8                  mov   al, byte ptr [ebp+var_38]
.text:004507C6 88 44 15 F0               mov   byte ptr [ebp+edx+var_10], al ; 处理结果写到var_10处
.text:004507C6                                                               ; 注册码的字符应该为0-F,共有32位
.text:004507CA EB B7                     jmp   short loc_450783
.text:004507CC                         ; ---------------------------------------------------------------------------
.text:004507CC
.text:004507CC                         loc_4507CC:                           ; CODE XREF: sub_4506B5+DBj
.text:004507CC 8D 4D CC                  lea   ecx, [ebp+var_34]
.text:004507CF 51                        push  ecx
.text:004507D0 8D 55 F0                  lea   edx, [ebp+var_10]
.text:004507D3 52                        push  edx                           ; 注册码,为32位的十六进制数
.text:004507D4 B9 80 1B 4E 00            mov   ecx, offset table
.text:004507D9 E8 99 06 FF FF            call  sub_440E77                    ; 对注册码处理,注册码算法
.text:004507DE 6A 10                     push  10h                           ; size_t
.text:004507E0 8D 45 CC                  lea   eax, [ebp+var_34]             ; var_34为字符串
.text:004507E3 50                        push  eax                           ; void *
.text:004507E4 8D 4D E0                  lea   ecx, [ebp+var_20]             ; var_20为字符串
.text:004507E7 51                        push  ecx                           ; void *
.text:004507E8 E8 25 F3 00 00            call  memcmp                        ; 比较两个字符串是否相等
.text:004507ED 83 C4 0C                  add   esp, 0Ch                      ; 相等则注册成功!
.text:004507F0 85 C0                     test  eax, eax
.text:004507F2 0F 85 E5 00 00 00         jnz   loc_4508DD
.text:004507F8 8D 95 E8 FE FF FF         lea   edx, [ebp+hKey]
.text:004507FE 52                        push  edx                           ; phkResult
.text:004507FF 68 3F 00 0F 00            push  0F003Fh                       ; samDesired
.text:00450804 6A 00                     push  0                             ; ulOptions
.text:00450806 68 04 F9 4D 00            push  offset SubKey                 ; "SOFTWARE\\Microsoft\\sabdu"
.text:0045080B 68 02 00 00 80            push  80000002h                     ; hKey
.text:00450810 FF 15 08 00 4C 00         call  ds:RegOpenKeyExA
.text:00450816 85 C0                     test  eax, eax
.text:00450818 74 31                     jz    short loc_45084B
.text:0045081A 6A 00                     push  0                             ; lpdwDisposition
.text:0045081C 8D 85 E8 FE FF FF         lea   eax, [ebp+hKey]
.text:00450822 50                        push  eax                           ; phkResult
.text:00450823 6A 00                     push  0                             ; lpSecurityAttributes
.text:00450825 68 3F 00 0F 00            push  0F003Fh                       ; samDesired
.text:0045082A 6A 00                     push  0                             ; dwOptions
.text:0045082C 6A 00                     push  0                             ; lpClass
.text:0045082E 6A 00                     push  0                             ; Reserved
.text:00450830 68 20 F9 4D 00            push  offset aSoftwareMicr_0        ; "SOFTWARE\\Microsoft\\sabdu"
.text:00450835 68 02 00 00 80            push  80000002h                     ; hKey
.text:0045083A FF 15 04 00 4C 00         call  ds:RegCreateKeyExA
.text:00450840 85 C0                     test  eax, eax
.text:00450842 74 07                     jz    short loc_45084B
.text:00450844 33 C0                     xor   eax, eax
.text:00450846 E9 94 00 00 00            jmp   loc_4508DF
.text:0045084B                         ; ---------------------------------------------------------------------------
.text:0045084B
.text:0045084B                         loc_45084B:                           ; CODE XREF: sub_4506B5+163j
.text:0045084B                                                               ; sub_4506B5+18Dj
.text:0045084B 8B 4D 08                  mov   ecx, [ebp+lpData]             ; 写注册表
.text:0045084E 51                        push  ecx                           ; char *
.text:0045084F E8 3C EF 00 00            call  strlen
.text:00450854 83 C4 04                  add   esp, 4
.text:00450857 83 C0 01                  add   eax, 1
.text:0045085A 50                        push  eax                           ; cbData
.text:0045085B 8B 55 08                  mov   edx, [ebp+lpData]
.text:0045085E 52                        push  edx                           ; lpData
.text:0045085F 6A 01                     push  1                             ; dwType
.text:00450861 6A 00                     push  0                             ; Reserved
.text:00450863 68 3C F9 4D 00            push  offset ValueName              ; "name"
.text:00450868 8B 85 E8 FE FF FF         mov   eax, [ebp+hKey]
.text:0045086E 50                        push  eax                           ; hKey
.text:0045086F FF 15 18 00 4C 00         call  ds:RegSetValueExA
.text:00450875 85 C0                     test  eax, eax
.text:00450877 74 11                     jz    short loc_45088A
.text:00450879 8B 8D E8 FE FF FF         mov   ecx, [ebp+hKey]
.text:0045087F 51                        push  ecx                           ; hKey
.text:00450880 FF 15 00 00 4C 00         call  ds:RegCloseKey
```
