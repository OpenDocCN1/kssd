
例如其中的 `BL` 指令并未修复，这个其实还容易修复，可以替换成 `LDR LR, 8`（应该放到最后）；`BLR LR`；因为既然是 `BL` 指令，那么 `LR` 寄存器肯定是会被修改，那就说明 `LR` 寄存器已经保存了，所以我们就使用 `LR` 寄存器也是没有关系的。

如果是 `B` 指令才麻烦，因为不能操作 `PC`，那么就要至少使用一个寄存器且还不能是 `LR`，只能使用 `X16`、`X17` 了。

后面看了下 `Android_Inline_Hook` 写了 `B` 指令的修复（是错的），`BL` 指令未实现。后来自己写了 `BL` 指令的实现，只是暂时用，不准备在这个基础上再写了。缺少的待修复指令还有不少，且已实现的也有一些错误，待重构。

![图片描述](upload/attach/202001/670707_MPBB6EEC9YCY2N2.png)

![图片描述](upload/attach/202001/670707_6TNDZYAC7HWG8D4.png)

修复之后，Android 9.0 arm64 的 `dlopen` 可以 `hook` 了。

## 2、arm64 从备份/修复的原方法跳回跳板 0 之后执行

采用的方式是在备份/修复的原方法后面加入：

```asm
STP X1, X0, [SP, #-0x10]
LDR X0, 8
BR X0
ADDR(64)
LDR X0, [SP, -0x8]
```

和跳板 0 是一样的，但是跳板 0 是一个函数的开头的概率很大，所以一般都是会栈平衡的，`SP` 指向栈顶，所以暂时存放寄存器不会被覆盖。

而放在后面就不太确定能保证 `SP` 指向栈顶了，所以可能要考虑解析前几条指令是否有操作 `SP`，或者尽可能地把寄存器储存在 `SP - 较大的值`，或者使用 `X16`、`X17` 寄存器……

当然这个可能概率较小，可以先不管。

## 3、无法重复 `hook` 一个函数

当然这个是说在其他框架或者本框架的另一个版本（或者无法维护保存 `hook` 的容器。如果是本框架内，是可以重复 `hook` 同一个函数的，比如解除 `hook` 再 `hook`，或者再扩展成链表的形式，依次调用 `hook`/`dump` 函数）。

其实就是指令修复不完整，`ARM` 的应该是 `LDR PC` 没有正确修复。这个事情排在后面，不是特别重要。

也可以参考 `Cydia Substrate` 框架，不过有点坑，只处理 `Thumb` 的情况。`Thumb` 可以多次 `hook`，`ARM` 下第二次 `hook` 不生效。直接判断第一条指令是不是 `LDR PC, [PC, #-4]`；是的话就认为是已经被 `hook` 过的，直接返回。

```cpp
if (result != NULL) {
    if (backup[0] == A$ldr_rd_$rn_im$(A$pc, A$pc, 4 - 8)) {
        *result = reinterpret_cast<void *>(backup[1]);
        return;
    }
}
```

可见只有第一次 `hook` 生效。

![图片描述](upload/attach/202001/670707_X4AKPPZ3JKSX9W4.png)

而 `Thumb` 模式，把上次的 `shellcode` 赋给 `result`，再把跳板中的地址替换为新的地址。

```cpp
f (
    (align == 0 || area[0] == T$nop) &&
    thumb[0] == T$bx(A$pc) &&
    thumb[1] == T$nop &&
    arm[0] == A$ldr_rd_$rn_im$(A$pc, A$pc, 4 - 8)
) {
    if (result != NULL)
        *result = reinterpret_cast<void *>(arm[1]);
    SubstrateHookMemory code(process, arm + 1, sizeof(uint32_t) * 1);
    arm[1] = reinterpret_cast<uint32_t>(replace);
    return sizeof(arm[0]);
}
```

![图片描述](upload/attach/202001/670707_W9ZQCNHSEYXJHG9.png)

感觉是不是写代码的忘了处理 `ARM` 的，因为 `ARM` 也是可以同样处理的。呃，无语的 bug。

例如这么修复即可，前面再加一行 `SubstrateHookMemory code(process, symbol, used);` 修改权限/刷新缓存。

![图片描述](upload/attach/202001/670707_CAYGBG3358JANJQ.png)

应该 `VirtualApp` `hook` 系统函数之后使用 `syscall` 调用，就是因为梆梆等一些壳 `hook` 了函数，而 `VirtualApp` 使用 `Cydia Substrate` 未修复该 bug，所以导致 `hook` 失败。

但是一推理应该是错误的，`VirtualApp` 是先 `hook` 的，壳应该是后 `hook` 的，所以只能是壳 `hook` 失败。因此，应该是某些系统函数修复指令不正确导致调用原函数失败，和本篇其实没多大关系，只是推理下。

其实以上 bug 根本原因还是指令的修复，这是最麻烦的，也是暂时没办法达到百分百修复的，尤其是 `arm64`，只能抽时间逐步完善了。

