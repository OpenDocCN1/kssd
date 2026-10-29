
# [原创]如何写一个Android inline hook框架

## Android\_Inline\_Hook

<mark class="encrypted">e80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2g2r3!0S2k6q4)9J5c8V1q4F1k6s2u0G2K9h3c8Q4y4h3k6u0L8X3I4A6L8X3g2Q4y4h3k6t1L8$3!0C8i4K6g2X3b7g2u0y4y4U0b7</mark>.

有32和64的实现，但是是分离的，要用的话还要自己把两份代码合并在一起。

缺点：

1. 不支持函数替换（即hook后不执行原函数），现在只能修改参数寄存器，无法修改返回值。
2. 不支持定义同类型的hook函数来接受处理参数，只能通过修改寄存器的方式修改参数。

多余4个/或者占两个字节的参数，那么参数还要自己从栈上捞取。所以issues中说的把`mov r0，sp`去掉用来接收参数也是有问题的，就是参数在栈上的情况，传过来的时候sp不是原来的sp了。

因为以上的两个缺点，想来没太多人用也是情理之中了，因为自己解析参数、不能修改返回值、不能不执行原函数，局限太大了。看来要想用起来，还得自己修改代码。。。

## whale

<mark class="encrypted">560K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7@1I4G2k6s2W2Q4x3V1k6%4K9r3q4D9k6b7</mark>.

移植好像比较简单，记得好像移植过，但是hook了某个系统函数回调原函数就崩溃了。所以用之前可能要先帮他找一遍bug、修复。

后记：art hook的之前看过，应该说是xposed/frida的另一份代码，frida也是用的xposed的方式，只不过不修改系统文件、通过动态调用系统函数实现。frida是js的代码，这个是c/c++的实现。

内联hook大概看了下其实也是一样的套路，32位采用`ldr pc`的方式跳到hook函数，64位使用`x17`寄存器跳到hook函数。剩下的修复原函数也是一样的。只是没时间完整看一遍定位bug了。

## HookZz

<mark class="encrypted">656K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6B7L8i4m8W2N6%4y4Q4x3V1k6t1L8$3!0C8h3Y4Z5</mark>.

对于安卓程序员来说不太友好，编译需要`cmake`。

windows：

```
mkdir build_for_android_arm64 && cd build_for_android_arm64
set ANDROID_NDK=D:\android\NDK\android-ndk-r16b
C:\Users\EDZ\AppData\Local\Android\Sdk\cmake\3.10.2.4988404\bin\cmake .. -DCMAKE_TOOLCHAIN_FILE=%ANDROID_NDK%/build/cmake/android.toolchain.cmake -DCMAKE_BUILD_TYPE=Release -DANDROID_ABI="arm64-v8a" -DANDROID_NATIVE_API_LEVEL=android-21 -DSHARED=OFF -DHOOKZZ_DEBUG=OFF -G "Unix Makefiles" -D"CMAKE_MAKE_PROGRAM:PATH=D:\app\mingw-w64\x86_64-8.1.0-posix-seh-rt_v6-rev0\mingw64\bin\mingw32-make.exe"
```

需要指定`-G "Unix Makefiles"`，默认的NMake Makefiles编译不过，指定`make`，因为未设置环境变量，`-D"CMAKE_MAKE_PROGRAM:PATH="`

编译动态库，`-DSHARED=ON`；编译静态库，`-DSHARED=OFF`。

接下来为了方便还是移植到Android工程中吧。

呃。。。基本不可用！Android：arm/arm64均crach，arm64可以修复，<mark class="encrypted">fbfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4K9i4c8E0k6h3#2G2M7Y4W2Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8%4g2F1k6r3E0W2P5g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9&6b7#2!0m8z5p5!0F1k6f1I4A6j5W2)9#2b7%4y4@1k6r3y4^5P5q4)9#2b7@1I4A6N6r3g2u0N6r3g2J5j5i4c8G2M7W2)9J5k6h3y4U0i4@1f1@1i4@1t1^5i4@1q4p5i4@1f1#2i4K6R3%4i4@1u0p5i4@1f1$3i4K6V1#2i4@1t1H3K9h3&6A6N6q4N6A6N6r3S2o6L8$3I4D9k6h3y4@1K9h3!0F1i4@1f1$3i4@1t1%4i4@1u0n7i4@1f1#2i4K6S2m8i4@1p5H3K9h3&6o6L8$3I4D9k6h3y4@1K9h3!0F1i4K6u0V1</mark>>`initIterator(innerIterator);`

