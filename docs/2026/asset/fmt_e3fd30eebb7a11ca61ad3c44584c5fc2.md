
```c
struct Method {
    ClassObject* clazz; /* method所属的类 public、native等 */
    u4 accessFlags; /* 访问标记 */
    u2 methodIndex; //method索引
    //三个size为边界值，对于native函数，这3个size均等于参数列表的size
    u2 registersSize; /* ins + locals */
    u2 outsSize;
    u2 insSize;
    const char* name; //函数名称
    /*
     * Method prototype descriptor string (return and argument types)
     */
    DexProto prototype;
    /* short-form method descriptor string */
    const char* shorty;
    /*
     * The remaining items are not used for abstract or native methods.
     * (JNI is currently hijacking "insns" as a function pointer, set
     * after the first call. For internal-native this stays null.)
     */
    /* the actual code */
    const u2* insns; /* instructions, in memory-mapped .dex */
    /* cached JNI argument and return-type hints */
    int jniArgInfo;
    /*
     * Native method ptr; could be actual function or a JNI bridge. We
     * don't currently discriminate between DalvikBridgeFunc and
     * DalvikNativeFunc; the former takes an argument superset (i.e. two
     * extra args) which will be ignored. If necessary we can use
     * insns==NULL to detect JNI bridge vs. internal native.
     */
    DalvikBridgeFunc nativeFunc;
    /*
     * Register map data, if available. This will point into the DEX file
     * if the data was computed during pre-verification, or into the
     * linear alloc area if not.
     */
    const RegisterMap* registerMap;
};
```

```
function replaceDalvikImplementation (fn) {
    if (fn === null && dalvikOriginalMethod === null) {
        return;
    }
    //备份原来的method,
    if (dalvikOriginalMethod === null) {
        dalvikOriginalMethod = Memory.dup(methodId, DVM_METHOD_SIZE);
        dalvikTargetMethodId = Memory.dup(methodId, DVM_METHOD_SIZE);
    }
    if (fn !== null) {
        //自定的代码
        implementation = implement(f, fn);
        let argsSize = argTypes.reduce((acc, t) => (acc + t.size), 0);
        if (type === INSTANCE_METHOD) {
            argsSize++;
        }
        // 把method变成native函数
        /*
         * make method native (with kAccNative)
         * insSize and registersSize are set to arguments size
         */
        const accessFlags = (Memory.readU32(methodId.add(DVM_METHOD_OFFSET_ACCESS_FLAGS)) | kAccNative) >>> 0;
        const registersSize = argsSize;
        const outsSize = 0;
        const insSize = argsSize;
        Memory.writeU32(methodId.add(DVM_METHOD_OFFSET_ACCESS_FLAGS), accessFlags);
        Memory.writeU16(methodId.add(DVM_METHOD_OFFSET_REGISTERS_SIZE), registersSize);
        Memory.writeU16(methodId.add(DVM_METHOD_OFFSET_OUTS_SIZE), outsSize);
        Memory.writeU16(methodId.add(DVM_METHOD_OFFSET_INS_SIZE), insSize);
        Memory.writeU32(methodId.add(DVM_METHOD_OFFSET_JNI_ARG_INFO), computeDalvikJniArgInfo(methodId));
        //调用dvmUseJNIBridge为这个Method设置一个Bridge,本质上是修改结构体中的nativeFunc为自定义的implementation函数
        api.dvmUseJNIBridge(methodId, implementation);
        patchedMethods.add(f);
    } else {
        patchedMethods.delete(f);
        Memory.copy(methodId, dalvikOriginalMethod, DVM_METHOD_SIZE);
        implementation = null;
    }
}
```

### art 模式

art模式也是需要将java 函数变成 native 函数但是不同于dalvik，art下有两种解释器一种汇编解释器一种smali解释器

```
quick code 模式：执行 arm 汇编指令
Interpreter 模式：由解释器解释执行 Dalvik 字节码
```

