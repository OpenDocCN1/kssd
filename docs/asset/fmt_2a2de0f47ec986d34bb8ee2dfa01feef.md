
# simonzh 的 unpackme 脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-559.htm`](https://www.pediy.com/kssd/pediy07/pediy7-559.htm)

目标程序：附件下载

工具: `Driver Studio v3.2/IceExt v0.67,LoadPE,ImportRec`

步骤:

`bpmb SetThreadContext x do "d (*(esp+8) + b8)"`

`bpmb WaitForDebugEvent x`

运行,断在`WaitForDebugEvent`,F12 回程序空间(即注入`winlogon`的代码)

`bd 2`(禁止`WaitForDebugEvent`)

`bpx xxxx076C do "d eax"` ; 对测试`dwDebugEventCode`处下断,`xxxx0000`为分配内存的基地址,offset=076C

壳代码引起的`EXCEPTION_DEBUG_EVENT`有 3 次:

1. 地址`010204E4`,代码为`int 3`
   然后`SetThreadContext,eip->0102060A`
2. 地址`01020F88`,代码为`lock cmpxchg8b eax`
   `SetThreadContext,eip->01026583`
3. 地址`01027945`代码`int 3`,这里明显在解码

可以看到,3 次异常地址都在壳代码的 section 内。显然,原程序与壳代码之间没有联系。

对`ContinueDebugEvent`下断:

`bpmb ContinueDebugEvent x`

断下后:

`proc ->查 calc 的 KTEB`

`addr KTEB`

`u 01027945`

在`int 3`后的`NOP`下断,禁止除此之外的其他断点,F5 继续调试,断到子进程内。直接用`esp`定律:

`bpmb 23:6FFC0 w`

断下 2 次后 F8 单步很快到这里:

```
01028ED8: jmp eax ; eax=00550EED
```

`stolen code`:

```
00550EED push 00000070
 jmp 00550EF4
-> push 01001606
 jmp 00550EFE
 add dword ptr [ecx],-1b
-> push 00550F09 ; 变形的 call 的返回
 push 01012C18
 ret
00550F09 jmp 00550F0E
 _JUNKCODE
-> push 010128B1
 ret ; 回原程序
```

得到`OEP`为`010128A5`,`stolen codes`为:

```
push 70
push dumped.01001608
call dumped.01012C18
```

`Dump`: Correct Imagesize, dump full, rebuild PE.
修复 IAT,可以直接用`ImportRec`手工完成,可看出真正的`api`(不仔细点容易认错),但数量多了点,还是跟一跟.
`iat`开始: `01001000`, 结束`01001238`, `size=0x238`

在 2 次`SetThreadContext`后,可以看到`iat`中还未填入`redirected api`(实际`iat`内是加密`api name`的`rva`).

`bpmb LoadLibraryA x`
F12 返回到`01027BDC`,单步跟,很快会出现第 1 个`iat`项的地址: `01001020`
到这里:

```
01027FFF call [ebp+40BAFB] -> 550000
```

跳到`GetProcAddress+0xF`的地址。取到`api`地址,第 1 个为`WaitForSingleObject`
patch 下面的标记检测,即可得到干净的`IAT`。

```
or ecx,ecx (此时=1)
01028257 jz .... -> Patch 为 jmp
```

建议:

1. 花指令过于单一,作用不大(是否源代码每行都用了,实在太多)
2. 对内核 debugger 防范不够,不用`IceExt`都可以跑
3. `anti-debug`不够,缺少自校验,可较轻松地设断,`Hying`的壳就难缠得多
4. `IAT`加密太容易被跳过了.
5. 被保护程序与壳代码联系不够,可加入`replaced code`.
6. `stolen code`温柔了点

个人意见,纯属信口开河。不喜欢用`inject`的壳,脱壳不好调试 :-)

(手伸得太长,有被列入病毒的危险吧)

请问`simonzh`:

我开始是用`OD`的,使代码注入到自己而不是`winlogon`,这样可以继续用`OD`调试.
但用`WriteProcessMemory`向子进程写入`\??\d:\windows\system32\winlogon.exe`
(用`winlogon`的路径替代`calc`)后,下一个调试事件是`EXIT_PROCESS_DEBUG_EVENT`,
即子进程退出了,能否解释一下?(我没有调试子进程,而且`DebugActiveProcessStop`我用不来)

`softworm`兄弟强. ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

关于`IAT`的哪个跳转都是被`MS`给害了.

`calc.exe`, `notepad.exe`等用了

`msvcrt.dll`中一些比较特殊的`API`.

为了兼容, 比较了一下.

对内核调试器检测有什么好的方法?

花指令, 我是借用`cyclotron`兄弟的, 我自己只加了一组.

至于`codereplace`等, `res`压缩, 限于能力和时间, 以后再说了.

至于`OD`退出, 兄弟再分析一下子进程.

![](img/b16bec246e75299c87cf57e1fb808ac9.jpg) ![](img/b16bec246e75299c87cf57e1fb808ac9.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
