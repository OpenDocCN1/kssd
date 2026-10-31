
# [原创]栈迁移原理（图示）

主要用的就是利用 `leave;ret;` 这样的 gadgets。

![](https://bbs.kanxue.com/upload/attach/202003/837755_K35U4D8EUPCF27Y.jpg)

假设，我们有一个程序，存在栈溢出漏洞，我们把内容覆盖成了下面这样子

当然此时 `bss` 段或者 `data` 段还没有内容，待会会通过 `read` 函数输入：
![](https://bbs.kanxue.com/upload/attach/202003/837755_6E545YD2GRG373N.jpg)

而实际上在程序调用完成 `call` 返回的时候就会有这样的指令

```
mov esp,ebp
pop ebp
ret
```
![](https://bbs.kanxue.com/upload/attach/202003/837755_CMQPGBY5PRG77ZA.jpg)

当我们挨个去执行的时候会出现这样的情况

首先是 `mov esp,ebp` 执行完以后变成了这个样子：
![](https://bbs.kanxue.com/upload/attach/202003/837755_M3P54EM8DDT93JM.jpg)

然后 `pop ebp` 执行完后就是

别忘了，`pop` 指令是把栈顶的值弹到指定的寄存器，也就是说 `esp` 会自动的减一个单位
![](https://bbs.kanxue.com/upload/attach/202003/837755_8SYDECJHS5NSYFV.jpg)

这时候就到 `ret` 了，我们可以通过 `read` 函数来把内容输入到 `fake ebp1` 的地址处

构造的内容主要是把 `fake ebp1` 处写成 `fake ebp2` 的地址
![](https://bbs.kanxue.com/upload/attach/202003/837755_4C54UMNUWCSQ4BM.jpg)

`read` 函数执行完成以后程序返回到了 `leave_ret`，这样就会在执行一遍上面说的那样

首先是 `mov esp,ebp` 执行完成后效果如下：
![](https://bbs.kanxue.com/upload/attach/202003/837755_4EKU7Z29HP7ECAK.jpg)

然后是 `pop ebp`，执行完成后：
![](https://bbs.kanxue.com/upload/attach/202003/837755_ZSA7KKY8MAZCM4W.jpg)

此时在执行 `ret` 命令，他就会执行我们构造在 `bss` 段后者 `data` 段的那个函数
![](https://bbs.kanxue.com/upload/attach/202003/837755_Y2XQXFCXQBHQQTD.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
