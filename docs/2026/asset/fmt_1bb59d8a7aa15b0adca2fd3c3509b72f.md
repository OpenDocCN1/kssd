
假设你想运行下面这个有错误的程序，它试图把2写到内存偏移0处：

```
int main()
{
    *(int *) 0 = 2;
}
```

正如你所料，偏移0不是一个可用的程序地址。例如，在Windows NT中，内存的第一个4KB页面被标记为“不存在”，用以阻止使用NULL指针的程序问题。试图写这个地址将引发一个页错误（异常0xE）。看一下上面的IDT图，你会看到这个异常是由`NTOSKRNL.EXE`中的`_KiTrap0E`处理的。

我已经多次在调试器中跟踪到`_KiTrap0E`的代码中，但这个代码相当复杂，想全面描述得另用一篇文章才行。眼下，只要知道Ring 0的`_KiTrap0E`代码检查各种各样的特殊条件就足够了。因此，`KiTrap0E`调用了`IRETD`指令把控制权传到了Ring 3的`NTDLL`开头的`KiUserExceptionDispatcher`函数中。我在这里不讲`KiUserExceptionDispatcher`，因为我已经在我的文章“A Crash Course on the Depths of Win32 Structured Exception Handling”（MSJ，1997年一月）中详细讲了这个函数。重点是要知道`KiUserExceptionDispatcher`被告知异常代码是`0xC0000005`（`STATUS_ACCESS_VIOLATION`），并不是由CPU产生的那个异常代码`0xE`。

像`0xC0000005`之类的Win32异常代码是哪里来的？答案可以在Win32 SDK或你的C++编译器中的`WINERROR.H`头文件中找到。几乎在最上面，你会看到一个注释：

```
//  Values are 32 bit values layed out as follows:
```

继续读这个注释，你就会知道，最高的两位（位31和30）代表严重程度。接下来的位（29）表示定义者。位28是保留的。高位字中剩下的12位是设备代码。低位字（位0到15）是异常代码。

比较有趣的一点是，Win32的Last Error代码也是通过用位域来分类信息的。因此，你会知道像`0x80010002`（`RPC_E_CALL_CANCELED`）之类的错误代码来自哪里。顺便说一下，使用严重程度，定义者和设备位域并不是起源于Windows NT。IBM的OS/2使用了相同的机制，它是在20世纪80年代后期合并分别由Microsoft和IBM完成的操作系统的工作的一个副产品。

回到异常中，看一下严重程度位，位31和30。值0代表成功，1代表信息，2代表警告，3（两个位均置位）代表错误。一个致命的异常相当于一个错误，因此任何32位的致命异常代码最高的两位都是置位的。接下来的两个位，定义者和保留位，通常都被设置为0，因为很少使用它们。

仅仅知道上面那些有限的异常代码构造方面的知识，你就能推断出致命异常代码都是以0xC开头的。因此，遇到像`0xC0000005`（`STATUS_ACCESS_VIOLATION`）和`0xC000001D`（`STATUS_ILLEGAL_INSTRUCTION`）之类的异常代码，你知道它们就属于这一类。比这严重程序低一些的异常，也就是警告，它的严重系数是2，因此你看到类似`0x80000003`（`STATUS_BREAKPOINT`）和`0x80000004`（`STATUS_SINGLE_STEP`）之类的代码，你知道它们就属于这一类。在`WINNT.H`中搜索`STATUS_`可以找到一份相当完整的可能的异常代码列表。当你看这个列表时要记住，并不是支持Win32的每一个处理器都可以生成所有Win32异常代码。

在写这个专栏时，我到底能导致多少个Win32异常引起了我的兴趣。我对操作系统到底能赋予我有意导致的许多错误什么样的异常代码也充满好奇。为了帮助解决这些问题，我写了一个能以各种方式产生处理器错误并且报告它们被映射到的Win32异常代码的程序框架。这就是我的`GenException`程序（见图5）。

图5 `GenException.CPP`