但是arm还是crash，

```
#00 pc 00025ef0 [anon:libc_malloc]
12-12 15:12:53.685 181-181/? I/DEBUG: #01 pc 000081f1 /data/app-lib/com.zhuotong.myhkzz-1/libhookzz.so (LiteCollectionIterator::getNextObject()+28)
12-12 15:12:53.685 181-181/? I/DEBUG: #02 pc 00007531 /data/app-lib/com.zhuotong.myhkzz-1/libhookzz.so (gen_thumb_relocate_code(void*, int*, unsigned int, unsigned int)+312)
12-12 15:12:53.685 181-181/? I/DEBUG: #03 pc 00007ac1 /data/app-lib/com.zhuotong.myhkzz-1/libhookzz.so (InterceptRouting::Prepare()+56)
12-12 15:12:53.685 181-181/? I/DEBUG: #04 pc 00007cc1 /data/app-lib/com.zhuotong.myhkzz-1/libhookzz.so (FunctionInlineReplaceRouting::Dispatch()+12)
12-12 15:12:53.685 181-181/? I/DEBUG: #05 pc 00007d4d /data/app-lib/com.zhuotong.myhkzz-1/libhookzz.so (ZzReplace+120)
```

且通过arm64测试来看，`open`、`fopen`可以hook成功。`__system_property_get`函数，第二个参数既是入参也做返回参数的情况无法正确hook，可以hook到，但是回调原函数，无论是使用第二个参数还是new参数均无法得到值，所以肯定哪里存在bug。

一个函数只能被hook一次，再次hook调用原函数(备份的第一个hook函数)崩溃，所以两次hook不能再调用原函数。

基于以上种种情况，可能还是自己实现Android\_Inline\_Hook比较好，毕竟Android\_Inline\_Hook代码易懂，hookZz太多无关代码，没时间看架构了。

后记：后来大概看了一下，32位也是`ldr pc`实现，好像也做了保存寄存器等操作，和Android\_Inline\_Hook基本是一样的，64位好像也是使用`x17`寄存器。其他也都是大同小异。所以也是没时间完整看一遍定位bug了。

## 自实现inline hook

因为以上的问题，目前/或者之前用过的一些hook框架或多或少都有些较大的bug(hookzz之前的某个版本应该是可以的)，而对其进行修复成本较高，还不如自己写一个。

首先统一一些概念。把覆盖被hook函数的指令称为跳板0，因为这部分指令越短越好，所以其大多数情况下只是一个跳板，跳到真正的用于hook指令处，这部分指令称为shellcode(shellcode不是必须的，比如跳板0直接跳到hook函数去执行，那么就不需要shellcode)。

这里假设都有一些arm指令的基础了，或者对hook已经有些理解了。后来我想了下我这篇更偏向于怎么写一个稳定可用hook框架，更偏向设计、编程，所以适合已经有些基础的，不是从0讲述hook到实现，虽然接下来的部分也会有很细节的部分，但是是针对一些特定的点。建议可以先看下其他人，比如ele7enxxh、GToad写的一些文章。

inline hook这种东西，我是感觉当你掌握汇编、自己有需求的情况下，不经过学习也是可以从0写出一个hook框架的，确实是原理很简单的。

### 最简单的实现

最容易想到的一种实现方式，使用跳板0覆盖一个函数的指令，当执行到这个函数的时候其实就是执行跳板0，跳板0在不修改寄存器的情况跳到执行hook函数。如果在hook函数内不需要执行原函数是最简单的，到这hook就是一个完整的hook。

