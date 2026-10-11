
# 时间限制一

> 原文：[`www.pediy.com/kssd/pediy03/forum390.htm`](https://www.pediy.com/kssd/pediy03/forum390.htm)

注：`SetTimer()`、`KillTimer()`、`WM_TIMER` 的破解

`X-Win32` 是个 `Win95` 下的 `X-server` 程序。

在 `http://www.starnet.com` 或 `tucows` 的 mirror 站点可以找到它。

其 4.0.1 版的演示版只允许你每次运行两个小时，之后必须重新运行它。

这个软件在安装时要输入注册码，不过这时候不一定要你输入正确的注册码。

我的破解方法如下：

1. 用 `SoftIce` 加载其主程序 `XWin32.exe`
2. 用 `bpx messageboxa` 设置断点，按 `F5` 运行。
3. 过一会就会激活 `SoftIce`，按一下 `F11` 返回调用 `MessageBoxA` 的地方，可以看到：

```
  00421A18:     CALL   [USER32!SetTimer]

               MOV    EAX,[004938FC]

               TEST   EAX,EAX

  00421A25:     JNZ    00421A63

               MOV    EDX,[0049BB20]

               LEA    ECX,[ESP+00]

               PUSH   00000100

               PUSH   ECX

               PUSH   09

               PUSH   EDX

  00421A3A:     MOV    DWORD PTR [004938FC],00000078 ;120 分钟,即 2 小时

               CALL   [USER32!LoadStringA]

               MOV    ECX,[0049BA60]

               PUSH   40

               LEA    EAX,[ESP+04]

               PUSH   004A2D1C

               PUSH   EAX

               PUSH   ECX

               CALL   [USER32!MessageBoxA]       ;显示"demon version"

  00421A63:     ADD    ESP,00000100               ;亮条停在此行

               RET
```

4. 稍加分析，就可以知道上面的程序段的作用。程序先把 `DS:[004938FC]` 中的值取出来，判断它是否为 0，为零则显示对话框，不为零就跳过对话框。我们还可以看到它在显示对话框之前把 `DS:[004938FC]` 赋值为 `78H`，即 10 进制的 120。如果 120 表示 120 分钟的话，那不正好为两小时吗？如果确实是在这个地方设置两小时限制的话，那么程序在以后肯定会来读写 `DS:[004938FC]`，所以用 `bpm 004938FC` 设置个新断点。
5. 按 `F5` 继续执行，过了一会儿又进入了 `SoftIce`！

   这次看仔细了：

```
  00421A7F: MOV EAX,[004938FC]

            DEC EAX                  ;亮条停在此行

            MOV [004938FC],EAX

  00421A8A: JZ    00421AC9           ;剩下 0 分钟,中止进程

            CMP   EAX,0F             ;剩下 15 分钟吗 ?

            JNZ   00421AE4           ;不是,继续运行

            MOV   ECX,[0049BB20]     ;显示"只剩 15 分钟"

            .

            .
```

   程序显然是在作递减工作，把那个计数器减到零的时候，也就是到了两小时的时候，就中止进程。在只剩下 15 分钟的时候它会显示个对话框，告诉你"只剩 15 分钟好用了"。
6. 我的破解方法是，把 `CS:00421A25` 处的 `JNZ 00421A63` 改为 `JMP 00421A63` 就行了。

   这样不仅跳过了对话框，而且计数器 `DS:[004938FC]` 的初值为 0，以后程序对它进行减一操作的话，它就会变成 -1，即 `FFFFFFF` 分钟，这可是个天文数字，所以你也根本不用担心程序会中止啦。

   `find: 75 3C 8B 15 20`
   `replace: EB -- -- -- --`

   以上只是一种破解方法而已，也可以把减一指令改为 `NOP`。能找到注册码当然最好。

`Netguy`, 1998.9.20

`4dd466d0e1e0cc123afbe504da2462ca`
