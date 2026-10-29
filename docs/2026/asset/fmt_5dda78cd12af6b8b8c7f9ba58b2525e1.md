
```
  uint32_t vmap_table_offset_ = 0u;

  uint32_t method_info_offset_ = 0u;

  QuickMethodFrameInfo frame_info_;

  // The code size in bytes. The highest bit is used to signify if the compiled

  // code with the method header has should_deoptimize flag.

  uint32_t code_size_ = 0u;

  // The actual code.

  uint8_t code_[0];

```

---

### 获取指令长度

`JitCompile->CommitCode->CommitCodeInternal`

```
  size_t alignment = GetInstructionSetAlignment(kRuntimeISA);

  size_t header_size = RoundUp(sizeof(OatQuickMethodHeader), alignment);

  size_t total_size = header_size + code_size;



  OatQuickMethodHeader* method_header = nullptr;

  uint8_t* code_ptr = nullptr;

  uint8_t* memory = nullptr;

  {

    ...

    {

      ....

      code_ptr = memory + header_size;



      std::copy(code, code + code_size, code_ptr);

      method_header = OatQuickMethodHeader::FromCodePointer(code_ptr);

      new (method_header) OatQuickMethodHeader(

          (stack_map != nullptr) ? code_ptr - stack_map : 0u,

          (method_info != nullptr) ? code_ptr - method_info : 0u,

          frame_size_in_bytes,

          core_spill_mask,

          fp_spill_mask,

          code_size);

      FlushInstructionCache(reinterpret_cast<char*>(code_ptr),

                            reinterpret_cast<char*>(code_ptr + code_size));

```

---

那么可得出 `CodeEntry - 4` 就是存放 Code Size 的地址

---

### Thumb 指令

- ARM32 模式根据 PC 来区别是 ARM 指令还是 Thumb 指令。

- 故 inline 的跳板也需要根据这两种模式进行分别实现

- 并且在跳转的时候要注意入口地址符合要求

```
bool isThumbCode(Size codeAddr) {

            return (codeAddr & 0x1) == 0x1;

}

```

---

### 指令对齐

- ART 使用 Thumb2 作为 Thumb 编译实现，Thumb2 指令又分为 Thumb16 和 Thumb32 指令

- 例如 `mov` 是 16 位指令，`ldr` 是 32 位指令

- 那么问题来了，我们一级跳板是 4 字节对齐的，原始指令是 2 字节对齐

- 所以在备份原指令的时候一定要注意指令的完整性

---

### PC 寄存器相关指令

先看一个 Java 函数:

```
public void setCloseOnTouchOutsideIfNotSet(boolean close) {

  if (!mSetCloseOnTouchOutside) {

        mCloseOnTouchOutside = close;

        mSetCloseOnTouchOutside = true;

    }

}

```

---

#### 汇编代码

