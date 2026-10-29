
```c
// invoke-super 位描述符如下：A|G|op BBBB F|E|D|C
// methodCallRange depending on whether this is a "/range" instruction.
GOTO_TARGET(invokeSuper, bool methodCallRange)
{
    Method* baseMethod;
    u2 thisReg;
    EXPORT_PC();

    // 取出 AG 的值
    vsrc1 = INST_AA(inst);
    // 要调用的 method 索引
    ref = FETCH(1);
    // 要作为参数的寄存器的索引
    vdst = FETCH(2);

    // 取出 this 寄存器的索引，比如 thisReg 为 3 的话，表示第三个寄存器，放的是 this 参数。
    if (methodCallRange) {
        ILOGV("|invoke-super-range args=%d @0x%04x {regs=v%d-v%d}",
              vsrc1, ref, vdst, vdst + vsrc1 - 1);
        thisReg = vdst;
    } else {
        ILOGV("|invoke-super args=%d @0x%04x {regs=0x%04x %x}",
              vsrc1 >> 4, ref, vdst, vsrc1 & 0x0f);
        thisReg = vdst & 0x0f;
    }

    // 检查 this 是否为空
    if (!checkForNull((Object*) GET_REGISTER(thisReg)))
        GOTO_exceptionThrown();

    // 解析要调用的方法
    baseMethod = dvmDexGetResolvedMethod(methodClassDex, ref);
    if (baseMethod == NULL) {
        baseMethod = dvmResolveMethod(curMethod->clazz, ref, METHOD_VIRTUAL);
        if (baseMethod == NULL) {
            ILOGV("+ unknown method or access denied");
            GOTO_exceptionThrown();
        }
    }

    if (baseMethod->methodIndex >= curMethod->clazz->super->vtableCount) {
        /*
         * Method does not exist in the superclass. Could happen if
         * superclass gets updated.
         */
        dvmThrowNoSuchMethodError(baseMethod->name);
        GOTO_exceptionThrown();
    }

    methodToCall = curMethod->clazz->super->vtable[baseMethod->methodIndex];

#if 0
    if (dvmIsAbstractMethod(methodToCall)) {
        dvmThrowAbstractMethodError("abstract method not implemented");
        GOTO_exceptionThrown();
    }
#else
    assert(!dvmIsAbstractMethod(methodToCall) ||
           methodToCall->nativeFunc != NULL);
#endif

    LOGVV("+++ base=%s.%s super-virtual=%s.%s",
          baseMethod->clazz->descriptor, baseMethod->name,
          methodToCall->clazz->descriptor, methodToCall->name);

    assert(methodToCall != NULL);

    // 调用方法
    GOTO_invokeMethod(methodCallRange, methodToCall, vsrc1, vdst);
}
GOTO_TARGET_END
```

解析完要调用的方法后，跳转到 `invokeMethod` 结构来执行函数调用。`invokeMethod` 为要调用的函数创建虚拟寄存器栈，新的寄存器栈和之前的栈是重叠的。然后重新设置解释器执行环境的参数，调用 `FINISH(0)` 执行函数。

```cpp
GOTO_TARGET(invokeMethod, bool methodCallRange,
            const Method* _methodToCall, u2 count, u2 regs)
{
    // 节选
    if (!dvmIsNativeMethod(methodToCall)) {
        /*
         * "Call" interpreted code. Reposition the PC, update the
         * frame pointer and other local state, and continue.
         */
        curMethod = methodToCall; // 设置要调用的方法
        self->interpSave.method = curMethod;
        methodClassDex = curMethod->clazz->pDvmDex;
        pc = methodToCall->insns; // 重置 pc 到要调用的方法
        fp = newFp;
        self->interpSave.curFrame = fp;

#ifdef EASY_GDB
        debugSaveArea = SAVEAREA_FROM_FP(newFp);
#endif

        self->debugIsMethodEntry = true; // profiling, debugging
        ILOGD("> pc <-- %s.%s %s", curMethod->clazz->descriptor,
               curMethod->name, curMethod->shorty);
        DUMP_REGS(curMethod, fp, true); // show input args
        FINISH(0); // jump to method start
    }
}
```

## 数字壳解释器分析

### 数字壳解释执行前的准备工作

进入解释器的流程为 `onCreate->sub_D930->sub_3FE5C->sub_3FF5C`。`sub_3FF5C` 是真正的解释器入口，`sub_D930` 和 `sub_3FE5C` 负责执行前的准备工作。这部分准备工作和 Dalvik 解释器的准备工作类似。

