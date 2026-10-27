
- 这个字段用于对外部中断和NMI的处理进行相关的配置。在有需要对外部中断进行监控的时候可以使用。但是我们的目的是对一些软件的行为进行监控，并不需要对外部中断进行监控，因此这里把这个字段的可选位均填充为0，都不进行拦截即可。

  ![image-20220317200829803](https://bbs.kanxue.com/upload/attach/202203/899076_HV9FEAYBB5X6PU4.jpg)

### `processor-based vm-execution control` 字段

- 这个字段用来对一些软件的行为进行拦截。对一些特定的指令、一些特定寄存器的读写进行拦截。因为我们的目的是通过VT技术对一些行为进行监控，因此这个字段是我们需要重点关注的部分。详细解析可以查看《处理器虚拟化技术》的3.5.1章节。

- 这个字段第31位如果置为1，可以激活另一个 `SECONDARY_VM_EXEC_CONTROL` 寄存器。这个寄存器可以设置更多拦截的操作。

  ![image-20220317203750490](https://bbs.kanxue.com/upload/attach/202203/899076_UGRA47EWV8C3PAB.jpg)

- 本篇文章中暂时不对这些操作进行任何拦截，在之后的文章中有需要时再进行相应的拦截。

```cpp
void VmxInitControls()
{
    ULONG64 vmxBasic = __readmsr(IA32_VMX_BASIC);
    ULONG64 mseregister = ((vmxBasic >> 55) & 1) ? IA32_MSR_VMX_TRUE_PINBASED_CTLS : IA32_MSR_VMX_PINBASED_CTLS;
    ULONG64 value = VmxAdjustContorls(0, mseregister);
    __vmx_vmwrite(PIN_BASED_VM_EXEC_CONTROL, value);
    mseregister = ((vmxBasic >> 55) & 1) ? IA32_MSR_VMX_TRUE_PROCBASED_CTLS : IA32_MSR_VMX_PROCBASED_CTLS;
    value = VmxAdjustContorls(0, mseregister);
    __vmx_vmwrite(CPU_BASED_VM_EXEC_CONTROL, value);
    /*
    //扩展部分
    mseregister = IA32_MSR_VMX_PROCBASED_CTLS2;
    value = VmxAdjustContorls(0, mseregister);
    __vmx_vmwrite(SECONDARY_VM_EXEC_CONTROL, value);
    */
}
```

# 对vm-exit处理函数进行框架搭建

- 在之前`vmcs`的填写过程中，vmexit事件发生之后回到host中之后的`rip`被设置成了vm-exit处理函数的地址。在vmexit事件发生回到host之后自动从这个处理函数开始跑。这个函数还未被实现。这里将其简略实现一下，为下一篇文章的vmexit处理打好基础。

- 首先这个函数开始一定要保存所有的寄存器，并在返回虚拟机之前恢复所有的寄存器。否则退出虚拟机之前寄存器中的内容和返回虚拟机之后寄存器中的内容不一样的话一定会导致不可预知的结果。因此这个函数一定得是汇编写的裸函数。在将寄存器进行相应的把保存之后跳到C语言写的函数中继续执行。在C语言写的函数中完成对vmexit消息的处理并返回之后，再恢复之前保存的所有寄存器，然后通过`vmresume`指令恢复虚拟机的执行，重新跳回到虚拟机之中。具体汇编实现如下

```asm
AsmVmxExitHandler proc
    push r15;
    push r14;
    push r13;
    push r12;
    push r11;
    push r10;
    push r9;
    push r8;
    push rdi;
    push rsi;
    push rbp;
    push rsp;
    push rbx;
    push rdx;
    push rcx;
    push rax;

    mov rcx,rsp;
    sub rsp,0100h
    call VmxExitHandler
    add rsp,0100h;

    pop rax;
    pop rcx;
    pop rdx;
    pop rbx;
    pop rsp;
    pop rbp;
    pop rsi;
    pop rdi;
    pop r8;
    pop r9;
    pop r10;
    pop r11;
    pop r12;
    pop r13;
    pop r14;
    pop r15;
    vmresume
    ret
AsmVmxExitHandler endp
```

- 至于`vmxExitHandler`这个C语言的函数中应该如何进行vmexit事件的处理以及应该如何通过设置`processor-based vm-execution control`字段对特定事件进行拦截，将会在下一篇文章中会进行详细的讲解。

- **本篇文章对应的代码晚些时候会传到github仓库中。**

# 参考文献

1. intel白皮书

2. 邓志《处理器虚拟化技术》

3. B站周壑VT教学视频

4. a8dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4M7e0p5H3y4o6f1#2y4e0p5H3y4K6m8Q4x3V1k6h3N6q4c8G2e0h3f1`.

5. 火哥上课讲的内容（这里帮火哥打个广告，火哥qq471194425，上课会讲很多干货，也会在讲解技术的时候同时讲解一些运用的方法，报火哥的班绝对是物超所值）

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2022-3-18 16:39 被smallzhong_编辑，原因：

- [#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)
- [#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
- [#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)
- [#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)
- [#虚拟化](https://bbs.kanxue.com/forum-41-1-136.htm)
