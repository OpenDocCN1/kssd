
# 有毒的学Pin记录（二）

## 4. Callbacks

这部分主要介绍几个Pin的用于注册回调函数的API：

- `INS_AddInstrumentFunction` (`INSCALLBACK fun, VOID *val`)：注册以指令粒度插桩的函数
- `TRACE_AddInstrumentFunction` (`TRACECALLBACK fun, VOID *val`)：注册以trace粒度插桩的函数
- `RTN_AddInstrumentFunction` (`RTNCALLBACK fun, VOID *val`)：注册以routine粒度插桩的函数
- `IMG_AddInstrumentFunction` (`IMGCALLBACK fun, VOID *val`)：注册以image粒度插桩的函数
- `PIN_AddFiniFunction` (`FINICALLBACK fun, VOID *val`)：注册在应用程序退出前执行的函数，该类函数不进行插桩，可以有多个。
- `PIN_AddDetachFunction` (`DETACHCALLBACK fun, VOID *val`)：注册在Pin通过`PIN_Detach()`函数放弃对应用程序的控制权限之前执行的函数，一个进程只调用一次，可以被任何线程调用，此时Pin的内存并没有释放。

对于每个注册函数的第二个参数`val`将在“回调”时传递给回调函数。如果在实际的场景中不需要传递第二个参数，为了保证安全，可以传递将`val`的值设置为0进行传递。`val`的理想使用方式是传递一个指向类实例的指针，这样回调函数在取消引用该指针前需要将其转换回一个对象。

所有的注册函数都会返回一个`PIN_CALLBACK`对象，该对象可以在后续过程中用于操作注册的回调的相关属性。

### PIN callbacks manipulation API

在注册函数返回`PIN_CALLBACK`对象后，可以使用`PIN_CALLBACK`API对其进行操作，来检索和修改在Pin中已注册的回调函数的属性。

声明：

```cpp
typedef COMPLEX_CALLBACKVAL_BASE * PIN_CALLBACK
```

函数：

1. `CALLBACK_GetExecutionOrder()`

声明：

```cpp
VOID CALLBACK_GetExecutionOrder (PIN_CALLBACK callback)
```

作用：获取已注册回调函数的执行顺序。越靠前，越早被执行。

参数：`callback`，从`*_Add*Funcxtion()`函数返回的注册的回调函数

2. `CALLBACK_SetExecutionOrder()`

声明：

```cpp
VOID CALLBACK_SetExecutionOrder (PIN_CALLBACK callback, CALL_ORDER order)
```

作用：设置已注册回调函数的执行顺序。越靠前，越早被执行。

参数：`callback`，从`*_Add*Funcxtion()`函数返回的注册的回调函数；`order`，新设置的回调函数的执行顺序。

3. `PIN_CALLBACK_INVALID()`

声明：

```cpp
const PIN_CALLBACK PIN_CALLBACK_INVALID(0)
```

PIN回调的无效值。

### CALL_ORDER

`CALL_ORDER`是一个枚举类型，预定义了`IARG_CALL_ORDER`的值。其作用就是当指令有多个分析函数调用时，控制每个分析函数的调用顺序，默认值为`CALL_ORDER_DEFAULT`。

- `CALL_ORDER_FIRST`：首先执行该调用，整数值为100.
- `CALL_ORDER_DEFAULT`：未指定`IARG_CALL_ORDER`时的默认值，整数值为200.
- `CALL_ORDER_LAST`：最后执行该调用，整数值为300.

在进行数值设定时，可以使用类似`CALL_ORDER_DEFAULT + 5`的格式来设置。

针对在相同插桩回调环境中的针对同一指令的、具备同样`CALL_ORDER`的多个分析调用，Pin会按照插入的顺序进行调用。

## 5. Mopdifying Application Instructions

虽然Pin的主要用途是对二进制程序进行插桩，但是它也可以实现对程序指令的修改。

### 5.1 实现方式

最简单的实现方式是插入一个分析routine来模拟指令执行，然后调用`INS_Delete()`来删除指令。也可以通过直接或间接插入程序执行流分支（使用`INS_InsertDirectJump`和`INS_InsertIndirectJump`）实现，这种方式会改变程序的执行流，但是会更容易实现指令模拟。

1. `INS_InsertDirectJump()`

声明：

```cpp
VOID INS_InsertDirectJump(INS ins, IPOINT ipoint, ADDRINT tgt)
```

参数：

