
# Q 宠保姆 vc 版 Pro 2.1Service Pack5 脱壳教程

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-589.htm`](https://www.pediy.com/kssd/pediy08/pediy8-589.htm)

Q 宠保姆 vc 版 Pro 2.1Service Pack5 脱壳教程。

加壳方式：

`ASProtect 2.1x SKE -> Alexey Solodovnikov`

用 `od` 加载原程序 `QQPetNurse.exe`

先用脚本 `findoep.txt` 运行到 `oep`

复制 `asmcode` 二进制粘到，`QQPetNurse.exe` 空白内存处，

记下起点地址，因为下面脚本要用的。

再运行脚本 `asm.txt`,这里我们己经修复了大部分的代码了。

下面是手工活，`ctrl+s` 搜索 `call e20000`,总共有三十多个。

在入在点 `f7` 跟入，再回到上一个起点，再 `f4` 代码就解密出来了

再跟入，在被调用的 `dll` 里面搜索，调用的代码是，的输入函数

然后再修复，

这个程序输入表是没有加密的，但有一个地址要改一下。

在下面提到过的。

## `findoep.txt`

```
\-------------------------
var v1
eoe test1
test2:
run
test1:
mov v1,45f000
cmp [v1],77f4584b
jne test3
mov v1,eip
add v1,2
cmp [v1],855ccaef
jne test3
add v1,15
cmp [v1],C4380C
jne test3
bprm 401000,5e000 //设置内存断点
eob test4
esto
test3:
esto
test4:
bpmc 401000
cmt eip,"oep"
ret
\--------------------------
```

修复被加密的代码：

因为调用的函数代码被抽了，所有修复起来有点困难

我们从 `401000` 开始一直向下找到 `45f000`

搜索 `call e20000`

用脚本搜索速度太慢，所以直接在执行汇编代码。`asmcode.txt`

由脚本调用。

## `asmcode.txt`

```
\------------
var StartCode //保存 asm 代码起点
var tem //用做临时变量
var MyCode //
var oep //变量保存，原始 eip
var n //保存未知的
mov oep,eip //这里保存
mov StartCode,11880 //这里是 asmcode 加载的起点地址。
mov n,0
mov eip,StartCode
mov tem,eip
add tem,19
bp tem //当找到加密代码时，会在这里断下
mov tem,eip
add tem,24
bp tem //当查代代码结束时断下
eob le1 //当发生中断时执行这里的脚本
run
le1:
mov tem,StartCode
add tem,19
cmp eip,tem //中断时比较是否为查加密代码处
je break0
cmp eip,MyCode//执行壳里面的代码之后会在这里
je break1
mov tem,StartCode
add tem,24
cmp eip,tem //比较搜索是否结束
je break2
mov eax,3
ret
break0:
mov MyCode,edx
mov eip,edx
sti //跟入壳代码
go edx //跳出来程序代码，壳会修改这里的地址。
break1:
sti //进入，进入解密后的 call，
find eip,#68????????c3#//这里为 push XX retn 两步 f8
cmp $RESULT,0
jne l2
find eip,#e9??????ff#//这里为 jmp XX 只要一步 f8
cmp $RESULT,0
jne l3
sub esp,4
run
ret
l2:
mov eip,$RESULT
sto
sto //进入下一个 call，这里就可以读到输入表了
jmp le3
l3:
mov eip,$RESULT
sto //进入下一个 call，这里就可以读到输入表了
jmp le3
le3:
find eip,#68????????c3#//这里是 push ???? +retn
cmp $RESULT,0
jne le4
find eip,#c3#//如果没找到，就是 retn
cmp $RESULT,0
jne le4
find eip,#c20400# //这里是 retn 4 有几个函数后面是这样子的
cmp $RESULT,0
jne le4
add n,1
add edx,5
mov tem,StartCode
add tem,5
mov eip,tem
run
ret
le4:
mov eip,$RESULT
mov tem,eip
add tem,1
mov tem,[tem]
mov ebx,eip
and ebx,ff
sub tem,ebx
mov eax,tem
mov tem,eip
sub tem,4
mov tem,[tem]
sub tem,ebx
mov edi,tem
add esp,4
mov tem,StartCode
add tem,26
mov eip,tem
run
ret
break2:
mov eip,oep
mov eax,n
ret
\------------
```

## `asm.txt`

```
\-----------
BA 00 10 40 00 80 3A E8 75 11 8B 42 01 03 C2 83 C0 05
3D 00 00 E2 00 75 02 8B C0 42 81 FA 00 F0
45 00 76 E1 8B C0 33 F6 B9 00 F0 45 00 39 01 75 0A 89
4A 02 66 C7 02 FF 15 EB CA 83 C1 04 81 F9
20 F7 45 00 7C E7 83 FE 00 75 05 46 8B C7 EB D8 B8 00
00 E2 00 2B C2 83 E8 05 89 42 01 42 EB A5
\-----------

