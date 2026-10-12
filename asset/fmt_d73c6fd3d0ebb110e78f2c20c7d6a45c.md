
```asm
:00402086 8D4580          lea eax, dword ptr [ebp-80]
:00402089 E86A280D00      call 004D48F8
:0040208E FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:00402094 8B10            mov edx, dword ptr [eax]
:00402096 52              push edx
```

\* Possible StringData Ref from Data Obj -> `软件注册`

```
                  |
```

```asm
:00402097 BAD5904E00      mov edx, 004E90D5
:0040209C 8D4584          lea eax, dword ptr [ebp-7C]
:0040209F E854280D00      call 004D48F8
:004020A4 FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:004020AA 8B10            mov edx, dword ptr [eax]
:004020AC 8B8500FFFFFF    mov eax, dword ptr [ebp+FFFFFF00]
:004020B2 59              pop ecx
:004020B3 8B18            mov ebx, dword ptr [eax]
:004020B5 FF13           call dword ptr [ebx]
:004020B7 8D8578FFFFFF    lea eax, dword ptr [ebp+FFFFFF78]
:004020BD 8B00            mov eax, dword ptr [eax]
:004020BF E8D4210B00      call 004B4298
:004020C4 8BD8            mov ebx, eax
:004020C6 FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:004020CC 8D8578FFFFFF    lea eax, dword ptr [ebp+FFFFFF78]
:004020D2 BA02000000      mov edx, 00000002
:004020D7 E8C0290D00      call 004D4A9C
:004020DC FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:004020E2 8D857CFFFFFF    lea eax, dword ptr [ebp+FFFFFF7C]
:004020E8 BA02000000      mov edx, 00000002
:004020ED E8AA290D00      call 004D4A9C
:004020F2 FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:004020F8 8D4580          lea eax, dword ptr [ebp-80]
:004020FB BA02000000      mov edx, 00000002
:00402100 E897290D00      call 004D4A9C
:00402105 FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:0040210B 8D4584          lea eax, dword ptr [ebp-7C]
:0040210E BA02000000      mov edx, 00000002
:00402113 E884290D00      call 004D4A9C
:00402118 66C78514FFFFFF1400    mov word ptr [ebp+FFFFFF14], 0014
:00402121 EB10            jmp 00402133
:00402123 33DB            xor ebx, ebx
:00402125 66C78514FFFFFF4801    mov word ptr [ebp+FFFFFF14], 0148
:0040212E E85FFC0C00      call 004D1D92
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
| :00402121(U)
|
```

```asm
:00402133 66C78514FFFFFF5801    mov word ptr [ebp+FFFFFF14], 0158
:0040213C 8D8574FFFFFF    lea eax, dword ptr [ebp+FFFFFF74]
:00402142 E891060000      call 004027D8
:00402147 8BD0            mov edx, eax
:00402149 FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:0040214F 8B85FCFEFFFF    mov eax, dword ptr [ebp+FFFFFEFC]
:00402155 E89E200B00      call 004B41F8
:0040215A 8D9574FFFFFF    lea edx, dword ptr [ebp+FFFFFF74]
:00402160 52              push edx
:00402161 8D93A6D80000    lea edx, dword ptr [ebx+0000D8A6]
:00402167 8D8570FFFFFF    lea eax, dword ptr [ebp+FFFFFF70]
:0040216D E8A2280D00      call 004D4A14
:00402172 FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:00402178 8D9570FFFFFF    lea edx, dword ptr [ebp+FFFFFF70]
:0040217E 58              pop eax
:0040217F E8142A0D00      call 004D4B98
:00402184 50              push eax
:00402185 FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:0040218B 8D8570FFFFFF    lea eax, dword ptr [ebp+FFFFFF70]
:00402191 BA02000000      mov edx, 00000002
:00402196 E801290D00      call 004D4A9C
:0040219B FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:004021A1 8D8574FFFFFF    lea eax, dword ptr [ebp+FFFFFF74]
:004021A7 BA02000000      mov edx, 00000002
:004021AC E8EB280D00      call 004D4A9C
:004021B1 59              pop ecx
:004021B2 84C9            test cl, cl
:004021B4 0F847D020000   je 00402437
:004021BA 66C78514FFFFFF6401    mov word ptr [ebp+FFFFFF14], 0164
:004021C3 66C78514FFFFFF7001    mov word ptr [ebp+FFFFFF14], 0170
:004021CC 8D8558FFFFFF    lea eax, dword ptr [ebp+FFFFFF58]
:004021D2 E801060000      call 004027D8
:004021D7 50              push eax
:004021D8 8D8568FFFFFF    lea eax, dword ptr [ebp+FFFFFF68]
:004021DE FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:004021E4 E8EF050000      call 004027D8
:004021E9 8BC8            mov ecx, eax
:004021EB FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:004021F1 8D55F0          lea edx, dword ptr [ebp-10]
```

