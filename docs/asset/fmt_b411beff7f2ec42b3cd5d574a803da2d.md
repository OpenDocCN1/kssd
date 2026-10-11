
```asm
mov othertime, 0
jmp GetAddr

GetAddr:
esto
inc IncTime
cmp IncTime, 3
je CrCother
find eip, #2B848D143B0000#
cmp $RESULT, 0
je GetAddr
mov CrCAddr, $RESULT
jmp FindIAT

FindIAT:
find eip, #8919#
cmp $RESULT, 0
je IATOther
mov IATAddr, $RESULT
bprm IATAddr, 2 // 在 `mov dword ptr ds:[ecx], ebx` 处下内存访问断点，并执行到这里
jmp Exception

//若 IAT 没有加密，则直接去修复“跳转表”

IATOther:
find eip, #8901#
cmp $RESULT, 0
jne Abort

//Crc 的另外一个特征码

CrCother:
esto
inc othertime
cmp othertime, 3
je Abort
find eip, #2B848D5C310000#
cmp $RESULT, 0
mov CrCAddr, $RESULT
je CrCother
jmp FindIAT

Exception:
cmp eip, IATAddr
je Compar
esto
jmp Exception

Compar:
cmp eip, IATAddr //处理输入表的位置
je FixIAT
cmp eip, CrCAddr //CRC 的位置
jne GotoOEP
esto
jmp Compar

FixIAT:
ASM IATAddr, "mov [ecx], eax"
sti
ASM IATAddr, "mov [ecx], ebx"
esto
jmp Compar

Abort:
msg "输入表似乎没加密：）"
ret

GotoOEP:
BPMC
esto
BPRM cbase, csize
esto
BPMC
msg "脚本完成"
ret
```

2007-3-28 于北京

`4dd466d0e1e0cc123afbe504da2462ca`
