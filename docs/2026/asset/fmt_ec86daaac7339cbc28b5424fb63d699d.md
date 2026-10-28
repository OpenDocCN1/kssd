
行号 1-25：

```
// guard.cc
extern "C" void __cxa_guard_release (__guard *g) noexcept
{
  // If __atomic_* and futex syscall are supported, don't use any global
  // mutex.
  if (__gnu_cxx::__is_single_threaded())
  {
    int *gi = (int *) (void *) g;
    *gi = _GLIBCXX_GUARD_BIT; // 1
    return;
  }
  else
  {
    int *gi = (int *) (void *) g;
    const int guard_bit = _GLIBCXX_GUARD_BIT;
    const int waiting_bit = _GLIBCXX_GUARD_WAITING_BIT;
    int old = __atomic_exchange_n (gi, guard_bit, __ATOMIC_ACQ_REL);
    if ((old & waiting_bit) != 0)
      syscall (SYS_futex, gi, _GLIBCXX_FUTEX_WAKE, INT_MAX);
    return;
  }
  set_init_in_progress_flag(g, 0);
  _GLIBCXX_GUARD_SET_AND_RELEASE (g);
}
```

## 像 Windows MSVC 平台那样分 CASE 分析

初始状态 `*g = 0`，`__gnu_cxx::__is_single_threaded() = true`。`__gnu_cxx::__is_single_threaded()` 在 `pthread_create` 函数已经被设置为了 `false`，这个操作在执行 `func_b` 函数之前。

### CASE ONE

- a. 若 `t1` 线程首先进入 `func_b`，执行顺序如下：`__cxa_guard_acquire` -> `res = b + a` -> `__cxa_guard_release`，此时 `*g = 0x1`。
- b. `t2` 线程进入时，因为 `*g = 0x1`，`t2` 在 `__cxa_guard_acquire` 函数的第 41 行返回，因返回值为 0，因此不会对 `res` 再次初始化。`t3` 和 `t4` 所遇情况，同 `t2` 线程。

### CASE TWO

`res` 的初始化需要较长时间。此时的情况如下：

- a. `t1` 线程，`__cxa_guard_acquire` -> `res` 的初始化流程，此时 `*g = 0x100`。
- b. `t2` 线程进入到 `__cxa_guard_acquire` 函数中，会执行 `syscall` 系统调用进行阻塞，此时 `*g = 0x10100`。直到在 `t1` 调用 `__cxa_guard_release` 解除 `t2` 线程的阻塞，`*g = 1`。
- c. 若 `t3` 或 `t4` 线程同 `t2` 线程，也执行 `func_b` 函数，则会被 `syscall(SYS_futex)` 系统调用阻塞。若 `t3` 或 `t4` 在 `t1` 执行 `__cxa_guard_release` 函数后调用 `func_b`，这种情况同 CASE ONE 下。

## 总结

- 使用常量初始化静态局部变量，MSVC和G++实现的方法相同
- 使用变量初始化静态局部变量，包括：多线程和单线程，CASE ONE和CASE TWO各个线程面对情况是一样的，只不过是两种平台的同步机制不同而已。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2023-12-9 22:04 被 baolongshou 编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm) [#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

上传的附件：

- [windows.exe](javascript:void(0);)（1.83MB，13次下载）
- [linux](javascript:void(0);)（2.46MB，11次下载）
