
```cpp
if (dumped) {
  return JValue();
}

//++++++++++++++++++++++++++++

} while (!interpret_one_instruction);

// Record where we stopped.

shadow_frame.SetDexPC(inst->GetDexPc(insns));

return result_register;

}  // NOLINT(readability/fn_size)
```

这里就看到每个指令都执行了`PREAMBLE`函数。然后每个指令执行完都执行了`afterInstructionExecute`这个函数。在这里就可以判断，如果执行完的指令是`INVOKE_STATIC`，就可以直接`return`结束掉函数执行了。看看 Youpk 的处理：

```cpp
bool Unpacker::afterInstructionExecute(Thread* self,
                                        ArtMethod* method,
                                        uint32_t dex_pc,
                                        int inst_count) {
  const uint16_t* const insns = method->GetCodeItem()->insns_;
  const Instruction* inst = Instruction::At(insns + dex_pc);
  uint16_t inst_data = inst->Fetch16(0);
  Instruction::Code opcode = inst->Opcode(inst_data);

  if (inst_count == 2 &&
      (opcode == Instruction::INVOKE_STATIC ||
       opcode == Instruction::INVOKE_STATIC_RANGE) &&
      Unpacker::isRealInvoke(self, method)) {
    Unpacker::enableFakeInvoke();
    Unpacker::disableRealInvoke();
  }

  return false;
}
```

这里留意了一下。这个函数固定返回`false`，但是通过设置`enableFakeInvoke`和`disableRealInvoke`来控制下一个指令执行的时候来进行退出函数。我感觉这里退出应该也没啥问题。

到这里基本就走完大致的流程了。那么欣赏完别人的代码，可以开始我们的改造工作了。

### FartExt 更深的主动调用

和 Youpk 一样，第一步就是先把解释器改成使用 Switch 解释器。但是由于我使用的是 AOSP 10，所以发现修改部分果然不大一样了。

```cpp
#if ART_USE_CXX_INTERPRETER

static constexpr InterpreterImplKind kInterpreterImplKind = kSwitchImplKind;

#else

static constexpr InterpreterImplKind kInterpreterImplKind = kMterpImplKind;

#endif
```

发现这里变成可以通过编译参数来控制的了。搜索一下`ART_USE_CXX_INTERPRETER`的使用：

```cpp
if envTrue(ctx, "ART_USE_CXX_INTERPRETER") {
  cflags = append(cflags, "-DART_USE_CXX_INTERPRETER=1")
}
```

发现这个好像可以通过`cflags`来配置了。所以我修改了下`runtime`下的`Android.pb`。如果不想改全局的，也可以在源码里面直接判断是主动调用就强制走 Switch 解释器。

```text
cflags: [
  // ART is allowed to link to libicuuc directly
  // since they are in the same module
  "-DANDROID_LINK_SHARED_ICU4C",
  "-Wno-error",
  "-DART_USE_CXX_INTERPRETER=1"
],
```

接着就是`ArtMethod::Invoke`的时候不要直接结束了。但是这里我们需要留意的是，第一个参数的`Thread`是 fart 用来判断是否为主动调用的。为了让后面能正常执行，我就直接把第一个参数给赋值了。而后面的调用流程也是需要判断当前执行函数是否为主动调用。Youpk 是用线程和一个变量来控制判断是否为主动调用的。这里使用`result=111111`在后续判断是否为主动调用。

