
```
00477C0C   |.   8B5D E4       MOV EBX,DWORD PTR SS:[EBP-1C]  此值决定了什么版本。  

00477C0F   |.   EB 02         JMP SHORT BAKPHONE.00477C13  
```

10、版本：（好象不支持 12 线，为何？）bug 吗？  

```
00477C0A     /75 05         JNZ SHORT BAKPHONE.00477C11      注册码正确，不跳转  

00477C0C   |. |8B5D E4      MOV EBX,DWORD PTR SS:[EBP-1C]    此处的值决定了版本  

00477C0F   |. |EB 02        JMP SHORT BAKPHONE.00477C13  

00477C11   |> \33DB         XOR EBX,EBX                     到此处则为未注册  

00477C13   |>  8BC3         MOV EAX,EBX  

00477C15   |.  B9 0C000000  MOV ECX,0C  

00477C1A   |.  99           CDQ  

00477C1B   |.  F7F9         IDIV ECX  

00477C1D   |.  8B15 280A4800 MOV EDX,DWORD PTR DS:[480A28]          ;  BAKPHONE.00481C30  

00477C23   |.  8902         MOV DWORD PTR DS:[EDX],EAX     相除的值，小于 0 为未注册，0 为标准版，1 为专业版，2 为企业版  

00477C25   |.  8BC3         MOV EAX,EBX  

00477C27   |.  B9 0C000000  MOV ECX,0C          ECX=0C  

00477C2C   |.  99           CDQ  

00477C2D   |.  F7F9         IDIV ECX            相除  

00477C2F   |.  8BDA         MOV EBX,EDX          余数就为多少线注册  

00477C31   |.  33C0         XOR EAX,EAX  

00477C33   |.  5A           POP EDX  

00477C34   |.  59           POP ECX  

00477C35   |.  59           POP ECX  

00477C36   |.  64:8910      MOV DWORD PTR FS:[EAX],EDX  

00477C39   |.  68 797C4700  PUSH BAKPHONE.00477C79  
```

11、内存注册机：  

中断：`00477c05`  

次数：`1`  

指令：`E8`  

长度：`5`  

内存方式，在 `d` `edx`

```
 4dd466d0e1e0cc123afbe504da2462ca
```