![pc_relate.png](https://bbs.kanxue.com/upload/attach/202001/675677_BZUMMYFUAG8Z53F.jpg)

`CBNZ` 依赖于当前 `PC` 值。

虽然这种情况不多，但是检查是必要的，如果我们发现这种指令，直接转用入口替换即可。

---

## Xposed 兼容

- 为何需要兼容

- 可选方案

- 实现

- 性能优化

---

### 为何要兼容 Xposed？

- Xposed 已经拥有众多优秀的开源模块

- Xposed 特殊的(Callback)分发模式可支持多个模块同时 Hook 同一个函数

- 原版 Xposed 需要 Root 并且替换定制 ART，并且 8.0 已经停止更新

---

### 可选方案

我们目前的方案需要手写一个签名与原方法类似的 Hook 方法，而 Xposed API 则使用 Callback，所以我们需要运行期间动态生成方法。

- libffi 动态生成 Native 方法，将 origin 方法注册为 JNI 方法(Frida/Whale),相当于入口替换

- `DexMaker` 生成 Java 方法，第一次生成较慢，但不存在兼容性

- 或者自己根据调用约定从栈捞参数(epic)，Hook 时较快，但运行时较慢(一次 Hook 调入需要调用很多方法解析参数)，兼容性存疑

---

### `DexMaker`

最终我选择了 `DexMaker`

- 需要兼容前面的注解 API

- 性能可接受，使用缓存只需要生成一次

- 只需要完成参数打包的工作，生成的代码及其有限

- 稳定不会有兼容问题

- 运行时是最快的

---

### 性能优化

为了优化第一次生成 Hook 方法的性能缺陷，采取了一种折中的方法，既可以不需要从栈中解析参数，也可以不用动态生成方法。

- 依然是写一堆 Stub，不过这次 Stub 是 Hook 方法

- 为了兼容更多的参数情况(`Object` 类型以及基本类型(浮点除外))，将所有参数设为 `long`(32位为 `int`)，返回值类似

- 这样如果是基本类型参数，则可以简单转换得到值，`Object` 参数收到的则是内存地址

- 参数多的 stub 可以兼容较少参数的情况

---

写了一个 python 脚本以自动生成

```
//stub of arg size 3, index 13

    public static long stub_hook_13(long a0, long a1, long a2) throws Throwable {

        return hookBridge(getMethodId(3, 13), null , a0, a1, a2);

    }

```

---

#### 参数转换

```
public static Object addressToObject64(Class objectType, long address) {

        if (objectType == null)

            return null;

        if (objectType.isPrimitive()) {

            if (objectType == int.class) {

                return (int)address;

            } else if (objectType == long.class) {

                return address;

            } else if (objectType == short.class) {

                return (short)address;

            } else if (objectType == byte.class) {

                return (byte)address;

            } else if (objectType == char.class) {

                return (char)address;

            } else if (objectType == boolean.class) {

                return address != 0;

            } else {

                throw new RuntimeException("unknown type: " + objectType.toString());

            }

        } else {

            return SandHook.getObject(address);

        }

    }

```

---

#### 对象与地址互转

- 一切的基础在于，当一个对象在栈上时，是不会 Moving GC 的，保证了对象地址在传递中的有效性

- 地址 -> 对象, 使用 ART 内部的 `AddWeakGlobalRef`

- 对象 -> 地址, 直接使用 Java 层的 `Unsafe` 类

---

#### 结果

- 如此，几乎 9 成以上的函数 Hook 都走内部 Stub 的方式，Hook 耗时在 1ms 以内

- `DexMaker` 方式第一次大约需要 80ms 左右，后面直接加载约为 3 - 5ms，其实也能接受

---

## Inline 处理

- ART Inline 优化

- 阻止 JIT Inline

- 阻止 dex2oat Inline(Profile)

- 系统类中 Inline 的如何处理

---

### ART Inline 优化

ART 的 inline 类似其他语言的编译器优化，在 Runtime(JIT) 或者 `dex2oat` 期间, ART 将 “`invoke` 字节码指令” 替换成 callee 的方法体。

往往被 inline 的都是较为简单的方法。

---

### 阻止 JIT 期间的 Inline

观察 JIT Inline 的条件：

```
const CompilerOptions& compiler_options = compiler_driver_->GetCompilerOptions();

  if ((compiler_options.GetInlineDepthLimit() == 0)

      || (compiler_options.GetInlineMaxCodeUnits() == 0)) {

    return;

  }



 bool should_inline = (compiler_options.GetInlineDepthLimit() > 0)

      && (compiler_options.GetInlineMaxCodeUnits() > 0);

  if (!should_inline) {

    return;

  }



  size_t inline_max_code_units = compiler_driver_->GetCompilerOptions().GetInlineMaxCodeUnits();

  if (code_item->insns_size_in_code_units_ > inline_max_code_units) {

    VLOG(compiler) << "Method " << PrettyMethod(method)

                   << " is too big to inline: "

                   << code_item->insns_size_in_code_units_

                   << " > "

                   << inline_max_code_units;

    return false;

  }

```

---

当被 inline 方法的 code units 大于设置的阈值的时候，方法 Inline 失败。

这个阈值是 `CompilerOptions -> inline_max_code_units`。

经过搜索，`CompilerOptions` 一般与 `JitCompiler` 绑定：

```
class JitCompiler {

 public:

  static JitCompiler* Create();

  virtual ~JitCompiler();



..............

 private:

  std::unique_ptr<CompilerOptions> compiler_options_;



    ............

};



}  // namespace jit

}  // namespace art

```

---

ART 的 `JitCompiler` 为全局单例：

```
  // JIT compiler

  static void* jit_compiler_handle_;



  jit->dump_info_on_shutdown_ = options->DumpJitInfoOnShutdown();

  if (jit_compiler_handle_ == nullptr && !LoadCompiler(error_msg)) {

    return nullptr;

  }



  jit_compiler_handle_ = (jit_load_)(&will_generate_debug_symbols);



extern "C" void* jit_load(bool* generate_debug_info) {

  VLOG(jit) << "loading jit compiler";

  auto* const jit_compiler = JitCompiler::Create();

  CHECK(jit_compiler != nullptr);

  *generate_debug_info = jit_compiler->GetCompilerOptions()->GetGenerateDebugInfo();

  VLOG(jit) << "Done loading jit compiler";

  return jit_compiler;

}

```

---

#### 结论

ok，那么我们就得到了 “`static void* jit_compiler_handle_`“ 的 C++ 符号 “`_ZN3art3jit3Jit20jit_compiler_handle_E`“

最后修改里面的值就可以了。`SandHook.disableVMInline()`

---

### 阻止 dex2oat Inline

- Android N 以上默认的 ART 编译策略为 `speed-profile`

- 除了 JIT 期间的内联，系统在空闲时会根据这个所谓的 profile 进行 speed 模式的 dex2oat

- speed 模式包含 Inline 优化

- 现象是 App 多打开几次发现 Hook 不到了

---

#### 如何阻止

- 提前主动调用 `dex2oat`，附带 `--inline-max-code-units=0` 参数，要求 Dex 是动态加载的 (`ArtDexOptimizer`)

- 如果上述条件无法满足，则需要干掉 profile 文件以阻止空闲时 dex2oat，当然会对性能造成一定影响 (`SandHook.tryDisableProfile`)

---

#### profile

- 首先明确一点是 profile 是安装时创建的，并非由 ART 创建，ART 只负责读取以及写入

- profile 存储了方法的热度，以指导 dex2oat 进程编译高热度方法

- 如果你从 Google Play 下载某个热门 App，是有可能带有一份已经有内容的 profile 的

- 综上，我们只需要删除这个文件，如果不保险，则可以修改权限使之不可写

---

#### profile path

`/data/misc/profiles/cur/" + userId + "/" + selfPackageName + "/primary.prof"`
