
调用 `leave` 后，`%rsp` 指向返回地址；`ret` 指令，从栈顶弹出数据，并跳转到此数据指向的地址处。在 `leave` 执行后，`%rsp` 指向返回地址，因而 `ret` 的作用就是把 `%rsp` 上移一个位置，并跳转到返回地址执行。

所以，`leave` 指令用于恢复父函数的栈帧，`ret` 用于跳转到返回地址处，`leave` 和 `ret` 配合共同完成了子函数的返回。当执行完成 `ret` 后，`%rsp` 指向的是父栈帧的结尾处，父栈帧尾部存储的调用参数由编译器自动释放。

## 函数调用示例

程序源代码如下：

```
int add(int a, int b, int c, int d, int e, int f, int g, int h) { // 8 个参数

  int sum = a + b + c + d + e + f + g + h;  //相加求和

  return sum;

}



int main(void) {

  int i = 10;

  int j = 20;

  int k = i + j;

  int sum = add(11, 22,33, 44, 55, 66, 77, 88);

  int m = k; // 为了观察 %rax Caller Save 寄存器的恢复



  return 0;

}

```

上面程序生成的和子函数调用相关的汇编程序如下：

```
add:

.LFB2:

    pushq    %rbp

.LCFI0:

    movq    %rsp, %rbp

.LCFI1:

    movl    %edi, -20(%rbp)

    movl    %esi, -24(%rbp)

    movl    %edx, -28(%rbp)

    movl    %ecx, -32(%rbp)

    movl    %r8d, -36(%rbp)

    movl    %r9d, -40(%rbp)

    movl    -24(%rbp), %eax

    addl    -20(%rbp), %eax

    addl    -28(%rbp), %eax

    addl    -32(%rbp), %eax

    addl    -36(%rbp), %eax

    addl    -40(%rbp), %eax

    addl    16(%rbp), %eax

    addl    24(%rbp), %eax

    movl    %eax, -4(%rbp)

    movl    -4(%rbp), %eax

    leave

    ret



main:

.LFB3:

    pushq    %rbp

.LCFI2:

    movq    %rsp, %rbp

.LCFI3:

    subq    $48, %rsp

.LCFI4:

    movl    $10, -20(%rbp)

    movl    $20, -16(%rbp)

    movl    -16(%rbp), %eax

    addl    -20(%rbp), %eax

    movl    %eax, -12(%rbp)

    movl    $88, 8(%rsp)

    movl    $77, (%rsp)

    movl    $66, %r9d

    movl    $55, %r8d

    movl    $44, %ecx

    movl    $33, %edx

    movl    $22, %esi

    movl    $11, %edi

    call    add

    movl    %eax, -8(%rbp)

    movl    -12(%rbp), %eax

    movl    %eax, -4(%rbp)

    movl    $0, %eax

    leave

    ret

```

首先看 `main` 函数的前三条汇编语句：

```
.LFB3:

    pushq    %rbp

.LCFI2:

    movq    %rsp, %rbp

.LCFI3:

    subq    $48, %rsp

```

保存父函数栈帧，之后创建 `main` 函数的栈帧并且分配了 48 Byte 的空间。执行完成后，`main` 函数的栈帧如下图所示：

