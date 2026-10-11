
# Recover My Files v3.98 [5104] 脱壳脚本

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-599.htm`](https://www.pediy.com/kssd/pediy08/pediy8-599.htm)

只是在 `3.98[5104]` 测试过，里面有一个参数 `RealIatAddr`

把这行 `mov RealIatAddr,7B7280` 修改一下，也许能用于其他版本

跑到 `oep` 以后，可以用 `arminline0.96f` 完美修复 `Code Splicing`

然后 `dump`，修复 `iat` 就可以了

```
//
#log
var BpAddr
var TmpDw
var TmpDw1
var pMutexName
var CodeSecAddr
var Patch01
var Patch02
var SaveIat
var IatSize
var IatFileBin
var RealIatAddr
var SaveEip
var SplicCodeAddr
var SplicCodeSize
var TmpDw
var AddrMarkS
var AddrMarkE
var PushfdCnt
var PopfdCnt
var AddrS
var AddrNextPushfd
var AddrNextPopfd

mov RealIatAddr,7B7280    //这个参数需要自己找到修改
mov IatSize,1000
mov CodeSecAddr,401000

mov TmpDw,eip
gpa "OutputDebugStringA", "KERNEL32.dll"
mov [$RESULT], #C20400#
gpa "OpenMutexA", "KERNEL32.dll"
mov BpAddr,$RESULT
bp BpAddr
WaitMutex:
eob BpOpenMutex1
esto
BpOpenMutex1:
cmp eip,BpAddr
jne WaitMutex
mov TmpDw,esp
add TmpDw,0C
mov pMutexName,[TmpDw]
mov eip,CodeSecAddr
mov TmpDw,eip
mov [TmpDw],68909C60
add TmpDw,4
mov [TmpDw],pMutexName
add TmpDw,4
mov [TmpDw],6A006A
add TmpDw,4
asm TmpDw, "call CreateMutexA"
add TmpDw,5
mov [TmpDw],9090619D
add TmpDw,4
asm TmpDw, "jmp OpenMutexA"

eob BpOpenMutex2
esto
BpOpenMutex2:
bc BpAddr
fill CodeSecAddr,20,0

gpa "GetEnvironmentVariableA", "KERNEL32.dll"
mov BpAddr,$RESULT
bp BpAddr
eob BpGetEnV
LoopBpGetEnV:
esto
BpGetEnV:
mov TmpDw,esp
add TmpDw,4
mov TmpDw,[TmpDw]
cmp [TmpDw],53525F
jne LoopBpGetEnV
bc BpAddr
mov TmpDw,esp
add TmpDw,8
mov TmpDw,[TmpDw]
mov [TmpDw],38383838

gpa "VirtualAlloc", "KERNEL32.dll"
mov BpAddr,$RESULT
bphws BpAddr,"x"
eob BpWaitSplic
LoopWaitSplic:
esto
BpWaitSplic:
mov TmpDw,esp
mov TmpDw,[TmpDw]
add TmpDw,0C
mov TmpDw,[TmpDw]
cmp TmpDw,8B3C7400
jne LoopWaitSplic
bphwc BpAddr
mov TmpDw,esp
add TmpDw,4
mov SplicCodeAddr,[TmpDw]
mov TmpDw,esp
add TmpDw,8
mov SplicCodeSize,[TmpDw]

gpa "GetTickCount", "KERNEL32.dll"
mov BpAddr,$RESULT
bphws BpAddr,"x"
eob BpTickCount
LoopTickCount:
esto
BpTickCount:
mov TmpDw,esp
mov TmpDw,[TmpDw]
log TmpDw
add TmpDw,6
mov TmpDw,[TmpDw]
log TmpDw
cmp TmpDw,8558016A
jne LoopTickCount

bphwc BpAddr
mov TmpDw,[esp]
find TmpDw,#8378080074??6800010000#
cmp $RESULT,0
je Error
mov Patch01,$RESULT
mov [Patch01],#83780800EB#
find TmpDw,#6BC93281C1D00700003BC176#
cmp $RESULT,0
je Error
mov Patch02,$RESULT
mov [Patch02],#6BC93281C1D00700003BC1EB#

find TmpDw,#33D2B910270000F7F18985????????8B85????????8B00#
cmp $RESULT,0
je Error
mov BpAddr,$RESULT
add BpAddr,15
bp BpAddr
eob BpSaveIat
esto
BpSaveIat:
bc BpAddr
mov [Patch01],#8378080074#
mov [Patch02],#6BC93281C1D00700003BC176#
mov SaveIat,eax
log SaveIat
eval "SaveIat{SaveIat}.bin"
mov IatFileBin,$RESULT
mov TmpDw1,SaveIat
loc_0:
mov TmpDw,[TmpDw1]
sub TmpDw,eip
cmp TmpDw,80000000
jb loc_1
neg TmpDw
loc_1:
cmp TmpDw,80000
ja loc_2
mov TmpDw,TmpDw1
sub TmpDw,4
cmp [TmpDw],0
je loc_3
mov [TmpDw1],0
loc_2:
add TmpDw1,4
jmp loc_0
loc_3:
sub TmpDw1,SaveIat
mov IatSize,TmpDw1
dm SaveIat,IatSize,IatFileBin

