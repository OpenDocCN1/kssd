
现在就可以检测出来`Windows ME`或者`Windows 95`/`98`。这是解决问题的第一步，并没有到最后。有可能检测出来`Windows 95`和`Windows 98`之间的不同。

`Section II` showed the start up values and their rules. The start up value of `EBX` on non `NT`-based `OS` is `00530000h`. Exactly this value will be found inside the additional data part - close to the now resolved `LastErrorValue`. By analysing it’s location, the result will be:

第 II 部分说明了初始化值和他们的规律。对于非 `NT` 内核的系统，`EBX` 初始为 `00530000h`。并且这个值可以从附加数据中查出来 - 与 `LastErrorValue` 非常近。为了分析这个数据，下面列出来具体位置：

`Windows 95` - `TEB-base` + `58h`

`Windows 98` - `TEB-base` + `54h`

`Windows ME` - `TEB-base` + `7Ch`

Now we are able to differ between each non `NT`-based operating system.

现在我们就可以区分每个非 `NT` 内核的系统。

### IV. Code creation

### IV. 创建代码

Right now, we where able the detect the version information for each operating system. We want an operating system independent code, we need to structure the given information’s. Also it should be possible to resolve the version information workflow independent. `Section VI` will show the complete solution in Assembler.

现在，我们可以检测每个操作系统的版本信息。为了让代码不依赖于操作系统，我们需要把需要的信息组织成一个结构。并且有必要让类型信息独立起来。第 `VI` 部分将给出完全的汇编解决方案。

First of all, we get the base addresses of `PEB` and `TEB` and resolve the operating system base by analysing them:

首先，获取 `PEB` 和 `TEB` 的基地址，通过分析他们获取不同的操作系统信息。

```
assume fs:nothing
mov ebx,fs:[18h] ; get self pointer from TEB
mov eax,fs:[30h] ; get pointer to PEB / database

.if eax==7FFDF000h && ebx==7FFDE000h
; WinNT based
.else
; Win9X based
.endif ; of base check NT/9X
```

The version information for `NT`-based operation systems is stored inside `PEB`. We only have to analyse the values of

NT 内核的版本信息存储在 `PEB` 中。我们仅仅需要分析这些数据：

`OSMajorVersion` and `OSMinorVersion`:

```
mov ebx,[eax+0A8h] ; get OSMinorVersion
mov eax,[eax+0A4h] ; get OSMajorVersion

.if eax==5 && ebx==0 ; is it Windows 2000?
.elseif eax==5 && ebx==1 ; is it Windows XP?
.elseif eax==5 && ebx==2 ; is it Windows 2003?
.elseif eax<=4 ; is it Windows NT?
.endif
```

Non `NT`-based operating systems could be detected by analysing the additional data area behind `TEB`, searching the value `00530000h`:

非 `NT` 内核的操作系统可以通过分析 `TEB` 附加数据，搜索 `00530000h`：

```
mov edx,00530000h ; the value to search
mov eax,fs:[18h] ; get the TEB base address
mov ebx,[eax+58h] ; TEB-base + 58h (W95)
mov ecx,[eax+7Ch] ; TEB-base + 7Ch (WME)
mov eax,[eax+54h] ; TEB-base + 54h (W98)

.if ebx==edx ; is it Windows 95?
.elseif eax==edx ; is it Windows 98?
.elseif ecx==edx ; is it Windows ME?
.endif
```

### V. Conclusions

### V. 结论

Resolving the operating system by using this technique is only one possibility to avoid the usage of `Advanced Programming Interface` functions. Other functions, for example `GetCommandLine[1]`, `IsDebuggerPresent[1]` or named functions in this essay could be ”rewritten” in the same way. In other words: the reverse engineer isn’t able to set breakpoints on `API` function calls, because they didn’t exist. And mixing different operating system solutions together makes life even harder for him.

采用这个技术，不过是避免使用 `API` 函数一种方法。其他很多函数，例如 `GetCommandLine[1]`, `IsDebuggerPresent[1]` 或者这篇文章中的一些函数都可以采用同样的方法被重写。换句话说：逆向人员不会拦截到相关 `API` 函数，因为根本就没有使用他们。并且，如果把对不同操作系统的处理都混杂在一起，对他们分析来说，会更加困难。

### VI. Appendix

### VI. 附录

```
.const
;-- return values from OS_GetOS
OS_UNKNOWN equ -1
OS_WIN95 equ 1
OS_WIN98 equ 2
OS_WINME equ 3
OS_WINNT equ 4
OS_WIN2K equ 5
OS_WINXP equ 6
OS_WIN2K3 equ 7

.code
OS_GetOS proc
local _theReturnValue:DWORD
pushad ; store all registers
mov _theReturnValue,OS_UNKNOWN
assume fs:nothing
mov ebx,fs:[18h] ; get self pointer from TEB
mov eax,fs:[30h] ; get pointer to PEB / database

.if eax==7FFDF000h && ebx==7FFDE000h ; WinNT based
mov ebx,[eax+0A8h] ; get OSMinorVersion
mov eax,[eax+0A4h] ; get OSMajorVersion

.if eax==5 && ebx==0 ; is it Windows 2000?
mov _theReturnValue,OS_WIN2K
.elseif eax==5 && ebx==1 ; is it Windows XP?
mov _theReturnValue,OS_WINXP
.elseif eax==5 && ebx==2 ; is it Windows 2003?
mov _theReturnValue,OS_WIN2K3
.elseif eax<=4 ; is it Windows NT?
mov _theReturnValue,OS_WINNT
.endif
.else ; Win9X based
mov edx,00530000h ; the magic value to search
mov eax,fs:[18h] ; get the TEB base address
mov ebx,[eax+58h] ; TEB-base + 58h (W95)
mov ecx,[eax+7Ch] ; TEB-base + 7Ch (WME)
mov eax,[eax+54h] ; TEB-base + 54h (W98)

.if ebx==edx ; is it Windows 95?
mov _theReturnValue,OS_WIN95
.elseif eax==edx ; is it Windows 98?
mov _theReturnValue,OS_WIN98
.elseif ecx==edx ; is it Windows ME?
mov _theReturnValue,OS_WINME
.endif
.endif ; of base check NT/9X

popad ; restore all registers
mov eax,_theReturnValue
ret ; return to caller
OS_GetOS endp
```
