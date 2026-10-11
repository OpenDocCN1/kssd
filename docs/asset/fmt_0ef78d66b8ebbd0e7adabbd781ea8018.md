
Ring3 中这一点的解决方案就是保存每个内存页的信息，这样一旦发生了 `PAGE_GUARD` 异常，我们就可以确定这个 `PAGE_GUARD` 异常发生时，`EIP` 是在一个没有 `PAGE_GUARD` 属性的页中还是在一个标记了 `PAGE_GUARD` 的内存页中。是个不错的主意，但是需要编写更多的代码，还需要重新组织我的 ring3 的内存管理器，增加对每个内存页的保护功能。因为我懒得重新编写，所以我就又为 `KiUserExceptionDispatcher` 添加了一小段代码。

当然也可能 hook `KiTrap0E` 并且在 `context` 结构中返回 `cr2` 寄存器的内容。嗯，不错的想法。`cr2` 寄存器将会保存有错误发生的 `va`（虚拟地址），因为对受保护页的访问只不过是触发异常，我们除了得到 `cr2` 寄存器的值还要确定异常是由于执行产生的还是由于读写产生的。当然，`KiUserExceptionDispatcher` 是不会替我们返回出错地址的，就是因为糟糕的这一点我们必须改进它，使它能返回 `cr2` 寄存器的值：

```
cr2 = faulting_address0 //aal 注：出错地址

eip = faulting_address0 //aal 注：出错地址

exceptioncode = EXCEPTION_GUARD_PAGE //错误代码 = EXCEPTION_GUARD_PAGE
```

爽，登入它，去除这一页的保护，然后等待下次访问。但是如何得到 `cr2` 寄存器的内容呢？？？是啊，这的确是个头痛的问题，但是实际上确实是可以做到的。来看看 `KiTrap0E` 是怎么做的：

```
.text:804DAF25 _KiTrap0E:

.text:804DAF25 mov word ptr [esp+2], 0

.text:804DAF2C push ebp

.text:804DAF2D push ebx

.text:804DAF2E push esi

...

.text:804DB0ED mov ecx, 3

.text:804DB0F2 mov edi, eax

.text:804DB0F4 mov eax, 0C0000006h

.text:804DB0F9 call CommonDispatchException
```

到这里所有的 `KiTrap` 调用将会调用 `CommonDispatchException`，这个函数会将 `CONTEXT` 和 `ERROR_CODE` 的内容保存到堆栈中并且将 `EIP` 重新引导到 `KiUserExceptionDispatcher`。继续跟踪 `CommonDispatchException`，我们来到这里：

```
.text:804D8A8D CommonDispatchException proc near

.text:804D8A8D

.text:804D8A8D

.text:804D8A8D

.text:804D8A8D sub esp, 50h

.text:804D8A90 mov [esp+50h+var_50], eax

.text:804D8A93 xor eax, eax

.text:804D8A95 mov [esp+50h+var_4C], eax

.text:804D8A99 mov [esp+50h+var_48], eax

.text:804D8A9D mov [esp+50h+var_44], ebx

.text:804D8AA1 mov [esp+50h+var_40], ecx

.text:804D8AA5 cmp ecx, 0

.text:804D8AA8 jz short loc_804D8AB6

.text:804D8AAA lea ebx, [esp+50h+var_3C]

.text:804D8AAE mov [ebx], edx

.text:804D8AB0 mov [ebx+4], esi

.text:804D8AB3 mov [ebx+8], edi

.text:804D8AB6

.text:804D8AB6 loc_804D8AB6:

.text:804D8AB6 mov ecx, esp

.text:804D8AB8 test dword ptr [ebp+70h], 20000h

.text:804D8ABF jz short loc_804D8AC8

.text:804D8AC1 mov eax, 0FFFFh

.text:804D8AC6 jmp short loc_804D8ACB

.text:804D8AC8

---------------------------------------------------------------------

.text:804D8AC8

.text:804D8AC8 loc_804D8AC8:

.text:804D8AC8 mov eax, [ebp+6Ch]

.text:804D8ACB

.text:804D8ACB loc_804D8ACB:

.text:804D8ACB and eax, 1

.text:804D8ACE push 1

.text:804D8AD0 push eax

.text:804D8AD1 push ebp

.text:804D8AD2 push 0

.text:804D8AD4 push ecx

.text:804D8AD5 call _KiDispatchException@20

.text:804D8ADA mov esp, ebp

.text:804D8ADC jmp Kei386EoiHelper@0

.text:804D8ADC CommonDispatchException endp
```

