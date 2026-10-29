
```
//./libgcc/unwind-dw2.c

/*

    此函数调用前, context代表栈回溯过程中已经计算出的某个栈帧对应函数的各个寄存器/CFA的值(指针), 其中context->ra记录的是其返回到caller函数的地址.

    此函数负责根据caller执行到context->ra时候的各个寄存器计算方法(在参数fs中,来自CIE/FDE的解析结果) 更新context,

    更新后的context' 记录的则是caller在执行到context->ra时CFA/各个寄存器的值(指针), context'->ra指向caller的父函数.

*/

static void uw_update_context (struct _Unwind_Context *context, _Unwind_FrameState *fs)

{

  uw_update_context_1 (context, fs);

 

  /* context->ra指向 caller的返回地址(其父函数中某地址), 若返回地址未定义,则代表栈回溯结束 */

  if (fs->regs.reg[DWARF_REG_TO_UNWIND_COLUMN (fs->retaddr_column)].how == REG_UNDEFINED)

    context->ra = 0;

  else

  {

      /* 设置context->ra指向caller的返回地址 */

    context->ra = __builtin_extract_return_addr(_Unwind_GetPtr (context, fs->retaddr_column));

    .......

  }

}

 

/* 根据fs中的计算规则,更新context中CFA/各个寄存器的值(指针),更新后context中则记录caller调用callee时各个寄存器的状态 */

static void uw_update_context_1 (struct _Unwind_Context *context, _Unwind_FrameState *fs)

{

  struct _Unwind_Context orig_context = *context;        /* 先复制一份context, 以确保修改context时可以引用callee栈帧的计算结果 */

  void *cfa;

  long i;

  ......

  _Unwind_SpTmp tmp_sp;

 

  /* 在如aarch64平台中, CIE/FDE中没有sp的规则,在栈回溯过程中context->reg[sp]总是未定义的, 对于这些平台需要手动设置 SP = context->cfa */

  if (!_Unwind_GetGRPtr (&orig_context, __builtin_dwarf_sp_column ()))

    _Unwind_SetSpColumn (&orig_context, context->cfa, &tmp_sp);

 

  /* SP的值不应该继承自上一个栈帧,这没有意义，故这里主动清空 */

  _Unwind_SetGRPtr (context, __builtin_dwarf_sp_column (), NULL);

 

  switch (fs->regs.cfa_how)        /* 先更新CFA */

    {

    case CFA_REG_OFFSET:           /* CFA = reg + offset, reg默认来自sp(=callee的CFA) */

      cfa = _Unwind_GetPtr (&orig_context, fs->regs.cfa_reg);

      cfa += fs->regs.cfa_offset;

      break;

    case CFA_EXP:

    {

      .......

      execute_stack_op (exp, exp + len, &orig_context, 0);    /* 若CFA是表达式,则通过此函数解析表达式 */

      break;

    }

    default:

      gcc_unreachable ();

    }

 

  context->cfa = cfa;               /* caller的CFA已经计算出来,保存到context->cfa中; caller的其他寄存器的计算依赖于此CFA */

 

  /* 遍历所有寄存器的计算方法,最终确定caller各个寄存器的值/指针 */

  for (i = 0; i < __LIBGCC_DWARF_FRAME_REGISTERS__ + 1; ++i)

    switch (fs->regs.reg[i].how)

  {

      case REG_UNSAVED:            /* same/未定义寄存器保持和callee中结果一致 */

      case REG_UNDEFINED:

        break;

 

      case REG_SAVED_OFFSET:       //reg[i] = *(cfa + reg.offset)

        _Unwind_SetGRPtr (context, i,(void *) (cfa + fs->regs.reg[i].loc.offset));

        break;

 

      case REG_SAVED_REG:

        ........

  }

  ......

}
```

# 九、基于DWARF的栈回溯(6)—libunwind与libgcc在保存上下文时的区别

虽然都满足IA-64 C++ABI标准，但不同库对基于DWARF的unwind的实现是略有不同, 如libunwind/libgcc在context上下文初始化时的实现就有所区别,同样是`_Unwind_Backtrace`函数:

- 在libunwind中`_Unwind_Backtrace`函数入口最开始就会通过汇编代码将所有硬件寄存器的当前值保存到context中,所以使用libunwind栈回溯时其回调函数中总是可以通过调用`_Unwind_GetGR`获取任何一个寄存器的值,测试输出如下(见附录中的代码std_callback):

```
Unwind Frame(0): CFA:00000040007ffe10, SP:00000040007ffe10, RA:000000000023a7a8                                                                                    

R00:00000040007ff950,R01:00000000002ae8e8,R02:00000040007fffc8,R03:000000000023a770,

R04:00000040007ffe30,R05:b7591a76500cc58e,R06:00000000002afcf8,R07:0000400000000000,

R08:00000000002ae000,R09:000000000023a624,R10:0000000800000020,R11:0000000000000000,

R12:0000000000000001,R13:0000000000000000,R14:00000000002b1750,R15:0000000000000000,

R16:0000000000000000,R17:0000000000000000,R18:0000000000000000,R19:0000000000241860,

R20:0000000000241924,R21:0000000000000000,R22:0000000000200338,R23:00000000002ae8a0,

R24:0000000000000018,R25:0000000000200338,R26:00000000002b0000,R27:00000000002b0000,

R28:0000000000000000,R29:00000040007ffe20,R30:0000000000240b44,R31:00000040007ffe10,



Unwind Frame(1): CFA:00000040007ffe30, SP:00000040007ffe30, RA:000000000024128c

R00:00000040007ff950,R01:00000000002ae8e8,R02:00000040007fffc8,R03:000000000023a770,

R04:00000040007ffe30,R05:b7591a76500cc58e,R06:00000000002afcf8,R07:0000400000000000,

R08:00000000002ae000,R09:000000000023a624,R10:0000000800000020,R11:0000000000000000,

R12:0000000000000001,R13:0000000000000000,R14:00000000002b1750,R15:0000000000000000,

R16:0000000000000000,R17:0000000000000000,R18:0000000000000000,R19:0000000000241860,

R20:0000000000241924,R21:0000000000000000,R22:0000000000200338,R23:00000000002ae8a0,

R24:0000000000000018,R25:0000000000200338,R26:00000000002b0000,R27:00000000002b0000,

R28:0000000000000000,R29:00000040007ffe30,R30:0000000000240b44,R31:00000040007ffe30,
```

- 在libgcc中`_Unwind_Backtrace`通过调用一个子函数`uw_init_context`配合`_Unwind_Backtrace`自身的DWARF信息完成context初始化, 此时只有callee-saved/x29,x30寄存器被初始化了,调用`_Unwind_GetGR`获取非callee-saved寄存器可能会导致crash(见附录测试函数中gcc_callback).

**实际上libgcc的实现并没有问题,只是在IA-64 ABI下的接口不太友好**(`_Unwind_GetGR`的crash无法预测,为确保不crash只能根据AAPCS64标准选择只打印callee-saved寄存器)。

因为打印非callee-saved寄存器通常没有意义, 这些寄存器在函数调用过程中随时都可能被修改, 若某函数没有将其保存到栈中那么即使context中有值通常也是错误的。如上面libunwind测试结果中Unwind Frame 0/1中的通用寄存器的值完全相同，但并不复合实际情况(其每次输出的都只是`_Unwind_Backtrace`时这些寄存器的值).

**但libgcc的实现对一些指令的解析存在影响,**如Shadow Call Stack需要在在异常处理之前插入".cfi_escape 0x16, 0x12, 0x02, 0x82, 0x78" 指令(即x18=x18-8), 此指令在libunwind中可以正常运行,但在libgcc中就会由于x18寄存器未初始化而crash[10]

# 参考资料:

[1] [Stack Computers: 1.2 WHAT IS A STACK?](https://bbs.kanxue.com/elink@6f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2M7$3g2J5M7#2)9J5k6h3g2U0k6g2)9J5k6h3y4E0N6g2)9J5k6h3g2V1N6g2)9J5c8W2)9%4c8h3E0G2L8%4m8E0j5h3&6Q4x3V1k6K6N6r3q4U0K9#2)9#2k6X3y4G2L8i4m8#2N6r3g2J5M7#2)9J5c8Y4y4W2j5K6q4Q4y4h3j5J5i4K6u0W2K9s2c8E0L8l9`.`.)

[2] <Using The GNU Compiler Collection>, `__builtin_frame_address`的定义

[3] [Download DWARF Standards](https://bbs.kanxue.com/elink@eb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6$3q4J5k6Y4y4@1k6q4)9J5k6h3!0J5k6#2)9J5c8V1c8G2N6$3&6D9L8$3q4V1i4K6u0W2M7r3S2H3)

[4] [Dwarf Home](https://bbs.kanxue.com/elink@5c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6$3q4J5k6Y4y4@1k6q4)9J5k6h3!0J5k6#2)9J5c8R3`.`.)

[5] [Exception Frames](https://bbs.kanxue.com/elink@fdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3k6K6M7r3g2U0M7#2)9J5k6h3I4A6L8Y4g2^5k6X3!0#2L8X3c8S2N6r3W2G2L8W2)9J5k6h3!0J5k6#2)9J5c8V1I4e0b7W2)9#2k6U0c8Q4x3X3f1I4i4K6u0W2x3q4)9J5c8V1I4e0b7W2)9J5k6p5y4G2M7X3g2Q4x3X3c8Y4k6h3&6W2M7X3W2U0i4K6u0r3e0q4y4n7i4K6u0V1b7$3!0J5k6g2)9J5k6r3N6W2L8X3g2J5K9h3y4Q4x3V1k6W2K9r3k6J5j5h3#2W2j5$3S2H3N6q4)9J5k6h3S2@1L8h3H3`.)

[6] [assembly - Why GCC compiled C program needs .eh_frame section? - Stack Overflow](https://bbs.kanxue.com/elink@e05K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3U0j5K6x3o6l9^5x3e0W2Q4x3V1k6%4K9s2W2Q4x3X3c8Y4j5$3y4Q4x3X3c8U0L8$3#2H3K9h3I4W2k6q4)9J5k6r3y4Q4x3X3c8H3M7X3!0Y4M7X3q4E0i4K6u0V1L8X3g2W2k6s2y4Q4x3X3c8W2K9q4)9J5k6r3k6J5j5h3#2W2i4K6u0V1M7$3g2U0N6r3W2G2L8R3`.`.)

[7] [C++ ABI for Itanium: Exception Handling](https://bbs.kanxue.com/elink@d2eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6N6r3q4F1K9i4g2E0i4K6u0V1j5%4S2^5i4K6u0V1j5h3u0A6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6U0P5s2S2Q4x3X3c8S2j5X3W2Q4x3V1k6S2j5X3W2Q4x3X3c8W2K9q4)9J5k6h3S2@1L8h3H3`.)

[8] [`_Unwind_Backtrace`](https://bbs.kanxue.com/elink@0feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3k6K6M7r3g2U0M7#2)9J5k6h3I4A6L8Y4g2^5k6X3!0#2L8X3c8S2N6r3W2G2L8W2)9J5k6h3!0J5k6#2)9J5c8V1I4e0b7W2)9#2k6U0c8Q4x3X3f1I4i4K6u0W2x3q4)9J5c8V1I4e0b7W2)9J5k6p5y4G2M7X3g2Q4x3X3c8Y4k6h3&6W2M7X3W2U0i4K6u0r3e0q4y4n7i4K6u0V1b7$3!0J5k6g2)9J5k6r3N6W2L8X3g2J5K9h3y4Q4x3V1k6T1j5i4y4W2L8r3W2T1i4K6u0V1i4K6u0V1N6h3&6%4K9h3&6V1i4K6u0V1j5X3q4U0K9%4c8J5j5h3y4W2i4K6u0W2K9s2c8E0L8l9`.`.)

[9] [Download DWARF Standards](https://bbs.kanxue.com/elink@d69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1N6$3q4J5k6Y4y4@1k6q4)9J5k6h3!0J5k6#2)9J5c8V1c8G2N6$3&6D9L8$3q4V1i4K6u0W2M7r3S2H3)
