
# [原创]SEH源码赏析之C篇

SEH结构化异常处理源码赏析(C篇)

关键字: C SEH 结构化异常处理 `_try` `_finally` `_except` `_except_handler3` VC

工程使用工具: `VC7.1.3088` `IDA pro 4.9.0.863` `cdb Windbg Editplus`

## 1. 起因

C++程序员对`try`, `catch`, `throw`都应该很熟悉,能知道VC怎么实现它的人就不多了,不过网络世界使很多人知道了它与SEH(structured exception handling）有密切关系,我也不例外,也是在若干年前从网络知道了SEH,并且大致也知道SEH的流程.但是和多数人一样在我的实践也很少直接使用SEH,对SEH也就仅限于网络上一些文章的介绍.曾经在用Windbg对某些软件作分析,我遇到了断点失效的情况,查找资料介绍是SEH中的Handler清除了调试寄存器,在分析SEH代码中由于VC没有SEH的源码,于是我产生了一种想法,详细完整地翻译VC的SEH的代码,一劳永逸的解决问题.C和C++的SEH有所不同,C++的要复杂些,我在此介绍的仅为C的SEH代码,也就是`__try`,`__finally`,`__except`,`__leave`所产生的SEH代码,C++篇有时间的话我再作.我以前看过的资料大都以比较专业的语言介绍,在此我仅以我自己感觉比较通俗的语言介绍,希望能有更多的人能认识,认清SEH.

## 2. SEH术语

SEH中术语虽然不多,但由于没有SDK的明确定义有时很难区别,各家说的表述也不太统一,因此为此文特定义如下术语,这些术语可能与其它文献有点冲突或细微差别,有的也是我自己的定义:

A. SEH(structured exception handling): 在C语言中关键字是`__try`(`_try`),`__finally`(`_finally`),`_except`(`__except`),而C++使用的关键字是`try`,`catch`.在以下的表述中所有的`try`均不再特别声明为C关键字,一律默认为C关键字.

B. EH3_List:`_EH3_EXCEPTION_REGISTRATION`链表,表头位于`FS:[0]`, `0xFFFFFFFF`为链表结束标志.编译器在编译一个函数时只要检测到含有`__try`或`__try`则为此函数生成一个`_EH3_EXCEPTION_REGISTRATION`节点,并插入到表头.因为每个函数编译只生成一个节点,因此在一个函数中C和C++的SEH不能同时存在,如果代码中同时有`catch`和`except`则不能通过编译就是此原因.

C. EH3_ScopeTable:是一个由编译器在`data section`生成的一张表(数组),实质可看作是二叉树结构(可能有多个二叉树顺序存放),节点为`_SCOPETABLE_ENTRY`类型,其中`_SCOPETABLE_ENTRY.ParentLevel`是父节点在数组中的位置,`EH3_ScopeTable[0]`是根节点,`_SCOPETABLE_ENTRY.ParentLevel=0xFFFFFFFF`.由此可见`ParentLevel`很重要,是SEH判断`try`嵌套层次的唯一依据.编译器从函数入口点开始遍历`try`,每遇到一个`try`生成一个节点`_SCOPETABLE_ENTRY`,并放在表最后,注意节点的先后与`try`的嵌套层次无关.

D. filter handler:是异常发生后让用户决定是否认识此异常,通过修改异常语句的上下文环境(`CONTEXT`)可使应用程序能继续正常运行.其返回值有三.

`EXCEPTION_EXECUTE_HANDLER`(1): 去执行`exception handler`,然后进程终止,且不显示出错提示框.

`EXCEPTION_CONTINUE_SEARCH`(0): 不执行`exception handler`,显示出错提示框,进程终止或者进入调试器进行调试.

`EXCEPTION_CONTINUE_EXECUTION`(-1): 不执行`exception handler`,系统用`CONTEXT`重新设置CPU环境,进程继续执行,如果修改了`EIP`则从新的`EIP`开始执行,否则从原异常点开始执行.

E. exception handler:是异常发生后检测到该异常无法被处理,而进程终止前提醒应用程序执行的收尾工作

F. termination handler:如果`try`语句被过早终止,不管是正常离开或者是非正常离开,包括`goto`,`leave`及异常时均执行此`handler`,具体执行过程可查MSDN.

G. 展开(unwind):这个名词很让人费解,翻译也确实不好命名,我也沿用此名.展开的目的是执行`finally`对应的`termination handler`,对照在下面的源代码中我们就能很容易理解MSDN关于`finally`的解释了.展开分为本地局部展开(`__local_unwind`)和全局展开(`__global_unwind`);本地展开:展开此`try`所在函数`try`嵌套关系并分别执行其`finally`对应的`termination handler`;全局展开:当`exception handler`不在本`try`函数时,执行`exception handler`前需要先执行这之前的`termination handler`,全局展开就是查找这些`termination handler`并执行它.需要说明的是全局展开不含本地展开.

## 3. SEH数据结构

```c
typedef struct  // (sizeof=0xC)
{
    DWORD ParentLevel ;           // 当前Handler父层TRY在EH3_ScopeTable中的位置,根没有上一层,故值=-1
                                  // 形成TRY层次的二叉树结构,与_EH3_EXCEPTION_REGISTRATION.TryLevel物理意义一样
    DWORD FilterFunc;      // 非NULL则HandlerFunc是exception handler,否则termination handler
    DWORD HandlerFunc;     // exception handler or termination handler
} _SCOPETABLE_ENTRY;
```

```c
typedef struct  // (sizeof=0x10)
{
    _EH3_EXCEPTION_REGISTRATION* pPrev;// 栈上一级EH3_List节点,=0xFFFFFFFF则为最后一个节点
    EXCE_HANDLER ExceptionHandler;           // VC7.1中统一指向__except_handler3
    _SCOPETABLE_ENTRY* pScopeTable;           // 指向一个_SCOPETABLE_ENTRY数组,函数有n个TRY,则数组有n个元素
                                              // p[0]->Try0为根,p[1]->Try1,p[2]->Try2...
    DWORD TryLevel;     // 指示当前指令在Try的层次级别,-1未进入TRY,进第一个TRY为0,第二个为1,...
                        // 但它与嵌套层次无关,在编译时确定,从函数代码开始处开始计数
} _EH3_EXCEPTION_REGISTRATION;
```

```c
typedef struct  // (sizeof=0x10)还有待进一步分析其用处
{
    DWORD unKnown;                // 未知:被编译器赋值
    DWORD HandlerFunc;  // _SCOPETABLE_ENTRY.HandlerFunc
    DWORD firstPara;    // Try所在函数第一个参数:crtMain!EBP+8
    DWORD TryEBP;                // Try所在函数EBP
} _NLGDestination;
```

以下在MSDN中均有定义,不再作解释

```c
typedef struct _EXCEPTION_RECORD
{
    DWORD    ExceptionCode;
    DWORD ExceptionFlags;
    struct _EXCEPTION_RECORD *ExceptionRecord;
    PVOID ExceptionAddress;
    DWORD NumberParameters;
    ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
} EXCEPTION_RECORD,*PEXCEPTION_RECORD;
```

```c
typedef struct _CONTEXT
{
    ...         // 依据CPU类型有不同定义,具体可见winnt.h
} CONTEXT,*PCONTEXT;
```

```c
typedef struct _EXCEPTION_POINTERS
{
    PEXCEPTION_RECORD ExceptionRecord;
    PCONTEXT ContextRecord;
} EXCEPTION_POINTERS,*PEXCEPTION_POINTERS;
```

## 4. SEH汇编要点

A. 函数中`try`的数据模型:VC将会为有`try`函数在栈上首先建立三个变量:

`EBP-18h`: `SaveESP`                                        // TRY前保存当时ESP,因此有:SaveESP<&Eh3Exception

`EBP-14h`: `pExceInfo`                                // GetExceptionPointers(),在handler中调用filter前赋值

`EBP-10h`: `Eh3Exception`                        // 一个`_EH3_EXCEPTION_REGISTRATION`,其大小刚好为10h哦

`EBP+00h`: `ebp`                                                // 上一frame的EBP

`EBP+04h`: `EIP`                                                // CALL返回地址

在这里我们应该注意到这个公式是成立的:

`EBP=&Eh3Exception+10h`

而在`__except_handler3`中参数: `pEh3Exce`刚好为`&Eh3Exception`,所以每当在`__except_handler3`中要回调`filter`,`exception`,`termination handler`时,在这之前汇编中均有一句:

```asm
lea ebp, [ebx+10h]                // ebx=__except_handler3!arg_pEh3Exception
```

明白了这点就不难明白如何在`__except_handler3`中访问`pExceInfo`及`SaveESP`了!

B. `try`块的终止:异常发生,`goto`,`return`,`leave`,正常终止.`goto`和`return`可能跨过其它`try`,所以必须要展开到目的地所在的`TryLevel`,但是`leave`关键字不会跨过其它`try`,只是跳出自己这层`try`,如果编译器检测到这层`try`有`termination handler`,则用`CALL xxx`直接调用.当然也有例外,这也是VC聪明的地方,如果函数只有一个`try`,则根本不用展开而直接使用`CALL`了,这种情况有时可见.这样就不难理解如下代码:

`goto 004013fb` 终止代码:

```asm
// ---------------goto跳出try----------------------------------------
goto try0;
push    0                                                        或者        (仅一个try时)                CALL 00401070        // 直接调termination handler
lea     eax, [ebp+var_Eh3Exce]                                                                jmp loc_4013FB
push    eax
call    __local_unwind2                                        // 展开到0(即第一个try内,goto目的地肯定在第一个try内)
add     esp, 8
jmp     loc_4013FB
```

`return` 终止代码:(无返回值)

```asm
// ---------------return跳出try--------------------------------------
push        0FFFFFFFFh
lea         eax,[ebp-10h]
push        eax
call        __local_unwind2 (401786h)        // 展开到-1(即所有try外)
add         esp,8
return ;
jmp         $L19800 (401139h)
```

`return var_i` 终止代码:(有返回值)

```asm
// ---------------return(带返回值)跳出try----------------------------
mov         eax,dword ptr [i]
mov         dword ptr [ebp-100h],eax        // 返回值先被保存
push        0FFFFFFFFh
lea         ecx,[ebp-10h]
push        ecx
```
