
# 脱 hying 壳的脚本，测试版

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-606.htm`](https://www.pediy.com/kssd/pediy07/pediy7-606.htm)

```
///////////////////////////////////////////////////////////  
//  
//  走到 oep 处，只测试在传神外挂上的 hying 壳版本  
//  选忽略所有异常，执行脚本即可，支持 hotcs.exe，main.dat 2 个文件  
//  感谢 forgot 的指导，呵呵  
//  2005-7-30 17:26 by jskew  
//  
//  
#log 

var SetThrPri  
var GetProHeap  
var mbase  
var msize  

gpa "SetThreadPriority","Kernel32.dll"  
find $RESULT,#C20800#  
mov  SetThrPri, $RESULT  
bp SetThrPri  
GMI eip, MODULEBASE  
mov mbase,$RESULT  
GMI eip, MODULESIZE  
mov msize,$RESULT  
esto //从 ep 开始，脚本开始  
rtu  
find eip, #2440#  
mov GetProHeap,$RESULT  
mov \[GetProHeap\], #2400# //修改 GetProcessHeap  
find eip, #6A006A006A11# //躲避 ZwSetInformationThread 破坏  
bp $RESULT  
esto  
mov \[GetProHeap\], #2440# //恢复 GetProcessHeap  
bc $RESULT  
sti  
sti  
sti  
sti  
sti  
mov eip,eax  
add esp,10  
find eip,#6A04680010000068002000006A00# //处理 iat 代码解码结束  
bp $RESULT  
esto  
bc $RESULT  
find eip,#0BC00F84610100000F31# //躲避 ZwQueryInformationProcess 检查  
bp $RESULT  
esto  
bc $RESULT  
mov eax,0  
esto  
bc SetThrPri  
rtu  
bprm mbase,msize  
esto  
bpmc  
ret
```

再修改，支持`hotcs.exe`文件，没`2k3`的机器测试，不知道怎么改了

其实我的`2k sp4`机器上用不了上面的代码，因为我不能停在`ep`处

这个脚本是对应可以停在`ep`的机器

可以到

`http://bbs.92wy.com/forumdisplay.php?fid=52`

下一个最新的传神外挂，我在`win2k`下，测试`199`，`199a`，`199b`通过

只是我的不能停在`ep`处，别人的可以，如果你不可以就自己把这里改下

先把`events`改为`make first pause`改成`system breakpoint`

```
var ZwSetInfo  
var SetThrPri  
var GetProHeap  
var mbase  
var msize  

gpa "ZwSetInformationThread","ntdll.dll"  
mov  ZwSetInfo, $RESULT  
gpa "SetThreadPriority","Kernel32.dll"  
find $RESULT,#C20800#  
mov  SetThrPri, $RESULT  
bp SetThrPri  
bp ZwSetInfo  
esto  
bc ZwSetInfo  
find ZwSetInfo, #C21000#  
mov eip, $RESULT  
esto //过 ZwSetInfo  
GMI eip, MODULEBASE  
mov mbase,$RESULT  
GMI eip, MODULESIZE  
mov msize,$RESULT  
esto //从 ep 开始，脚本开始
```

走到了`oep`，为了能对齐指令，我多加了一个`nop`

可是我的脚本没修复`iat`功能，`dump`功能。。

```
00423F5A    5F               POP EDI  
00423F5B    5D               POP EBP  
00423F5C    C3               RETN  
00423F5D    0000             ADD \[EAX\],AL  
00423F5F    0000             ADD \[EAX\],AL  
00423F61    0000             ADD \[EAX\],AL  
00423F63    0000             ADD \[EAX\],AL  
00423F65    0000             ADD \[EAX\],AL  
00423F67    0000             ADD \[EAX\],AL  
00423F69    0000             ADD \[EAX\],AL  
00423F6B    90               NOP  
00423F6C    64:A1 00000000   MOV EAX,FS:\[0\]     ;伪 oep  
00423F72    50               PUSH EAX  
00423F73    64:8925 0000000>MOV FS:\[0\],ESP  
00423F7A    83EC 58          SUB ESP,58  
00423F7D    53               PUSH EBX  
00423F7E    56               PUSH ESI  
00423F7F    57               PUSH EDI  
00423F80    8965 E8          MOV \[EBP-18\],ESP  
00423F83    90               NOP  
00423F84    E8 21CCF2FF      CALL 00350BAA       ;又进壳里，不知道干什么..  
00423F89    33D2             XOR EDX,EDX  
00423F8B    8AD4             MOV DL,AH  
00423F8D    8915 8C9F4500    MOV \[459F8C\],EDX  
00423F93    8BC8             MOV ECX,EAX  
00423F95    81E1 FF000000    AND ECX,0FF
```

怎么不讨论了呢？？我看好像这样就不异常了。

即使这样，离脱壳还远着呢，`IAT`的处理？

```
//=========================================================  
//解决 OutputDebugStringA 问题  
//var fnOutputDebugStringA//自行定义  
gpa "OutputDebugStringA","Kernel32.dll"  
mov fnOutputDebugStringA,$RESULT  
mov \[fnOutputDebugStringA\],#8BFFC204#//mov edi,edi  
add fnOutputDebugStringA,4  
mov \[fnOutputDebugStringA\],#00#//retn 4  
//=========================================================
```

```
//stolen byte(s)  
//查找入口  
bp fnSetThreadPriority//为原文档中的 SetThrPri,我习惯我的风格  
esto  
bc  fnSetThreadPriority  
rtu  
mov OEPFinder,eip  
and OEPFinder,FFFFF000//?忘记了,VirtualAlloc 是 1000 对齐还是 10000 对齐  
loop\_finding:  
find OEPFinder,#6150#//POPAD,PUSH EAX  
cmp $RESULT, 0  
sub OEPFinder,1000  
je loop\_finding  
mov OEP,$RESULT//保存临时结果  
sub OEPFinder,1000  
find OEPFinder,#6150#  
cmp $RESULT, 0  
je end\_find  
mov OEP,$RESULT  
end\_find:  
bp OEP  
esto  
bc OEP  
sti//POPAD, 到此处到 stolen byte(s)已经很近了,  
   //此处已经堆栈平衡
```

我已经脱完修复好了，在忙破解，等破解完，有时间整理再发

可以找下面的帖子看看，说的非常详细

`window`的“`hying 0.7x`的脱壳过程”

`loveboom`的“`Hying's pelock v0.7x`脚本”

`hnhuqiong`的“某人最新`0.7x`完整分析”

`4dd466d0e1e0cc123afbe504da2462ca`
