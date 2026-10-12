
# sd protector 脱壳脚本

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-529.htm`](https://www.pediy.com/kssd/pediy09/pediy09-529.htm)

> 已经修改......  

```
//log  
//sd protector  
//code by skylly  
msg "忽略所有异常"  
var seccount  
var secaddr  
var oep  
var codebase  
var elfnew  
var ntheader  
var oepaddr  
var iidaddr  
var iidsizeaddr  
var iidstart  
var iidsize  
var neediat  
mov neediat,0     //这个标志位是调试 iat 用的  
var ccccadr  
var eipvar  
var issd  
var EP  
mov EP,eip  
var temp  
mov temp,eip  
mov temp,\[temp\]  
and temp,FF  
cmp temp,55  
je sd  
mov issd,0  
jmp api  
sd:  
mov issd,1  

api:  
gpa "VirtualProtectEx","kernel32.dll"  
cmp $RESULT,0  
je err  
find $RESULT,#C2??00#  
cmp $RESULT,0  
je err  
var VirtualProtect  
mov VirtualProtect,$RESULT  

gpa "GetSystemInfo","kernel32.dll"  
cmp $RESULT,0  
je err  
var GetSystemInfo  
mov GetSystemInfo,$RESULT  
find GetSystemInfo,#7C20#  
cmp $RESULT,0  
je err  
mov \[$RESULT\],#EB#  
find GetSystemInfo,#C2??00#  
cmp $RESULT,0  
je err  
mov GetSystemInfo,$RESULT  

gpa "GetVersion","kernel32.dll"  
cmp $RESULT,0  
je err  
find $RESULT,#C3#  
cmp $RESULT,0  
je err  
var GetVersion  
mov GetVersion,$RESULT  

gpa "CreateFileA","kernel32.dll"  
cmp $RESULT,0  
je err  
find $RESULT,#C2??00#  
cmp $RESULT,0  
je err  
var CreateFileA  
mov CreateFileA,$RESULT  

gpa "CloseHandle","kernel32.dll"  
cmp $RESULT,0  
je err  
find $RESULT,#C2??00#  
cmp $RESULT,0  
je err  
var CloseHandle  
mov CloseHandle,$RESULT  

allstart:  
bp GetVersion  
esto  
esto  
esto  
bc GetVersion  
mov eax,80000000  

bp CreateFileA  
esto  
cmp eip,CreateFileA  
jne err  

bc CreateFileA  
rtu  

bp CloseHandle  
esto  
bc CloseHandle  
cmp eip,CloseHandle  
jne err  
rtu  
rtr  

//查找转单线程地址  
var single  
find eip,#837C24??01#  
cmp $RESULT,0  
je start  
var single1  
mov single1,$RESULT  
add $RESULT,1  
find $RESULT,#837C24??01#  
cmp $RESULT,0  
je start  
mov single,$RESULT  
log single  
log single1  

start:  
//查找转单进程地址  
find EP,#0F84????0000E8010000#  
cmp $RESULT,0  
je err  
log $RESULT  
mov ccccadr,$RESULT  
bphws $RESULT,"x"  
bp GetVersion  
esto  
bphwc $RESULT  
bc GetVersion  
cmp eip,GetVersion  
jne singlepro  
//这里是单进程程序的特殊处理  
msg "这是单进程程序"  
rtu  
jmp sdproiat  

singlepro:  
//转单进程  
mov !ZF,1  
msg "转换为单进程"  

var iataddr  
mov ccccadr,eip  
//查找 iat 判断地址  
find eip,#8B8424????000085C00F#  
cmp $RESULT,0  
je err  
mov iataddr,$RESULT  
//softdefender  
bp GetSystemInfo  
bprm iataddr,1  
jmp sdproiat  

good:  
esto  
cmp eip,GetSystemInfo  
je sing  
cmp eip,iataddr  
je iatpro  
jmp good  

iatpro:  
bphwc iataddr  
bpmc  
var temp  
var espvar  
mov espvar,esp  
mov temp,eip  
add temp,3  
mov temp,\[temp\]  
add espvar,temp  
//iat 保护  
mov \[espvar\],0  
jmp sdproiat  
ret  

sing:  
//处理转换单线程  
sti  
cmp eip,70000000  
ja good  
msg "转单线程"  
log eip  
var addr  
mov addr,esp  
sub addr,4  
mov addr,\[addr\]        //让外壳认为不是单 CPU  
add addr,14  
mov \[addr\],0  
var count  
inc count  
cmp count,2  
je final  
cmp issd,1  
je sdproiat  
jmp good  
ret  