gpa "VirtualProtect", "KERNEL32.dll"
mov BpAddr,$RESULT
bp BpAddr
LoopVirtualProtect:
eob BpVirtualProtect
esto
BpVirtualProtect:
mov TmpDw,esp
add TmpDw,4
mov TmpDw,[TmpDw]
cmp TmpDw,CodeSecAddr
jne LoopVirtualProtect
bc BpAddr
mov TmpDw,ebp
add TmpDw,0FFFFB0CC
sub [TmpDw],1
mov TmpDw,[esp]
find TmpDw,#E9????FFFFA0????????8885#
cmp $RESULT,0
je Error
mov BpAddr,$RESULT
add BpAddr,5
bp BpAddr
eob BpFixIat
esto
BpFixIat:
bc BpAddr
lm RealIatAddr,IatSize,IatFileBin
alloc 1000
cmp $RESULT,0
je Error
mov FixIatCode,$RESULT
mov SaveEip,eip
mov eip,FixIatCode
mov TmpDw,FixIatCode
mov [TmpDw],#608BB5E4D7FFFF8BBD24D9FFFF833F00742E8B0705000040008B10BB#
add TmpDw,1C
mov [TmpDw],RealIatAddr
add TmpDw,4
mov [TmpDw],#81FA9090909074138B123B13740B83C30481FB#
add TmpDw,13
mov [TmpDw],RealIatAddr
add [TmpDw],IatSize
add TmpDw,4
mov [TmpDw],#7CF1891883C704EBCD6168#
add TmpDw,0B
mov BpAddr,SaveEip
add BpAddr,33
mov [TmpDw],BpAddr
add TmpDw,4
mov [TmpDw],#C3#
bp BpAddr
eob FixIatE
esto
FixIatE:
bc BpAddr
//
mov PushfdCnt,0
mov PopfdCnt,0
mov AddrS,SplicCodeAddr
mov TmpDw,[AddrS]
and TmpDw,0FF
cmp TmpDw,9C
jne SpLoc_loop
mov TmpDw,AddrS
jmp SpLoc_3
SpLoc_loop:
findop AddrS,#9C#
mov TmpDw,$RESULT
cmp TmpDw,0
je SplicEnd
SpLoc_3:
mov AddrMarkS,TmpDw
add PushfdCnt,1
findop AddrMarkS,#9D#
mov TmpDw,$RESULT
cmp TmpDw,0
je SplicEnd
mov AddrMarkE,TmpDw
add PopfdCnt,1
mov AddrS,AddrMarkS
SpLoc_0:
findop AddrS,#9C#
mov TmpDw,$RESULT
cmp TmpDw,0
je SplicEnd
mov AddrNextPushfd,TmpDw
cmp AddrNextPushfd,AddrMarkE
ja SpLoc_2
add PushfdCnt,1
mov AddrS,AddrNextPushfd
jmp SpLoc_0

SpLoc_1:
findop AddrMarkE,#9D#
mov TmpDw,$RESULT
cmp TmpDw,0
je SplicEnd
mov AddrMarkE,TmpDw
add PopfdCnt,1
SpLoc_2:
cmp PopfdCnt,PushfdCnt
jne SpLoc_1
mov Len,AddrMarkE
sub Len,AddrMarkS
Add Len,1
fill AddrMarkS,Len,90
mov AddrS,AddrMarkE
jmp SpLoc_loop
SplicEnd:

gpa "VirtualProtect", "KERNEL32.dll"
mov BpAddr,$RESULT
bphws BpAddr,"x"
LoopVirtualProtect1:
eob BpVirtualProtect1
esto
BpVirtualProtect1:
mov TmpDw,esp
add TmpDw,4
mov TmpDw,[TmpDw]
cmp TmpDw,CodeSecAddr
jne LoopVirtualProtect1
mov TmpDw,esp
add TmpDw,0C
cmp [TmpDw],100
jb LoopVirtualProtect1
mov [TmpDw],40
bphwc BpAddr
mov TmpDw,[esp]

find TmpDw,#6A00FF770C8B503833502C3350242BCAFFD1#
cmp $RESULT,0
je Error
mov BpAddr,$RESULT
add BpAddr,10
bphws BpAddr,"x"
LoopWaitOep:
eob BpOep
esto
BpOep:
cmp eip,BpAddr
jne LoopWaitOep
bphwc BpAddr
sti

eval "OEP reached.Use arminline to fix Code_Splicing:{SplicCodeAddr}"
msg $RESULT

ret

Error:
msg "error!"
ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