好了，因为知道早晚都会调用\_KiDispatchException 所以我们就直接跟到这里了：

```
.text:804F318D ; __stdcall KiDispatchException(x,x,x,x,x)

.text:804F318D _KiDispatchException@20 proc near

.text:804F318D

.text:804F318D push 390h

.text:804F3192 push offset dword_804F3278

.text:804F3197 call __SEH_prolog

.text:804F319C mov eax, ds:___security_cookie

.text:804F31A1 mov [ebp-1Ch], eax

.text:804F31A4 mov esi, [ebp+8]

.text:804F31A7 mov [ebp-2ECh], esi

.text:804F31AD mov ecx, [ebp+0Ch]

.text:804F31B0 mov [ebp-2F0h], ecx

.text:804F31B6 mov ebx, [ebp+10h]

.text:804F31B9 mov [ebp-2F8h], ebx

.text:804F31BF db 3Eh

.text:804F31BF mov eax, ds:0FFDFF020h

.text:804F31C5 inc dword ptr [eax+504h]

.text:804F31CB mov dword ptr [ebp-2E8h], 10017h

.text:804F31D5 cmp byte ptr [ebp+14h], 1

.text:804F31D9 jz loc_804F5A76

.text:804F31DF cmp ds:_KdDebuggerEnabled, 0

.text:804F31E6 jnz loc_804F5A76

.text:804F31EC

.text:804F31EC loc_804F31EC:

.text:804F31EC

.text:804F31EC lea eax, [ebp-2E8h]

.text:804F31F2 push eax

.text:804F31F3 push ecx

.text:804F31F4 push ebx

.text:804F31F5 call _KeContextFromKframes@12

.text:804F31FA mov eax, [esi]

.text:804F31FC cmp eax, 80000003h

.text:804F3201 jnz loc_804F5A20

.text:804F3207 dec dword ptr [ebp-230h]

.text:804F320D

.text:804F320D loc_804F320D:

.text:804F320D

.text:804F320D xor edi, edi

.text:804F320F

.text:804F320F loc_804F320F:

.text:804F320F cmp byte ptr [ebp+14h], 0

.text:804F3213 jnz loc_804F58C3

.text:804F3219 cmp byte ptr [ebp+18h], 1

.text:804F321D jnz loc_80516D98

.text:804F3223 mov eax, ds:_KiDebugRoutine

.text:804F3228 cmp eax, edi

.text:804F322A jz loc_80505721

.text:804F3230 push edi

.text:804F3231 push edi

.text:804F3232 lea ecx, [ebp-2E8h]

.text:804F3238 push ecx

.text:804F3239 push esi

.text:804F323A push dword ptr [ebp-2F0h]

.text:804F3240 push ebx

.text:804F3241 call eax

.text:804F3243 test al, al

.text:804F3245 jz loc_80505721

.text:804F324B

.text:804F324B loc_804F324B:

.text:804F324B push dword ptr [ebp+14h]

.text:804F324E push dword ptr [ebp-2E8h]

.text:804F3254 lea eax, [ebp-2E8h]

.text:804F325A push eax

.text:804F325B push dword ptr [ebp-2F0h]

.text:804F3261 push ebx

.text:804F3262 call _KeContextToKframes@20

.text:804F3267

.text:804F3267 loc_804F3267:

.text:804F3267

.text:804F3267 mov ecx, [ebp-1Ch]

.text:804F326A call @xHalReferenceHandler@4

.text:804F326F call __SEH_epilog

.text:804F3274 retn 14h

.text:804F3274 _KiDispatchException@20 endp ; sp = -14h

.text:804F3274
```

如果不实际的跟踪一下，就看不出这些代码有什么意义，所以我们可以使用 Softice 开始跟踪，很快我们就会发现我们想要的字节：

```
.text:804F5959 call _ProbeForWrite@12

.text:804F595E mov ecx, 0B3h

.text:804F5963 lea esi, [ebp+var_2E8]

.text:804F5969 rep movsd
```

`804F5969` 处的指令负责将 `CONTEXT` 拷贝到用户堆栈，当然是为 `KiUserExceptionDispatcher` 做这个工作。如果我们能够 hook 那条指令，我们当然也就能够保存 `cr2` 寄存器的内容，并将它的值放到 `CONTEXT` 结构的某个域中传回 ring3。这些域就是 `dr0/4` `regs`。如果我们不处理它，我们的非侵入式跟踪器应该清除 `cr2` 的值。

