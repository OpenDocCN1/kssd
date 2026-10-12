
# Private\_exe\_Protector 2.71 Full-Protection Unpacking

> 原文：[`www.pediy.com/kssd/pediy10/97579.html`](https://www.pediy.com/kssd/pediy10/97579.html)

## 1. 保护特性分析

PEP 的脱文比较少，高版本 `ANTI-DUMP`、资源保护、`IAT` 加密的处理也比较有意思。最高的注册版为 2.71 版。选择 `DELPHI7` 无壳试炼品，`entry point virtualization mode` 可以按照语言特征很快恢复，故不勾选，其他保护选项全选。

这里有必要一提的是，PEP 的 `live protection at run-time` 不太稳定，大家抱怨 PEP 兼容性差也多是勾选了这个选项的缘故。

加壳完毕后发现第二个段虚拟大小为 `25100000`，这就是 PEP 的典型 `ANTI-DUMP` 特征。兼容性差也与这个有莫大关系。很多 `OD` 无法正确识别加壳后文件的区段，就是这里在作怪，可以选用 `SOD` 来辅助处理。

使用原版 `OD`，勾选 `SOD` 全选项，并使用 `PATCH` 之后的 `phantom1.54` 保护 `DRX`，方便下硬件断点。

## 2. 定位 OEP 与分析入口

按下 `SHIFT+F9` 跑起来，到 `401000` 搜索：

引用: `A3??????00A1??????00A3??????0033C0A3??????0033C0A3??????00`

找到 `DELPHI` 程序的第一个 `CALL` 为 `00405BC8`，从而找到 `OEP` 为 `0044CA98`。跟进第一个函数，观察被处理成了：

引用:  

```
00405B04  - E9 2DB11D25     jmp 255E0C36                     ; 255E0C36  
00405B09    2E:8BC0         mov eax,eax  
00405B0C  - E9 CFAF1D25     jmp 255E0AE0                     ; 255E0AE0  
00405B11    F2:             prefix repne:  
00405B12    8BC0            mov eax,eax  
00405B14  - E9 5FAE1D25     jmp 255E0978                     ; 255E0978  
00405B19    25 8BC0E9F6     and eax,F6E9C08B  
00405B1E    AC              lods byte ptr ds:[esi]  
00405B1F    1D 25BF8BC0     sbb eax,C08BBF25 
```

这是勾选了 `do not restore iat` 的缘故，否则这里只会加密 `IAT` 而不处理 `FF25`。单步走：

引用:  

```
2557F690    55                       push ebp  
2557F691    8BEC                     mov ebp,esp  
2557F693  - E9 F1880A00              jmp 25627F89                     ; 25627F89  
2557F698    2D 9D0E7EE6              sub eax,E67E0E9D  
2557F69D    57                       push edi  
2557F69E    BF 30A011E4              mov edi,E411A030  
2557F6A3    81F1 62D243B3            xor ecx,B343D262  
2557F6A9    24 8C                    and al,8C  
2557F6AB    FC                       cld  
2557F6AC    9A 6EE6A9A5 5747         call far 4757:A5A9E66E  
2557F6B3    6D                       ins dword ptr es:[edi],dx  
2557F6B4    35 56C7E438              xor eax,38E4C756  
2557F6B9    90                       nop  
2557F6BA    90                       nop  
2557F6BB    90                       nop  
2557F6BC    90                       nop  
2557F6BD    90                       nop  
2557F6BE    5D                       pop ebp  
2557F6BF    C2 0400                  retn 4 
```

这里我 `NOP` 了几个字节，方便看后面的 `RETN 4`。

`DarkBull` 在以前的文章中将此 `RETN` 命名为 `VM_RETN`，暂且借用这个叫法。

在这里下个断，中断下来后在 `esp+8` 处出现了真实的 `GetModuleHandleA`。这里还有一种情况，就是在 `esp+0` 处出现真实函数。

## 3. IAT 处理与 Hook 机制

`PEP2.3` 之后的版本在 `IAT` 处理上有点变化，壳会 `HOOK` 掉 `FindResourceA`、`LoadResource`、`SizeofResource` 等几个函数。以前的版本把代码中的 `FF25` 直接指向 `HookFindResourceA` 等。这几个函数在 `PEPVM.DLL` 中导出，`PEPVM.DLL` 是 PEP 的功能 `DLL`，可以从主程序中想想办法抓出来。

而新版则指向真实函数，当然真实函数入口被 `HOOK` 了。

所以在写脚本的时候，我们需要判断的情况也就少了一种。等于是新版本变相降低了强度。

中断在此 `RETN` 时，还有一种情况，就是返回壳模拟的几个函数。因为数量很少，所以脚本中不需要考虑这几个函数，留待后续手工处理。

## 4. 自动化修复脚本

脚本如下：

引用:  

```
var fi  
var oep  
var tmpesp  
var tmp  
var sFile  

bc  
bphwc  
bphws 0044CA98  ,"x" // oep  
esto  
bphwc  

mov sFile,"left.txt"  
mov oep,eip  
mov tmpesp,esp  
mov fi,401000  

bp 2557F6BF  // vm_retn  

loop:  
mov esp,tmpesp  
inc fi  
find fi,#E9??????25# // ff25-sign  
cmp $RESULT,0  
je exit  
mov fi,$RESULT  
gci fi,DESTINATION  
cmp $RESULT,25672000 //last section va+ size  
jae loop  
mov tmp,$RESULT  
find tmp,#E9??????FF#,5 // api left  
cmp $RESULT,0  
jnz left  
mov eip,fi  
esto  
cmp [esp],5ADC0000  //first system dll va  
ja fix1  
cmp [esp+8],5ADC0000 //first system dll va  
ja fix2  

left:  
wrta sFile,fi  
wrta sFile,"\\r\\n"  
jmp loop  

fix1:  
mov tmp,[esp]  
eval "jmp {tmp}"  
asm fi,$RESULT  
jmp loop  

fix2:  
mov tmp,[esp+8]  
eval "jmp {tmp}"  
asm fi,$RESULT  
jmp loop  

exit:  
bc  
bphwc  
mov eip,oep  
mov esp,tmpesp  
ret 
```

重载试炼品，修改脚本中相关数值，直接跑脚本。把 `OD` 窗口缩小，可以有效提高脚本运行速度。脚本跑完后会把大部分的 `FF25` 修复成 `JMP API`。

这里有 4 个漏掉的，分别是壳模拟的 4 个函数：

引用:  

```
401224-ExitProcess  
40126C-GetProcAddress  
405E68-GetProcAddress  
405F00-LoadLibraryA 
```

单步跟一下就出来了，具体跟踪过程，详见附件中动画。

至此 `API` 全部找出来了。用 `UIF` 修复成 `FF25` 即可。

## 5. 修复与资源提取

用 `LOADPE` 修正镜像大小，用区域转存 `400000-460000`。如果用完整转存的话，那个超级大的 `ANTI-DUMP` 区段就会让 `LOADPE` 假死。

修改区段数量为 `1`，修改 `ROFFSET`、`RSIZE` 与 `VOFFSET`、`VSIZE` 一致。修改文件对齐为 `1000`，部首大小为 `1000`，`RVA` 数及大小为 `10`。清空输入表，资源表，`TLS` 目录。

用 `IMPORTREC` 重建输入表。保存为 `unpacked.exe`。

发现文件没有图标，这是因为 PEP 会把资源分块加密保存到壳里，运行调用时再解密，这是这个壳的一大特色。对此目前没有好的解决办法，只能从 `LoadResource` 等函数的返回值确定资源块的位置，然后保存下来贴进 `unpacked.exe` 里。

用 `FixResDemo` `DUMP` 加壳过后程序里面的图标，版本等资源信息。能找回一点是一点 ^\_^ 保存为 `.rsrc`，贴到 `unpacked.exe` 里。

引用:  

```
.rsrc va-> 63000
```

为了方便后续资源的修复，扩充 `.rsrc` 段 `RSIZE` 到 `1000`。

对 `HookFindResourceA` 下断，中断下来从堆栈找回资源名为 `TFrom1`。紧接着断 `HookLoadResource` 和 `HookSizeofResource`，找回：

引用:  

```
name: TForm1  
va:   25C6C5D0  
size: 118
```

把这部分数据贴入 `TForm1.bin`，用 `Stud_PE` 载入 `unpacked.exe`。

新增资源 -> `number-RT_RCDATA` -> `string-TForm1`。

资源文件选择 `TForm1.bin` 即可。

这样处理之后的 `unpacked.exe` 就可以运行了。针对 PEP 的资源加密，实际上没必要全部恢复，恢复必须的就行了。

用 `PE Optimizer` 优化下大小。

处理完毕大小为 `341KB`。原版无壳文件为 `359KB`。

收工。附上视频，很详细。

[`www.brsbox.com/filebox/down/f...944c8731bb8f29`](http://www.brsbox.com/filebox/down/fc/51025fd16183a8b723944c8731bb8f29)

## 6. 资源加密深入分析

最近一直在玩 PEP，陆陆续续对它的 `IAT` 加密、`VM`、`ANTI`、资源加密都有了点认识。`IAT` 和 `ANTI` 前面已经写过了，在这里 [`http://bbs.pediy.com/showthread.php?t=97579`](http://bbs.pediy.com/showthread.php?t=97579)。

今天来简单谈谈他的资源加密。

从头说起。

PEP 会把原始文件的资源片加密后分散保存，然后 `Hook`：

引用: `FindResourceExW`  
`FindResourceExA`  
`FindResourceW`  
`FindResourceA`  
`SizeofResource`  
`LoadResource`

这么几个资源相关的函数，并判断调用是否来自目标程序。如果是则指向壳里面 `PEPVM.DLL` 对应的 `hookxx` 函数，反之还原被 `hook jmp` 占用的几个字节，并调往原始函数继续执行。

关于它的资源加密，`fxyang` 以及 `DarkBull` 两位前辈都是通过 `LoadResource`、`LoadStringX`、`LoadBitmapX`、`LoadCursorX`、`LoadIconX` 观察返回值确定资源片地址。这样做不仅体力活重且容易漏，因为不是每个程序都会一一调用到这些函数。

所以我们不得不寻求更高效的修复法：

既然要在调用资源的时候能够正确被找到，那么肯定要查表。壳通过查表才知道被他加密的资源片保存在哪里。既然查表，肯定有索引。

所以壳通过 `FindResourceA(W)` 返回索引值，通过 `LoadResource` 查表得到资源片地址，通过 `SizeofResource` 得到当前资源片大小。

索引值从 `1` 开始，与原始文件的资源片在顺序上一一对应。

这里我们可以强行喂给它索引值，并递加此索引，让壳查表解密，然后再强行跳往 `hookSizeofResource`，让壳计算资源片大小。这样一来，体力活全部都由壳自己来完成了。^\_^

当 `hookLoadResource` 返回值为 `0` 时，表示当前索引不存在，也就是说所有的资源片已经全部被我们找到了。

分析完毕，可以写脚本了。

引用: `////////////////////////////////////////////`  
`//  PEP2.x~3.x Resource Dumper.txt`  
`//  Kissy[UpK]`  
`//  `  
`//  info:reach to oep,run the script,then you'll get all the pieces of Resource which were encrypted by Pep 2.x~3.x.`  
`////////////////////////////////////////////`

```
var imgbase  
var Loadrc  
var Loadret  
var Sizeofrc  
var Sizeofret  
var num  
var dumpva  
var dumpname  
var dumpsize  
var tmp  

cmp $VERSION, "1.52"  
jb odver  

mov num,1  

gmi eip,MODULEBASE  
mov imgbase,$RESULT //获取基址  

gpa "LoadResource","Kernel32.dll"  
mov Loadrc,$RESULT  
find Loadrc,#E9#,1  
cmp $RESULT,0  
je error  
gci Loadrc,DESTINATION  
mov Loadret,$RESULT  
find Loadret,#68????????C3#  
mov tmp,$RESULT  
mov Loadret,[tmp+1]  
find Loadret,#C20800#  
mov Loadret,$RESULT  
bp Loadret  

//hookLoadResource  
//查找壳导出函数 hookLoadResource 的返回 RETN  

gpa "SizeofResource","Kernel32.dll"  
mov Sizeofrc,$RESULT  
find Sizeofrc,#E9#,1  
cmp $RESULT,0  
je error  
gci Sizeofrc,DESTINATION  
mov Sizeofret,$RESULT  
find Sizeofret,#68????????C3#  
mov tmp,$RESULT  
mov Sizeofret,[tmp+1]  
find Sizeofret,#C20800#  
mov Sizeofret,$RESULT  
bp Sizeofret  

//hookSizeofResource  
//查找壳导出函数 hookSizeofResource 的返回 RETN  

loopdumprc:  
mov [esp+4],imgbase  //镜像基址,告诉壳当前调用来自程序,让他放心干苦力 ^\_^  
mov [esp+8],num  //num 为索引  
mov eip,Loadrc  
esto  
cmp eax,0  //判断索引是否结束  
je exit  
mov dumpva,eax  //得到资源片起始地址  
mov eip,Sizeofrc  
esto  
cmp eax,0  
je error  
mov dumpsize,eax  //得到资源片大小  
eval "{num}.bin"  
mov dumpname,$RESULT  
dm dumpva,dumpsize,dumpname  
inc num  //索引递加  
jmp loopdumprc  

error:  
msg "no resource needed to be dumped or you made some mistake."  
pause  

odver:  
msg "plz update ODbgScript.dll to 1.65 or above."  
ret  

exit:  
dec num  
eval "All {num} pieces of resource were dumped! You can fix them to your target later."  
msg $RESULT  
msg "Kissy[UpK]"  
ret
```



脚本我写成了全自动的，停在`OEP`后跑脚本，跑完后原始程序中所有的资源片就全部`DUMP`出来了。贴回`dump+fix`的程序中即可。
