

# 随便说说 compileMethod 方法

> 原文：[`www.pediy.com/kssd/pediy10/71000.html`](https://www.pediy.com/kssd/pediy10/71000.html)

闲来无事，随便写点，老话题了，高手略过。  
    compileMethod 方法在各类.NET 内核的文章中出现频率相当高，因为它是 JIT 引擎工作的关键函数。其原型如下（参考 sscli 代码）：  

代码:

```
CorJitResult \_\_stdcall FJitCompiler::compileMethod (
            ICorJitInfo\*               compHnd,            /\* IN \*/
            CORINFO\_METHOD\_INFO\*        info,               /\* IN \*/
            unsigned                flags,              /\* IN \*/
            BYTE \*\*                 entryAddress,       /\* OUT \*/
            ULONG  \*                nativeSizeOfCode    /\* OUT \*/
            )

```

    一般的用法是通过该函数的第二个参数 COFINFO\_METHOD\_INFO 取得代码的 IL 和大小。但其实还可以更加深入，注意第一个传入参数很有意思，指向了 ICorJitInfo 接口。该接口定义如下：  
代码:

```
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
 \* a ICorJitInfo is the main interface that the JIT uses to call back to the EE and
 \*   get information
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/
class ICorJitInfo : public virtual ICorDynamicInfo
{//省略}

```

    这说明 ICorJitInfo 继承了 ICorDynamicInfo 接口。而后者的定义如下：  
代码:

```
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
 \* ICorDynamicInfo contains EE interface methods which return values that may
 \* change from invocation to invocation.  They cannot be embedded in persisted
 \* data; they must be requeried each time the EE is run.
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

class ICorDynamicInfo : public virtual ICorStaticInfo
{//省略}

```

    这说明 ICorDynamicInfo 又继承了 ICorStaticInfo。接着来，继续看 ICorStaticInfo 的代码：  
  
代码:

```
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
 \* ICorStaticInfo contains EE interface methods which return values that are
 \* constant from invocation to invocation.  Thus they may be embedded in
 \* persisted information like statically generated code. (This is of course
 \* assuming that all code versions are identical each time.)
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/
class ICorStaticInfo : public virtual ICorMethodInfo, public virtual ICorModuleInfo,
                       public virtual ICorClassInfo,  public virtual ICorFieldInfo,
                       public virtual ICorDebugInfo,  public virtual ICorArgInfo,
                       public virtual ICorLinkInfo,   public virtual ICorErrorInfo

```

    这一次牛了，ICorStaticInfo 继承了 8 个接口的方法，其中第一个为 ICorMethodInfo。该接口定义了许多与方法相关的函数，如果能调用它，在 hook 中是非常爽的。那可不可以调用呢？当然可以，compileMethod 的第一个参数就是我们需要的。  
    比如我们需要取得当前 hook 方法的方法名，可以调用 ICorMethodInfo 中的如下方法：  
代码:

```
    virtual const char\* \_\_stdcall getMethodName (
            CORINFO\_METHOD\_HANDLE       ftn,        /\* IN \*/
            const char                \*\*moduleName  /\* OUT \*/
            ) = 0;

```

    这里需要传入 CORINFO\_METHOD\_HANDLE 这个参数，该参数是.NET 内核中表示方法的核心结构，在 sscli 中也未公开。不过它的值是可以取得的，从 compileMethod 第二个参数 CORINFO\_METHOD\_INFO 结构的第一项中便可以取得：  
代码:

```
struct CORINFO\_METHOD\_INFO
{
    CORINFO\_METHOD\_HANDLE       ftn;
    CORINFO\_MODULE\_HANDLE       scope;    
    BYTE \*                      ILCode;
    unsigned                    ILCodeSize;
    unsigned short              maxStack;
    unsigned short              EHcount;
    CorInfoOptions              options;
    CORINFO\_SIG\_INFO            args;
    CORINFO\_SIG\_INFO            locals;
};

```

    同样，在调用 ICorModuleInfor 接口中的许多方法时，也需要传入 CORINFO\_MODULE\_HANDLE 作为参数，同样可以从 CORINFO\_METHOD\_INFO 结构中获得该值。比如调用 findClass 函数，以取得 CORINFO\_CLASS\_HANDLE，定义如下：  
代码:

```
    virtual CORINFO\_CLASS\_HANDLE \_\_stdcall findClass (
            CORINFO\_MODULE\_HANDLE       module,     /\* IN  \*/
            unsigned                    metaTOK,    /\* IN  \*/
            CORINFO\_CONTEXT\_HANDLE      context,    /\* IN  \*/
            CorInfoTokenKind            tokenKind = CORINFO\_TOKENKIND\_Default /\* IN  \*/
            ) = 0;

```

    其中出现了 CORINFO\_CONTEXT\_HANDLE。该结构也非常容易取得，具体参考 sscli，就不详述了。  
    具体在 VS 中编程时，可以添加 corinfo.h 和 corjit.h，并在同一目录下添加 corhdr.h，便可顺利编译通过。  
    最后还有一个问题，sscli 毕竟是早期框架了，还是精简版，现在还能直接使用吗？不妨分析一下。随便运行一个.NET 程序，用 WinDbg 调试并中断在 compileMethod 处，查看 ICorJitInfo 值所指的内存：  
代码:

```
0012ea38 79f10654 mscorwks!CEEJitInfo::\`vbtable'
0012ea3c 00174d18 
0012ea40 00997850 
0012ea44 00107210 
0012ea48 00000000 
0012ea4c 0018a610 
0012ea50 00000000 
0012ea54 00000000 
0012ea58 00000000 
0012ea5c 0012ea40 
0012ea60 00000000 
0012ea64 00000000 
0012ea68 00000000 
0012ea6c 79f105b8 mscorwks!CEEJitInfo::\`vftable'
0012ea70 00000000 
0012ea74 79f10584 mscorwks!CEEJitInfo::\`vftable'
0012ea78 00000000 
0012ea7c 79f104e0 mscorwks!CEEJitInfo::\`vftable'
0012ea80 00000000 
0012ea84 79f104bc mscorwks!CEEJitInfo::\`vftable'
0012ea88 00000000 
0012ea8c 79f104a4 mscorwks!CEEJitInfo::\`vftable'
0012ea90 00000000 
0012ea94 79f10498 mscorwks!CEEJitInfo::\`vftable'
0012ea98 00000000 
0012ea9c 79f10494 mscorwks!CEEJitInfo::\`vftable'
...//下略

```

    这里又涉及到 VC 编译器对类的 vftable 和 vbtable 在内存中的布局问题了，最先两项是类自身定义的虚方法表和虚基址表，相关资料请自己查阅。我们跟进 vftable：  
代码:

```
79f10624 79f106ac mscorwks!CEEJitInfo::getMemoryManager
79f10628 79f11d82 mscorwks!CEEJitInfo::allocMem
79f1062c 79f11f39 mscorwks!CEEJitInfo::allocGCInfo
79f10630 7a12b8cb mscorwks!CEEJitInfo::getEHInfo
79f10634 7a12b6cf mscorwks!CEEJitInfo::yieldExecution
79f10638 79f16373 mscorwks!CEEJitInfo::setEHcount
79f1063c 79f16491 mscorwks!CEEJitInfo::setEHinfo
79f10640 7a12ed41 mscorwks!CEEJitInfo::logMsg
79f10644 7a27ffcc mscorwks!ZapperModule::doAssert
79f10648 7a12eeb6 mscorwks!CEEJitInfo::allocBBProfileBuffer
79f1064c 7a2c2fa1 mscorwks!MDInternalRO::ConvertTextSigToComSig
79f10650 79f0efee mscorwks!CEEJitInfo::isVerifyOnly
79f10654 fffffffc 

```

      这些地址在静态编译时就已经确定了，因此如果用 IDA 反编译 mscorwks.dll，同样会得到这些值。这便是最新的.NET 框架内核中 ICorJitInfo 定义的方法。和 sscli 对比下（在 corjit.h 文件中），完全一样！（也可以再对比其它关键接口的方法，应该也是一样，我没有详细对比了。）  
    这样，我们就得到如下的结论：.NET 内核框架从 2.0 开始，内核变化不大，包括 sscli 的内核代码，这些可以从内部函数的定义看出来；通过 hook compileMethod，可以得到 ICorJitInfo 等关键接口，并调用其中的许多内部方法；具体编写时，可以在 VS 中引入 sscli 的 corinfo.h、corjit.h 和 corhdr.h 等文件，之后便可以直接调用。因此，通过 compileMethod，我们可以做的事很多很多，远不限于仅获得某个方法的 IL。  
    个人观点，若有错误，欢迎指正！