我们将要 hook `lea esi, [ebp+var_2E8]` 处，因为它的大小足够保存 hook-也就是 push/ret 组合，注意千万不要 hook `rep movsd` 处，否则驱动卸载的时候有可能发生 BSOD（aal 注：Blue Screen of Death，蓝屏死机）的错误。只是有可能发生，正像 Mark Russinovich 所描述的，当驱动的地址空间中的代码在执行时，如果打断了某个线程，而我们在这个时候卸载驱动，当被中断的线程打算再次执行时就会返回到空内存而你会得到一个内存页错误（`PAGE_FAULT`），看到的就是 BSOD。我还没有遇到过这种情况，但是这的确是有可能发生的。只要你遭遇到 BSOD 却一点也不知道是什么原因造成的时候，脑子中想到这一点就行。

同样我们必须标识我们的进程，完成这一点，我使用的是 `cr3` 技巧，因为 `cr3` 中存储了 `PDE` 的物理地址，因为每个进程都有自己的内存空间，那样我们就能毫无问题的标识进程了 \[11\]。注意我使用的 `KeStackAttachProcess` 是不必要的， 我们可以直接在 `KPROCESS` 结构(`EPROCESS` 的一部分)中得到 `cr3` 的值：

```
kd> dt nt!_KPROCESS

+0x000 Header : _DISPATCHER_HEADER

+0x010 ProfileListHead : _LIST_ENTRY

+0x018 DirectoryTableBase : [2] Uint4B <-- 就是这里了

+0x020 LdtDescriptor : _KGDTENTRY
```

`ntoskrnl.exe` 基址的偏移地址 `25963h` 处，我们现在就可以准备完成这个技巧了。开始干吧！！！！你必须在你自己的 `ntoskrnl.exe` 中看看这个偏移地址，因为这个偏移地址可能会有所不同。

另外还有一种条件非常非常非常重要，这就是 `cr2` 的保存!!!

`cr2` 的值没有被保存，如果其它的内存页错误，`cr2` 就会保存那个错误地址。 这对我们来说使非常危险的，因为内存页错误的处理程序同样也用来将内存页存储在页面文件中。有一种存储 `cr2` 的值的方法就是 hook `int 0eh`，它能够在当且仅当 `cr2` 的值属于我们的进程时存储它。接下来的代码中，我们的 hook 将会得到已经存储了的 `cr2` 的值，并且将它的值放到 `context.context_dr0` 中传给我们的代码。

```
hookint0eh label dword

push eax

mov eax, cr3

cmp eax, c_cr3

jne __exit_int0e

mov eax, cr2

mov c_cr2, eax

__exit_int0e:

pop eax

jmp cs:[oldint0eh]

;将 cr2 的内容保存在 context.context_dr0 中

hookmycode label dword

push eax

mov eax, cr3

cmp eax, c_cr3

jne __exithook

lea esi, [ebp-2e8h]

mov eax, c_cr2

mov [esi.context_dr0], eax

__exithook:

pop eax

lea esi, [ebp-2e8h]

retaddr:

push 0deac0deh

ret
```

现在在你的非侵入式跟踪器中你只要测试是否 `EIP = DR0`，如果相等的话我们就登入访问，而如果不是我们就去除 `PAGE_GUARD` 然后在错误指令后设置 `int 3h`。我们有改进后的 `KiUserExceptionDispatcher` 替我们作我们想做的事。

同样这也将帮助我们分别访问我们的内存页时产生的 `ACCESS_VIOLATION` 时我们是不是在使用 `PAGE_NOACCESS` 而不是在使用 `PAGE_GUARD`。

### 4.4. 进入内存区域并访问

一旦我们知道了如何解决问题，我们就得进入我们的内存页面。这可以通过向文件进行写入操作来实现，或者更好一点的方法是使用 www.sysinternals.com 的 `OutputDebugStringA` 和 `DbgView` 得到输出：

示例代码：

```
__log: lea ecx, [ebp+format5]

    lea ebx, [ebp+buffer]

    push eax

    push ecx

    push ebx

    call [ebp+wsprintfA]

    add esp, 0ch

    push ebx

    call [ebp+OutputDebugStringA] ;登入 访问

    ...

    format5 db "eip log : 0x%.08X", 13, 10, 0
```