```cpp
extern "C" void fartextInvoke(ArtMethod* artmethod)
    REQUIRES_SHARED(Locks::mutator_lock_) {
  if (artmethod->IsNative() || artmethod->IsAbstract()) {
    return;
  }

  JValue result;

  // 模拟参数
  Thread* self = Thread::Current();

  uint32_t temp[100] = {0};
  uint32_t* args = temp;

  uint32_t args_size =
      (uint32_t)ArtMethod::NumArgRegisters(artmethod->GetShorty());

  if (!artmethod->IsStatic()) {
    args_size += 1;
  }

  // 靠这个值，在后续来判断当前函数是否为主动调用。
  result.SetI(111111);

  LOG(ERROR) << "fartext fartextInvoke";

  Unpacker_self_ = self;

  artmethod->Invoke(self,
                    args,
                    args_size,
                    &result,
                    artmethod->GetShorty());
}

void ArtMethod::Invoke(Thread* self,
                       uint32_t* args,
                       uint32_t args_size,
                       JValue* result,
                       const char* shorty) {
  ...

  // add
  if (result != nullptr && result->GetI() == 111111) {
    LOG(ERROR) << "fartext artMethod::Invoke Method "
               << this->PrettyMethod().c_str();

    if (IsStatic()) {
      art::interpreter::EnterInterpreterFromInvoke(
          self,
          this,
          nullptr,
          args,
          result,
          /*stay_in_interpreter=*/true);
    } else {
      // 注意这里是把非静态的也当静态的方式处理的。避免使用引用类型参数。
      art::interpreter::EnterInterpreterFromInvoke(
          self,
          this,
          nullptr,
          args + 1,
          result,
          /*stay_in_interpreter=*/true);
    }

    LOG(ERROR) << "fartext artMethod::Invoke Method Over "
               << this->PrettyMethod().c_str();

    self->PopManagedStackFragment(fragment);

    return;
  }

  // add end

  ...

}
```

这里有个问题是上面这种模拟参数的方式，碰到引用类型的参数会报错。所以在处理参数入栈的时候，也要进行判断处理一下。

```cpp
void EnterInterpreterFromInvoke(Thread* self,
                                ArtMethod* method,
                                ObjPtr<mirror::Object> receiver,
                                uint32_t* args,
                                JValue* result,
                                bool stay_in_interpreter) {
  ...

  if (!method->IsStatic()) {
    // add 避免使用引用类型的参数
    if (result != nullptr && result->GetI() == 111111) {
      shadow_frame->SetVReg(cur_reg, args[0]);
    } else {
      CHECK(receiver != nullptr);
      shadow_frame->SetVRegReference(cur_reg, receiver);
    }

    // add end

    // shadow_frame->SetVRegReference(cur_reg, receiver);
    ++cur_reg;
  }

  uint32_t shorty_len = 0;
  const char* shorty = method->GetShorty(&shorty_len);

  for (size_t shorty_pos = 0, arg_pos = 0;
       cur_reg < num_regs;
       ++shorty_pos, ++arg_pos, cur_reg++) {
    DCHECK_LT(shorty_pos + 1, shorty_len);

    switch (shorty[shorty_pos + 1]) {
      case 'L': {
        // add 避免使用引用类型的参数
        if (result != nullptr && result->GetI() == 111111) {
          shadow_frame->SetVReg(cur_reg, args[0]);
          break;
        }

        // add end

        ObjPtr<mirror::Object> o =
            reinterpret_cast<StackReference<mirror::Object>*>(
                &args[arg_pos])->AsMirrorPtr();

        shadow_frame->SetVRegReference(cur_reg, o);
        break;
      }

      case 'J':
      case 'D': {
        uint64_t wide_value =
            (static_cast<uint64_t>(args[arg_pos + 1]) << 32) |
            args[arg_pos];

        shadow_frame->SetVRegLong(cur_reg, wide_value);

        cur_reg++;
        arg_pos++;

        break;
      }

      default:
        shadow_frame->SetVReg(cur_reg, args[arg_pos]);
        break;
    }
  }

  ...

  if (LIKELY(!method->IsNative())) {
    // 这里把我们主动调用函数的标志继续往后面传递
    if (result != nullptr && result->GetI() == 111111) {
      JValue r = Execute(
          self,
          accessor,
          *shadow_frame,
          *result,
          stay_in_interpreter);

      if (result != nullptr) {
        *result = r;
      }

      return;
    } else {
      JValue r = Execute(
          self,
          accessor,
          *shadow_frame,
          JValue(),
          stay_in_interpreter);

      if (result != nullptr) {
        *result = r;
      }
    }
  }

  ...

}
```

接下来就开始修改解释器部分的逻辑了。我们只要做到几点处理，就可以搞定这种壳了。