#### `sub_D930`

`sub_D930` 分为两部分：调用 `sub_66BD4` 之前为第一部分，之后为第二部分。这两部分主要做的事情如下：

- 第一部分：
  1. JNI 的一些初始化工作，例如 `FindClass`、`GetMethodID` 等。
  2. 利用 `java.lang.Thread.getStackTrace` 获取调用当前方法的类名以及函数名。

- 第二部分：
  1. 调用 `sub_66BD4` 获取一些全局信息，以及待解释函数的信息。
  2. 构建解释器的虚拟寄存器栈。
  3. 解析待解释函数的简单声明，将函数参数放入虚拟寄存器。

主要分析第二部分。首先引入一些数据结构，这类数据结构是动态分析出来的，有些字段的含义还不清楚，标记为 `unkonw`。

```cpp
struct GlobalInfo {
    DexInfo *dex_info;
    vector<MethodInfo*> method_list;
};

struct DexInfo {
    void *dexBase; // 指向内存中 dex 的基址
    int unknow[12];
    void *dexHeader; // 指向 dex header，用来解析 dex
    .....
    optable[];
};

struct MethodInfo {
    int dex_type_id; // 该方法在 dex 文件中 DexTypeId 列表的索引
    int dex_class_def_id; // 该方法所在类在 dex 文件中 DexClassDef 列表的索引
    int unknow;
    int codeoff; // 该方法的 DexCode 结构距离 dex 头的偏移
};

struct StackInfo {
    JNIEnv *jni_env;
    int registerSize; // 函数所需寄存器数量
    DexCode *dexCode; // 指向 DexCode 结构
    int unkonw;
    void *registerSpace; // 放入原始参数，malloc(4 * registerSize)
    void *registerSpace2; // 新建一个 object 引用上面的参数，malloc(4 * registerSize)
    int unkonw2;
    char key; // 解密指令的 key
    char origin_key; // 计算解密指令 key 所需的一个数据
};
```

`sub_66BD4` 返回的是指向 `GlobalInfo` 结构的指针。这个全局信息里面包含了 dex 相关的信息和待解释函数的信息。有了这些信息就可以构建解释器所需的虚拟寄存器栈，完成准备工作。

```cpp
int __fastcall sub_D930(unsigned int a1, JNIEnv *a2, _DWORD *a3)
{
    // 节选

    // 获取全局信息
    global_info = sub_66BD4(v109, &v114);

    if (v114 & 1)
        j_j_j__ZdlPv(*(v47 + 5));

    if (global_info &&
        (v52 = *(global_info + 4), (*(global_info + 8) - v52) >> 2 > v4))
    {
        v53 = v52 + 4 * v4;
        a2c = v3;
        method_info = *v53;
        dexInfo = *global_info;
        DexCode = (**global_info + *( *v53 + 12)); // DexCode

        ((*v3)->PushLocalFrame)();

        // 创建并初始化 StackInfo 结构
        stackinfo = malloc_1(0x20u); // 创建 StackInfo 结构
        v56 = *DexCode;
        *stackinfo = a2c; // stackinfo->jni_env
        *(stackinfo + 4) = v56; // stackinfo->registerSize
        *(stackinfo + 8) = DexCode; // stackinfo->dexCode
        *(stackinfo + 12) = 0;

        v57 = 4 * v56;
        v58 = malloc_0(4 * v56); // 创建虚拟寄存器栈
        *(stackinfo + 16) = v58;
        memset(v58, v57, 0);

        v59 = malloc_0(v57); // 创建虚拟寄存器栈
        *(stackinfo + 20) = v59;
        memset(v59, v57, 0);

        *(stackinfo + 29) = 0;
        *(stackinfo + 28) = 0;

        v60 = sub_3E268();
        v61 = DexCode;
        *(stackinfo + 28) =
            *DexCode ^ *(v60 + 24) ^ *(method_info + 4) ^ *method_info;
        // stackinfo->key
        *(stackinfo + 29) = *(v60 + 24);
    }
}
```

通过创建并初始化 `StackInfo` 结构，就完成了虚拟寄存器栈的创建，可以看到这里分配了两个虚拟寄存器栈。后面调试发现主要使用的是第二个虚拟寄存器栈。猜测这两个虚拟寄存器栈和 Dalvik 拥有两个虚拟寄存器栈一样，原因是一个用来执行 native 方法，一个用来执行 Java 方法。