00011A00        BA 00104000            mov edx,401000 脚本开始运行就调用这里。
00011A05        803A E8              cmp byte ptr ds:[edx],0E8
00011A08        75 11                jnz short 00011A1B
00011A0A        8B42 01              mov eax,dword ptr ds:[edx+1]
00011A0D        03C2                 add eax,edx
00011A0F        83C0 05              add eax,5
00011A12        3D 0000E200          cmp eax,0E20000
00011A17        75 02                jnz short 00011A1B
00011A19        8BC0                 mov eax,eax   脚本在这里设下断点，只要中断就会执行脚本
00011A1B        42                   inc edx
00011A1C        81FA 00F04500        cmp edx,45F000
00011A22      ^ 76 E1                jbe short 00011A05
00011A24        8BC0                 mov eax,eax  断在这里，代码己搜索完了。脚本运行结束
00011A26        33F6                 xor esi,esi  当脚本执行到计算出函数调用地址后时会断在这里
 地址可以在，eax,也可能在 edi 里面
00011A28        B9 00F04500          mov ecx,45F000  //输入表的开始地址
00011A2D        3901                 cmp dword ptr ds:[ecx],eax
00011A2F        75 0A                jnz short 00011A3B
00011A31        894A 02              mov dword ptr ds:[edx+2],ecx
00011A34        66:C702 FF15        mov word ptr ds:[edx],15FF
00011A39      ^ EB CA              jmp short 00011A05
00011A3B        83C1 04              add ecx,4
00011A3E        81F9 20F74500        cmp ecx,45F720
00011A44      ^ 7C E7              jl short 00011A2D
00011A46        83FE 00              cmp esi,0
00011A49        75 05                jnz short 00011A50
00011A4B        46                   inc esi  在这里如果函数地址不在 eax 里面
00011A4C        8BC7                 mov eax,edi  查 edi 里面的地址
00011A4E      ^ EB D8              jmp short 00011A28
00011A50        B8 0000E200          mov eax,0E20000 如果都没找到的话，
00011A55        2BC2                 sub eax,edx    就还原原来的代码
00011A57        83E8 05              sub eax,5      己方便手工修复查代代码
                      只要探索 call e20000 就可以代到未能修复的代码了
                      当初我不知道这点，靠单步执行，来找，确实很累，
                      还没做好。