\* Possible StringData Ref from Data Obj -> `序列号: `

```
                  |
```

```asm
:004021F4 B8FF904E00      mov eax, 004E90FF
:004021F9 E8D22C0D00      call 004D4ED0
:004021FE 8D9568FFFFFF    lea edx, dword ptr [ebp+FFFFFF68]
:00402204 8D8560FFFFFF    lea eax, dword ptr [ebp+FFFFFF60]
:0040220A 52              push edx
:0040220B E8C8050000      call 004027D8
:00402210 50              push eax
:00402211 FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
```

\* Possible StringData Ref from Data Obj -> `     `

```
                  |
```

```asm
:00402217 BA08914E00      mov edx, 004E9108
:0040221C 8D8564FFFFFF    lea eax, dword ptr [ebp+FFFFFF64]
:00402222 E8D1260D00      call 004D48F8
:00402227 FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:0040222D 8D9564FFFFFF    lea edx, dword ptr [ebp+FFFFFF64]
:00402233 59              pop ecx
:00402234 58              pop eax
:00402235 E8BA280D00      call 004D4AF4
:0040223A 8D9560FFFFFF    lea edx, dword ptr [ebp+FFFFFF60]
:00402240 8D855CFFFFFF    lea eax, dword ptr [ebp+FFFFFF5C]
:00402246 8B0A            mov ecx, dword ptr [edx]
```

\* Possible StringData Ref from Data Obj -> `00 燃撞`

```
                  |
```

```asm
:00402248 BA0E914E00      mov edx, 004E910E
:0040224D 51              push ecx
:0040224E E8A5260D00      call 004D48F8
:00402253 FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
```

\* Possible StringData Ref from Data Obj -> `请注册!!!    蓝帆打支票`

```
                  |
```

```asm
:00402259 BAE7904E00      mov edx, 004E90E7
:0040225E 8B08            mov ecx, dword ptr [eax]
:00402260 8D856CFFFFFF    lea eax, dword ptr [ebp+FFFFFF6C]
:00402266 51              push ecx
:00402267 E88C260D00      call 004D48F8
:0040226C FF8520FFFFFF    inc dword ptr [ebp+FFFFFF20]
:00402272 8B00            mov eax, dword ptr [eax]
:00402274 59              pop ecx
:00402275 5A              pop edx
:00402276 E88D680700      call 00478B08
```

\> \> \> 请注册！呵呵，到这里出来要求注册的窗口！

```asm
:0040227B 8D8558FFFFFF    lea eax, dword ptr [ebp+FFFFFF58]
:00402281 8B00            mov eax, dword ptr [eax]
```

\> \> \> EAX=13572468 试炼码

```asm
:00402283 E810200B00      call 004B4298
```

\> \> \> 把试炼码转换成 16 进制值

```asm
:00402288 8BD8            mov ebx, eax
```

\> \> \> EBX=EAX=00CF1974（H）=13572468（D）

```asm
:0040228A FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
:00402290 8D8558FFFFFF    lea eax, dword ptr [ebp+FFFFFF58]
:00402296 BA02000000      mov edx, 00000002
:0040229B E8FC270D00      call 004D4A9C
:004022A0 FF8D20FFFFFF    dec dword ptr [ebp+FFFFFF20]
```