```
//==========================================

// Matt Pietrek

// Microsoft Systems Journal, October 1997

// FILE: GenException.CPP

// 使用命令行CL GenException.CPP编译

//==========================================

#define WIN32_LEAN_AND_MEAN

#include <windows.h>

#include <stdio.h>

#include <float.h>

#include <assert.h>

typedef void (* PFNGENERATEEXCEPTION)(void);

void GenerateSTATUS_BREAKPOINT( void )
{
    __asm   int 3   // 普通的断点指令
}

void GenerateSTATUS_SINGLE_STEP( void )
{
    // 这比使用硬件断点寄存器生成int 1更容易
    __asm   int 1
}

void GenerateSTATUS_ACCESS_VIOLATION( void )
{
// 通过读取地址在2GB以上的内存来产生
// 一个页错误（异常代码0xE）
    int i = *(int *)0xFFFFFFF0;
}

void GenerateSTATUS_ILLEGAL_INSTRUCTION( void )
{
    __asm _emit 0x0F    // 无效指令导致产生异常0xD
    __asm _emit 0xFF
}

void GenerateSTATUS_ARRAY_BOUNDS_EXCEEDED( void )
{
    DWORD arrayBounds[2] = { 10, 48 };
    __asm   mov eax, 12
    __asm   bound eax, arrayBounds  // 这条BOUND指令运行正常
    __asm   mov eax, 7
    __asm   bound eax, arrayBounds  // 这条BOUND指令会产生异常0x5
}

void UnmaskFPExceptionBits( void )
{
    unsigned short cw;
    __asm   fninit      // 初始化数值协处理器
    __asm   fstcw [cw]
    cw &= 0xFFE0;       // 关闭大部分异常位（除了精度异常）
    __asm   fldcw [cw]
}

void GenerateSTATUS_FLOAT_DIVIDE_BY_ZERO( void )
{
    double a = 0;
    a = 1 / a;
    __asm fwait;
}

void GenerateSTATUS_FLOAT_OVERFLOW( void )
{
    double a = DBL_MAX;
    a *= a;
    __asm fwait;
}

void GenerateSTATUS_FLOAT_STACK_CHECK( void )
{
    unsigned a;
    __asm   fistp [a]
    __asm   fwait;
}

void GenerateSTATUS_FLOAT_UNDERFLOW( void )
{
    double a = DBL_MIN;
    a /= 10;
    __asm fwait;
}

void GenerateSTATUS_INTEGER_DIVIDE_BY_ZERO( void )
{
    // 除以0导致异常0x0
    int i = 0;
    i = 2 / i;
}

void GenerateSTATUS_INTEGER_OVERFLOW( void )
{
    __asm   mov eax, 07FFFFFFFh     // 带符号数的最大值
    __asm   add eax, 2              // 结果 = 0x80000001 -> 溢出！
    __asm   into                    // 产生异常0x4
}

void GenerateSTATUS_PRIVILEGED_INSTRUCTION( void )
{
    // HLT指令只能在ring 0下执行
    __asm   hlt
}

void GenerateSTATUS_STACK_OVERFLOW( void )
{
    DWORD myArray[512];
    // “无穷”递归导致堆栈溢出
    GenerateSTATUS_STACK_OVERFLOW();
}

DWORD GetExceptionNumber( PFNGENERATEEXCEPTION pfn )
{
    DWORD exceptionCode = 0;
    __try
    {
        pfn();
    }
    __except( exceptionCode = GetExceptionCode(), EXCEPTION_EXECUTE_HANDLER )
    {
    }
    return exceptionCode;
}

#define SHOW_EXCEPTION( x )                                 \
    dwExceptionNumber = GetExceptionNumber( Generate##x );  \
    printf( "%X %s\n", dwExceptionNumber, #x );             \
    assert( dwExceptionNumber == x );

int main(int argc, char *argv[])
{
    DWORD dwExceptionNumber;
    SHOW_EXCEPTION( STATUS_BREAKPOINT )
    SHOW_EXCEPTION( STATUS_SINGLE_STEP )
    SHOW_EXCEPTION( STATUS_ACCESS_VIOLATION )
    SHOW_EXCEPTION( STATUS_ILLEGAL_INSTRUCTION )
    SHOW_EXCEPTION( STATUS_ARRAY_BOUNDS_EXCEEDED )
    UnmaskFPExceptionBits();
    SHOW_EXCEPTION( STATUS_FLOAT_DIVIDE_BY_ZERO )
    UnmaskFPExceptionBits();
    SHOW_EXCEPTION( STATUS_FLOAT_OVERFLOW )
    UnmaskFPExceptionBits();
    SHOW_EXCEPTION( STATUS_FLOAT_STACK_CHECK )
    UnmaskFPExceptionBits();
    SHOW_EXCEPTION( STATUS_FLOAT_UNDERFLOW )
    SHOW_EXCEPTION( STATUS_INTEGER_DIVIDE_BY_ZERO )
    SHOW_EXCEPTION( STATUS_INTEGER_OVERFLOW )
    SHOW_EXCEPTION( STATUS_PRIVILEGED_INSTRUCTION )
    SHOW_EXCEPTION( STATUS_STACK_OVERFLOW );
    return 0;
}
```