如果需要执行原函数，那么在跳板0覆盖指令之前先备份指令，执行原函数之前把备份的指令再覆盖回去，执行之后再覆盖回跳板0。确实是最简单的方法、也确实可以，但是也有一个很大的问题，就是多线程的问题，在把备份的指令覆盖回去之后，其他线程执行到这里不就hook不到了，甚至crash。

因为无法加锁(真正有效的锁)，而暂停所有线程的太重了，所以基本上只有自己确定某个函数不存在多线程问题或者无需调用原函数才可用。写个demo，熟悉下hook还行，真的实际使用是不行的。

也是因为这个问题，基本上目前的inline hook都会避免再次覆盖指令。不能覆盖回去，那么就直接执行备份的指令，执行后再跳到跳板0之后再继续执行。

也确实是可行的，大部分指令是可以这么做的，但是也有例外。比如备份的指令中有`b`/`bl`指令跳到一个偏移地址，跳转到的地址等于当前地址+偏移。而备份后的指令取得当前地址肯定是不等于原来的当前地址的，所以就跳到错误的地址去执行了。

好在我们可以进行修复，计算出原来要跳到的绝对地址，把这条`b`指令替换成`ldr pc`指令。其他指令的一些修复也是类似的道理。

### 四种hook形式应用不同的场景

#### dump读写寄存器、堆栈--一种hook形式

Android\_Inline\_Hook就是这样的实现，只能接收读写寄存器和堆栈，原函数还运行。那么其实这种方式主要作用就是读写参数寄存器、栈，不能读写函数返回值，如果是不受参数控制流程的函数就无能为力了（当然是可以逆向分析，在已经得到返回值的指令处hook，但是应用场景太小了）。

那么实现的核心就是，跳板0->`dumpshellcode`。`dumpshellcode`把寄存器以数组的形式存放（栈就是天然的数组），把这个数组传递给`dump`函数，`dump`函数接收处理寄存器(未生效)。`dump`函数返回shellcode，恢复数组数据到寄存器，完成恢复或者修改。执行`backupshellcode`(备份的原函数的开头部分，修复`pc`，跳回原函数之后的部分)完成原函数的执行流程。

#### 替换/拦截函数，接收处理参数，调用原函数、读写返回值--一种hook形式

最常用、最符合使用习惯的的方式。

实现的核心，跳板0->`funshellcode`。`funshellcode`可以在上面的`dumpshellcode`基础上实现也可以全新实现。

在`dumpshellcode`基础上可以取巧一些，前面的保存`dump`寄存器保留，把后面的执行`backupshellcode`换成执行hook函数。

全新实现就是不保存寄存器，那么就可以把hook函数放在跳板shellcode中，直接跳到hook函数；也可以跳板0->`funshellcode`，`funshellcode`中再跳转到hook函数。

之后就进入hook函数，如果不调用原函数，直接返回一个返回值或者`void`函数什么都不做即可(如果是参数也做返回值的情况就修改参数)。如果调用原函数，就通过一个结构体/`map`等查询被hook函数地址对应的`backupshellcode`，把`backupshellcode`转成函数指针，传参调用，即可完成原函数的调用、或者返回值。

#### dump读写寄存器、堆栈，调用原函数、读写返回值/寄存器--一种hook形式

在`dump`的基础上，执行`backupshellcode`(备份的原函数的开头部分，修复`pc`，跳回原函数之后的部分)完成原函数的执行流程之后返回到`dumpshellcode`，调用另一个`dump`函数，读写返回值(`r0/x0`,`r1/x1`寄存器)。不同于`dump`的地方在于如果要返回到`dumpshellcode`，那么在调用`backupshellcode`之前应该备份原来的`lr`寄存器。考虑到多线程的问题，肯定是不能用一个固定的变量/地址去存储`lr`寄存器的，可能被覆盖，同一个线程哪怕是递归调用函数也是有顺序的，所以每个线程的被hook函数使用一个顺序的容器保存`lr`，后进先出。

#### dump读写寄存器、堆栈，读写返回值/寄存器--一种hook形式

