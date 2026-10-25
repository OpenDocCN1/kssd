
Here we put sum of command line arguments to `rax` register, set `r12` to zero and jump to `int_to_str`. Ok now we have base of our program. We already know how to print string and we have what to print. Let’s see at `str_to_int` and `int_to_str` implementation.

```nasm
str_to_int:
xor rax, rax
mov rcx, 10
next:
cmp [rsi], byte 0
je return_str
mov bl, [rsi]
sub bl, 48
mul rcx
add rax, rbx
inc rsi
jmp next
return_str:
ret
```

在`str_to_int`开始时，我们将`rax`设置为0，`rcx`设置为10。然后我们转到下一个标签。正如您在上面的示例中所看到的（`str_to_int`调用之前的第一行），我们从堆栈中将`argv[1]`放在`rsi`中。现在我们将`rsi`的第一个字节与0进行比较，因为每个字符串都以NULL结尾，如果是，我们将返回。如果不是0，我们将它的值复制到一个字节的`bl`寄存器，并从中减去48。为什么是48？从0到9的所有数字在asci表中都有48到57个代码。所以如果我们从数字符号48（例如从57）中减去，我们得到数字。然后在`rcx`上乘以`rax`（值为-10）。在此之后，我们增加`rsi`以获得下一个字节并再次循环。算法很简单。例如，如果`rsi`指向‘5’ ‘7’ ‘6’ ‘\000’序列，则将执行以下步骤：

At the start of `str_to_int`, we set up `rax` to 0 and `rcx` to 10. Then we go to next label. As you can see in above example (first line before first call of `str_to_int`) we put `argv[1]` in `rsi` from stack. Now we compare first byte of `rsi` with 0, because every string ends with NULL symbol and if it is we return. If it is not 0 we copy it’s value to one byte `bl` register and substract 48 from it. Why 48? All numbers from 0 to 9 have 48 to 57 codes in asci table. So if we substract from number symbol 48 (for example from 57) we get number. Then we multiply `rax` on `rcx` (which has value - 10). After this we increment `rsi` for getting next byte and loop again. Algorthm is simple. For example if `rsi` points to ‘5’ ‘7’ ‘6’ ‘\000’ sequence, then will be following steps:

```nasm
rax = 0
get first byte - 5 and put it to rbx
rax * 10 --> rax = 0 * 10
rax = rax + rbx = 0 + 5
Get second byte - 7 and put it to rbx
rax * 10 --> rax = 5 * 10 = 50
rax = rax + rbx = 50 + 7 = 57
and loop it while rsi is not \000
```

在`str_to_int`之后我们将得到`rax`中的数字。现在让我们看看`int_to_str`：

After `str_to_int` we will have number in `rax`. Now let’s look at `int_to_str`:

```nasm
int_to_str:
mov rdx, 0
mov rbx, 10
div rbx
add rdx, 48
add rdx, 0x0
push rdx
inc r12
cmp rax, 0x0
jne int_to_str
jmp print
```

这里我们把0放入`rdx`，10放入`rbx`。然后我们执行除以`rbx`。如果我们在`str_to_int`调用之前查看上面的代码。我们将看到`rax`包含两个命令行参数的整数和。通过这个指令，我们将`rax`值除以`rbx`值，并在`rdx`中得到提醒，在`rax`中得到整个部分。接下来我们添加到`rdx` 48和`0x0`。在添加48之后，我们将得到这个数字的asci符号，所有字符串都以`0x0`结尾。在此之后，我们将符号保存到堆栈中，增量`r12`（在第一次迭代时为0，在开始时设置为0）并将`rax`与0进行比较，如果是0，则意味着我们结束将整数转换为字符串。算法的一步一步如下：例如我们有23

Here we put 0 to `rdx` and 10 to `rbx`. Than we exeute div `rbx`. If we look above at code before `str_to_int` call. We will see that `rax` contains integer number - sum of two command line arguments. With this instruction we devide `rax` value on `rbx` value and get reminder in `rdx` and whole part in `rax`. Next we add to `rdx` 48 and `0x0`. After adding 48 we’ll get asci symbol of this number and all strings much be ended with `0x0`. After this we save symbol to stack, increment `r12` (it’s 0 at first iteration, we set it to 0 at the `_start`) and compare `rax` with 0, if it is 0 it means that we ended to convert integer to string. Algorithm step by step is following: For example we have number 23

```nasm
123 / 10. rax = 12; rdx = 3
rdx + 48 = "3"
push "3" to stack
compare rax with 0 if no go again
12 / 10. rax = 1; rdx = 2
rdx + 48 = "2"
push "2" to stack
compare rax with 0, if yes we can finish function execution and we will have "2" "3" ... in stack
```

我们实现了两个有用的函数`int_to_str`和`str_to_int`，用于将整数转换为字符串，反之亦然。现在我们有两个整数的和，它们被转换成字符串并保存在堆栈中。我们可以打印结果：

We implemented two useful function `int_to_str` and `str_to_int` for converting integer number to string and vice versa. Now we have sum of two integers which was converted into string and saved in the stack. We can print result:

```nasm
print:
;;;; calculate number length
mov rax, 1
mul r12
mov r12, 8
mul r12
mov rdx, rax
;;;; print sum
mov rax, SYS_WRITE
mov rdi, STD_IN
mov rsi, rsp
;; call sys_write
syscall
jmp exit
```