00011A5A        8942 01              mov dword ptr ds:[edx+1],eax
00011A5D        42                   inc edx
00011A5E      ^ EB A5              jmp short 00011A05
\-----------
```

上面的脚本己经可以修复，大部分被抽的代码了。

但还有一些系统函数因为太短，所有的部分全部被抽掉了

只要一个一个的搜索，才能把它们代出来。

下面一共有几十个。。

## 函数调用地址修复

```
00409142     E8 B96EA100      GetCurrentThreadId ->004D6270
00409268     E8 936DA100      GetCurrentThreadId ->004D6270
00414761     E8 9AB8A000      GetCurrentThreadId ->004D6270
0041A76C     E8 8F58A000      GetCurrentThreadId ->004D6270
0041C09F     E8 5C3FA000      GetCurrentThreadId ->004D6270
0042913A     E8 C16E9F00      GetCurrentThreadId ->004D6270
004310B1     E8 4AEF9E00      GetCurrentThreadId ->004D6270
00431376     E8 85EC9E00      GetCurrentThreadId ->004D6270
004373BF     E8 3C8C9E00      GetCurrentThreadId ->004D6270
004091B5     E8 466EA100     GetCurrentProcess ->004D628C
00430E4F     E8 ACF19E00     GetCurrentProcess ->004D628C
00431E09     E8 F2E19E00     GetCurrentProcess ->004D628C
00446169     E8 929E9D00     GetCurrentProcess ->004D628C
0044650C     E8 EF9A9D00     GetCurrentProcess ->004D628C
004468CC     E8 2F979D00     GetCurrentProcess ->004D628C
0041B7BE     E8 3D48A000     GetCurrentProcessId ->004D6250
004373B7     E8 448C9E00     GetCurrentProcessId ->004D6250
00441D77     E8 84E29D00     GetCurrentProcessId ->004D6250
0041A763     E8 9858A000     GetCurrentThread ->004D6300
00445915     E8 E6A69D00     GetCurrentThread ->004D6300
00445C89     E8 72A39D00     GetCurrentThread ->004D6300
0044D0B8     E8 432F9D00     GetCurrentThread ->004D6300
0044D5DE     E8 1D2A9D00     GetCurrentThread ->004D6300
0044D875     E8 86279D00     GetCurrentThread ->004D6300
0044E88A     E8 71179D00     GetCurrentThread ->004D6300
0040F212     E8 E90DA100     GetThreadLocale ->004D6240
0043F605     E8 F6099E00     GetThreadLocale ->004D6240
00424FC0     E8 3BB09F00     GetOEMCP ->004D61E4
0043166A     E8 91E99E00     GetOEMCP ->004D61E4
0042C9C3     E8 38369F00     GetCommandLineA ->004D6154
00430E13     E8 E8F19E00     IsDebuggerPresent ->004D6178
00431DDA     E8 21E29E00     IsDebuggerPresent ->004D6178
0043168D     E8 6EE99E00     GetACP ->004D617C
0043F640     E8 BB099E00     GetACP ->004D617C
004370C8     E8 338F9E00     FreeEnvironmentStringsW ->004D61A4
004370DF     E8 1C8F9E00     FreeEnvironmentStringsW ->004D61A4
```



## 汇编与调用分析

`0043730D` E8 EE8C9E00 `SetHandleCount` -> `004D61AC`

`0043F5B6` E8 450A9E00 `CreateToolhelp32Snapshot` -> `004D62F8`

`0043F5C2` E8 390A9E00 `Module32Next` -> `004D62F0`

`00455BC9` E8 32A49C00 `CloseClipboard` -> `004D658C`

`00457F80` E8 7B809C00 `CloseClipboard` -> `004D658C`

```
mov eax,dword ptr fs:[18]-->GetCurrentThreadId
mov eax,dword ptr ds:[eax+24]
or eax,FFFFFFFF -->GetCurrentProcess
mov eax,dword ptr fs:[18] GetThreadLocale
mov eax,dword ptr ds:[eax+C4]
push -2 GetCurrentThread
pop eax
mov eax,dword ptr fs:[18] GetCurrentProcessId
mov eax,dword ptr ds:[eax+20]
mov eax,dword ptr ds:[7C88B638] GetOEMCP
mov eax,dword ptr ds:[7C88B5D4] GetCommandLineA
mov eax,dword ptr fs:[18] IsDebuggerPresent
mov eax,dword ptr ds:[eax+30]
movzx eax,byte ptr ds:[eax+2]
mov eax,dword ptr ds:[7C88B358] GetACP
mov edi,edi FreeEnvironmentStringsW
push ebp
mov ebp,esp
push dword ptr ss:[ebp+8]
mov eax,dword ptr fs:[18]
mov eax,dword ptr ds:[eax+30]
push 0
push dword ptr ds:[eax+18]
call dword ptr ds:[<&ntdll.RtlFreeHea>;
ntdll.RtlFreeHeap
movzx eax,al
pop ebp
mov eax,114A CloseClipboard user32.dll
mov edx,7FFE0300
call dword ptr ds:[edx]
```

`0043F5B6` $ FF15 F8624D00 `call` `dword` `ptr` `ds`:`[<&kernel32.CreateToolhelp32Snapshot>]` ;

`|CreateToolhelp32Snapshot`

`0043F5BC` $ FF15 F4624D00 `call` `dword` `ptr` `ds`:`[<&kernel32.Module32First>]` ;

`|Module32First`

`0043F5C2` $ FF15 F0624D00 `call` `dword` `ptr` `ds`:`[<&kernel32.Module32Next>]` ;

`|Module32Next`

`0043F5C8` $ FF15 88624D00 `call` `dword` `ptr` `ds`:`[<&kernel32.Process32First>]` ;

`|Process32First`

`0043F5CE` $ FF15 84624D00 `call` `dword` `ptr` `ds`:`[<&kernel32.Process32Next>]` ;

`Process32Next`

## 修复思路

这里的 `call` 都要改为 `jmp`，因为修复的时候出错了。

修复完之后 `dump` 就可以了。

`SetHandleCount` 这个函数在输入表里面，可能是被删了，但我发现 `FreeEnvironmentStrings` 有两个，所以将一个改为了 `SetHandleCount`。

脱这个壳花了我一天多的时间。头都看大了。当初没用脚本。大部分都是手工活。不然几天都难弄好。

被加密的 `call` 总共有 `0x150` 之多，每一个调用的函数，前部分代码都被抽了，代码少的就抽空了。

## 脚本代码

`asmEx.txt`

修复代码全被抽的 `call`，不用手工了：

```
var StartCode //保存 asm 代码起点 4167ef
var tem  //用做临时变量
var MyCode //
var oep  //变量保存，原始 eip
var n //保存未知的
var address//用来保存 call 地址

mov oep,eip  //这里保存
mov StartCode,11880 //
mov n,0
mov eip,StartCode
mov tem,eip
add tem,19
bp tem //当找到加密代码时，会在这里断下
mov tem,eip
add tem,24
bp tem //当查代代码结束时断下
eob le1 //当发生中断时执行这里的脚本
run

le1:
mov tem,StartCode
add tem,19
cmp eip,tem //中断时比较是否为查加密代码处
je break0
cmp eip,MyCode//执行壳里面的代码之后会在这里
je break1
mov tem,StartCode
add tem,24
cmp eip,tem //比较搜索是否结束
je break2
mov eax,3
ret

break0:
mov MyCode,edx
mov eip,edx
sti //跟入壳代码
go edx //跳出来程序代码，壳会修改这里的地址。

break1:
sti //进入，进入解密后的 call，
find eip,#68????????c3#//这里为 push XX retn 两步 f8
cmp $RESULT,0
jne l2
find eip,#e9??????ff#//这里为 jmp XX 只要一步 f8
cmp $RESULT,0
jne l3
sub esp,4
run
ret

l2:
mov eip,$RESULT
sto
sto //进入下一个 call，这里就可以读到输入表了
jmp le3

l3:
mov eip,$RESULT
sto
jmp le3

le3://这下面是查找各个函数代码
find eip,#64A1180000008B4024C3#//GetCurrentThreadId 7C82BC6D
cmp $RESULT,0
jne a1
find eip,#83C8FFC3#//GetCurrentProcess 7C82EC9D
cmp $RESULT,0
jne a2
find eip,#64A1180000008B4020C3#//GetCurrentProcessId 7C827B9E
cmp $RESULT,0
jne a3
find eip,#6AFE58C3#//GetCurrentThread 7C82C212
cmp $RESULT,0
jne a4
find eip,#64A1180000008B80C4000000C3#//GetThreadLocale 7C82E15F
cmp $RESULT,0
jne a5
find eip,#A138B6887CC3#//GetOEMCP 7C80A67D
cmp $RESULT,0
jne a6
find eip,#A1D4B5887CC3#//GetCommandLineA 7C814A34
cmp $RESULT,0
jne a7
find eip,#64A1180000008B40300FB64002C3#//IsDebuggerPresent 7C824860
cmp $RESULT,0
jne a8
find eip,#A158B3887CC3#//GetACP 7C82C0B5
cmp $RESULT,0
jne a9
find eip,#8BFF558BECFF750864A1#//FreeEnvironmentStringsW 7C814B5D
cmp $RESULT,0
jne a10
find eip,#8BFF558BEC8B45085DC20400#//SetHandleCount 7C82DA8F
cmp $RESULT,0
jne a11
find eip,#B84A110000BA0003FE7FFF12C3#//CloseClipboard 77E3D354
cmp $RESULT,0
jne a12
add n,1
add edx,5
mov eax,2121 //这个随便输填，只要输入表里面没有就可以了
jmp le4//如果没代到时，要还原代码

mov tem,StartCode
add tem,5
mov eip,tem
run
ret

le4:
add esp,4//这里是平衡椎
mov tem,StartCode
add tem,26
mov eip,tem//运行我们的汇编代码，来找函数
run
ret

break2:
mov eip,oep
mov eax,n
ret

a1:
mov eax,7C82BC6D
jmp le4

a2:
mov eax,7C82EC9D
jmp le4

a3:
mov eax,7C827B9E
jmp le4

a4:
mov eax,7C82C212
jmp le4

a5:
mov eax,7C82E15F
jmp le4

a6:
mov eax,7C80A67D
jmp le4

a7:
mov eax,7C814A34
jmp le4

a8:
mov eax,7C824860
jmp le4

a9:
mov eax,7C82C0B5
jmp le4

a10:
mov eax,7C814B5D
jmp le4

a11:
mov eax,7C82DA8F
jmp le4

a12:
mov eax,77E3D354
jmp le4
```

有几个 `call` 还是要手动改一下，改成 `jmp`。

## 高级脚本代码

```
var StartCode //保存 asm 代码起点 4167ef
var tem  //用做临时变量
var MyCode //
var oep  //变量保存，原始 eip
var n //保存未知的
var TemEsp
var sticode
var stisave
var EndOfCode
var aspCall
var EndOfAti
var Bk1
alloc 1000
mov sticode,$RESULT
mov stisave,sticode
mov oep,eip  //这里保存
mov StartCode,11880 //
mov EndOfCode,46D000
mov EndOfAti,46D780
mov aspCall,1E40000
mov Bk1,CFC875
mov [StartCode],#BA00104000803AE875118B420103C283C0053D0000E40175028BC04281FA00C0460076E18BC033F6B900C046003901750A894A0266C702FF15EBCA83C10481F980C746007CE783FE007505468BC7EBD8B80000E4012BC283E80589420142EBA5#

//init StartCode
mov tem,StartCode
add tem,13
mov [tem],aspCall
mov tem,StartCode
add tem,1E
mov [tem],EndOfCode
mov tem,StartCode
add tem,29
mov [tem],EndOfCode
mov tem,StartCode
add tem,40
mov [tem],EndOfAti
mov tem,StartCode
add tem,51
mov [tem],aspCall
mov n,0
mov eip,StartCode
mov tem,eip
add tem,19
bp tem //当找到加密代码时，会在这里断下
mov tem,eip
add tem,24
bp tem //当查代代码结束时断下
bphws Bk1, "x"
eob le1 //当发生中断时执行这里的脚本
run

le1:
bphwc MyCode
mov tem,StartCode
add tem,19
cmp eip,tem //中断时比较是否为查加密代码处
je break0
cmp eip,Bk1
je break3
cmp eip,MyCode//执行壳里面的代码之后会在这里
je break1
mov tem,StartCode
add tem,24
cmp eip,tem //比较搜索是否结束
je break2
mov eax,3
ret

break0:
mov MyCode,edx
mov eip,edx
sti //跟入壳代码
bphws edx,"x"
mov TemEsp,esp
run
//go edx
//跳出来程序代码，壳会修改这里的地址。

break3:
sub TemEsp,DC
mov tem,[TemEsp]
mov eax,0
dec eax
//ollyscript.dll 有 bug
//不能用 mov eax,-1
//这里是比较 call 后面是否有代码被抽
cmp tem,eax
je stst
//mov eax,100
//cmp tem,eax
//jae stst
mov [sticode],MyCode
add sticode,4
stst:
mov eax,[esp]
mov tem,StartCode
add tem,26
add esp,184
mov eip,tem
mov edx,MyCode// 恢复 edx
run
jmp le1
ret

break1:
sti //进入，进入解密后的 call，
find eip,#68????????c3#//这里为 push XX retn 两步 f8
cmp $RESULT,0
jne l2
find eip,#e9??????ff#//这里为 jmp XX 只要一步 f8
cmp $RESULT,0
jne l3
sub esp,4
run
ret

l2:
mov eip,$RESULT
sto
sto //进入下一个 call，这里就可以读到输入表了
jmp le3

l3:
mov eip,$RESULT
sto //进入下一个 call，这里就可以读到输入表了
jmp le3

le3:
find eip,#68????????c3#
cmp $RESULT,0
jne le4
find eip,#c3#
cmp $RESULT,0
jne le4
find eip,#c20400#
cmp $RESULT,0
jne le4
add n,1
add edx,5
mov tem,StartCode
add tem,5
mov eip,tem
run
ret

le4:
mov eip,$RESULT
mov tem,eip
add tem,1
mov tem,[tem]
mov ebx,eip
and ebx,ff
sub tem,ebx
mov eax,tem //保存第一种情况
mov tem,eip
sub tem,4
mov tem,[tem]
sub tem,ebx
mov edi,tem //保存第二种情况
add esp,4
mov tem,StartCode
add tem,26
mov eip,tem
```



`run`

`ret`

`break2:`

`mov eip,oep`

`mov eax,n`

`dma stisave, 1000, "sti.bin"`

`ret`
