
```
SYM_CODE_START(secondary_startup_64)

        UNWIND_HINT_END_OF_STACK

        ANNOTATE_NOENDBR

        ......

1:      cmpl    %edx, %eax
        je      1f
        xor     %edx, %edx
        wrmsr                           /* Make changes effective */
1:

        /* Setup cr0 */
        movl    $CR0_STATE, %eax
        /* Make changes effective */
        movq    %rax, %cr0

        /* zero EFLAGS after setting rsp */
        pushq $0
        popfq

        /* Pass the boot_params pointer as first argument */
        movq    %r15, %rdi

.Ljump_to_C_code:
        xorl    %ebp, %ebp      # clear frame pointer
        ANNOTATE_RETPOLINE_SAFE
        callq   *initial_code(%rip)
        ud2
SYM_CODE_END(secondary_startup_64)
```

#### 流程图
