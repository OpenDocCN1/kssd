
| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16 | 17 | 18 | 19 | 20 | 21 | 22 | 23 | 24 | 25 | 26 | 27 | 28 | 29 | 30 | 31 | 32 | 33 | 34 | 35 | 36 | 37 | 38 | 39 | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 | 48 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|

1. 获取 `HookZz` 对象

```java
IHookZz hookZz = HookZz.getInstance(emulator);
```

加载 `HookZz`，支持 `inline hook`，文档看 <https://github.com/jmpews/HookZz>。

2. `enable hook`

```java
hookZz.enable_arm_arm64_b_branch();
```

测试 `enable_arm_arm64_b_branch`，可有可无。

```java
index = 0;

hookZz.replace(module.findSymbolByName("lrand48"), new ReplaceCallback() {
    @Override
    public void postCall(Emulator<?> emulator, HookContext context) {
        ((EditableArm32RegisterContext) context).setR0(0x12345678);
        int ptrace_args0 = context.getIntArg(0);
        System.out.println("lrand48=" + ptrace_args0);
    }
}, true);
```

`aesdecode hook`：

```java
hookZz.wrap((module.base) + 0x39634 + 1, new WrapCallback<RegisterContext>() {
    UnidbgPointer addr = null;

    @Override
    public void preCall(Emulator<?> emulator, RegisterContext ctx, HookEntryInfo info) {
        addr = ctx.getPointerArg(0);
        UnidbgPointer pointerArg = ctx.getPointerArg(1);
        UnidbgPointer pointer = pointerArg.getPointer(12);
        int anInt = pointerArg.getInt(8);
        byte[] byteArray = pointer.getByteArray(0, anInt);
        String s = xuzi1(byteArray);
        System.out.println("aes aesdecode= " + s);
    }

    @Override
    public void postCall(Emulator<?> emulator, RegisterContext ctx, HookEntryInfo info) {
        byte[] aaaa = addr.getPointer(0).getPointer(12).getByteArray(0, 0x30);
        String s = xuzi1(aaaa);
        System.out.println("aes aesdecode1= " + s);
    }
});
```

同理，此框架也支持导出函数 `HOOK` 以及 `InlineHOOK`。有了这个方法，在你分析一些函数的时候，可以充当 `Log` 的效果，或者强行改变一些函数的返回值，让你更容易地分析。比如本例中笔者改变了 `Lrand48` 的返回值，让函数每次都强行返回 `0x12345678`，这样在逆向分析的时候能让一些不确定性变成可控性。

---

最后感谢 `Unidbg` 作者带来这么好的工具，并且希望大家能一起进步，在逆向的道路越走越远。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-8-27 12:05

被至尊小仙侠编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)