`GetException`程序的代码被分成三部分。第一部分是一系列函数，它们的名字以`Generate`开头，后面是它们要产生的Win32异常的名字。例如，`GenerateSTATUS_ILLEGAL_INSTRUCTION`引起一个非法指令异常。第二部分是`GetExceptionNumber`函数。它使用Win32结构化异常处理（SEH）来确定各个`GenerateXXX`函数引起的Win32异常代码，并且将这个异常代码返回它的调用者。`GetExceptionNumber`函数带有一个参数，这个参数是指向它要调用的`GenerateXXX`函数的指针。

`GenException.CPP`的最后一部分是`main`函数。它是一系列C++预处理器宏的调用，这个宏被我命名为`SHOW_EXCEPTION`。对`SHOW_EXCEPTION`的每一次调用就会产生一个Win32异常。`SHOW_EXCEPTION`带一个预定义的异常名称（例如`STATUS_ACCESS_VIOLATION`），然后将它合成一个与其相应的`GenerateXXX`函数的调用。我使用`SHOW_EXCEPTION`宏来省略大量模板代码，这些模块代码只有实际调用的异常代码不同。通过使用预处理器符号粘贴（preprocessor token pasting）和字符串化（stringizing）宏，这一行

```
SHOW_EXCEPTION( STATUS_BREAKPOINT )
```

被扩展成：

```
dwExceptionNumber = GetExceptionNumber( GenerateSTATUS_BREAKPOINT );
printf( "%X %s\n", dwExceptionNumber, "STATUS_BREAKPOINT" );
assert( dwExceptionNumber == STATUS_BREAKPOINT );
```

在写`GetException`时，一些异常非常容易产生，例如`STATUS_ACCESS_VIOLATION`。创建那些不常见的异常也很重要，例如`STATUS_ILLEGAL_INSTRUCTION`。许多情况下，我不得不借助于内联汇编。两个比较好的例子是CPU异常4和5，它们分别由`INTO`指令和`BOUND`指令产生。我不详细讲述各种异常是如何产生的，`GenException.CPP`代码中包含了许多相关注释。

生成浮点异常需要一些技巧，因为Win32初始化浮点单元时不会产生异常。我不得不明确关闭协处理器控制字中的某些位来产生浮点异常，像`STATUS_FLOAT_DIVIDE_BY_ZERO`。如果你对此好奇，可以看`UnmaskFPExceptionBits`函数，它包含了处理那些位的代码。因为在执行浮点指令时，只有执行到实际出错指令的下一条指令时才引发异常，因此我使用`__asm fwait`指令强制在一个有意出错的指令后引发一个异常。

可能`GetException`不是你曾经运行过的程序中最令人兴奋或最有用的程序，但是我相信你一定能从如何产生各种Win32异常中受到启发。在大多数情况下，CPU生成一个异常`0xD`，然后Win32异常处理程序分析这个代码并构造一个更有意义，更加明确的异常代码。我的目的是描述这些机制，解释硬件级别和操作系统级别的异常，并且向你展示它们之间的联系。

[（Microsoft System Journal 1997年10月Under The Hood专栏）](<https://bbs.kanxue.com/elink@2a2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3#2K6K9W2)9J5c8X3c8W2k6X3q4#2L8s2c8X3M7X3q4E0k6g2)9J5k6h3q4K6M7q4)9K6c8Y4m8S2k6$3g2Q4x3@1c8Q4x3V1k6E0M7$3A6Q4x3V1j5I4x3o6V1%4i4K6u0r3K9r3!0G2k6o6p5H3z5e0N6Q4x3X3g2Z5N6r3@1`.)>)

译者：SmartTech   电子信箱：zhzhtst@163.com

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [Interrupt and Exception.pdf](javascript:void(0);)

  （232.32kb，512次下载）