创建完虚拟寄存器栈的下一步工作就是放入函数参数。

```cpp
int __fastcall sub_D930(unsigned int a1, JNIEnv *a2, _DWORD *a3)
{
    // 节选

    v107 = stackinfo;
    proto = (*dexInfo
             + *( *dexInfo
                  + *(dexInfo[4] + 60)
                  + 4 * *( *dexInfo
                           + *(dexInfo[4] + 76)
                           + 12 * *( *dexInfo
                                    + *(dexInfo[4] + 92)
                                    + 8 * *(method_info + 4)
                                    + 2))));

    do
        v63 = proto++; // 方法的简单声明：VL
    while (*v63 < 0);

    v64 = j_strlen(proto);
    v99 = v64;
    v65 = *v61 - v61[1]; // 函数内部使用寄存器个数：DexCode.registerSize - DexCode.insSize
    v105 = v65 - 1;
    v66 = v95 + 1;
    v67 = v107;

    if (!*(method_info + 8)) {
        v105 = v65;
        v68 = 4 * v65;
        v69 = *(*(v107 + 20) + v68); // 函数的第一个参数 thisReg
        v96 = *v95;

        // 节选
        v111 = v68;
        v73 = (*(v67 + 20) + v68);
        v74 = *v73;

        if (*v73 && !*(v74 + 4)) {
            *(v74 + 4) = 1;
            v78 = v111;
            v77 = v96;
            **(*(v67 + 20) + v111) = v96;
        } else {
            v75 = v64;
            v76 = malloc_1(8u); // 重新创建了一个 MainActivity object，并放入第二个 register space 位置
            *v76 = 0;
            *(v76 + 4) = 0;
            v77 = v96;
            *v76 = v96;
            *(v76 + 4) = 1;
            *v73 = v76;
            v67 = v107;
            v64 = v75;
            v78 = v111;
        }

        *(*(v67 + 16) + v78) = v77; // 直接将 Activity 实例放入第一个 register space 的相应位置
    }

    v108 = v67;

    if (v64 >= 2) { // 处理除了 this 之外的其他参数
        v79 = 1;
        v112 = 1;

        do {
            v80 = v63[v79++ + 1];

            if (v80 > 89) {
                if (v80 == 90)
                    *(*(v67 + 16) + 4 * (v112++ + v105)) = *v66;
                ++v66;
            } else {
                v81 = v80 - 66;

                if (v81 <= 0x11)
                    JUMPOUT(__CS__, *(&off_E308 + v81) + 58120);
                // 调用 jni->newGlobalRef，构造将传递给 onCreate 的参数
            }
        } while (v79 < v64);
    }
}
```

这里是从 dex 文件中提取出函数的简要声明，`onCreate` 的简单声明为 `VL`，然后根据声明放入参数。首先放入 `this` 参数，然后根据后续参数的类型，将参数放入相应位置。和 Dalvik 虚拟机流程类似。

#### `sub_3FE5C`

`sub_D930` 完成了虚拟寄存器栈的构建并放入参数后，调用了 `sub_3FE5C`。`sub_3FE5C` 主要负责初始化解释器的一些状态，主要是 `InterpState` 结构。

```asm
struct InterpState {
    void *pc;
    char key;
    DexInfo *dex_info;
};

LOAD:0003FF18 BL malloc_1 ; 创建 InterpState 结构
LOAD:0003FF1C PUSH {R4}
LOAD:0003FF1E POP {R1}
LOAD:0003FF20 PUSH {R0}
LOAD:0003FF22 POP {R4}
LOAD:0003FF24 STR R4, [SP,#0x74+var_24]
LOAD:0003FF26 LDRB R0, [R1,#0x1C]
LOAD:0003FF28 ADDS R6, #0x10
LOAD:0003FF2A STR R6, [SP,#0x74+var_2C]
LOAD:0003FF2C STR R6, [R4] ; InterpState->pc
LOAD:0003FF2E STRB R0, [R4,#4] ; InterpState->key
LOAD:0003FF30 STR R5, [R4,#8] ; InterpState->dex_info
```

### 数字壳解释器模型

`sub_D930` 和 `sub_3FE5C` 完成了解释器的准备工作。`sub_3FF5C` 负责解释执行。数字壳的解释器模型就是上面提到的那种最直观的模型：循环取指令，然后判断指令类型，去相应分支执行；执行完成后，再返回到 switch 执行下一条指令。
