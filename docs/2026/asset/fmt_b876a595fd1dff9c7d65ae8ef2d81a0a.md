
> Hook1 对应 `Java_com_example_x64_JNI_aal` 函数的入口位置( `0x7fac430b70` )，可以看到前16字节已经被替换掉，新指令为利用 `x16` 寄存器制作的一个跳板（trampoline），其目标为 `0x7face7c600`：

![hoo1_trampoline_analysis1](https://bbs.kanxue.com/upload/attach/202206/772122_YBPNBJUJU9QAJWS.png)

> Hook2 情况与 Hook1 类似，也是生成了16字节的跳板指令（依然使用 `x16` 寄存器）来替换掉 `0xBBA0` 位置的16字节原始指令，在此不做展示

**P.S.** > 在后续多次测试中发现，偶尔也会出现使用单条 Branch 指令（4字节）来替换掉被 Hook 地址的单条指令（4字节）的情况发生，如下图所示

![singal_Branch_inst](https://bbs.kanxue.com/upload/attach/202206/772122_5Y5EYSAFVDRAKMN.png)

> 因为 Branch 指令存在跳转范围（+-128MB），所以 Frida 使用这种形式的 trampoline 需要对被 Hook 地址前后 128MB 范围进行检测，寻找空闲地址，不过这对本文实现一个简单的 inlineHook 模型并无太大影响，故不做深入讨论

**P.P.S.** > 其实 Frida 还有一种跳转范围扩大至 +-4GB 大小的 trampoline 生成规则，在此也不做讨论了，因为在原理上大同小异，单纯属于细节优化问题

> 另外还有不得不提的一点，当 trampoline 使用 `x16` 寄存器作为跳板寄存器时，Hook 结束后 `x16` 寄存器无疑会被污染，然而事实上 Frida 同时使用了 `x16` 与 `x17` 寄存器，那么关于这两个寄存器有什么说法呢？官方对这两个寄存器作用的描述如下：

![x16x17_official_desc](https://bbs.kanxue.com/upload/attach/202206/772122_GETM6JFBJHP3ZEY.png)

> 描述中提到 `x16`、`x17` 寄存器作为内部过程调用中的临时寄存器，结合下图便能更好的理解官方的定义

![x16x17_plt_table](https://bbs.kanxue.com/upload/attach/202206/772122_HXFT5ZEY3XKRE8C.png)

> 关于 trampoline 的研究就到此为止了，接下来我们看他生成的 shellCode

## -- Step3 --

> 接下来我们开始分析 shellcode 部分，以 Hook1 为例

> `Java_com_example_x64_JNI_aal` 函数入口: `0x7fac430b70`

> 入口处 trampoline 汇编代码如下：

![shellcode_analysis1](https://bbs.kanxue.com/upload/attach/202206/772122_SHRSW242SP2SGFF.png)

> 进入 `0x7face7c600` 位置，分析如下图：

![shellcode_analysis2_debugger](https://bbs.kanxue.com/upload/attach/202206/772122_BSYYUZMNFPPCWBV.png)

> 首先 mmap 了一段匿名内存( `7face7c000-7face83000 rwxp` )，在 `0x7face7c600` 位置放置了以下几条汇编指令构成第二段跳板

```
ldr x17, =0x7facec12e0
ldr x16, =0x7face7c000
br x16
```

> 其中 `x17` 寄存器装载了一个地址( `0x7facec12e0` )，这个地址内部保存着 `0x7fac430b70` ，正是 `Java_com_example_x64_JNI_aal` 函数入口地址

> 而 `x16` 寄存器装载了此番生成的 shellCode 的地址( `0x7face7c000` )，将该段内存 dump 下来，拖入 ida 进行分析：

![shellcode1_ida](https://bbs.kanxue.com/upload/attach/202206/772122_X2AD4ZJKKXRE2C3.png)

> 绿色、蓝色部分合并完成了栈平衡、寄存器保护与恢复工作；

> 我们在外部用 JS 编写的 Hook 功能代码( onEnter 部分 )，由 `BLR X4` ( `0x7F7D8D8360` ) 跳转至 `frida-agent-64.so` （见下图）来完成；

> 在 JS 中可以打印，甚至修改函数入参的原因是因为入参（前8个在 X0-X7 寄存器上，后面的在栈上）已全部由绿色块指令压入栈中保存，所以在 `BLR X4` 进行函数调用时，合理设置 X0-X3 寄存器，使其正确的指向栈上某位置尤为关键

![findmem_frida_agent](https://bbs.kanxue.com/upload/attach/202206/772122_QF4SG2KQAZWYURG.png)

> 我们接下来在 shellcode 最后一条 `BR X16` 指令上插入断点，分析函数的运行情况

![brk_br_x16_analysis](https://bbs.kanxue.com/upload/attach/202206/772122_Y943MAFCEB3DN9X.png)

> 当断点触发时 `BR X16` 欲跳转至内存 `0x7face7c630`，其对应的汇编代码如上图所见，其中包含 `Java_com_example_x64_JNI_aal` 函数开头被替换的4条原始指令

> 之后再次使用 `x16` 寄存器跳转至 `0x7fac430b80`，即函数 `Java_com_example_x64_JNI_aal` 开头偏移 `0x10` 的位置，以完成原函数的执行动作

> 此时 `hook1 on enter` 打印完毕，但 `hook1 on leave` 还未打印，所以注意到 `x30` 寄存器中保存的返回地址是 `0x7face7c60c`，即前文中暂未分析的第三段跳板指令，汇编代码如下：

```
ldr x17, =0x7facec12e0
ldr x16, =0x7face7c100
br x16
```

> `x17` 寄存器行为与之前一致，`x16` 寄存器装载了第二段 shellCode 的地址( `0x7face7c100` )，刚才已经一起 dump 下来了，直接在 ida 分析

![shellcode2_ida](https://bbs.kanxue.com/upload/attach/202206/772122_EXG9YZ52V99ZTS4.png)

> 绿色、蓝色部分代码作用不变，由 `BLR X3` ( `0x7F7D8D86C8` ) 跳转至 `frida-agent-64.so` 来完成外部 JS 写的 Hook 功能代码中 onLeave 的部分

> 最后由 `BR X16` 返回 `Java_com_example_x64_JNI_aal` 函数被调用时真正的 LR

> 至此 shellcode 部分也大体分析完毕了，此时我们应该能够写出一款简单的 AArch64 inlineHook 工具模型了

> # --> 2. AArch64 inlineHook 开发

> 结合前文的分析，我们的 inlineHook 应该具备以下这几点功能：

1. Hook 导出函数：即在函数开头进行 Hook ，能够执行原函数，并提供 `onEnter` 以及 `onLeave` 两层代码注入点，达到类似 Frida 那种 "代码托管" 一样的效果

2. Hook 函数内指定地址：Hook 指定位置的汇编指令，仅提供 `onEnter` 一层代码注入点，因为考虑到在指定位置上 X30( LR ) 寄存器可能已经发生变化，此时用该寄存器做返回判断并不准确，故放弃 `onLeave`

3. 在 `onEnter` 中提供入参的打印/修改操作 ( 本质是寄存器/堆栈内存打印/修改操作 )

4. 在 `onLeave` 中提供返回值的打印/修改操作 ( 本质是寄存器/堆栈内存打印/修改操作 )

> OK，有了以上几点需求，我们现在可以开始开发了 ( **源码下载见文章末尾** )

## -- Step1 --

> 我们首先来设计 shellcode 部分

> 在本简易版工具中，我们的跳板指令选择使用 `x16` 寄存器的 16 字节 trampoline ，代码如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `_trampoline_:`  `LDR                 X16, x64code0`  `BR                  X16`  `x64code0:`  `_jmp_addr_:`  `.dword` `0x1111111111111111` |

> 接下需要做参数和返回地址入栈工作，以及全寄存器状态保护，代码如下：

![codding_shellcode_start](https://bbs.kanxue.com/upload/attach/202206/772122_B3WAY9ZQYZYYGS2.png)

> 接下来调用 Hook 功能函数的 `onEnter` 部分，并恢复寄存器及栈状态，最后取出返回地址并返回原函数执行

![codding_shellcode_middle](https://bbs.kanxue.com/upload/attach/202206/772122_P6AYHS4WHSJD38Y.png)

> 对于 onLeave 部分的 shellcode 与之大体类似，就不贴图展示了；

## -- Step2 --

> 接下来开始编写函数完成 inlineHook 的插入

```
```
//## Hook目标函数
extern "C" JNIEXPORT jstring JNICALL
Java_com_cs_inline_MainActivity_stringFromJNI(
JNIEnv* env,
jobject /* thisobj */,
jstring jstr) {
std::string hello = "Hello from C++: ";
hello.append(env->GetStringUTFChars(jstr, nullptr));
return env->NewStringUTF(hello.c_str());
}

//## 该函数内部完成了对Java_com_cs_inline_MainActivity_stringFromJNI函数的inlineHook
extern "C" JNIEXPORT void JNICALL
Java_com_cs_inline_MainActivity_inlineHook1(JNIEnv* env,
jobject /* thisobj */)
{
 //## Hook target函数为：Java_com_cs_inline_MainActivity_stringFromJNI
u_long func_addr = (u_long)Java_com_cs_inline_MainActivity_stringFromJNI;
extern u_long _shellcode_start_, _the_func_addr_, _end_func_addr_, _ori_ins_set1_, _retback_addr_, _shellcode_end_, _trampoline_, _jmp_addr_, _shellcode_part2_;
//## 计算shellcode整体长度
u_long total_len = (u_long)&_shellcode_end_ - (u_long)&_shellcode_start_;
LOGD(ANDROID_LOG_DEBUG, "[+] ShellCode len: %d, target func: %p", total_len, func_addr);

//## 使用mmap分配匿名内存存放shellcode
u_long page_size = getpagesize();
u_long shellcode_mem_start = (u_long)mmap(0, page_size, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_ANONYMOUS | MAP_PRIVATE, 0, 0);
memset((void *)shellcode_mem_start, 0, page_size);
memcpy((void *)shellcode_mem_start, (void *)&_shellcode_start_, total_len);
LOGD(ANDROID_LOG_DEBUG, "[+] shellcode_mem_start: %p", shellcode_mem_start);

//## 设置trampoline跳转的目标地址
*(u_long*)&_jmp_addr_ = shellcode_mem_start;

u_long mem_the_func_addr_ = (u_long)&_the_func_addr_ - (u_long)&_shellcode_start_ + shellcode_mem_start;
u_long mem_end_func_addr_ = (u_long)&_end_func_addr_ - (u_long)&_shellcode_start_ + shellcode_mem_start;
u_long mem_ori_ins_set1_ = (u_long)&_ori_ins_set1_ - (u_long)&_shellcode_start_ + shellcode_mem_start;
u_long mem_retback_addr_ = (u_long)&_retback_addr_ - (u_long)&_shellcode_start_ + shellcode_mem_start;
if(!off_shellcode_part2_)
off_shellcode_part2_ = (u_long)&_shellcode_part2_ - (u_long)&_shellcode_start_;

//## 设置onEnter及onLeave函数
*(u_long*)mem_the_func_addr_ = (u_long)on_enter_1;
*(u_long*)mem_end_func_addr_ = (u_long)on_leave_1;
//## 设置返回地址为距离Hook点0x10长度的指令地址，即偏移为trampoline的长度
*(u_long*)mem_retback_addr_ = (u_long)func_addr + 0x10;

//## 原指令保存，并未做任何解析，PC-relative address相关指令暂不支持
*(u_long*)mem_ori_ins_set1_ = *(u_long*)func_addr;
*(u_long*)(mem_ori_ins_set1_ + 8) = *(u_long*)(func_addr + 8);

//## 页权限修改并完成inlineHook
u_long entry_page_start = (u_long)(func_addr) & (~(page_size-1));
mprotect((u_long*)entry_page_start, page_size, PROT_READ | PROT_WRITE | PROT_EXEC);
*(u_long*)func_addr = *(u_long*)&_trampoline_;
*(u_long*)(func_addr + 8) = *(u_long*)(((u_long)&_trampoline_) + 8);
}
```

> `inlineHook1` 函数主要作用是分配 shellcode 的内存及设置其中的关键数据，并使用 trampoline 替换原指令完成 Hook，函数内注释较为详细，就不做过多解释了

> 最后我们来编写 `onEnter` 及 `onLeave` 函数

|  |  |
| --- | --- |