sdproiat:  
bc GetSystemInfo  
bp GetVersion  
//mov eipvar,eip  
//sub eipvar,1000  
find EP,#03F5#     //#03F589??24??E8#  
cmp $RESULT,0  
je err  
var iid  
mov iid,$RESULT  

mov iataddr,0  
bprm iid,1  
gogo:  
esto  
cmp eip,GetVersion  
je gogo  
cmp eip,GetSystemInfo  
je sing  
cmp eip,iataddr  
je iatpro  
cmp eip,iid  
jne gogo  

log iid  
bpmc iid  
msg "下面开始处理输入表"  
bpmc  
var crcaddr  
var iidend  
var procend  
var oldprcend  
//自校验 1  
find EP,#5F5E85C05B#  
cmp $RESULT,0  
je err  
mov crcaddr,$RESULT  
add crcaddr,5  
mov \[crcaddr\],#EB#  
log crcaddr  

//iid 结束  
find eip,#83??148B#  
cmp $RESULT,0  
je err  
find $RESULT,#E9????FFFF#  
cmp $RESULT,0  
je err  
mov iidend,$RESULT  
add iidend,5  
bp iidend  

//自校验函数  nop 掉 因为它会清理硬件断点  
find eip,#83C50483C604#  
cmp $RESULT,0  
je err  
mov procend,$RESULT  
add procend,B  
mov \[procend\],#90#  
log procend  
add procend,1  
mov oldprocend,\[procend\]  
mov \[procend\],#90909090#  
sub procend,1  

//是否加密 IAT 的标志  
var encflag1  
var encflag2  
var encflag1var  
var encflag2var  
find eip,#8D7B??????57E8????????8B??24#  
cmp $RESULT,0  
je err  
find $RESULT,#0F84#  
mov encflag1,$RESULT     
mov encflag1var,encflag1  
add encflag1var,2  
//保存旧值  
mov encflag1var,\[encflag1var\]  
find $RESULT,#85C0#  
cmp $RESULT,0  
je err  
find $RESULT,#0F84#   
cmp $RESULT,0  
je err  
mov encflag2,$RESULT   
mov encflag2var,encflag2  
add encflag2var,2  
//保存旧值  
mov encflag2var,\[encflag2var\]  

mov \[encflag1\],#909090909090#  
log encflag1  
mov \[encflag2\],#E9#  
log encflag2  
add encflag2,5  
mov \[encflag2\],#90#  
sub encflag2,4  
add encflag2var,1  
mov \[encflag2\],encflag2var  
sub encflag2var,1  
sub encflag2,1  

cmp neediat,0  
je comeon  
//填充序号输入表地址  
find eip,#A900000080#  
cmp $RESULT,0  
je err  
find $RESULT,#25FFFFFF7F50#  
cmp $RESULT,0  
je err  
find $RESULT, #8906E9#  
cmp $RESULT,0  
je err  
var number  
mov number,$RESULT  
mov \[number\],#9090#  
log number  
//填充输入表地址  
find eip,#EB??8B??24??57??E8#  
cmp $RESULT,0  
je err  
add $RESULT,1  

//第二处才是  
find $RESULT,#EB??8B??24??57??E8#  
cmp $RESULT,0  
je err  
find $RESULT,#8906#  
//mov     dword ptr \[esi\], eax  
cmp $RESULT,0  
je err  
var thunk  
mov thunk,$RESULT  
mov \[thunk\],#9090#  
var oldclear  
var clear  
var clearoff  
var clearok  
//清除输入表地址  
find eip,#66C7030000#  
//mov     word ptr \[ebx\], 0  
cmp $RESULT,0  
je err  
mov clear,$RESULT  
mov clearok,$RESULT  
sub clear,C  
mov oldclear,\[clear\]  
//清除输入表完毕地址  
find clear,#74??6A#  
cmp $RESULT,0  
je err  
var hoho  
mov hoho,$RESULT  
var oldhoho  
mov oldhoho,\[hoho\]  
mov \[hoho\],#EB#  
log hoho  
add clearok,5  

mov clearoff,clearok  
sub clearoff,clear  
sub clearoff,2  
mov \[clear\],#EB#  
log clear  
inc clear  
mov \[clear\],clearoff  
dec clear  