在`dump`的基础上，不执行`backupshellcode`(备份的原函数的开头部分，修复`pc`，跳回原函数之后的部分)，直接修改`r0/x0`、`r1/x1`寄存器，返回。和`dump`函数很多地方是一致的，应用于只需要返回值，并不需要原函数执行的情况。

以上四种场景应该是足够满足hook的需要了。

### arm64 实现难点

因为无法直接操作`pc`，那么实现跳转(通用情况)需要占用一个寄存器。要么使用一个不会被使用的寄存器(哪有绝对不会被使用的寄存器)，要么先保存这个寄存器，通过栈保存(之前就是忽略了这个问题在固定地址保存寄存器，那么多线程情况下就可能被覆盖)，跳过去之后先恢复这个寄存器。例如：

```
stp X1, X0, [SP, #-0x10];//保存寄存器
...
ldr x0, [sp, #-0x8];//恢复x0寄存器
```

对应shellcode我们很容易实现，但是如果是c/c++函数(我们的hook函数)就麻烦了，直接在函数开头插代码肯定是不行的。在源码中函数第一行内嵌汇编恢复寄存器？很可惜，这种只在无参、无返回值(空实现)、只有内嵌汇编的情况会成功，其他情况下源码中的第一行并不是汇编中的第一行。。。

所以似乎陷入了无解的状态，在llvm中函数开头插指令？似乎太重了。所以回到原点了，就要考虑真的没有不使用寄存器跳转的可能吗？其实还是有的，`b`或者`bl`到偏移

ARM64:

`B`：`0x17`向前跳转，`0x14`向后跳转

`BL`：`0x97`向前跳转 `0x94`向后跳转

偏移地址计算过程：

```
（目标地址 - 指令地址）/ 4 = 偏移

// 减8，指令流水造成。

// 除4，因为指令定长，存储指令个数差，而不是地址差。
```

完整指令：

```
.text:000000000008CC84 8D B3 FF 97 BL je_arena_malloc_hard
.text:0000000000079AB8 je_arena_malloc_hard
```

计算偏移：

```
(79AB8-8CC84) / 4 = FFFFFFFFFFFFB38D
FFB38D | 0x97000000 = 97FFB38D
```

所以理论上，如果被hook的函数和hook函数/跳板/shellcode的距离在正负67108860/64m的范围内，64m=67108864，还有0，比如`BL 0=00000094`。那么这样就可以省一个寄存器了，针对arm64不能直接操作`pc`的问题，这应该是一个解决方案。

那么单指令hook除了异常的方式是不是就是指的这种方式呢？只需要覆盖一条指令，关键是怎么确保被hook函数和hook函数地址在正负64m内呢。怎么能申请到这块地址的内存呢。

也许可以查看`/proc/pid/maps`，在要hook的so附近寻找未使用的空间，然后使用`mmap`申请，不确定是否可行。

暂时可用的一些方式如下(最终未采用)：

#### 自定义section，增加蹦床，可读写执行

实际上到内存中还是被和代码段放在一起都是可读可执行，没有写的权限。

```
.section ".my_sec", "awx"
.global _myopen_start
//.text
_myopen_start:
ldr x0, [sp, #-0x8];
b my_open;
.end
//用于手动生成蹦床代码，因为hook代码和这个蹦床一起编译的，所以基本上不会超出加减64m，那么就可以使用b跳转
//到偏移，就不用占用一个寄存器了。需要每增加一个hook函数，就加一个蹦床，相应的生成shellcode中跳转到hook
//函数的地方都要改成这个蹦床的地址。难就难在这个不好通过内嵌汇编实现，因为b跟的是个偏移值，在汇编中可以使用
//函数名称，编译器替换，但是内嵌汇编中不行。而自己计算
```

shellcode如下，

```
//用于演示，因为不是最终方案，不写完整代码了。
//修改trampoline指向蹦床即可
.data
replace_start:
ldr x0, trampoline; //如果每一个函数都在源码中新建一个shellcode的话，而不是动态复制生成，那么这个shellcode和蹦床可以合为一个。
br x0; //不能改变lr寄存器
trampoline: //蹦床的地址
.double 0xffffffffffffffff
replace_end:
```
