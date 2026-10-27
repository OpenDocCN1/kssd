
在上述过程中，我们可以发现，遍历过程中 `RtlDispatchException` 等系统函数被频繁遍历到。于是就有了前面提到的全局展开历史表 `RtlpUnwindHistoryTable`，这个表中存放着 `RtlDispatchException`、`RtlUnwindEx` 等函数的 `RUNTIME_FUNCTION` 和 `ImageBase` 信息，这样就不用每次都去解析 PE+ 中的 `ExceptionDirectory`，实现了加速。

到这里，我们就讲完了 x64 SEH 的实现。可以发现，x64 和 x86 的 SEH 思想或者说框架是一样的：

1. `RtlDispatchException` 和 `RtlUnwindEx` 都对异常注册信息进行遍历。前者是为了分发异常而遍历，后者是为了展开而遍历。
2. MSC 提供的异常处理函数按照“异常解决”和“展开”两个分支，对 `SCOPE_TABLE`/`scopetable` 进行遍历。前者是为了找到 `EXCEPT_FILTER` & `EXCEPT_HANDLER`，后者是为了找到 `FINALLY_HANDLER`。
3. `RtlDispatchException` 和 `RtlUnwindEx` 借助 MSC 提供的异常处理函数这个桥梁，配合处理异常。

主要的改变有两点：

1. `RtlDispatchException` 和 `RtlUnwindEx` 通过调用 `RtlVirtualUnwind` 推动遍历。
2. 所有的非叶函数都参与到 SEH，尽管大部分的函数都没有使用到 SEH。

以上我们主要讲述的是 x64 SEH 的内部实现。对于使用者，也有一个好消息，我们来看看，

### C 代码：

``` 
    VOID SehTest()
    {
        __try
        {
            __try
            {
                ExRaiseStatus(STATUS_INVALID_PARAMETER);
                DbgPrint("%u [%s] __try \n", __LINE__, __FILE__);
            }
            __except((STATUS_INVALID_PARAMETER == GetExceptionCode()) ? EXCEPTION_CONTINUE_SEARCH : EXCEPTION_EXECUTE_HANDLER)
            {
                DbgPrint("%u [%s] __except \n", __LINE__, __FILE__);
            }
        }
        __finally
            DbgPrint("%u [%s] __finally \n", __LINE__, __FILE__);
        }
    }
```

### 反汇编码：

```
    kd> uf passthrough!SehTest
    PassThrough!SehTest:
    fffffadf`f1100020 sub     rsp,38h
    fffffadf`f1100024 mov     ecx,0C000000Dh
    fffffadf`f1100029 call    qword ptr [PassThrough!_imp_ExRaiseStatus (fffffadf`f1101050)]
    fffffadf`f110002f lea     r8,[PassThrough! ?? ::FNODOBFM::`string` (fffffadf`f1100500)]
    fffffadf`f1100036 mov     edx,39h
    fffffadf`f110003b lea     rcx,[PassThrough! ?? ::FNODOBFM::`string` (fffffadf`f1100540)]
    fffffadf`f1100042 call    PassThrough!DbgPrint (fffffadf`f11004a6)
    fffffadf`f1100047 jmp     PassThrough!SehTest+0x42 (fffffadf`f1100062)

    PassThrough!SehTest+0x42:
    fffffadf`f1100062 lea     r8,[PassThrough! ?? ::FNODOBFM::`string` (fffffadf`f1100500)]
    fffffadf`f1100069 mov     edx,42h
    fffffadf`f110006e lea     rcx,[PassThrough! ?? ::FNODOBFM::`string` (fffffadf`f1100570)]
    fffffadf`f1100075 call    PassThrough!DbgPrint (fffffadf`f11004a6)
    fffffadf`f110007a add     rsp,38h
    fffffadf`f110007e ret
```

我们发现 `SehTest` 内部完全没有任何 SEH 的踪迹，不像 x86 那样会有创建、销毁 `EXCEPTION_REGISTRATION_RECORD` 和调整 `EXCEPTION_REGISTRATION_RECORD::trylevel` 等操作。

这样的好处就是使用者无需再担心性能损耗，可以放心大胆的使用 SEH 机制了。

## 附录一 boxr 扩展

为了方便自己分析，我写了一个简单的 windbg 扩展，提供了几个 x64 seh 常用功能：

`!boxr.unwindinfo`    `module-name`    `unwindinfo_addr`

功能：

用于查询指定 `UNWIND_INFO` 结构的详细信息。

参数说明：

`module-name` —— 待查询的 `UNWIND_INFO` 结构对应函数的模块名

`unwindinfo_addr` —— `UNWIND_INFO` 结构的绝对地址

`!boxr.rtfn`    `option`    `module`    `runtimefunction_addr`

功能：

用于查询指定 `RUNTIME_FUNCTION` 结构的详细信息。`rtfn` 表示 `RunTime_FunctioN`

参数说明：

`option` —— 参数选项，目前支持两种：

`/a` 表示 `module` 参数为模块基地址

`/n` 表示 `module` 参数为模块名称

`module` —— `RUNTIME_FUNCTION` 结构对应函数所在的模块，具体形式根据 `option` 而定。

`runtimefunction_addr` —— 需要查询的 `RUNTIME_FUNCTION` 结构体的绝对地址。支持 `@rax` 操作方式，但不支持复杂的组合，比如 `@rax+8`。

使用的方法是：用 `.extpath+` 命令将 `boxr.dll` 所在的目录添加到 windbg 的搜索路径中，然后就可以使用了。需要卸载时就 `.unload`。

简单说明一下这两个命令。

比如我们要查看下面这个函数的 `UNWIND_INFO` 信息：
