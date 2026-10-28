
```
static gint
test_arm64_stalker_fixture_follow_and_invoke (TestArm64StalkerFixture * fixture,
    StalkerTestFunc func,
    gint arg)
{
  GumAddressSpec spec;
  guint8 * code;
  GumArm64Writer cw;
  gint ret;
  GCallback invoke_func;

  spec.near_address = gum_strip_code_pointer (gum_stalker_follow_me);
  spec.max_distance = G_MAXINT32 / 2;
  //1. 创建新代码页用来储存接下来将要生成的代码
  code = gum_alloc_n_pages_near (1, GUM_PAGE_RW, &spec);

  gum_arm64_writer_init (&cw, code);
  //2. 保存寄存器
  gum_arm64_writer_put_push_reg_reg (&cw, ARM64_REG_X29, ARM64_REG_X30);
  gum_arm64_writer_put_mov_reg_reg (&cw, ARM64_REG_X29, ARM64_REG_SP);
  //3. 调用 gum_stalker_follow_me
  gum_arm64_writer_put_call_address_with_arguments (&cw,
      GUM_ADDRESS (gum_stalker_follow_me), 3,
      GUM_ARG_ADDRESS, GUM_ADDRESS (fixture->stalker),
      GUM_ARG_ADDRESS, GUM_ADDRESS (fixture->transformer),
      GUM_ARG_ADDRESS, GUM_ADDRESS (fixture->sink));

  /* call function -int func(int x)- and save address before and after call */
  gum_arm64_writer_put_ldr_reg_address (&cw, ARM64_REG_X0, GUM_ADDRESS (arg));
  fixture->last_invoke_calladdr = gum_arm64_writer_cur (&cw);
  //4. 调用原本的代码
  gum_arm64_writer_put_call_address_with_arguments (&cw, GUM_ADDRESS (func), 0);
  fixture->last_invoke_retaddr = gum_arm64_writer_cur (&cw);
  gum_arm64_writer_put_ldr_reg_address (&cw, ARM64_REG_X1, GUM_ADDRESS (&ret));
  gum_arm64_writer_put_str_reg_reg_offset (&cw, ARM64_REG_W0, ARM64_REG_X1, 0);
  //5. 取消跟踪
  gum_arm64_writer_put_call_address_with_arguments (&cw,
      GUM_ADDRESS (gum_stalker_unfollow_me), 1,
      GUM_ARG_ADDRESS, GUM_ADDRESS (fixture->stalker));

  gum_arm64_writer_put_pop_reg_reg (&cw, ARM64_REG_X29, ARM64_REG_X30);
  gum_arm64_writer_put_ret (&cw);

  gum_arm64_writer_flush (&cw);
  gum_memory_mark_code (cw.base, gum_arm64_writer_offset (&cw));
  gum_arm64_writer_clear (&cw);

  invoke_func =
      GUM_POINTER_TO_FUNCPTR (GCallback, gum_sign_code_pointer (code));
  invoke_func ();

  gum_free_pages (code);

  return ret;
}
```

逻辑比较清晰，相当于将原本的代码注入到另外一片内存，然后对其进行插桩执行，并在插桩代码中记录覆盖率相关的信息。这里我们先从 `gum_stalker_follow_me` 开始，它是一个由汇编实现的函数：

```
#ifdef __APPLE__

  .globl _gum_stalker_follow_me

_gum_stalker_follow_me:

#else

  .globl gum_stalker_follow_me

  .type gum_stalker_follow_me, %function

gum_stalker_follow_me:

#endif

  stp x29, x30, [sp, -16]!

  mov x29, sp

  mov x3, x30

#ifdef __APPLE__

  bl __gum_stalker_do_follow_me

#else

  bl _gum_stalker_do_follow_me

#endif

  ldp x29, x30, [sp], 16

  br x0
```

此处对于 Apple 架构和其他架构选用了两个不同的函数，不过我在源代码中并没有找到 `__gum_stalker_do_follow_me` 的声明或实现，这里我们将就这用 `_gum_stalker_do_follow_me` 进行理解吧：

```
gpointer
_gum_stalker_do_follow_me (GumStalker * self,
    GumStalkerTransformer * transformer,
    GumEventSink * sink,
    gpointer ret_addr)
{
  GumExecCtx * ctx;
  gpointer code_address;

  ctx = gum_stalker_create_exec_ctx (self, gum_process_get_current_thread_id (),
      transformer, sink);
  g_private_set (&gum_stalker_exec_ctx_private, ctx);

  ctx->current_block = gum_exec_ctx_obtain_block_for (ctx, ret_addr,
      &code_address);

  if (gum_exec_ctx_maybe_unfollow (ctx, ret_addr))
  {
    gum_stalker_destroy_exec_ctx (self, ctx);
    return ret_addr;
  }

  gum_event_sink_start (ctx->sink);
  ctx->sink_started = TRUE;

  return code_address + GUM_RESTORATION_PROLOG_SIZE;
}
```

关键内容跟入 `gum_event_sink_start`，里面是用于记录覆盖率信息的具体函数，分别有两套实现，一套是用 `quickjs`，另外一套是 `v8` 的实现，细节这里笔者就不深究了，大致逻辑如图：