### `Android_Inline_Hook` 中较严重的一些 bug

#### 1、`init_arry` 函数和普通函数引用 `vector`

但其实不是同一个 `vector` 的，导致无法保存已 `hook` 的信息，应该用指针，不能直接引用 `vector`。属于代码逻辑 bug。

#### 2、内存权限问题

虽然 `mprotect` 的时候取一页内存，但是因为使用的 `malloc` 申请的内存可能是夹缝中内存，前后被使用了？也可能是申请后剩余的被其他占用更改了权限？

虽然 `mprotect` 不报错，修改权限成功了，但是有几率 `signal 11 (SIGSEGV), code 2 (SEGV_ACCERR)`。还有一种可能是 `malloc` 返回的地址是奇数的，未测试，理论上如果是奇数的也会 crash，因为内存没有对齐。

所以使用：

```cpp
long pagesize = sysconf(_SC_PAGE_SIZE);
void *pNewShellCode = NULL; // = malloc(sShellCodeLength);
int code = posix_memalign(&pNewShellCode, pagesize, pagesize);
```

或者：

```cpp
mmap(NULL, PAGE_SIZE, PROT_READ | PROT_WRITE | PROT_EXEC,
     MAP_ANONYMOUS | MAP_PRIVATE, 0, 0);
```

申请一整页的内存，但是这样肯定是浪费的，所以之后应该在这个内存上填充，占满之后再申请新的一页内存。

#### 3、修改内存权限

`ChangePageProperty` 中如果超过一页的内存，第二页和之后的都没有设置权限，属于代码逻辑 bug。但是没有超过 2 页的情况，所以不会触发。

#### 4、ARM 刷新缓存错误/未生效，arm64 未刷新缓存

```cpp
// 通过测试：cacheflush 无法刷新缓存，不过不确定是不是第三个参数的问题，ICACHE、DCACHE 或 BCACHE，抽时间再看吧。

// 不过 gtoad 的把地址转成无符号指针再取值肯定是不正确的。

// cacheflush(*((uint32_t *)(info->pBeHookAddr)), info->backUpLength, 0); // 测试也无法刷新缓存

// cacheflush(PAGE_START((uintptr_t)info->pBeHookAddr), PAGE_SIZE, 0); // 测试也无法刷新缓存

// cacheflush(info->pBeHookAddr, info->backUpLength, 0); // 测试也无法刷新缓存

// 而确实有效的刷新缓存的是：可以直接传地址和结束地址，也可以刷新一页，效果来看都成功了。

// if (TEST_BIT0((uint32_t)info->pBeHookAddr)) {

//     __builtin___clear_cache((char *)info->pBeHookAddr - 1,
//                             (char *)info->pBeHookAddr - 1 + info->backUpLength);

// } else {

//     __builtin___clear_cache((char *)info->pBeHookAddr,
//                             (char *)info->pBeHookAddr + info->backUpLength);

// }

// __builtin___clear_cache(PAGE_START((uintptr_t)info->pBeHookAddr),
//                         PAGE_END((uintptr_t)info->pBeHookAddr + info->backUpLength));

// 后记：
// 测试 cacheflush 可以，但是和 int cacheflush(long __addr, long __nbytes, long __cache);
// 不一致的是，从命名来看第二个参数应该是大小，不应该是结束地址，而且 Linux man 中
// Some or all of the address range addr to (addr+nbytes-1) is not accessible。
// 哎，奇怪，抽时间再逆向看看吧。

/*
if (TEST_BIT0((uint32_t)info->pBeHookAddr)) {
    cacheflush((char *)info->pBeHookAddr - 1,
               (char *)info->pBeHookAddr - 1 + info->backUpLength, 0);
} else {
    cacheflush((char *)info->pBeHookAddr,
               (char *)info->pBeHookAddr + info->backUpLength, 0);
}
*/

// 目前来看使用 __builtin___clear_cache 吧，应该是都实现了且确实可用。
// 如果某些系统不生效在抠出系统调用吧，先使用函数吧。
```

## 写在后面

为了避免不必要的纠纷，前面列举的框架只是为了引出为什么在写一个的原因，没有其他的意思。只是想写一个让大多数人更容易使用、了解实现细节、能更容易参与开发完善的 Android inline hook 项目。

所以暂时不考虑代码的艺术，而尽量地写成易懂的代码，且代码写了较多、完整的注释。

也是希望不止是为了使用，而能让使用者能自己了解原理，自己就能解决错误，这方面感谢下 `Android_Inline_Hook`。

之前也写过一些自己个人用的临时 `hook`，但是时间久了代码都找不到了，且也都是最简单的实现，临时用下的，所以也打算重新实现下，没时间一直拖着。

看到 `Android_Inline_Hook`，发现很多思路和我之前是一样的，代码也易懂，看一下就全明白了，所以一开始在 `Android_Inline_Hook` 上面修了一些 bug 凑合用。

不过后来修的多了，发现不如重新实现，因为很多地方不兼容、架构有冲突了。

