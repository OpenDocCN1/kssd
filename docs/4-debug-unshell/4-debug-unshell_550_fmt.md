
# 发一个简单修复 aspr2.1 iat 的脚本

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-604.htm`](https://www.pediy.com/kssd/pediy08/pediy8-604.htm)

```asm
//还原 ASProtect 2.1x SKE 代码段 call 011B0000 的 api  

//这个版本的 aspr 异常特点是 2 个 int3 异常  

//ollydbg,ollyscript,patch 结合的例子  

//program by jskew 2006-9-15 19:12  

#log  

var GetModuleFileNameA  
var patch\_addr  
var patch\_end  
var tmp  
var tmp2  
var oep\_addr  
var saveapi  
var cbase  
var csize  
var getapi  

gpa "GetModuleFileNameA", "kernel32.dll"  

mov GetModuleFileNameA,$RESULT  

GMI eip, CODEBASE  

mov cbase,$RESULT  

GMI eip, CODESIZE  

mov csize,$RESULT  

bp GetModuleFileNameA //壳 delphi 程序入口  

esto  

bc GetModuleFileNameA  

rtu  

find eip,#8B551429D08945FC# //得到 api  

mov getapi,$RESULT  

find eip,#31C031DB648F0383C404# //int3 异常后出来的地址  

bp $RESULT  

esto  

esto  

bc $RESULT  

bprm cbase,csize  

esto  

bpmc  

mov oep\_addr,eip  

exec  

push 40h  

push 1000h  

push 1000h  

push 0  

call VirtualAlloc  

ende  

mov patch\_addr,eax  

mov tmp,eax  

/*  

\_codebegin  equ 0h  

\_codeend  equ 4h  

\_iatbegin  equ 8h  

\_iatend    equ 0ch  

\_esp    equ 10h //保存 esp 的地址，建议用\_codeend 的地址  

\_api    equ 14h //保存 api 的地址，建议用\_codeend+4 的地址  

\_magiccall  equ 18h  

*/  

mov [tmp],cbase  

add tmp,4  

mov tmp2,cbase  

add tmp2,csize  

sub tmp2,10  

mov [tmp],tmp2  

add tmp,4  

mov [tmp],0047E000  

add tmp,4  

mov [tmp],0047E5C0  

add tmp,4  

mov [tmp],tmp2  

add tmp,4  

add tmp2,4  

mov [tmp],tmp2  

add tmp,4  

mov [tmp],010C0000  

add tmp,4  

mov [tmp],0  

add tmp,4  

add patch\_addr,50  

mov [patch\_addr],#60E8000000005F81E700FFFFFF8B57108997910000008997B60000008B57148997AF0000008997BE0000008B17803AE8755E8B5A0103DA83C3053B5F18755160B8246987248920FFE290909064FF35000000006AFF648925000000005B5BBB246987248903B8246987248B2061B8246987248B188B47083918740A83C0043B470C72F4EB1166C702FF1589420283C205423B5704729761909090EBFB#  

mov eip,patch\_addr  

find eip,#90909064FF35# //patch 保存 api  

mov saveapi,$RESULT  

find eip,#9090EBFB# //patch 结束  

mov patch\_end,$RESULT  

bp patch\_end  

bphws getapi,"x"  

leb1:  

esto  

cmp eip,getapi  

jne over  

mov eip,saveapi  

jmp leb1  

over:  

bphwc getapi  

bc patch\_end  

mov eip,oep\_addr  

ret  

\------------------------------------------------------  

修改了下代码，省去手动输入 4 个 patch 参数  

测试其他程序要设置 3 个参数  

\_iatbegin  equ 8h  

\_iatend  equ 0ch  

\_magiccall  equ 18h  

测试软件，如果要测试，可能还要修改\_magiccall 的参数  

iat 用例子的就可以  

http://www.xlightftpd.com/download/setup.exe 

 4dd466d0e1e0cc123afbe504da2462ca
```
