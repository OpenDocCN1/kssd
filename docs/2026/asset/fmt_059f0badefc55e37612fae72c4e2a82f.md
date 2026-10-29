
```python
def _hook(self, mu, address, size, user_data):

    # Check if instruction is "IT AL"

    if size != 2 or self._emu.mu.mem_read(address, size) != b"\xE8\xBF":

        return



    # Find hook.

    hook_id = self._emu.mu.reg_read(UC_ARM_REG_R4)

    hook_func = self._hooks[hook_id]



    # Call hook.

    try:

        hook_func(self._emu)

    except:

        # Make sure we catch exceptions inside hooks and stop emulation.

        mu.emu_stop()

        raise
```

## 使用 keystone 动态编译 stub

[`Keystone`](https://bbs.kanxue.com/elink@587K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8k6i4W2K6N6r3!0F1k6g2)9J5k6r3g2F1k6$3W2F1k6g2)9J5c8X3E0W2P5i4y4@1L8$3&6W2) 是一款很牛逼的汇编框架，Unicorn的兄弟！

使用`pip`安装

```
pip install keystone
```

`keystone` 使用方法十分简单，不需要额外学习即可理解接下来的代码。`write_function`函数将python的`func`函数映射到Unicorn虚拟机，并返回虚拟机中的函数地址， 如果在虚拟机中调用该地址就会被Python捕获并调用`func`函数。

```python
def write_function(self, func):

    # Get the hook id.
    hook_id = self._get_next_id()
    hook_addr = self._hook_current


    # Create the ARM assembly code.
    # Make sure to update STACK_OFFSET if you change the PUSH/POP.
    asm = "PUSH {R4,LR}\n" \
          "MOV R4, #" + hex(hook_id) + "\n" \
          "IT AL\n" \
          "POP {R4,PC}"


    asm_bytes_list, asm_count = self._keystone.asm(bytes(asm, encoding='ascii'))


    if asm_count != 4:
        raise ValueError("Expected asm_count to be 4 instead of %u." % asm_count)


    # Write assembly code to the emulator.
    self._emu.mu.mem_write(hook_addr, bytes(asm_bytes_list))


    # Save results.
    self._hook_current += len(asm_bytes_list)
    self._hooks[hook_id] = func


    return hook_addr
```

## symbol hook

上一个小结中讲到如何实现stub，那么如何hook 呢？AndroidNativeEmu实现了Symbol Hook。这种HOOK与平时常见的IAT HOOK、GOT Hook 原理是一样的。 调用`add_symbol_hook`函数，可以将符号和新地址关联起来， 模块加载的时候，查找符号优先从该表中获取地址。 这种实现方式有点bug，比如有一些Native 程序，自身会got hook，那么这种方式可能就会出问题。

## 例子：使用Hook 实现 AAssetManager_open

```
modules.add_symbol_hook('AAssetManager_open', hooker.write_function(self.AAssetManager_open) + 1)

@native_method

def AAssetManager_open(self, mu, mgr, filename_ptr, mode):
    filename = memory_helpers.read_utf8(mu, filename_ptr)
    filename = "assert/" + filename
    logger.info("AAssetManager_open(%s,%d)" % (filename, mode))
    fd = self._open_file(filename, 0, mode)
    if fd == -1:
        fd = 0
    return fd
```

## python 实现 Native 函数修饰 @native_method

大家可能对Python 装饰器的语法不太熟悉，可以参考廖雪峰老师博客：[`Python 装饰器`](https://bbs.kanxue.com/elink@174K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9K9h3q4G2P5s2g2W2k6X3g2F1k6#2)9J5k6h3y4G2L8g2)9J5c8Y4N6A6K9$3W2Q4x3V1j5I4x3o6p5$3z5e0f1&6y4U0j5K6y4U0l9J5y4o6l9H3i4K6u0r3x3e0l9I4y4K6b7#2x3e0j5$3x3U0t1&6y4e0f1^5y4l9`.`.)

简单的讲，装饰器可以自定义修改函数，为目标函数套一层外壳。`native_method`主要功能是处理参数数据，这样就能很优雅地编写Native 函数，不用再函数中冗杂地调用寄存器读写函数。

```python
def native_method(func):
    def native_method_wrapper(*argv):
        """
        :type self
        :type emu androidemu.emulator.Emulator
        :type mu Uc
        """
        emu = argv[1] if len(argv) == 2 else argv[0]
        mu = emu.mu


        args = inspect.getfullargspec(func).args  # 判断参数个数
        args_count = len(args) - (2 if 'self' in args else 1)


        if args_count < 0:
            raise RuntimeError("NativeMethod accept at least (self, mu) or (mu).")


        native_args = []


        if args_count >= 1:  # 从寄存器取参数
            native_args.append(mu.reg_read(UC_ARM_REG_R0))


        if args_count >= 2:
            native_args.append(mu.reg_read(UC_ARM_REG_R1))


        if args_count >= 3:
            native_args.append(mu.reg_read(UC_ARM_REG_R2))


        if args_count >= 4: 
            native_args.append(mu.reg_read(UC_ARM_REG_R3))


        sp = mu.reg_read(UC_ARM_REG_SP) # 参数大于4个，从栈中取参数
        sp = sp + STACK_OFFSET  # Need to offset by 4 because our hook pushes one register on the stack. 


        if args_count >= 5:
            for x in range(0, args_count - 4):
                native_args.append(int.from_bytes(mu.mem_read(sp + (x * 4), 4), byteorder='little'))


        if len(argv) == 1:
            result = func(mu, *native_args)
        else:
            result = func(argv[0], mu, *native_args)


        if result is not None:
            native_write_arg_register(emu, UC_ARM_REG_R0, result)
        else:
            mu.reg_write(UC_ARM_REG_R0, JNI_ERR)
    return native_method_wrapper
```

## 小结

这篇文章讲了如何实现 Unicorn 内部的函数级Hook，原理是在模块加载重定位阶段，填充stub函数地址到目标函数的 got表。 stub函数是Unicorn 内部环境与外部环境的一个桥梁，使用小巧的IT AL和R4寄存器实现交互，像极了系统调用。使用Python的装饰器，简化了Python 编写Hook 函数的难度。

# Unicorn 调用SO之系统调用

系统调用是操作系统给应用程序提供的最底层的基础接口。应用程序读写文件、访问网络等操作都需要操作系统支持。

Unicorn 拦截系统调用只需要添加`UC_HOOK_INTR`的callback，该callback的参数定义如下:

```
typedef void (*uc_cb_hookintr_t)(uc_engine *uc, uint32_t intno, void *user_data);
```

- `intno`: 中断号
- `user_data`: user data

我们只需要用`hook_add` 添加一个 `UC_HOOK_INTR` 的callback 就能够处理中断了。根据`intno` 中断号分发不同的中断处理函数。

## 实现getpid

```python
self._syscall_handler.set_handler(0xe0, "getpid", 0, self._getPid)

def _getpid(self, mu):
    return 0x1122
```

## 实现文件系统需要拦截的syscall

我们将在文件系统章节详细讨论如何实现文件系统。

```python
syscall_handler.set_handler(0x3, "read", 3, self._handle_read)
syscall_handler.set_handler(0x4, "write", 3, self._handle_write)
syscall_handler.set_handler(0x5, "open", 3, self._handle_open)
syscall_handler.set_handler(0x6, "close", 1, self._handle_close)
syscall_handler.set_handler(0x92, "writev", 3, self._handle_writev)
syscall_handler.set_handler(0xC5, "fstat64", 2, self._handle_fstat64)
syscall_handler.set_handler(0x142, "openat", 4, self._handle_openat)
syscall_handler.set_handler(0x147, "fstatat64", 4, self._handle_fstatat64)
syscall_handler.set_handler(0x14e, "faccessat", 4, self._faccessat)
syscall_handler.set_handler(0x14d, "fchmodat", 4, self._fchmodat)
syscall_handler.set_handler(0x8c, "_llseek", 5, self._llseek)
syscall_handler.set_handler(0x13, "lseek", 3, self._lseek)
```

## 实现内存管理的syscall

```python
self._syscall_handler.set_handler(0x5B, "munmap", 2, self._handle_munmap)
self._syscall_handler.set_handler(0x7D, "mprotect", 3, self._handle_mprotect)
self._syscall_handler.set_handler(0xC0, "mmap2", 6, self._handle_mmap2)
self._syscall_handler.set_handler(0xDC, "madvise", 3, self._handle_madvise)
```

## 其它杂项syscall

这些实现都非常的简单，就不过多展开了。

```python
self._syscall_handler.set_handler(0x4E, "gettimeofday", 2, self._handle_gettimeofday)
self._syscall_handler.set_handler(0xAC, "prctl", 5, self._handle_prctl)
self._syscall_handler.set_handler(0xF0, "futex", 6, self._handle_futex)
self._syscall_handler.set_handler(0x107, "clock_gettime", 2, self._handle_clock_gettime)
self._syscall_handler.set_handler(0x119, "socket", 3, self._socket)
self._syscall_handler.set_handler(0x11b, "connect", 3, self._connect)
self._syscall_handler.set_handler(0x159, "getcpu", 3, self._getcpu)
self._syscall_handler.set_handler(0x14e, "faccessat", 4, self._faccessat)
self._syscall_handler.set_handler(0x14, "getpid", 0, self._getpid)
self._syscall_handler.set_handler(0xe0, "gettid", 0, self._gettid)
self._syscall_handler.set_handler(0x180, "null1", 0, self._null)
self._syscall_handler.set_handler(0x7e, "sigprocmask", 0, self._null)
self._syscall_handler.set_handler(0xaf, "rt_sigprocmask", 0, self._null)
self._syscall_handler.set_handler(0x10c, "sigaction", 0, self._tgkill)
self._syscall_handler.set_handler(0x43, "sigaction", 0, self._sigaction)
self._syscall_handler.set_handler(0xf8, "exit", 0, self._null)
self._syscall_handler.set_handler(0x16e, "accept4", 0, self.accept4)
```

# Unicorn 调用SO之文件系统

如果想尽可能完美的模拟Android Native程序，那就必须加入虚拟文件系统的支持。虚拟文件系统可以将Unicorn 虚拟机内部访问文件的操作映射到主机。

## 思路

Hook 拦截文件操作相关的系统调用，转换成对主机的文件操作。

为了安全性，我们要在主机上划分一个用于专门虚拟文件系统的目录。 处理系统调用的时候，将路径转换成虚拟文件系统目录的路径。

## 实现

需要处理的syscall已经在上一篇文章介绍过了
