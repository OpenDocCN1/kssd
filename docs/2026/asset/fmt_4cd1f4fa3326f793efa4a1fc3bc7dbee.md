
`Log.d(TAG, "test", new Exception());`或者`e.printStackTrace();`等函数被调用，虚拟机就会崩溃。崩溃的位置在`StackVisitor::WalkStack`函数里。

经过我的实验发现，在调用上面类型的函数时，函数会回溯调用栈，如果发现在调用栈里出现了没有和`dex`对应的汇编指令（就是我们自己定义的跳转函数等）就会报错。

所以我们解决这个问题的办法需要走2步：

1. 坚决杜绝在调用hook处理函数前，调用到输出堆栈类型的函数；并且在调用到hook处理函数前要做好堆栈和寄存器的处理，保证在回溯的时候发现不了任何跳转函数的足迹

2. 去掉`dex`和机器码之间的mapping关系。在4.4上的`art_method`类里是有一个mapping table的成员，我的做法是直接将其值置为`Null`。但是还没有在5.1上发现类似的成员，所以5.1上的hook可能出现类似的问题，不过既然知道了方向，想解决也不难。

下面我们看一下一个hook的示例：

```
ENTRY art_quick_dispatcher
    push    {r4, r5, lr}           @ sp - 12
    …
blx     artQuickToDispatcher
pop     {r4, r5, pc}           @ success, r0 and r1 hold the result
END art_quick_dispatcher
```

上面一段的意思就是说一个被hook的函数被调用后，其实先调用了`art_quick_dispatcher`这个函数，接着这个函数又调用了`artQuickToDispatcher`。

那么这就出现了问题，如果原函数或者你自己写的hook处理函数中出现了输出调用栈的代码，那么我们预期的调用关系会是：调用者->`art_quick_dispatcher`->`artQuickToDispatcher`->被调用者。可是`art_quick_dispatcher`和`artQuickToDispatcher`是没有java代码与其对应的，结果就是虚拟机直接崩溃。

所以我对这种代码做了一个升级。要达到的目的就是每次调用一个我们自己写的函数，都是有java代码与其对应的。举例说来就是上面的调用者->`art_quick_dispatcher`->`artQuickToDispatcher`->被调用者关系，其中的`art_quick_dispatcher`，`artQuickToDispatcher`都有真实的java代码对应。所以我就预写了相关的java函数代码，然后取出其`art_method`的首地址，在hook后直接在汇编代码层进行调用。这样对于虚拟机来说就是正常的函数调用关系了。

调用形式如下：

```
ENTRY art_quick_proxy
    push   {r0-r7}
    mov    r7, lr
    mov    r0, #9//标号为9的函数是我预写的java函数
    bl     exe_switch_entry//执行这个java函数，exe_switch_entry也是我写的一段汇编代码
…
    mov    lr, r7
    pop    {r0-r7}
…
```

这里保存了所有寄存器和返回地址，在调用了想要调用的java函数后，所有寄存器和堆栈以及返回地址都恢复正常，这样对于虚拟机来说，就相当于正常的java调用。

取消mapping关系的方法，在4.4上我将`art_method`类mapping table置为`Null`，方法如下：

下面的define都是4.4上的，5.1上没有mapping table，还没有研究。

```
#define METHOD_ACCESS_FLAG 20
#define METHOD_MAPPING_TABLE 60
#define kAccNative 0x0100
#define kAccStatic 0x0008
…
	int* access_flag = (int*) (artmeth + METHOD_ACCESS_FLAG);
	*access_flag = *access_flag | kAccNative;

	int* mapping_table = (int*) (artmeth + METHOD_MAPPING_TABLE);
	*mapping_table = 0;
…
```

## 0x02 如何处理参数

处理参数问题就和主流的参数处理方法一致了，我这里就是遍历堆栈，获取参数，然后通过调用java函数对基本类型装箱，最后由一个`Object`数组的形式封装所有的参数。下面具体介绍一下。

首先要介绍一下art虚拟机上参数是如何传递的。在汇编层面，参数组织如下：

```
r0 = method
r1 = this
r2 = arg0
r3 = arg1
[sp] = N/A
[sp + 4] = N/A
[sp + 8] = N/A
[sp + 12] = N/A
[sp + 16] = arg2
```

需要注意的就是堆栈中的前4个存储单元里存的东西未知，不管是什么，肯定是我们不需要的，但是又不建议妄自修改的东西。

然后我们可以从`r2`寄存器开始遍历，取出所有的参数。基本参数的装箱就是指将`int`、`short`等类型转换为`Integer`、`Short`这样的`Object`，java里已经有这样的函数供我们使用了：

```
Integer.valueOf(int);
Short.valueOf(short);
…
```

## 0x03 如何处理返回值

我想到的办法就是，为每一个被hook的函数都分配一个与其返回值对应的hook处理函数，由于返回值类型是确定的(8种基本类型加`Object`)，所以我枚举构造了9种不同返回值的hook处理函数（直接java编写）。

```
    private static int onHookInt(Object artmethod, Object receiver, Object[] args) {
        return (Integer) HookManager.onHooked(artmethod, receiver, args);
    }

    private static long onHookLong(Object artmethod, Object receiver, Object[] args) {
        return (Long) HookManager.onHooked(artmethod, receiver, args);
    }

    private static double onHookDouble(Object artmethod, Object receiver, Object[] args) {
        return (Double) HookManager.onHooked(artmethod, receiver, args);
    }

    private static char onHookChar(Object artmethod, Object receiver, Object[] args) {
        return (Character) HookManager.onHooked(artmethod, receiver, args);
    }

    private static short onHookShort(Object artmethod, Object receiver, Object[] args) {
        return (Short) HookManager.onHooked(artmethod, receiver, args);
    }

    private static float onHookFloat(Object artmethod, Object receiver, Object[] args) {
        return (Float) HookManager.onHooked(artmethod, receiver, args);
    }

    private static Object onHookObject(Object artmethod, Object receiver, Object[] args) {
        return HookManager.onHooked(artmethod, receiver, args);
    }

    private static boolean onHookBoolean(Object artmethod, Object receiver, Object[] args) {
        return (Boolean) HookManager.onHooked(artmethod, receiver, args);
    }

    private static byte onHookByte(Object artmethod, Object receiver, Object[] args) {
        return (Byte) HookManager.onHooked(artmethod, receiver, args);
}
```

而`HookManager.onHooked`返回的是`Object`类型，对于基本类型来说，我们只要对其拆箱就可以了。

## 0x04 结束

至此，关于android5.1上的hook就完成了，本文主要是为了解决前辈们做的hook demo遗留下来的一些问题，立志于对这一体系做一种补充，感谢大家。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