1. 如果是主动调用并且第一个指令如果不是`GOTO`的，就直接脱壳并结束。
2. 如果是主动调用并且第一个指令是`GOTO`的，让它继续执行。
3. 如果第三个指令是`INVOKE_STATIC`的，执行完后直接结束掉。

接下来准备改代码，然后碰到一个问题。同样也是 AOSP 10 的版本导致的。Switch 解释器的逻辑发生了较大的变动。先看看变成了啥样子：

```cpp
template <bool do_access_check, bool transaction_active>
ATTRIBUTE_NO_SANITIZE_ADDRESS void ExecuteSwitchImplCpp(
    SwitchImplContext* ctx) {
  ...

  bool const interpret_one_instruction = ctx->interpret_one_instruction;

  while (true) {
    dex_pc = inst->GetDexPc(insns);
    shadow_frame.SetDexPC(dex_pc);

    TraceExecution(shadow_frame, inst, dex_pc);

    inst_data = inst->Fetch16(0);

    {
      bool exit_loop = false;

      InstructionHandler<do_access_check, transaction_active> handler(
          ctx,
          instrumentation,
          self,
          shadow_frame,
          dex_pc,
          inst,
          inst_data,
          exit_loop);

      // PREAMBLE 变成这种方式调用了
      if (!handler.Preamble()) {
        if (UNLIKELY(exit_loop)) {
          return;
        }

        if (UNLIKELY(interpret_one_instruction)) {
          break;
        }

        continue;
      }
    }

    switch (inst->Opcode(inst_data)) {
#define OPCODE_CASE(OPCODE, OPCODE_NAME, pname, f, i, a, e, v) \
  case OPCODE: {                                               \
    bool exit_loop = false;                                    \
    InstructionHandler<do_access_check, transaction_active> handler( \
        ctx,                                                     \
        instrumentation,                                         \
        self,                                                    \
        shadow_frame,                                            \
        dex_pc,                                                  \
        inst,                                                    \
        inst_data,                                               \
        exit_loop);                                              \
    handler.OPCODE_NAME();                                       \
    /* TODO: Advance 'inst' here, instead of explicitly in each handler */ \
    if (UNLIKELY(exit_loop)) {                                  \
      return;                                                    \
    }                                                            \
    break;                                                       \
  }

      DEX_INSTRUCTION_LIST(OPCODE_CASE)

#undef OPCODE_CASE
    }

    if (UNLIKELY(interpret_one_instruction)) {
      break;
    }
  }

  // Record where we stopped.
  shadow_frame.SetDexPC(inst->GetDexPc(insns));

  ctx->result = ctx->result_register;

  return;
}  // NOLINT(readability/fn_size)
```

看到了这两个部分都发生了较大的变化。那个超大的`case`都不见了。不过也只是处理的方式发生变化。我们跟着调整下就行了。

```cpp
template <bool do_access_check, bool transaction_active>
ATTRIBUTE_NO_SANITIZE_ADDRESS void ExecuteSwitchImplCpp(
    SwitchImplContext* ctx) {
  ...

  // add
  int32_t regvalue = ctx->result_register.GetI();

  // 这里很重要。需要把我们用来作为主动调用的值给改了。不然调用另外一个函数也会当成 fart 的主动调用的。
  ctx->result_register = JValue();

  int inst_count = -1;  // 当前第几个指令
  bool flag = false;    // 第一个指令是否为 goto
  // add end

  bool const interpret_one_instruction = ctx->interpret_one_instruction;

  while (true) {
    ...

    // add
    inst_count++;

    uint8_t opcode = inst->Opcode(inst_data);

    // 如果是主动调用
    if (regvalue == 111111) {
      // 第一个指令是 goto 的处理
      if (inst_count == 0) {
        if (opcode == Instruction::GOTO ||
            opcode == Instruction::GOTO_16 ||
            opcode == Instruction::GOTO_32) {
          LOG(ERROR)
              << "fartext ExecuteSwitchImplCpp Switch inst_count=0 "
                 "opcode==GOTO "
              << shadow_frame.GetMethod()->PrettyMethod().c_str();

          flag = true;
        } else {
          ...
        }
      }
    }
  }
}
```