我们已经知道如何用`sys_write` syscall打印字符串，但这里有一个有趣的部分。我们必须计算字符串的长度。如果您查看`int_to_str`，您将看到我们每次迭代都递增`r12`寄存器，因此它包含我们的字符串的字数。我们必须将它乘以8（因为我们将每个符号都推到堆栈中）,它将是需要打印字符串的长度。在这之后，我们每次都把1放在`rax`（`sys_write` number）、1放在`rdi`（`stdin`）、字符串长度放在`rdx`和指向栈顶的指针放在`rsi`（start of string）上。完成我们的程序：

We already know how to print string with `sys_write` syscall, but here is one interesting part. We must to calculate length of string. If you will look on the `int_to_str`, you will see that we increment `r12` register every iteration, so it contains amount of digits in our number. We must multiple it to 8 (because we pushed every symbol to stack) and it will be length of our string which need to print. After this we as everytime put 1 to `rax` (`sys_write` number), 1 to `rdi` (`stdin`), string length to `rdx` and pointer to the top of stack to `rsi` (start of string). And finish our program:

```nasm
exit:
mov rax, SYS_EXIT
exit code
mov rdi, EXIT_CODE
syscall
```

就这些了!

That's all.

### Say hello to x86_64 Assembly [part 3]

最近在学习x64汇编,在github上面找到了一点学习资料,入门级别的,因为想细致的学习一下,所以顺便久把作者的内容都翻译了一下,也不知道自己翻译的是否合适,请大家看看有问题的地方请批评指正.第一次做翻译,做的不好请大家原谅,

[作者原文](https://bbs.kanxue.com/elink@3baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5H3P5p5q4j5i4K6u0r3j5i4y4E0)

---

堆栈是存储器中的一个特殊区域，其工作原理是后进先出。

The stack is special region in memory, which operates on the principle lifo (Last Input, First Output).

我们有16个通用寄存器用于临时数据存储。它们是`RAX`、`RBX`、`RCX`、`RDX`、`RDI`、`RSI`、`RBP`、`RSP`和`R8-R15`。对于应用程序来说太少了。所以我们可以在堆栈中存储数据。堆栈的另一个用法是：当我们调用函数时，返回地址的副本保存在堆栈中。函数执行结束后，返回地址被复制到（`RIP`）中应用程序将继续从函数后的下一个位置执行。

We have 16 general-purpose registers for temporary data storage. They are `RAX`, `RBX`, `RCX`, `RDX`, `RDI`, `RSI`, `RBP`, `RSP` and `R8-R15`. It’s too few for serious applications. So we can store data in the stack. Yet another usage of stack is following: When we call a function, return address copied in stack. After end of function execution, address copied in commands counter (`RIP`) and application continue to executes from next place after function.

例子 :

```
global _start



section .text



_start:

        mov rax, 1

        call incRax

        cmp rax, 2

        jne exit

        ;;

        ;; Do something

        ;;



incRax:

        inc rax

        ret

```

这里我们可以看到，在应用程序运行之后，`rax`等于1。然后我们调用`incRax`函数，它将`rax`值增加到1，现在`rax`值必须是2。在这个执行之后，继续从8行开始，我们将`rax`值与2进行比较。同样，我们可以在[System V AMD64 ABI](https://bbs.kanxue.com/elink@87dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5r3q4^5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6S2M7$3#2Q4y4h3j5K6i4K6u0r3N6%4N6%4i4K6u0W2P5o6R3$3i4K6u0V1y4U0c8Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4#2L8h3g2F1N6r3q4@1K9h3!0F1i4K6u0r3j5h3u0A6i4K6u0W2M7r3c8X3)中读取，前六个函数参数在寄存器中传递。他们是：

Here we can see that after application runnning, `rax` is equal to 1. Then we call a function `incRax`, which increases `rax` value to 1, and now `rax` value must be 2. After this execution continues from 8 line, where we compare `rax` value with 2. Also as we can read in [System V AMD64 ABI](https://bbs.kanxue.com/elink@fdeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5H3P5r3q4^5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6S2M7$3#2Q4y4h3j5K6i4K6u0r3N6%4N6%4i4K6u0W2P5o6R3$3i4K6u0V1y4U0c8Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4#2L8h3g2F1N6r3q4@1K9h3!0F1i4K6u0r3j5h3u0A6i4K6u0W2M7r3c8X3), the first six function arguments passed in registers. They are:

- `rdi`-第一个参数

- `rsi`-第二个参数

- `rdx`-第三个参数

- `rcx`-第四个参数

- `r8`-第五个参数

- `r9`-第六个参数

- `rdi` - first argument

- `rsi` - second argument

- `rdx` - third argument

- `rcx` - fourth argument

- `r8` - fifth argument

- `r9` - sixth

下一个参数将在堆栈中传递。如果我们有这样的函数：

Next arguments will be passed in stack. So if we have function like this:

```
int foo(int a1, int a2, int a3, int a4, int a5, int a6, int a7)

{

    return (a1 + a2 - a3 - a4 + a5 - a6) * a7;

}
```

然后，前六个参数将在寄存器中传递，但7个参数将在堆栈中传递。

Then first six arguments will be passed in registers, but 7 argument will be passed in stack.

#### 堆栈指针

Stack pointer

正如我所说，我们有16个通用寄存器，有两个有趣的寄存器-`RSP`和`RBP`。`RBP`是基指针寄存器。它指向当前堆栈帧的基部。`RSP`是堆栈指针，它指向当前堆栈帧的顶部。
