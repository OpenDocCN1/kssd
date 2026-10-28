
```asm
\_KiSystemService proc
ENTER\_SYSCALL kss\_a, kss\_t ; set up trap frame and save state
?FpoValue = 0
;
; (eax) = Service number
; (edx) = Callers stack pointer
; (esi) = Current thread address
;
; All other registers have been saved and are free.
;
; Check if the service number within valid range
;
\_KiSystemServiceRepeat:
mov edi, eax ; copy system service number
;edi = eax 等于了系统调用号
shr edi, SERVICE\_TABLE\_SHIFT ; isolate service table number
and edi, SERVICE\_TABLE\_MASK ;
; 这里用来确定是去找系统服务表还是驱动的表
mov ecx, edi ; save service table number
add edi, [esi]+ThServiceTable ; compute service descriptor address
; 指向系统服务表
mov ebx, eax ; save system service number
; ebx位系统调用号
and eax, SERVICE\_NUMBER\_MASK ; isolate service table offset
; eax为系统服务下标
;
; If the specified system service number is not within range, then attempt
; to convert the thread to a GUI thread and retry the service dispatch.
;
cmp eax, [edi]+SdLimit ; check if valid service
jae Kss\_ErrorHandler ; if ae, try to convert to GUI thread
; 检查系统调用的下标是否超过了表的大小
;
; If the service is a GUI service and the GDI user batch queue is not empty,
; then call the appropriate service to flush the user batch.
;
cmp ecx, SERVICE\_TABLE\_TEST ; test if GUI service
; 跳转GUI API服务
jne short Kss40 ; if ne, not GUI service
mov ecx, PCR[PcTeb] ; get current thread TEB address
; ecx指向KPCR
xor ebx, ebx ; get number of batched GDI calls
KiSystemServiceAccessTeb:
or ebx, [ecx]+TbGdiBatchCount ; may cause an inpage exception
jz short Kss40 ; if z, no batched calls
push edx ; save address of user arguments
push eax ; save service number
call [\_KeGdiFlushUserBatch] ; flush GDI user batch
pop eax ; restore service number
pop edx ; restore address of user arguments
;
; The arguments are passed on the stack. Therefore they always need to get
; copied since additional space has been allocated on the stack for the
; machine state frame. Note that we don't check for the zero argument case -
; copy is always done regardless of the number of arguments because the
; zero argument case is very rare.
;
Kss40: inc dword ptr PCR[PcPrcbData+PbSystemCalls] ; system calls
; 直接跳转到这里进行++系统调用数
FPOFRAME ?FpoValue, 0
mov esi, edx ; (esi)->User arguments
; 3环的指针
mov ebx, [edi]+SdNumber ; get argument table address
; 系统服务表
xor ecx, ecx
mov cl, byte ptr [ebx+eax] ; (ecx) = argument size
mov edi, [edi]+SdBase ; get service table address
mov ebx, [edi+eax\*4] ; (ebx)-> service routine
; 指向的函数
sub esp, ecx ; allocate space for arguments
shr ecx, 2 ; (ecx) = number of argument DWORDs
mov edi, esp ; (es:edi)->location to receive 1st arg
cmp esi, \_MmUserProbeAddress ; check if user address
jae kss80 ; if ae, then not user address
KiSystemServiceCopyArguments:
rep movsd ; copy the arguments to top of stack.
; Since we usually copy more than 3
; arguments. rep movsd is faster than
; mov instructions.
; 复制参数
;
; Make actual call to system service
;
kssdoit:
CAPSTARTX <\_KiSystemService,ebx>
call ebx ; call system service
; call 系统服务
CAPENDX <\_KiSystemService>
kss60:
```

两张表一张是找内核函数，另一个是驱动函数，那么我们看的`eax`里存一个值，系统调用号或者服务号，这个东西的低12位就是函数参数表和函数地址表的下标，而第13位如果是0，表示找系统服务表，如果是1，那么找驱动函数的的表

![image-20210812184030416](upload/attach/202109/873515\_EHU3G27KK2M7PXU.png)

总结来说就是`KiSystemService`会根据服务ID从`SSDT`中找调用的服务地址和参数描述，然后将参数从用户态复制到线程的内核态，然后调用真正的函数执行操作，操作结束后返回给`KiSystemService`，该函数会把结果返回到用户态，在进行系统调用的时候，CPU需要从内存中加载门描述符和段描述符才可以得到`KiSystemServer`的地址，然后进行权限检查，检查源代码位置和目标代码位置的代码段的权限

#### 快速系统调用

系统调用是非常繁琐的事情，那么肯定想要减少这些开销的啦！

- 系统调用服务例程的地址放到寄存器中来避免读`IDT`这样的内存操作（??因为`IDT` 是`INT xxx IDT`表的东西），因为读寄存器的速度比读内存的速度要快很多

- 避免权限检查，也就是使用特殊的指令让CPU省去对系统服务调用来说根本不需要的权限检查

- 奔腾 II 处理器引入的`SYSENTER`/`SYSEXIT` 指令正是按照这一思路设计的，AMD K7 引入的 `SYSCALL`/`SYSRETURN`指令 都是因为这个速度的目的来设计的，对比`INT 2E`来说，使用这些指令可以加快系统调用的速度，因此利用这些指令来进行系统调用的方式叫做快速系统调用

因为Windows 2000 或者之前的Windows的操作系统不支持快速系统调用，他们只能使用`INT 2E`的方式来进行系统调用，这时候我们需要去小路哥哥那里白嫖一个windows XP！???♂?白嫖回来！

![image-20210817120007616](upload/attach/202109/873515\_JANRJV63PC6T7D5.png)

这让人兴奋的界面！！！??直接来了把扫雷

