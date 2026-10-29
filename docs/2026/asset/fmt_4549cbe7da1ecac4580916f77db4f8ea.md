
```
//libgcc/unwind-dw2.c
/*
此函数需要`noinline`属性以确保其不会在其`caller`函数中展开,否则无法确定`__builtin_return_address(0)`获取的是哪一级`caller`的返回地址.

`context`: 一个要初始化的`_Unwind_Context`上下文的指针

`outer_cfa`: 是当前函数`caller`的`CFA`, 在`caller`函数可通过调用`__builtin_dwarf_cfa()`函数传入

`outer_ra`: 是当前函数`caller`的返回地址, 在`caller`函数中可通过调用`__builtin_return_address(0)`函数传入

此函数的作用是通过基于`DWARF`的栈回溯为其父函数初始化栈回溯的上下文`context`,返回的`context`中记录着父函数调用 `uw_init_context_1`之前`CFA`和各个寄存器的值/指针.

需要注意的是,此函数的实现只依赖`.eh_frame`中的`CIE`/`FDE`信息和三个硬件寄存器的值:

* `uw_init_context_1`返回到其`caller`(这里也就是`_Unwind_Backtrace`函数)的地址,此地址用来确定其`caller`的`CIE`/`FDE`信息在`.eh_frame`中的位置

* `caller`的`CFA`, 这个值是从`caller`中传来的, 其是计算`caller`其他各个寄存器值的基础, 此`CFA`实际就是`caller`函数入口时的`SP`

* `caller`的返回地址, 这个值也是从`caller`中传来的,用于继续回溯上一级栈帧(实际上这个值不传也可以从栈上获取)

需要注意的是,基于`DWARF`的栈回溯并不能保证所有寄存器都被初始化,只有`caller`中入栈的寄存器可以被初始化,这也是为什么在其`caller`中都要调用 `__builtin_unwind_init` 函数的原因, `__builtin_unwind_init`可以确保`_Unwind_Backtrace`函数的`pro/epilogue`保存了所有`callee-saved register`, 但其他寄存器均没有保存.

所以此函数返回的`context`中存在很多未初始化的寄存器(如`_Unwind_Backtrace`的`context`中`x0-x18`均为初始化), 后续继续栈回溯时除非某一级栈帧主动保存并定义了这些寄存器,否则基于`x0-x18`的`.cfi`指令可能直接导致系统`crash`(可参考后面`SCS`的例子).
*/

static void __attribute__((noinline)) uw_init_context_1 (struct _Unwind_Context *context, void *outer_cfa, void *outer_ra)
{
  _Unwind_FrameState fs;
  _Unwind_SpTmp sp_slot;
  _Unwind_Reason_Code code;

  /* 获取当前函数`uw_init_context_1`的返回地址, 如当前函数`caller`为`_Unwind_Backtrace`,那么此`ra`就是`_Unwind_Backtrace`中的一个地址 */
  void *ra = __builtin_extract_return_addr (__builtin_return_address (0));
    ........

  /* 利用`DWARF`信息, 为`uw_init_context_1`的父函数初始化一个上下文 `context`, `context`中记录父函数调用`uw_init_context_1`时各个`callee-saved`寄存器的值/指针 */
  memset (context, 0, sizeof (struct _Unwind_Context));    /* 初始化`_Unwind_Context`结构体 */

  context->ra = ra;    /* `context->ra`是要回溯的父函数`_Unwind_Backtrace` 调用`uw_init_context_1`处的地址 */

  /* 根据`context->ra`确定其所在函数(`caller`)的`CIE`/`FDE`(这里是`_Unwind_Backtrace`函数), 并解析其中的`.cfi`指令序列,确定`caller`执行到地址`context->ra`时各个寄存器的回溯规则
     此函数不修改`context`结构体,只是将指令解析结果记录到`fs`中, `fs`结构体中记录的是`caller` `CFA`/各个寄存器的计算规则,不涉及具体值/指针. */
  code = uw_frame_state_for (context, &fs);
  .......

  /*
    这里还是以`caller`为`_Unwind_Backtrace`为例, 正常`_Unwind_Backtrace`函数的 `CFA1 = SP1 + offset`; 其中`SP1`是`_Unwind_Backtrace`执行到`uw_init_context_1`时`SP`寄存器的值,
   这种计算并没有错误,但由于传入的`outer_cfa`本身就是`CFA1`了,所以这里将规则修改为 `SP1 = CFA1`; `CFA1 = SP1 + 0`; 再调用`uw_update_context_1`结果也是一样的.
    这里实际上有多种写法, 直接设置`CFA1=SP0`也是可以的(`SP0`可以通过在`uw_init_context_1`函数中调用`__builtin_dwarf_cfa`获取)
  */
  _Unwind_SetSpColumn (context, outer_cfa, &sp_slot);
  fs.regs.cfa_how = CFA_REG_OFFSET;
  fs.regs.cfa_reg = __builtin_dwarf_sp_column ();
  fs.regs.cfa_offset = 0;

  /
  /* 此函数执行前 `context`记录的是`callee`的运行时信息, 此函数根据`caller` `CFA`/各寄存器的计算方法(`fs`),和`callee`的`context`计算出
    `caller`(如这里`_Unwind_Backtrace`函数)的`context'`,最终结果写回到`context`中, 此函数并未更新`caller`的返回地址 */
  uw_update_context_1 (context, &fs);

  /* 下一次要回溯的函数是`_Unwind_Backtrace`的父函数,故这里将`context->ra`设置为`_Unwind_Backtrace`函数的返回地址 */
  context->ra = __builtin_extract_return_addr (outer_ra);
  ......
}
```

`uw_frame_state_for`函数负责解析`caller`的`CIE`/`FDE`,代码如下;