var clear2  
var oldclear2  
var clearok2  
var clear2off  
log clearok  
find clearok,#68FF000000#  
cmp $RESULT,0  
je err  
mov clear2,$RESULT  
//清除输入表地址 2  

dec clear2  
mov oldclear2,\[clear2\]  
//清除输入表完毕地址 2  
find clear2,#83????8B#  
cmp $RESULT,0  
je err  
mov clearok2,$RESULT  
mov clear2off,clearok2  
sub clear2off,clear2  
sub clear2off,2  
mov \[clear2\],#EB#  
inc clear2  
log clear2  
mov \[clear2\],clear2off  
dec clear2  

mov codebase,ebp  
mov iidstart,esi  
cmp issd,1  
je comeon  
//soft defender 的起始地址存在 ebx 上  
mov codebase,ebx  
log iidend  
msg "执行输入表修复过程"  

comeon:  
esto  
cmp eip,iidend  
jne comeon  
msg "输入表修复完毕"  
jmp finishiat  

finishiat:  
cmp neediat,0  
je finishiat1  

mov iidsize,esi  
cmp iidsize,0  
```



```
je bugan
cmp issd,1
je tianchong
mov iidsize,edi
jmp tianchong

bugan:
mov iidsize,ebx

tianchong:
sub iidsize,codebase
//减去基址
sub iidsize,iidstart
//减去起始地址
add iidsize,14
//添加一个空白 iid

//计算 elfnew
mov elfnew,codebase
add elfnew,3C
mov elfnew,[elfnew]
//计算 ntheader
mov ntheader,codebase
add ntheader,elfnew
//计算 oep 地址
mov oepaddr,ntheader
add oepaddr,28 //16 进制
//计算区段数地址
mov secaddr,ntheader
add secaddr,6
mov seccount,[secaddr]
and seccount,FFFF
dec seccount
mov [secaddr],seccount

//计算 iid 地址
mov iidaddr,ntheader
add iidaddr,80 //16 进制
mov [iidaddr],iidstart
//计算 iid 大小
mov iidsizeaddr,ntheader
add iidsizeaddr,84 //16 进制
mov [iidsizeaddr],iidsize

//这是输入表修复时修改的地方
mov [number],#8906#
mov [thunk],#8906#
mov [clear],oldclear
mov [clear2],oldclear2
mov [hoho],oldhoho

finishiat1:
//IAT 处理完毕
//还原修改的地方
bc iidend
mov [crcaddr],#74#
mov [procend],#E8#
add procend,1
mov [procend],oldprocend

mov [encflag1],#0F84#
add encflag1,2
mov [encflag1],encflag1var

mov [encflag2],#0F84#
add encflag2,2
mov [encflag2],encflag2var

bphws VirtualProtect,"x"

final:
//找 OEP 了
bc GetSystemInfo
bp GetVersion
bpwm ccccadr,1

esto
cmp eip,GetVersion
je final
cmp eip,VirtualProtect
jne ending
bphwc VirtualProtect

rtu
find eip,#83C4??C3#
cmp $RESULT,0
je err
sub $RESULT,4
//绕开壳的 ANTI-DUMP
mov eip,$RESULT

//绕开壳偷 OEP 代码
find EP, #8854241C#
cmp $RESULT,0
je final
bphws $RESULT,"x"
esto
bphwc $RESULT
cmp eip,$RESULT
jne err
mov oep,esi
sub oep,codebase
find eip,#755A#
cmp $RESULT,0
je err
go $RESULT
mov !ZF,0
find eip,#7415#
cmp $RESULT,0
je err
go $RESULT
mov !ZF,0
find eip,#0F85??000000#
cmp $RESULT,0
je err
go $RESULT
mov !ZF,0
jmp final

ending:
//这个 GetVersion 用于奇怪的延时
bphwc VirtualProtect
bc GetVersion
bpmc
sti
sto

var nowcode
lop:
sti
mov nowcode,eip
mov nowcode,[nowcode]
and nowcode,FF
cmp nowcode,C3 //retn
//自动走 retn
je lop
cmp nowcode,9C //pushfd sdprotector 会是这个
jne allend
sti
var espvar
mov espvar,esp
bphws espvar,"r"
esto
esto
bphwc espvar
jmp lop

allend:
mov oep,eip
sub oep,codebase
log iidstart
log iidsize
log oep
msg "dump(去掉粘贴 PE 头选项)"
mov [oepaddr],oep
ret

err:
msg "err"
ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
