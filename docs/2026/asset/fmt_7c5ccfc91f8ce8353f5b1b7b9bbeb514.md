
```asm
import kernel,\
         ExitProcess,'ExitProcess'

;szCaption  db 'test',0

;

;     section '.import' import data readable writeable

;     library kernel32, 'kernel32.dll',user32, 'user32.dll'

;     include 'api\kernel32.inc'

;     include 'api\user32.inc'
```

///////////////////////////////////////////////

注释的很详细了 需要说明的有这么几点

### 1
堆栈平衡是重重之重，如果你记不住，就在每一行代码后面加好注释

搞清楚这个时候堆栈里面都还有些什么

### 2
`PE32+`改变比较大的也就是`NT`头变成了`IMAGE_NT_HEADERS64`，使一些偏移发生了变化，可以自行`dt`查看

### 3
导出表没有发生变化

### 4
导出表的后三个成员一定要好好看看，这是看懂代码的关键所在。

### 4 对`call WinExec`的说明

最后的代码我是这样写的

```asm
         mov rdx,5
         lea rcx,[calc]
         call        rax
```

实际上`googlecode`上面的那个作者也是用的`rdx rcx`传递的参数的，他的代码是这样的:

```asm
    PUSH    B2DW('c', 'a', 'l', 'c')      ; Stack = "calc", 0
    PUSH    RSP
    POP     RCX                           ; RCX = &("calc")
    PUSH    RCX                           ; WinExec messes with stack -
    CDQ                                   ; RDX = 0
    CALL    RDI                           ; WinExec(&("calc"), 0);
```

你可能问，用`r9`/`r8`传递参数不可以吗，我用`r9 r8`传递参数结果程序`crash`。我们写个简单的小程序，使用`WinExec`调用`calc`，`IDA`中显示这样的结果

```asm
; int __cdecl main(int argc, const char **argv, const char **envp)
main proc near
sub     rsp, 28h
lea     rcx, CmdLine    ; "calc.exe"
mov     edx, 5          ; uCmdShow
call    cs:__imp_WinExec
xor     eax, eax
add     rsp, 28h
retn
```

调用`WinExec`的时候 `windows`本身就用的是`rcx`/`rdx`传递的参数，我们也还是老老实实的用`rdx`/`rcx`传递参数吧

`Fasm` 完整可编译工程

[noprintfwinexec.7z](https://bbs.kanxue.com/attach-download-70050.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [noprintfwinexec.7z](javascript:void(0);)

  （3.56kb，47次下载）
