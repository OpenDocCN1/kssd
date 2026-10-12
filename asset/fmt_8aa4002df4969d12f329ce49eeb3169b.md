
# VProtect 1.XX Pass HWID And OEP Finder

> 原文：[`www.pediy.com/kssd/pediy12/129993.html`](https://www.pediy.com/kssd/pediy12/129993.html)

为了庆祝苦力 VP2.0 的诞生，就把这个发出来大家一块娱乐吧

支持的版本我测试的 `1.XX-1.93` 的 `EXE` 文件均通过。是否支持 `DLL` 没有测试，大家娱乐而已，不要当真！

代码:

```
///////////////////////////////////////////////////////////////////////
//VProtect 1.XX Pass HWID And OEP Finder By wuqing1501
//1.支持版本 VProtect1.xx-1.93
//2.可能不支持 DLL 文件，如果是 DLL 的话要修改 codebase 和 codeend 两个地址
//  因为 DLL 申请的空间可能小于基址，所以这两个值可能要互换一下
//3.脚本运行过程中可能会卡，请耐心等待即可!
//4.可以简单查找 OEP，针对部分程序或者选项无效
//  CODE BY wuqing1501(笨笨鼠)
///////////////////////////////////////////////////////////////////////
var VirtualAlloc
var bypassaddr
var deax
var deax1
var deax2   
var deip
var n
var decx
var debx
var codebase
var codesize
var codeaddr
var newcode
var oldcode
var m
var temeax
var version
var VirtualProtect
var bpaddr
//////////////////////////////////////////////////////////
//配置区
//////////////////////////////////////////////////////////
mov newcode,"12345678901234567890123456789012"   //此处为正确的机器码
mov codebase,00401000
mov codeend,01370000  /////// 让程序直接运行，属于程序的最大的内存地址不算 dll 地址哦
///////////////////////////////////////////////////////////
mov version,0
bc
bpmc
bphwc
mov n,0
mov m,20
GMI codebase, CODESIZE
mov codesize,$RESULT
gpa "VirtualProtect", "kernel32.dll"
cmp $RESULT,0
je erro
mov VirtualProtect, $RESULT
add VirtualProtect,13
gpa "VirtualAlloc", "kernel32.dll"
cmp $RESULT,0
je erro
mov VirtualAlloc, $RESULT
add VirtualAlloc,19
bp VirtualAlloc
run
mov deax,eax
mov deax1,eax
run
run
run
run
run
run
mov deax2,eax

findlvbypassaddr:
cmp deax,codeend
ja findhv
find deax,#8B4DF483C101894DF48B55F43B550C73638B45080345F40FB60881E1F0000000884DFF8A55FFC0FA048855FF0FBE45FF83E00F8845FF0FB64DFF51#
cmp $RESULT,0
jnz next1
add deax,1000
jmp findlvbypassaddr

next1:
mov bypassaddr,$RESULT
mov deax,$RESULT
add deax,1
add bypassaddr,06e
bp bypassaddr
add version,1
jmp findlvbypassaddr

findhv:
mov deax,deax1
cmp version,0
jnz finded

findbypassaddr:
cmp deax,codeend
ja finded
find deax,#33C08B47448B1883C0048?088?0BC7474C05000000#
cmp $RESULT,0
jnz next2
add deax,1000
jmp findbypassaddr

next2:
mov bypassaddr,$RESULT
mov deax,$RESULT
add deax,1
add bypassaddr,0C
bp bypassaddr
jmp findbypassaddr

finded:
bc VirtualAlloc
run
find eip,#88445101#,4
mov bpaddr,eip
cmp $RESULT,0
jnz VPLV
find eip,#880B#,2
mov bpaddr,eip
cmp $RESULT,0
jnz VPHV


erro:
msg "Something Wrong!"
ret

VPLV://VP 低版本


VPLV\_addr:
mov n,0
mov bypassaddr,ecx
cmp ecx,codebase
jb VPLV\_next


VPLV\_pathaddr:
cmp edx,0f
jnz VPLV\_next
sti
alloc 1000
mov codeaddr,$RESULT
mov \[codeaddr\],##+newcode
mov temeax,eax
mov m,0

path:
mov eax,\[codeaddr\]
mov \[bypassaddr\],al
add bypassaddr,1
add codeaddr,1
add m,1
cmp m,20
jnz path
mov eax,temeax
free codeaddr
bc bpaddr
msgyn "HWID PATH SUCCESSED!GOTO OEP?"
cmp $RESULT,1
je findoep

VPLV\_next:
run
jmp VPLV\_addr


VPHV://VP 高版本
mov n,0

VPHV\_addr:
cmp ebx,codebase
jb VPHV\_next
mov bypassaddr,ebx
add n,1
VPHV\_pathaddr:
cmp n,20
jnz VPHV\_next

sti
alloc 1000
mov codeaddr,$RESULT
mov \[codeaddr\],##+newcode
mov temeax,eax
mov m,0
sub bypassaddr,1f

HVpath:
mov eax,\[codeaddr\]
mov \[bypassaddr\],al
add bypassaddr,1
add codeaddr,1
add m,1
cmp m,20
jnz HVpath
mov eax,temeax
free codeaddr
bc bpaddr
msgyn "HWID PATH SUCCESSED!GOTO OEP?"
cmp $RESULT,1
je findoep

VPHV\_next:
run
jmp VPHV\_addr


findoep:
bp VirtualAlloc
bp VirtualProtect
loop:
run
cmp eip,VirtualProtect
jnz loop
bc VirtualAlloc
bc VirtualProtect
bprm codebase,codesize
run
bpmc
log eip,"OEP Or Near OEP:"
CMT eip,"OEP Or Near OEP Finded By wuqing1501"
msg "OEP Or Near OEP Finded!"
ret


exit:
bc
run
ret

```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `VProtect 1.XX Pass HWID And OEP Finder By wuqing1501.rar`  

 |