整理下，最近开源了，希望各位大佬多参与，完善修复 bug，多谢！

<mark class="encrypted">422K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6\*7K9s2g2G2N6r3!0F1k6#2)9J5c8V1q4F1k6s2u0G2K9h3c8Q4y4h3k6u0L8X3I4A6L8X3g2t1L8$3!0C8</mark>

## `Android_Inline_Hook`

b15K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2g2r3!0S2k6q4)9J5c8V1q4F1k6s2u0G2K9h3c8Q4y4h3k6t1L8$3!0C8i4K6g2X3b7g2u0y4y4U0b7`.

有 32 位和 64 位的实现，但是是分离的，要用的话还要自己把两份代码合并在一起。

### 缺点

1. 不支持函数替换（即 `hook` 后不执行原函数），现在只能修改参数寄存器，无法修改返回值。
2. 不支持定义同类型的 `hook` 函数来接受处理参数，只能通过修改寄存器的方式修改参数。

多余 4 个或者占两个字节的参数，那么参数还要自己从栈上捞取。所以 issues 中说的把 `mov r0, sp` 去掉用来接收参数也是有问题的，就是参数在栈上的情况，传过来的时候 `sp` 不是原来的 `sp` 了。

因为以上的两个缺点，想来没太多人用也是情理之中了。因为自己解析参数、不能修改返回值、不能不执行原函数，局限太大了。看来要想用起来，还得自己修改代码……

## `whale`

868K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7@1I4G2k6s2W2Q4x3V1k6%4K9r3q4D9k6b7`. 

移植好像比较简单，记得好像移植过，但是 `hook` 了某个系统函数回调原函数就崩溃了。所以用之前可能要先帮它找一遍 bug、修复。

后记：`ART hook` 的之前看过，应该说是 `Xposed/Frida` 的另一份代码，`Frida` 也是用的 `Xposed` 的方式，只不过不修改系统文件、通过动态调用系统函数实现。`Frida` 是 `JS` 的代码，这个是 `C/C++` 的实现。

内联 `hook` 大概看了下其实也是一样的套路，32 位采用 `LDR PC` 的方式跳到 `hook` 函数，64 位使用 `X17` 寄存器跳到 `hook` 函数。剩下的修复原函数也是一样的。只是没时间完整看一遍定位 bug 了。

## `HookZz`

453K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2Y4W2N6%4y4Q4x3V1k6t1L8$3!0C8h3Y4Z5`.

对于安卓程序员来说不太友好，编译需要 `cmake`。

### Windows

```bat
mkdir build_for_android_arm64 && cd build_for_android_arm64

set ANDROID_NDK=D:\android\NDK\android-ndk-r16b

C:\Users\EDZ\AppData\Local\Android\Sdk\cmake\3.10.2.4988404\bin\cmake .. -DCMAKE_TOOLCHAIN_FILE=%ANDROID_NDK%/build/cmake/android.toolchain.cmake -DCMAKE_BUILD_TYPE=Release -DANDROID_ABI="arm64-v8a" -DANDROID_NATIVE_API_LEVEL=android-21 -DSHARED=OFF -DHOOKZZ_DEBUG=OFF -G "Unix Makefiles" -D"CMAKE_MAKE_PROGRAM:PATH=D:\app\mingw-w64\x86_64-8.1.0-posix-seh-rt_v6-rev0\mingw64\bin\mingw32-make.exe"
```

需要指定 `-G "Unix Makefiles"`，默认的 `NMake Makefiles` 编译不过，指定 `make`。因为未设置环境变量，所以使用 `-D"CMAKE_MAKE_PROGRAM:PATH="`。

编译动态库：`-DSHARED=ON`；编译静态库：`-DSHARED=OFF`。

接下来为了方便还是移植到 Android 工程中吧。

呃……基本不可用！Android：`arm`/`arm64` 均 crash，`arm64` 可以修复，adbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4K9i4c8E0k6h3#2G2M7Y4W2Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8%4g2F1k6r3E0W2P5g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9&6b7#2!0m8z5p5!0F1k6f1I4A6j5W2)9#2b7%4y4@1k6r3y4^5P5q4)9#2b7@1I4A6N6r3g2u0N6r3g2J5j5i4c8G2M7W2)9J5k6h3y4U0i4@1f1@1i4@1t1^5i4@1q4p5i4@1f1#2i4K6R3%4i4@1u0p5i4@1f1$3i4K6V1#2i4@1t1H3K9h3&6A6N6q4N6A6N6r3S2o6L8$3I4D9k6h3y4@1K9h3!0F1i4@1f1$3i4@1t1%4i4@1u0n7i4@1f1#2i4K6S2m8i4@1p5H3K9h3&6o6L8$3I4D9k6h3y4@1L8%4u0Q4x3U0S2A6L8X3&6W2M7V1W2@1k6i4u0S2N6r3!0J5i4K6t1&6i4K6y4n7`

但是 `arm` 还是 crash。