//2022.7.17 寒冰大佬指出问题 安卓7.0 有三种解释器switch结构，goto跳转表，汇编

```
enum InterpreterImplKind {
    kSwitchImplKind, // Switch-based interpreter implementation.
    kComputedGotoImplKind, // Computed-goto-based interpreter implementation.
    kMterpImplKind // Assembly interpreter
};
```

![图片描述](upload/attach/202206/928079_ZRDSMC7S2VH5CDY.png)

1.如果函数已经存在quick code, 则指向这个函数对应的 quick code的起始地址，而当quick code不存在时，它的值则会代表其他的意义。

2.当一个 java 函数不存在 quick code时，它的值是函数`artQuickToInterpreterBridge`的地址，用以从 quick 模式切换到 Interpreter 模式来解释执行 java 函数代码。

3.当一个 java native（JNI）函数不存在 quick code时，它的值是函数`art_quick_generic_jni_trampoline`的地址，用以执行没有quick code的 jni 函数。

所以 frida 要将 java method 转为 native method，需要将ARTMethod 结构进行如下修改：

```
patchMethod(methodId, {
    //jnicode入口entry_point_from_jni_改为自定义的代码
    'jniCode': implementation,
    //修改为access_flags_为native
    'accessFlags': (Memory.readU32(methodId.add(artMethodOffset.accessFlags)) | kAccNative | kAccFastNative) >>> 0,
    //art_quick_generic_jni_trampoline函数的地址
    'quickCode': api.artQuickGenericJniTrampoline,
    //artInterpreterToCompiledCodeBridge函数地址
    'interpreterCode': api.artInterpreterToCompiledCodeBridge
});
```

# 参考文献

```
https://evilpan.com/2022/04/05/frida-internal/
https://blog.drov.com.cn/2021/04/hook.html
https://bbs.pediy.com/thread-229215.htm
https://frida.re/docs/home/
https://www.youtube.com/watch?v=uc1mbN9EJKQ
```

# 添加补充

```
// 2022.7.17
一个非常不错的项目 纯frida实现的smali追踪
trace smali代码一般都是通过修改art源码去强制解析执行然后记录执行的指令和寄存器的值
https://github.com/SeeFlowerX/frida-smali-trace
```

# frida介绍

frida是一款便携的,自由的,支持全平台的,hook框架，可以通过编写JavaScript,Python代码来和`frida_server`端进行交互,还记得当年用xposed时那种写了一大堆代码每次修改都要重新打包安装重启手机,那种调试调到头皮发麻的痛苦百分之30的时间都是在那里安装重启安装重启直到有一天遇到了小甜甜。

# frida的代码结构

| | |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `frida-core: Frida 核心库` `frida-gum: inline-hook 框架` `bindings:` `frida-python: python` `frida-node: Node.js` `frida-qml: Qml` `frida-swift: Swift` `frida-tools: CLI tools` `capstone: instruction disammbler` |

Frida的核心是c编写的有多种语言绑定例如 Node.js、 Python、 Swift、 .NET、 Qml

一般我们都使用js去编写frida脚本因为js的异常处理机制非常棒相比于其他语言更高效好用。

## frida-core

frida-core的功能有进程注入、进程间通信、会话管理、脚本生命周期管理等功能，屏蔽部分底层的实现细节并给最终用户提供开箱即用的操作接口。而这一切的实现都在 frida-core 之中，正如名字所言，这其中包含了 frida 相关的大部分关键模块和组件，比如 frida-server、frida-gadget、frida-agent、frida-helper、frida-inject 以及之间的互相通信底座。

## frida-gum

frida-gum是基于inline-hook实现的他还有很多丰富的功能比如用于代码跟踪 Stalker、用于内存访问监控的MemoryAccessMonitor，以及符号查找、栈回溯实现、内存扫描、动态代码生成和重定位等。

### Interceptor

Interceptor 是 inline-hook 的封装

| | |
| --- | --- |
