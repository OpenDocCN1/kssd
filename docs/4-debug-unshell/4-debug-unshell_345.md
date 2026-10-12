
# asprotect 旧版脱壳脚本

> 原文：[`www.pediy.com/kssd/pediy10/62512.html`](https://www.pediy.com/kssd/pediy10/62512.html)

仅支持 `1.0`, `1.1` (没见过 `1.1` 的，姑且把 `chap708` 当作 `1.1` 吧), `1.2x` 的，不支持 `1.30` 及以上的版本

```
//for asprotect 1.0  

//code by skylly  

var cbase  

var csize 

#log

gmi eip, CODEBASE 

mov cbase, $RESULT 

log cbase 

gmi eip, CODESIZE 

mov csize, $RESULT 

log csize 

var GetCommandLineARes

var GetVersionRes

var espvar

mov espvar,esp

sub espvar,4

gpa "GetCommandLineA","kernel32.dll"

cmp $RESULT,0

je err

var GetCommandLineA

mov GetCommandLineA,$RESULT

gpa "GetVersion","kernel32.dll"

cmp $RESULT,0

je err

var GetVersion

mov GetVersion,$RESULT

mov [espvar],eip

mov esp,espvar

bp eip

mov eip,GetVersion

esto

bc eip

mov GetVersionRes,eax

mov [espvar],eip

mov esp,espvar

bp eip

mov eip,GetCommandLineA

esto

bc eip

mov GetCommandLineARes,eax

var hookedgpa

#log

gpa "LocalAlloc","kernel32.dll"

cmp $RESULT,0

je err

var LocalAlloc

mov LocalAlloc,$RESULT

find LocalAlloc,#7407C745#

cmp $RESULT,0

je err

mov [$RESULT],#9090#    //anti chap008 的 heapmagic trick

gpa "LoadLibraryA","kernel32.dll"

cmp $RESULT,0

je err

var LoadLibraryA

mov LoadLibraryA,$RESULT

gpa "VirtualFree","kernel32.dll"

cmp $RESULT,0

je err

var VirtualFree

mov VirtualFree,$RESULT

gpa "GetProcAddress","kernel32.dll"

cmp $RESULT,0

je err

var GetProcAddressorg

mov GetProcAddressorg,$RESULT

find GetProcAddressorg,#C2??00#

cmp $RESULT,0

je err

mov GetProcAddress,$RESULT

gpa "GetModuleHandleA","kernel32.dll"

cmp $RESULT,0

je err

var GetModuleHandleA

mov GetModuleHandleA,$RESULT

go VirtualFree

go GetProcAddress

rtu

var temp

mov temp,[eip]

and temp,FFFF

cmp temp,C085

je again

rtr

again:

find eip,#617508#

cmp $RESULT,0

je err

flying:

var ending

mov ending,$RESULT

play:

go ending

cmp eip,ending

jne play

sti

sti

sti

sti

var dllbase

mov dllbase,edi

log dllbase

ver10:

find dllbase,#E09E760FBA#    //aspack1.08 主程序的过期 nag

cmp $RESULT,0

je ver11

add $RESULT,2

mov [$RESULT],#EB#

jmp startnow

ver11:

find dllbase,#ABEBDC61837D0800750933C0#   //aspr 1.1 iat store addr

cmp $RESULT,0

je mayver12

log "ver11"

/* crc 代码

001B876A        8B15 04FB1B00       mov edx,dword ptr ds:[1BFB04]  //code start

001B8770        A1 00FB1B00         mov eax,dword ptr ds:[1BFB00]  //code size

001B8775        E8 0AE0FFFF         call 001B6784                  //crc

001B877A        A3 08FB1B00         mov dword ptr ds:[1BFB08],eax  //保存

001B877F        A1 08FB1B00         mov eax,dword ptr ds:[1BFB08]  //读取

001B8784        3B45 08             cmp eax,dword ptr ss:[ebp+8]   //与真值比较

001B8787        74 07              je short 001B8790

001B8789        66:B8 0F00         mov ax,0F                      

001B878D        FF55 18           call near dword ptr ss:[ebp+18] //完蛋

*/

find dllbase,#A3????????A1????????3B4508#   //1.1 版本 code 段 crc  一部分拿来比较 一部分拿来计算 oep...

cmp $RESULT,0

je err

mov [$RESULT],#8B45089090A3#    //完美 patch...

var crcaddr

mov crcaddr,$RESULT

find dllbase,#50E8F8FEFFFF#

cmp $RESULT,0

je err

mov [$RESULT],#909090909090#  //user32 kernel32 gdi32 中函数

find dllbase,#558BEC81C4F8FEFFFF#

cmp $RESULT,0

je err

var fixusecall

mov fixusecall,$RESULT

find fixusecall,#894504#

cmp $RESULT,0

je err

mov [$RESULT],#909090#  //不让填充返回地址

find dllbase,#8BC3EB55#

cmp $RESULT,0

je err

sub $RESULT,5

bp $RESULT

esto

bc $RESULT

var patchadr

mov patchadr,$RESULT

var tmpcall

mov tmpcall,$RESULT

inc tmpcall

mov tmpcall,[tmpcall]

add tmpcall,$RESULT

add tmpcall,5

alloc 1000 

cmp $RESULT,0

je err

var virmem

mov virmem,$RESULT

eval "call {virmem}"

asm patchadr,$RESULT

add patchadr,5

mov [patchadr],#9090#

eval "call {tmpcall}"

asm virmem,$RESULT

add virmem,5

mov [virmem],#438B1B53#

add virmem,4

eval "call {fixusecall}"

asm virmem,$RESULT

add virmem,5

mov [virmem],#C3#

bp crcaddr

esto

bc crcaddr

jmp jmpoep

ret

mayver12:

find dllbase,#E8????????E8????????8B??8902#     //aspr 1.2 iat patch

cmp $RESULT,0

je ver13

log "ver12"

add $RESULT,5

var iataddr

mov iataddr,$RESULT

var tmp

mov tmp,iataddr

inc tmp

mov tmp,[tmp]

mov [iataddr],#9090909090#         //patch iat patch 了就只有几个被模拟的 api 没修复了

find dllbase,#8B178902830704#

cmp $RESULT,0

je err

var tmpadr

var tmpapi

sub $RESULT,4

mov tmpadr,$RESULT

mov tmpapi,[tmpadr]

eval "addr:{tmpapi} fix as:GetProcAddress"

log $RESULT

mov [tmpadr],GetProcAddressorg

noget:

//查找 iat 处理完毕

find iataddr,#EB??61E8#

cmp $RESULT,0

je err

add $RESULT,2

bp $RESULT

esto

bc $RESULT

//还原代码

mov [iataddr],#E8#       

inc iataddr

mov [iataddr],tmp

gpa "DialogBoxParamA","user32.dll"

cmp $RESULT,0

je err

var DialogBoxParamA

mov DialogBoxParamA,$RESULT

gpa "FreeResource","kernel32.dll"

cmp $RESULT,0

je err

var FreeResource

mov FreeResource,$RESULT

gpa "GetVersionExA","kernel32.dll"

cmp $RESULT,0

je err

var GetVersionExA

mov GetVersionExA,$RESULT

gpa "GetCurrentProcess","kernel32.dll"

cmp $RESULT,0

je err

var GetCurrentProcess

mov GetCurrentProcess,$RESULT

gpa "GetCurrentProcessId","kernel32.dll"

cmp $RESULT,0

je err

var GetCurrentProcessId

mov GetCurrentProcessId,$RESULT

gpa "GetModuleHandleA","kernel32.dll"

cmp $RESULT,0

je err

var GetModuleHandleA

mov GetModuleHandleA,$RESULT

gpa "SetHandleCount","kernel32.dll"

cmp $RESULT,0

je err

var SetHandleCount

mov SetHandleCount,$RESULT

find eip,#6068????????8D45F4#    //kernel32 中预处理函数

cmp $RESULT,0

je err

add $RESULT,6

bp $RESULT

esto

bc $RESULT

var tmp

mov tmp,[esp]

eval "特殊 api 起始地址:{tmp}"

log $RESULT

msgyn "是否修复特殊 api?"

cmp $RESULT,0

je endfixapi

loopfixapi:

var tmpaddr

mov tmpaddr,tmp

var tmpapi

mov tmpapi,[tmpaddr]

cmp tmpapi,0

je endfixapi

var tmpproc

mov tmpproc,tmpapi

add tmpproc,6

mov tmpproc,[tmpproc]

and tmpproc,FFFFFF

cmp tmpproc,8B0845

je GetVersionExA

cmp tmpproc,458B08

je DialogBoxParamA

cmp tmpproc,75C085

je GetModuleHandleA

cmp tmpproc,C3C358

je mayGetVersion

cmp tmpproc,35FFFF

jne nene

//下面是 GetVersion 的几种情况

mov tmpproc,tmpapi

add tmpproc,E

mov tmpproc,[tmpproc]

and tmpproc,FFFF

cmp tmpproc,58B

je GetCommandLineA

mayGetVersion:

mov tmpproc,tmpapi

add tmpproc,2

mov tmpproc,[tmpproc]

mov tmpproc,[tmpproc]

cmp tmpproc,GetVersionRes

je GetVersion

mov tmpproc,tmpapi

add tmpproc,9

mov tmpproc,[tmpproc]

mov tmpproc,[tmpproc]

cmp tmpproc,GetVersionRes

je GetVersion

jne GetCommandLineA

ret

nene:

mov tmpproc,tmpapi

mov tmpproc,[tmpproc]

and tmpproc,FFFFFFFF

cmp tmpproc,5DEC8B55

je FreeResource

//log tmpproc

and tmpproc,FF

cmp tmpproc,A1

je mayGetCurrentProcess

mov tmpproc,tmpapi

add tmpproc,5

mov tmpproc,[tmpproc]

and tmpproc,FF

cmp tmpproc,A1

je GetCurrentProcessId

mov tmpproc,tmpapi

add tmpproc,D

mov tmpproc,[tmpproc]

and tmpproc,FFFF

cmp tmpproc,4

je SetHandleCount

mov tmpproc,tmpapi

add tmpproc,E

mov tmpproc,[tmpproc]

and tmpproc,FFFF

cmp tmpproc,4

je SetHandleCount

mov tmpproc,tmpapi

add tmpproc,1

mov tmpproc,[tmpproc]

cmp tmpproc,58BEC8B

jne mayGetVersion2

mov tmpproc,tmpapi

add tmpproc,5

mov tmpproc,[tmpproc]

mov tmpproc,[tmpproc]

cmp tmpproc,GetCommandLineARes

je GetCommandLineA

log tmpapi  
```



`jmp nextfixapi`

`mayGetVersion2:`

```
mov tmpproc,tmpapi
mov tmpproc,\[tmpproc\]
cmp tmpproc,E8EC8B55
jne nextfixapi
mov tmpproc,tmpapi
add tmpproc,4
mov tmpproc,\[tmpproc\]
add tmpproc,tmpapi
add tmpproc,8
add tmpproc,2
mov tmpproc,\[tmpproc\]
mov tmpproc,\[tmpproc\]
cmp tmpproc,GetVersion
je FreeResource
jne nextfixapi
```

`mayGetCurrentProcess:`

```
mov tmpproc,tmpapi
inc tmpproc
mov tmpproc,\[tmpproc\]
mov tmpproc,\[tmpproc\]
cmp tmpproc,0
je nextfixapi
cmp tmpproc,FFFFFFFF
je GetCurrentProcess
jmp GetCurrentProcessId
ret
```

`DialogBoxParamA:`

```
eval "addr:{tmpapi} fix as:DialogBoxParamA"
log $RESULT
mov \[tmp\],DialogBoxParamA
jmp nextfixapi
```

`FreeResource:`

```
eval "addr:{tmpapi} fix as:FreeResource"
log $RESULT
mov \[tmp\],FreeResource
jmp nextfixapi
```

`GetCommandLineA:`

```
eval "addr:{tmpapi} fix as:GetCommandLineA"
log $RESULT
mov \[tmp\],GetCommandLineA
jmp nextfixapi
```

`GetVersion:`

```
eval "addr:{tmpapi} fix as:GetVersion"
log $RESULT
mov \[tmp\],GetVersion
jmp nextfixapi
```

`GetVersionExA:`

```
eval "addr:{tmpapi} fix as:GetVersionExA"
log $RESULT
mov \[tmp\],GetVersionExA
jmp nextfixapi
```

`GetModuleHandleA:`

```
eval "addr:{tmpapi} fix as:GetModuleHandleA"
log $RESULT
mov \[tmp\],GetModuleHandleA
jmp nextfixapi
```

`GetCurrentProcess:`

```
eval "addr:{tmpapi} fix as:GetCurrentProcess"
log $RESULT
mov \[tmp\],GetCurrentProcess
jmp nextfixapi
```

`GetCurrentProcessId:`

```
eval "addr:{tmpapi} fix as:GetCurrentProcessId"
log $RESULT
mov \[tmp\],GetCurrentProcessId
jmp nextfixapi
```

`SetHandleCount:`

```
eval "addr:{tmpapi} fix as:SetHandleCount" //LockResource
log $RESULT
mov \[tmp\],SetHandleCount
jmp nextfixapi
```

`nextfixapi:`

```
add tmp,4
jmp loopfixapi
```

`endfixapi:`

`haha:`

```
find eip,#837804007417# //pre-dip
cmp $RESULT,0
je nopredip
cmt $RESULT,"pre-dip"
add $RESULT,4
bp $RESULT
esto
bc $RESULT
var tmp
mov tmp,!ZF
cmp tmp,0
jmp nopredip
```

`patchdip:`

```
mov tmp,eip
add tmp,19
bp tmp
esto
bc tmp
//patch user name //todo 有问题...
var tmp
mov tmp,eax
add tmp,2FFC
mov \[tmp\],#10000000437261636B6420427920736B796C6C7900#
add tmp,4
mov \[eax\],tmp
```

`nopredip:`

```
find eip,#8378100074??A1# //0x1500 以内
cmp $RESULT,0
je notimelimit
cmt $RESULT,"是否有时间限制"
bp $RESULT
esto
bc $RESULT
find eip,#80783000750B#
cmp $RESULT,0
je err
cmt $RESULT,"是否时间过期,跳则过期"
add $RESULT,F
cmt $RESULT,"必须跳"
bp $RESULT
esto
bc $RESULT
```

`notimelimit:`

```
find dllbase,#59595DC21000# //crc 自校验
cmp $RESULT,0
je err
bp $RESULT
esto
esto
bc $RESULT
//此时自校验结束
//其实下面这个查找可以忽略(下面不远我们可以看到壳作者名字)
//00EAFF88 B2 01 mov dl,1
//00EAFF8A B8 20CCEA00 mov eax,0EACC20 ; ASCII 0E,"TBlockOperator"
//00EAFF8F E8 9CCCFFFF call 00EACC30
find eip,#B201??????????E8????????8945E8#
cmp $RESULT,0
je err
bp $RESULT
esto
bc $RESULT
find eip,#FF30FF75F0FF75ECC3#
cmp $RESULT,0
je jmp212
add $RESULT,8
bp $RESULT
esto
bc $RESULT
sti
```

`sethwd:`

```
bphws espvar,"r"
esto
bphwc espvar
sti
var tmp
mov tmp,\[eip\]
and tmp,FFFF
cmp tmp,C350
je twosti
cmp tmp,E0FF
je onesti
cmt eip,"fake oep"
msg "fake oep"
jmp goingoep
ret
```

`jmp212: //aspack2.12 主程序的 jmp`

```
find eip,#FF30FF75F0FF65EC#
cmp $RESULT,0
je jmp212
add $RESULT,5
bp $RESULT
esto
bc $RESULT
sti
jmp sethwd
```

`twosti:`

```
sti
```

`onesti:`

```
sti
jmp final12
ret
```

`goingoep:`

```
bprm cbase,csize
esto
bpmc
```

`final12:`

```
cmt eip,"oep"
ret
```

`ver13:`

```
find dllbase,#89060FB7442404# //aspr 1.3x 跳转表加密
cmp $RESULT,0
je startnow
log "ver13"
find dllbase,#8B54240C89028B44240C89060FB7442404# //iat 加密
cmp $RESULT,0
je err
add $RESULT,5
mov \[$RESULT\],#2A#
find dllbase,#59595DC21000# //crc 自校验
cmp $RESULT,0
je err
bp $RESULT
esto
esto
bc $RESULT
//此时自校验结束
find eip,#C700DF??????B201??????????E8????????8B??A1????????8B0089????A1#
cmp $RESULT,0
je err
add $RESULT,6
bp $RESULT
esto
bc $RESULT
find eip,#8B65F8FF35????????C3#
cmp $RESULT,0
je err
sub $RESULT,2
bp $RESULT
esto
bc $RESULT
sti
cmp ecx,0
je someother
rtr
sti
cmt eip,"fake oep"
ret
```

`some131:`

```
find eip,#0F85????0000#
cmp $RESULT,0
je err
add $RESULT,6
bp $RESULT
esto
bc $RESULT
jmp ffe0
```

`someother:`

```
rtr
sti
find eip,#0F85????FFFF#
cmp $RESULT,0
je some131
add $RESULT,6
bp $RESULT
esto
bc $RESULT
```

`ffe0:`

```
find eip,#FFE0#
cmp $RESULT,0
je err
bp $RESULT
esto
bc $RESULT
sti
cmt eip,"OEP"
ret
bprm cbase,csize
esto
bpmc
find ebp,#33C08945??FFE5#
cmp $RESULT,0
je err
bp $RESULT
esto
bc $RESULT
bp ebp
esto
bc ebp
sti
sti
cmt eip,"oep"
ret
```

`startnow: //主要为 1.0 服务`

```
log "ver10"
//第一层 aspack 结束
bp GetProcAddress
lop:
esto
cmp eip,GetProcAddress
jne lop
bc GetProcAddress
rtu
find eip,#85C07508B8#
cmp $RESULT,0
je special
var fix1
mov fix1,$RESULT
add fix1,2
find fix1,#741A#
cmp $RESULT,0
je err
var fix2
mov fix2,$RESULT
mov \[fix2\],#9090# //patch kernel32.dll
find fix1,#753A#
cmp $RESULT,0
je err
var fix3
mov fix3,$RESULT
mov \[fix3\],#EB# //patch user32.dll
msgyn "是否修复 sdk?"
cmp $RESULT,0
je no10sdk
add fix1,2
bp fix1
esto
bc fix1
find eip,#891083C604#
cmp $RESULT,0
je err
bp $RESULT
esto
bc $RESULT
mov hookedgpa,eax //记录 hook 后的 getprocaddress 地址
sub fix1,2
jmp imp10ok
```

`no10sdk:`

```
mov \[fix1\],#EB# //patch GetProcAddress
```

`imp10ok:`

```
find fix1,#33C05A59#
cmp $RESULT,0
je err
bp $RESULT
esto
bc $RESULT //输入表完毕
```

`fixok: //还原代码`

```
mov \[fix1\],#75#
mov \[fix2\],#741A#
mov \[fix3\],#75#
```

`//走 oep`

`jmpoep:`

```
find dllbase,#6150C3#
cmp $RESULT,0
jne final10
find dllbase,#61EB01#
cmp $RESULT,0
je err
```

`final10:`

```
var jmpingoep
mov jmpingoep,$RESULT
bp jmpingoep
lpfinal:
esto
cmp eip,jmpingoep
jne lpfinal
bc jmpingoep
```

`lpa: //自动走路`

```
sti
var temp
mov temp,\[eip\]
and temp,FF
cmp temp,C3
jne lpa
sti
```

`atoep:`

```
cmt eip,"OEP"
cmp hookedgpa,0
je allend
msgyn "是否有 sdk?"
cmp $RESULT,0
jne fixesdk
mov \[hookedgpa\],GetProcAddressorg
```

`allend:`

```
ret
```

`fixsdk:`

```
var newgpa
mov newgpa,\[hookedgpa\]
add newgpa,18
bp newgpa
lo:
esto
cmp eip,newgpa
jne end
```

`fixstart:`

```
var addr
mov addr,\[esp+4\]
sub addr,5
var num
mov num,\[esp+c\]
bp eax
esto
bc eax
mov addr,\[esp\]
```

`reg1:`

```
cmp num,8
jne reg2
eval "addr:{addr}:call isreged"
ret
jmp nextapi
```

`reg2:`

```
cmp num,5
jne notregapi
eval "addr:{addr}:call getregname"
jmp nextapi
```

`notregapi:`

```
eval "addr:{addr}:call {num}"
jmp nextapi
```

`nextapi:`

```
log $RESULT
jmp lo
```

`end:`

```
bc GetProcAddressorg
ret
```

`special:`

```
log "新版本,脚本不支持"
ret
```

`err:`

```
msg "error"
ret
```
