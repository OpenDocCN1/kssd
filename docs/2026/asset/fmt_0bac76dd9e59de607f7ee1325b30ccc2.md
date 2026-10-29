
# [原创]【从源码过反调试】二、过PTRACE_TRACEME

## `PTRACE_TRACEME` 反调试

原理有两条

- 反调试先附加自己，防止其他 `ptrace` 附加。
- 反调试后附加自己失败，则被其他 `ptrace` 附加

其实就这样：

```
if (ptrace(PTRACE_TRACEME,...) == -1)
{
// exit ...
}
```

常见的过这个反调试是 `hook` `ptrace` 调用，过滤参数，返回。

我的反反调试思路也和这个差不多：

**伪造 `ptrace(PTRACE_ME,...)` 的返回值为 `0`，这样**

**反调试先附加自己，以为自己附加成功了，实际上失败了，我们调试器可以附加上。**

**反调试后附加自己，以为自己附加成功了，实际上他失败了，所以认为没有其他 `ptrace` 附加。**

这也就骗过了反调试。

可修改源码的情况下，这里直接 `return` 即可。

![图片描述](upload/attach/202210/792494_9XYU9CT7AQ4FXQH.png)

但是这样相当于禁用了全局的 `ptrace` 的 `PTRACE_TRACEME` 功能， 有没有其他影响我还没有发现，目前还没遇到有问题的地方，后面遇到的话打算在这里加个 `pid` 筛选功能，只针对特定进程开启过滤。

和这帖子的方法有异曲同工之处，也是自己改完想学学大佬的方法，才发现的。比较下来，还是感觉改系统源码这种方式一劳永逸，但也学习了大佬的思路。（如果没有破坏稳定性的话）

https://bbs.pediy.com/thread-212404.htm

如果有其他方式调用，绕过了这种反反调试，还是可以在 `ptrace_traceme` 函数内部直接 `return` 的

![图片描述](upload/attach/202210/792494_X7FWHVNPAWZY2H8.png)

## `PTRACE_TRACEME` 反调试

原理有两条

反调试先附加自己，防止其他 `ptrace` 附加。

反调试后附加自己失败，则被其他 `ptrace` 附加

其实就这样：

|  |  |
| --- | --- |
| 1 2 3 4 | `if` `(ptrace(PTRACE_TRACEME,...)` `==` `-1)` `{` `// exit ...` `}` |

常见的过这个反调试是 `hook` `ptrace` 调用，过滤参数，返回。

我的反反调试思路也和这个差不多：

**伪造 `ptrace(PTRACE_ME,...)` 的返回值为 `0`，这样**

**反调试先附加自己，以为自己附加成功了，实际上失败了，我们调试器可以附加上。**

**反调试后附加自己，以为自己附加成功了，实际上他失败了，所以认为没有其他 `ptrace` 附加。**

这也就骗过了反调试。

可修改源码的情况下，这里直接 `return` 即可。

![图片描述](https://bbs.kanxue.com/upload/attach/202210/792494_9XYU9CT7AQ4FXQH.png)

但是这样相当于禁用了全局的 `ptrace` 的 `PTRACE_TRACEME` 功能， 有没有其他影响我还没有发现，目前还没遇到有问题的地方，后面遇到的话打算在这里加个 `pid` 筛选功能，只针对特定进程开启过滤。

和这帖子的方法有异曲同工之处，也是自己改完想学学大佬的方法，才发现的。比较下来，还是感觉改系统源码这种方式一劳永逸，但也学习了大佬的思路。（如果没有破坏稳定性的话）

https://bbs.pediy.com/thread-212404.htm

如果有其他方式调用，绕过了这种反反调试，还是可以在 `ptrace_traceme` 函数内部直接 `return` 的

![图片描述](https://bbs.kanxue.com/upload/attach/202210/792494_X7FWHVNPAWZY2H8.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
