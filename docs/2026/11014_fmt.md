
# [原创][讨论]关于x64平台下的R3 Hook，windows 64 位系统 Hook

因为x86下的调用协议到了x64下都消失了，

vc++ x64 开发竟然取消了 内联汇编，使用汇编的话要不买inter的编译器（买不起....），另一种方法是编写 asm文件，转换成obj文件来在代码中声明，虽然可以汇编，但是变成了调用函数的形式。

哎....

有个hook函数的想法，想让大神看看如何 （望指点...错误的地方 笑纳）

一个函数

`proc`（`int a,int b,int c,int d,int e,int f`）

{

}

在 `_tmain` 下调用此函数

`proc`（1,2,3,4,5,6）

汇编形式如下

```
mov [rsp+28h],6
mov [rsp+20h],5
mov r9d,4
mov r8d,3
mov edx,2
mov ecx,1
call proc
```

`proc` 内部如下：

```
mov [rsp + 20h],r9d
mov [rsp + 18h],r8d
mov [rsp + 10h],rdx
mov [rsp + 08h],ecx
push rdi
```

此时栈的内容：

[`rsp`] == ret address

[`rsp` + 08h] == 1

[`rsp` + 10h] == 2

[`rsp` + 18h] == 3

[`rsp` + 20h] == 4

[`rsp` + 28h] == 5

[`rsp` + 30h] == 6

## Hook proc

```
mov [rsp+28h],6
mov [rsp+20h],5
mov r9d,4
mov r8d,3
mov edx,2
mov ecx,1
call proc
```

```
---- mov [rsp + 20h],r9d --   hook处 改   call MyHook  Call占5字节   mov 占 8字节
mov [rsp + 18h],r8d
mov [rsp + 10h],rdx
mov [rsp + 08h],ecx
push rdi
```

栈内容如下：

[`rsp`] == 【 `MyHook` 】 ret address

[`rsp` + 08h] == 【 `proc` 】 ret address

[`rsp` + 10h] == 1

[`rsp` + 18h] == 2

[`rsp` + 20h] == 3

[`rsp` + 28h] == 4

[`rsp` + 30h] == 5

[`rsp` + 38h] == 6

`MyHook` (`retAddr`/\* 被Hook函数的返回地址 \*/,`value1` /\* 参数1 \*/,`value2`,`value3`,`value4`,`value5`,`value6`)

{

}

这样指向到 自己的 `MyHook` 函数当中，通过修改 `[ESP]` 内容 确认返回地址

跳转到 申请的 heap 处理完成后 跳转

目的只为 不写汇编，因为x64下没内联汇编了（这是个痛处啊）

如果想在hook 完毕以后执行自己的另一端代码可以修改myhook 的第一个参数，

`retaddr` = `xxxx`

这样在 `proc` 执行完毕之后 会跳转到自己指定的地址。

可以判断 函数 的 返回内容. 个人感觉这样比 `jmp` 好用！

内容原创哦....

有不足的地方，祈求大神指点，或者更好的方法

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
