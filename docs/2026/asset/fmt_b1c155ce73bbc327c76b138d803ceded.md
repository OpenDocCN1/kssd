
# [翻译]GO语言逆向教程

[原文链接](https://bbs.kanxue.com/elink@d70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3c8F1j5h3N6S2i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6W2Q4x3V1j5J5x3g2)9J5c8Y4u0W2N6X3g2J5M7$3W2F1k6#2)9#2k6X3N6G2i4K6g2X3j5X3W2F1j5i4u0A6k6i4y4Q4y4h3k6D9K9h3E0W2i4K6g2X3j5g2)9#2k6Y4m8J5L8#2)9J5c8R3`.`.)

给出了一种golang清洗符号和字符串的方法

作者：RedNaga Security

GO编写的程序很奇怪，或者至少，在开始的地方是这样。在深入研究一些名为Rex的Linux恶意软件的同时，我意识到我可能需要了解的比我想象的要多。就在前一周，我一直在逆向Linux Lady，它也是用GO编写的，但它并没有去除符号表，所以它很容易。GO编译出的二进制文件相当大，包含许多我不关心的额外方法 - 虽然我真的不明白为什么。老实说 - 我仍然没有完全挖掘Golang代码并且还没有真正在Go中编写很多代码，因此其中一些可能是不正确的;这只是我在逆向一些ELF Go二进制文件时的经验！如果您不想阅读整个文章，或滚动到底部以获得完整git仓库的链接，请转到此处。 [原文的GitHub](https://bbs.kanxue.com/elink@c09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6s2u0S2P5Y4A6W2M7X3g2Q4x3V1k6Y4L8$3I4S2L8X3N6Q4y4h3k6D9L8$3q4V1k6i4u0Q4y4h3k6S2M7%4y4A6M7%4b7`.`)

为了弄清楚我的一些例子，我将使用一个非常简单的“Hello,World!”示例，并引用Rex恶意软件的代码。代码和Make文件非常简单。

```go
//hello.go

package main

import "fmt"

func main() {

    fmt.Println("Hello, World!")

}
```

```make
//Makefile

all:

	GOOS=linux GOARCH=386 go build -o hello-stripped -ldflags "-s" hello.go

	GOOS=linux GOARCH=386 go build -o hello-normal hello.go
```

由于我正在使用OSX机器，因此需要上面的`GOOS`和`GOARCH`变量来正确交叉编译。 第一行还添加了`ldflags`选项来去除二进制文件的符号表。 通过这种方式，我们可以同时分析的去除了符号表和原始的可执行文件，运行`make`然后反汇编编译出的GO程序。 如果我们在IDA Pro中打开未删除符号的二进制文件，我们很快就可以注意到一些神奇的事情;

![](https://bbs.kanxue.com/upload/attach/201908/845726_GAWRC4DE6AJ5S76.png)

我的妈 - 我们的5行代码变成了超过2058个函数。 由于看起来像运行库，我们在`main()`函数中也没有任何有趣的东西。 如果我们看的仔细，就能发现我们实际感兴趣的代码是在`main_main`中;

![](https://bbs.kanxue.com/upload/attach/201908/845726_KCFTE44QNJTGT9H.png)

这是很多我真诚地不想看的代码。 字符串加载也看起来有点奇怪 - 虽然IDA似乎在确定必要的位方面做得很好。 我们可以很容易地看到字符串加载实际上是一组三个`mov`;

```
；string load

mov     ebx, offset aHelloWorld ; "Hello, World!"

mov     [esp+3Ch+var_14], ebx ; Shove string into location

mov     [esp+3Ch+var_10], 0Dh ; length of string
```

这不完全是革命性的，尽管我不能说我以前见过这样的事情。 引起我注意的另一个代码是`runtime_morestack_context`调用;

```
;morestack_context

loc_80490CB:

call    runtime_morestack_noctxt

jmp     main_main
```

这种样式的代码块似乎总是出现在函数的末尾，它似乎总是循环回到同一函数的顶部。 通过查看函数的交叉引用验证了这一点。 好了，既然我们知道IDA Pro可以处理未除去符号的二进制文件，那么这次看看剥离版本的表现。

![](https://bbs.kanxue.com/upload/attach/201908/845726_VRHS77RDHXEEWEN.png)

我们马上看到一些......好吧，让我们称之为“差异”。 这回我们定义了1329个函数，导航器工具栏告诉我们有一些未定义的代码。 幸运的是，IDA仍然能够找到我们正在寻找的字符串引用，但是现在这个功能似乎不那么友好了。

![](https://bbs.kanxue.com/upload/attach/201908/845726_ZPEP83WNA6MN7VM.png)

我们不再有函数名称了 - 但是如果我们对`main.main`进行字符串搜索，那么函数名称似乎会保留在二进制文件的特定部分中。

```
.gopclntab

.gopclntab:0813E174                 db  6Dh ; m

.gopclntab:0813E175                 db  61h ; a

.gopclntab:0813E176                 db  69h ; i

.gopclntab:0813E177                 db  6Eh ; n

.gopclntab:0813E178                 db  2Eh ; .

.gopclntab:0813E179                 db  6Dh ; m

.gopclntab:0813E17A                 db  61h ; a

.gopclntab:0813E17B                 db  69h ; i

.gopclntab:0813E17C                 db  6Eh ; n
```

好吧，看来我们想要的符号还是有救。`gopclntab`段！在深入Google搜索结果并发布推文之后 - 一个友好的逆向工程师George（Egor？）Zaytsev向我展示了他的IDA Pro脚本，用于函数的重命名和添加类型信息。在略读这些之后，很容易弄清楚这一部分的格式，所以我在他的脚本基础上添加了一些功能。基本代码如下所示，简单来说，我们查看段`.gopclntab`并跳过前8个字节。然后我们创建一个指针（Qword或Dword，取决于程序是否为64位）。第一组数据实际上给出了`.gopclntab`表的大小，因此我们知道进入这个结构的偏移。现在我们可以开始处理其余的数据，这些数据似乎是以`function_offset`开始，后跟（function）`name_offset`。当我们创建指向这些偏移量的指针并告诉IDA创建字符串时，我们只需要确保我们不会向`MakeString`传递任何错误字符，因此我们使用`clean_function_name`函数去除任何错误。重命名的脚本如下。

```python
#renamer.py

def create_pointer(addr, force_size=None):

    if force_size is not 4 and (idaapi.get_inf_structure().is_64bit() or force_size is 8):

        MakeQword(addr)

	return Qword(addr), 8

    else:

	MakeDword(addr)

	return Dword(addr), 4

STRIP_CHARS = [ '(', ')', '[', ']', '{', '}', ' ', '"' ]

REPLACE_CHARS = ['.', '*', '-', ',', ';', ':', '/', '\xb7' ]

def clean_function_name(str):

    # Kill generic 'bad' characters

    str = filter(lambda x: x in string.printable, str)

    for c in STRIP_CHARS:

        str = str.replace(c, '')

    for c in REPLACE_CHARS:

        str = str.replace(c, '_')

    return str

def renamer_init():

    renamed = 0

    gopclntab = ida_segment.get_segm_by_name('.gopclntab')

    if gopclntab is not None:

        # Skip unimportant header and goto section size

        addr = gopclntab.startEA + 8

        size, addr_size = create_pointer(addr)

        addr += addr_size

        # Unsure if this end is correct

        early_end = addr + (size * addr_size * 2)

        while addr < early_end:

            func_offset, addr_size = create_pointer(addr)

            name_offset, addr_size = create_pointer(addr + addr_size)

            addr += addr_size * 2

            func_name_addr = Dword(name_offset + gopclntab.startEA + addr_size) + gopclntab.startEA

            func_name = GetString(func_name_addr)

            MakeStr(func_name_addr, func_name_addr + len(func_name))

            appended = clean_func_name = clean_function_name(func_name)

            debug('Going to remap function at 0x%x with %s - cleaned up as %s' % (func_offset, func_name, clean_func_name))

            if ida_funcs.get_func_name(func_offset) is not None:

                if MakeName(func_offset, clean_func_name):

                    renamed += 1

                else:

                    error('clean_func_name error %s' % clean_func_name)

    return renamed

def main():

    renamed = renamer_init()

    info('Found and successfully renamed %d functions!' % renamed)
```

上面的代码实际上还没有经过运行（当然repo中的完整代码是可用的），但我希望它足够简单，可以阅读并理解该过程。但是，这仍然无法解决IDA Pro没有把所有函数都分析出来的问题。所以这将创建指针，这些指针在任何地方都没有被引用（这里原作者的意思应该是脚本使用`ida_funcs.get_func_name(func_offset)`来进行重命名，如果 `.gopclntab`表中解析出的`func_offset`指向ida未分析出的函数，`get_func_name`就失效了）。这可能看起来很难办，但别忘了我们刚才发现的看上去没什么用的 `runtime_morestack_noctxt`。由于每个函数都使用它（当然只是基本上，它有一个边缘区域），如果我们找到这个函数并向后遍历到该函数的交叉引用，那么我们将知道每个函数的存在位置。 现在我们知道函数的结束和调用`runtime_morestack_noctxt`之后的下一条指令让我们跳转到函数的顶部。这意味着我们应该能够快速地给出IDA所需的函数的开始和停止的边界，同时将其与函数名称的解析分开。如果我们打开窗口以交叉引用函数`runtime_morestack_noctxt`，我们会看到有许多未定义的部分调用它。 函数的交叉引用总共1774个，这是IDA已经为我们定义的1329个函数，这由下图突出显示;

![](https://bbs.kanxue.com/upload/attach/201908/845726_RJ68S4QTUFQVP73.png)

在深入研究多个二进制文件后，我们可以看到`runtime_morestack_noctext`将始终调用`runtime_morestack`（带context）。 这是我之前引用的边框，所以在这两个函数之间我们应该能够看到交叉引用二进制文件中使用的其他函数。 查看两个函数中较大的函数，`runtime_more_stack`，多个二进制文件往往有一个有趣的布局;

![](https://bbs.kanxue.com/upload/attach/201908/845726_TQ3YBK22F5A8HKQ.png)

我感兴趣的部分是`mov large dword ptr ds:1003h,0`  - 这在我看到的所有64位二进制文件中看起来都是不变的。 所以在交叉编译之后我注意到32位二进制文件使用了`mov qword ptr ds:1003h,0`，所以我们将寻找这种特征来创建一个“hook”来向后遍历。 幸运的是，在ida中，我还没有看到一个实例无法定义这个特定函数，我们并不需要花费太多脑力来映射它或自己定义它。 所以，说够了的话，让我们写一些代码来找到这个函数;

```python
#find_runtime_morestack.py

def create_runtime_ms():

    debug('Attempting to find runtime_morestack function for hooking on...')

    text_seg = ida_segment.get_segm_by_name('.text')

    # This code string appears to work for ELF32 and ELF64 AFAIK

    runtime_ms_end = ida_search.find_text(text_seg.startEA, 0, 0, "word ptr ds:1003h, 0", SEARCH_DOWN)

    runtime_ms = ida_funcs.get_func(runtime_ms_end)

    if idc.MakeNameEx(runtime_ms.startEA, "runtime_morecontext", SN_PUBLIC):

        debug('Successfully found runtime_morecontext')

    else:

        debug('Failed to rename function @ 0x%x to runtime_morestack' % runtime_ms.startEA)

    return runtime_ms
```

在找到函数之后，我们可以递归地遍历所有函数调用。 这是因为GO函数的结构似乎总是：