![](https://bbs.kanxue.com/upload/attach/202308/924548_CTXSR93K4FSA6MM.png)

## 内存监控

这部分内容也不是笔者关心的重点，但笔者找了一圈似乎没找到 arm64 下的实现，倒是有 x86 平台下的测试样例。因此本文也就不过多赘述了，大致原理就是设置内存页的读写权限，从而在读写监控页面的时候引发中断来监视内容。

实现的内容分了 Windows 平台和 posix 平台两种，如下代码为 posix 平台：

```
GumMemoryAccessMonitor *
gum_memory_access_monitor_new (const GumMemoryRange * ranges,
    guint num_ranges,
    GumPageProtection access_mask,
    gboolean auto_reset,
    GumMemoryAccessNotify func,
    gpointer data,
    GDestroyNotify data_destroy)
{
  GumMemoryAccessMonitor * monitor;
  guint i;

  monitor = g_object_new (GUM_TYPE_MEMORY_ACCESS_MONITOR, NULL);
  monitor->ranges = g_memdup (ranges, num_ranges * sizeof (GumMemoryRange));
  monitor->num_ranges = num_ranges;
  monitor->access_mask = access_mask;
  monitor->auto_reset = auto_reset;
  monitor->pages_total = 0;

  for (i = 0; i != num_ranges; i++)
  {
    GumMemoryRange * r = &monitor->ranges[i];
    gsize aligned_start, aligned_end;
    guint num_pages;

    aligned_start = r->base_address & ~((gsize) monitor->page_size - 1);
    aligned_end = (r->base_address + r->size + monitor->page_size - 1) &
        ~((gsize) monitor->page_size - 1);
    r->base_address = aligned_start;
    r->size = aligned_end - aligned_start;

    num_pages = r->size / monitor->page_size;
    g_atomic_int_add (&monitor->pages_remaining, num_pages);
    monitor->pages_total += num_pages;
  }

  monitor->notify_func = func;
  monitor->notify_data = data;
  monitor->notify_data_destroy = data_destroy;

  return monitor;
}
```

```
gboolean
gum_memory_access_monitor_enable (GumMemoryAccessMonitor * self,
    GError ** error)
{
  if (self->enabled)
    return TRUE;
  // ...
  self->exceptor = gum_exceptor_obtain ();
  gum_exceptor_add (self->exceptor, gum_memory_access_monitor_on_exception,
      self);
  // ...
}
```

### `gum-js`

这部分主要是做一个扫盲。细节可以参考 evilpan 大佬的文章，里面也大致介绍了 `gum-js` 的实现。

简单来说就说，V8 支持对 JavaScript 的动态解析，并能够将其抽象到 C 语言层面进行调用。

```
Local<String> attach_name =
    String::NewFromUtf8Literal(GetIsolate(), "Attach");
// 判断对象是否存在，以及类型是否是函数
Local<Value> attach_val;
if (!context->Global()->Get(context, attach_name).ToLocal(&attach_val) ||
    !attach_val->IsFunction()) {
  return false;
}
// 如果是，则转换为函数类型
Local<Function> attach_func = attach_val.As<Function>();

// 将调用参数封装为 JS 对象
Local<Object> obj =
    templ->NewInstance(GetIsolate()->GetCurrentContext()).ToLocalChecked();
obj->SetInternalField(0, 0xdeadbeef);
// 使用自定义的参数调用该 JS 函数，并获取返回结果
TryCatch try_catch(GetIsolate());
const int argc = 1;
Local<Value> argv[argc] = {obj};
Local<Value> result;
attach_func->Call(context, context->Global(), argc, argv).ToLocal(&result);
```

通过这种交互面板，就能给允许用户动态传入脚本进行执行了。之所以选择的是 JavaScript 而不是其他语言，就笔者估计来看，大致上有两个原因（以下内容为笔者的猜测，各位读者可以当看个乐子）：

- 首先编译型的语言肯定是不行了，因为它不支持动态调整，每次都要编译一份再运行肯定不太灵活
- 而在解释型语言里就比较看重运行效率了。笔者曾写过一篇 V8 优化引擎 Turbofan 的原理分析，从其中可以大致理解到，V8 对 JavaScript 的优化效率几乎已经达到理论上限了，这意味着在各种解释型语言里，JS 的效率可能是最高的（这是区分场景的，但在 Frida 里，可能它是最合适的）

> Turbofan 可参考本文：<https://bbs.kanxue.com/thread-273791.htm>

# 参考

[410K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9@1i4K6u0r3x3o6g2Q4x3V1k6X3M7X3W2V1j5g2)9J5k6r3W2F1N6r3g2J5L8X3q4D9i4K6u0r3i4K6t1K6M7%4c8S2L8r3E0W2M7R3`.`.](https://bbs.kanxue.com/elink@f34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9@1i4K6u0r3x3o6g2Q4x3V1k6X3M7X3W2V1j5g2)9J5k6r3W2F1N6r3g2J5L8X3q4D9i4K6u0r3i4K6t1K6M7%4c8S2L8r3E0W2M7R3`.`.)