![](https://bbs.kanxue.com/upload/attach/202002/779730_6E379KAXT4RYJZG.jpg)

继续往后走，可以看到对 `k=i+j` 的处理过程：

```
    movl    $10, -20(%rbp)

    movl    $20, -16(%rbp)

    movl    -16(%rbp), %eax

    addl    -20(%rbp), %eax

    movl    %eax, -12(%rbp)  

    movl    $88, 8(%rsp)

    movl    $77, (%rsp)

    movl    $66, %r9d

    movl    $55, %r8d

    movl    $44, %ecx

    movl    $33, %edx

    movl    $22, %esi

    movl    $11, %edi

    call    add

```

需要注意的是，传统的栈空间的利用操作是使用一个栈空间，进行一次 `push` 操作。但是我们的代码里没有这样进行，而是利用之前分配的 48 字节的空间，以空间的缩减进行分配，本质上是和 `push` 操作是一样的。最终的计算结果保存在 `%eax` 中。

后续进行 `add` 函数调用，`add` 的返回值将会放在 `%eax` 中，当前 `%eax` 中保存的是 `k` 值。所以需要首先保存 `%eax` 中的值，然后在 `add` 函数中进行调用，最后再恢复 `%eax` 的值。`%eax` 是 Caller Save 的，所以由父函数 `main` 函数来进行保存（`movl %eax,-12(%rbp)`）。

再往后，开始参数入栈，前 6 个参数依次保存到对应的寄存器中，最后两个参数从右到左压入栈中。

进入 `add` 函数后之后的操作如下：

```
add:

.LFB2:

    pushq    %rbp ; 保存父栈帧指针

.LCFI0:

    movq    %rsp, %rbp  ; 创建新栈帧

.LCFI1:

    movl    %edi, -20(%rbp)  ; 在寄存器中的参数压栈

    movl    %esi, -24(%rbp)

    movl    %edx, -28(%rbp)

    movl    %ecx, -32(%rbp)

    movl    %r8d, -36(%rbp)

    movl    %r9d, -40(%rbp)

    movl    -24(%rbp), %eax

    addl    -20(%rbp), %eax

    addl    -28(%rbp), %eax

    addl    -32(%rbp), %eax

    addl    -36(%rbp), %eax

    addl    -40(%rbp), %eax

    addl    16(%rbp), %eax

    addl    24(%rbp), %eax

    movl    %eax, -4(%rbp)

    movl    -4(%rbp), %eax

    leave

    ret

```

首先创建新栈帧，然后进行参数入栈。

在参数入栈时，我们看到并未使用 `push` 之类的指令，也没有调整 `%esp` 指针的值，而是使用了 `-N(%rbp)` 这样的指令来使用新的栈空间。这种使用“基地址+偏移量” 来使用栈的方式和直接使用 `%esp` 指向栈顶的方式其实是一样的。

当 `add` 函数返回后，返回结果存储在 `%eax` 中，`%rbp` 和 `%rsp` 调整为指向 `main` 的栈帧，之后执行 `main` 函数中的如下指令：

```
movl    %eax, -8(%rbp)  ; 保存 add 函数返回值到栈中，对应 C 语句 int sum = add(...)

movl   -12(%rbp), %eax  ; 恢复寄存器 %eax 的值，与调用add前保存 %eax 相对应 

movl    %eax, -4(%rbp) ; 对应 C 语句 m = k，%eax 中的值就是 k。

movl    $0, %eax  ; main 函数返回值

leave   ; main 函数返回

ret

```

`add` 函数返回时，把返回值保存到了 `%eax` 中，使用完返回值后，会恢复 caller save 寄存器 `%eax` 的值，这时 `main` 栈帧恢复到调用 `add` 函数之前的状态。

需要注意的是，在调用 `add` 之前，`main` 中执行了一条 `subq 48, %rsp` 的指令，这主要是因为 `main` 函数并未再调用其他函数，结尾处的 `leave`、`ret` 两条指令直接覆盖了 `%rsp` 的值从而回到了父栈帧中。如果先调整 `main` 栈帧的 `%rsp` 值，之后 `leave` 再覆盖 `%rsp` 的值，相当于调整是多余的。因而省略 `main` 中 `add` 返回之后的 `%rsp` 的调整，而使用 `leave` 直接覆盖 `%rsp` 更为合理。

之前对这一块不是很懂，看了大佬的文章后就明白了～

## 参考链接

最后附上大佬的知乎链接，这篇文章解决了我很多疑惑。

1. [知乎大佬原文](https://bbs.kanxue.com/elink@0f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0t1%4x3K6x3&6x3e0V1I4)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