- `ins`：输入的指令
- `ipoint`：与`ins`相关的location（仅支持`IPOINT_BEFORE`和`IPOINT_AFTER`）
- `tgt`：target的绝对地址

作用：插入相对于给定指令的直接跳转指令，与`INS_Delete()`配合使用可以模拟控制流转移指令。

2. `INS_InsertIndirectJump()`

声明：

```cpp
VOID INS_InsertIndirectJump ( INS ins, IPOINT ipoint, REG reg)
```

参数：

- `ins`：输入的指令
- `ipoint`：与`ins`相关的location（仅支持`IPOINT_BEFORE`和`IPOINT_AFTER`
- `reg`：target的寄存器

作用：插入相对于给定指令的间接跳转指令，与`INS_Delete()`配合使用可以模拟控制流转移指令。

### 5.2 指令内存修改

对于原始指令使用到的内存的访问，可以通过使用`INS_RewriteMemoryOperand`来引用通过分析routine计算得到的值来替代。

需要注意的是，对于指令的修改操作，会在所有的指令插桩操作完成后进行，因此在进行指令插桩时，插桩routine看到的都是原始的、没有经过修改的程序指令。

`INS_RewriteMemoryOperand()`

声明：

```cpp
VOID INS_RewriteMemoryOperand(INS ins, UINt32 memindex, REG newBase)
```

参数：

- `ins`：输入指令
- `memindex`：控制需要重写的内存操作数（0，1，...）
- `newBase`：包含新操作数地址的寄存器，通常是通过`PIN_ClainToolRegister`分配的临时寄存器

作用：更改此内存访问指令以饮用包含在给定特定寄存器中的虚拟内存地址。

在IA-32和Intel 64平台上，修改后的操作数仅使用具有新基址寄存器`newBase`的基址寄存器进行寻址。原始指令中该操作数的任何index， scale或者offset filed都会被删除。

该函数可以用于重写内存操作数，包括隐式的（如call、ret、push、pop），唯一不能重写的指令是第二个操作数大于0的`enter`。

`newBase`中的地址是中是该操作数将访问的最低地址，如果操作数在内存访问之前被指令修改，如push，则`newBase`中的值将不是堆栈指针，而是指令访问的内存地址。

用于内存地址重写的一个样例插桩代码如下：

```cpp
// 映射originalEa到一个翻译后的地址
static ADDRINT ProcessAddress(ADDRINT originalEa, ADDRINT size, UINT32 access);
...
for (UINT32 op = 0; op<INS_MemoryOperandCount(ins); op++) // 首先遍历内存操作指令进行计数
{
UINT32 access = (INS_MemoryOperandIsRead(ins,op) ? 1 : 0) | // 判断是内存读还是内存写
(INS_MemoryOperandIsWritten(ins,op) ? 2 : 0);
INS_InsertCall(ins, IPOINT_BEFORE,
AFUNPTR(ProcessAddress),
IARG_MEMORYOP_EA, op,
IARG_MEMORYOP_SIZE, op,
IARG_UINT32, access,
IARG_RETURN_REGS, REG_INST_G0+i,
IARG_END); // 在指令处进行插桩
INS_RewriteMemoryOperand(ins, i, REG(REG_INST_G0+i)); // 重写内存指令的操作数
}
```

## 6. Applying a Pintool to an Application

命令行：

```shell
pin [pin-option]... -t [toolname] [tool-options]... -- [application] [application-option]..
```

### 6.1 Pin Cmdline Options

如下是Pin的命令行的完整option列表：

| Option | Description |
| ----------------------------- | ------------------------------------------------------------ |
| -follow_execv | 使用Pin执行由execv类系统调用产生的所有进程 |
| -help | 帮助信息 |
| -pause_tool <n> | 暂停并打印PID以可以在tool加载后attach到debugger，处理过程在‘n’秒后重启 |
| -logfile | 指定log文件的名字和路径，默认路径为当前工作目录，默认文件名为pin.log |
| -unique_logfile | 添加pid到log文件名中 |
| -error_file | 指定error文件的名字和路径，默认路径为当前工作目录。如果设置了error文件，则所有error都会写入到文件中，并且不会在console中显示。如果没有指定，则不创建文件。 |
| -unique_error_file | 添加pid到error文件名中 |
| -injection <mode> | <mode>的选项为dynamic， self， child， parent，只能在UNIX中使用，详看Injection，默认使用dynamic。 |
| -inline | 内联简单的分析routine |
