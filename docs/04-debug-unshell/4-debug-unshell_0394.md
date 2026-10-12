
# Advanced System Agent

> 原文：[`www.pediy.com/kssd/pediy03/forum1014.htm`](https://www.pediy.com/kssd/pediy03/forum1014.htm)

下载地址忘了，我是在`download.com`上下到的，很容易找到，`2.73`最新版，是个多功能小软件，占资源极小。

`ASA Version 2.73`是`ASPROTECT1.2`压的，`CASPR`可以脱呀！只是无法运行。

它的历史文件如下

## Advanced System Agent: Versions history

\=======================================

**simbols:**

\+ new features  
! improvement/modification or bug fix

### Version 2.73

! Program protection improved 《《《《看这里!!!  
! Resource leak in alarms feature is fixed  
! Alarms loading and saving time is greatly improved  
! Transparency performance under `Windows 2000` is greatly improved.  
! Access violation in atomic clock synchronization fixed.  
! Improved work of dial-up function.

`asa`脱壳后一运行就马上退出，问题出在`GetProcAddress`上，

`Asprotect`先判断`function name`是否为`-1`，是就返回其“Export functions”

```assembly
exp:
  cmp eax, -1
  jnz continue
  mov eax, [edx*4+xxxxxxxx]
  jmp return

continue:
  call GetProcAddress

return:
  ret
```

只要`bpx`上面的`mov eax, [edx*4+xxxxxxxx]`，

让原`asa`运行就知道哪里用到这些`Export functions`了。

`asa`里有两处`436723`，`452c8c`

第一处是判断是否被脱壳了，是就退出，

第二处是用来返回`get`剩下天数的地址，没有该地址就是过期了。

怎么改不用说了吧(:

还有就是`nag screen`了，要`get`完剩下天数才会出`nag screen`的，

所以那个`call nag screen`的地方应该在`asa`领空的不远处(`MFC`的就跳过吧)，

一番`F12`，`F8`后来到了`436c9f`，往上看的是：

```assembly
    jmp xxxxxxxx
    push ebx
    mov ecx, esi
    call 004362e0

xxxxxxxx:
    ....
```

把以上三条指令都`nop`掉，就可以了，为什么就自己看吧。`4dd466d0e1e0cc123afbe504da2462ca`