![image-20210817120130819](upload/attach/202109/873515\_J3E9XN4P3NK4VQN.png)

稳了！继续学习！

在Windows XP和Windows Server 2003或更高的版本在启动的过程中会通过`CPUID` 指令检测CPU是否支持快速系统调用指令（`EDX`寄存器的`SEP`标志位），`EDX`的11位代表了`SEP`标志位，来代表是否支持快速系统调用，如果系统不支持就还会使用`INT 2E`的方式

进行快速系统调用需要做的准备：

- 在`GDT`表中建立4个段描述符，用来描述供`SYSENTER`指令进入内核模式时使用的代码段(`CS`)和栈段(`SS`)，以及`SYSEXIT`指令从内核模式返回用户模式时使用的代码段和栈段，在`GDT`中他们的排列方式应该严格按照上述的方式进行排列

- 设置专门用于系统调用的MSR寄存器，`SYSENTER_EIP_MSR`用于指定新的程序指针，是`SYSENTER`指令要跳转的目标例程地址。Windows系统会将其设置为`KiFastCallEntry`的地址，因为该例程是Windows内核中专门用于受理快速系统调用的，`SYSENTER_CS_MSR`用来指定新的代码段，也就是`KiFastCallEntry`所在的代码段，`SYSENTER_ESP_MSR`用于指定真的栈指针(`ESP`)

| MSR Name | MSR Add | Useful |
| ---------------- | ------- | -------------------- |
| `SYSENTER_CS_MSR` | 174 | 目标代码段的CS选择子 |
| `SYSENTER_ESP_MSR` | 175 | 目标ESP |
| `SYSENTER_EIP_MSR` | 176 | 目标EIP |

该地址用于Kernel debug时，通过`rdmsr`/`wrmsr`指令来读/写这3个寄存器

![image-20210817131814717](upload/attach/202109/873515\_6WER8ZU5UF4X8Q9.png)

174的位置我们应该不陌生了，因为0环的时候切换寄存器看到的`cs`经常是08，那么我们看一下`EIP`的位置

![image-20210817132005018](upload/attach/202109/873515\_73DCZEBPKTR4UZW.png)

可以看到是`nt!KiFastCallEntry`的地址

- 会将一小段名为`SystemCallStub`的代码复制到`SharedUserData`的内存区，内存区会被映射到每个`Win32`的进程中，这样每次进行系统调用的时候，`NTDll.DLL`中的stub函数就会调用这段`SystemCallstub`代码，该代码根据系统硬件的不同而不同，对于IA-32就会使用`SYSENTER`，对于AMD就会使用`SYSCALL`指令

???♂?打开windbg开始 看一下ntdll中的`ReadFile`函数！这里碰到了点坑，就是符号老加载不上，如果想去看ntdll的东西，需要切换一下进程

![image-20210817150550703](upload/attach/202109/873515\_3F6CDF2BTDETEXF.png)

可以很清楚的看到 `SharedUserData!SystemCallstub` 的地址给了`edx`寄存器，Call的是`edx`中的地址

![image-20210817150754208](upload/attach/202109/873515\_V6ZJBSH5K4PUZX5.png)

将当前的`esp`的寄存器放入到`edx`寄存器当中 ( 个人认为这里应该是要去获取r3层的参数所以到时候可以通过`edx`寄存器来取参数，因为`esp`需要修改 )，因为`sysenter`要调用`KiFastCallEntry`，所以去看看！

```asm
8053e540 b923000000 mov ecx,23h
8053e545 6a30 push 30h
8053e547 0fa1 pop fs
8053e549 8ed9 mov ds,cx
8053e54b 8ec1 mov es,cx
```

在 `KPCR`（Processor Cotnrol Region）区域的 +0x40 位置是 `TSS` 指针（指向一个 `KTSS` 结构），`KPCR` 结构的地址在`0xffdff000`：

![image-20210817162344025](upload/attach/202109/873515\_MA7C2UCJBNWSFNM.png)

`0x80042000 _KTSS`为TSS的指针，所以`KTSS`的结构为：

![image-20210817162557752](upload/attach/202109/873515\_UM5VQ4JUTKNMWX7.png)

0级的`Esp`值，这指向一个 `KTRAP_FRAME` 结构 `V86Es` 成员

```asm
8053e54d 8b0d40f0dfff mov ecx,dword ptr ds:[0FFDFF040h]
8053e553 8b6104 mov esp,dword ptr [ecx+4]
8053e556 6a23 push 23h
```

这个！`ffdff040`就是`KPCR` `TSS`指针所指向的`KTSS`的指针，所以他将这个值给了`ecx`，再通过`ecx+4`获得`ESP0`的地址，因为当前的`esp`所在的是`KTRAP_FRAME`的`V86Es`，所以push 23修改了`V86Es`上层成员`HardwareSegSs`

##### KTRAP_FRAME 结构

在 `KiFastCallEntry()` 中将 context 信息保存在一个被称为 `KTRAP_FRAME` 的结构里，因为我们已经获取到了`ESP0`的地址，`KTRAP_FRAME`的基址为 `Esp0 - 0x7c`

![image-20210817164718816](upload/attach/202109/873515\_KJMJ2MRQGJ7QW3T.png)

在 `KiFastCallEntry()` 中将 context 信息保存在一个被称为 `KTRAP_FRAME` 的结构里，在前面我们看到 `KTRAP_FRAME` 结构的地址被赋予 `esp` 寄存器，因此：`KTRAP_FRAME` 结构就是 `KiFastCallEntry()` 函数的 stack 区域

![image-20210818110755176](upload/attach/202109/873515\_9AD5KCQE8D5UT89.png)

开始逆！
