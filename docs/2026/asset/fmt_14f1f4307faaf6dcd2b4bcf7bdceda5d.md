
# [旧帖][原创]大牛的小bug，对某Windows x64 shellcode的一点改进0.00雪花

本人新人，但是关注看雪前辈、大牛们的文章很久了，确实受益匪浅~最近一段时间对windows x64 shellcode比较感兴趣，发现看雪里有一些很好的文章，同时从互联网上看到几个页面，写了这方面，很好：1、[657K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4V1N6i4c8Q4x3X3c8T1L8%4W2Q4x3X3g2U0L8$3#2Q4x3V1k6@1j5h3N6Q4x3V1k6K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1j5`.](https://bbs.kanxue.com/elink@1eeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4V1N6i4c8Q4x3X3c8T1L8%4W2Q4x3X3g2U0L8$3#2Q4x3V1k6@1j5h3N6Q4x3V1k6K6K9r3g2D9L8r3y4G2k6r3g2Q4x3V1j5`.) 2、[996K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2U0k6r3g2J5L8h3!0@1N6r3y4&6j5X3g2J5M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8^5y4U0c8Q4x3X3c8K6K9r3g2D9L8r3y4G2k6r3f1`.](https://bbs.kanxue.com/elink@c3aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2U0k6r3g2J5L8h3!0@1N6r3y4&6j5X3g2J5M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8^5y4U0c8Q4x3X3c8K6K9r3g2D9L8r3y4G2k6r3f1`.)，其中第二篇写得更棒，Bill McDermott写的，崇拜and敬仰~

于是，拿他的代码去机器上调试，结果测出了一点问题，具体如下：

他说，

```
    lea rdx, loadlib_func
    lea rcx, kernel32_dll
    call lookup_api         ;get address of LoadLibraryA
    mov r15, rax            ;save for later use with forwarded exports

    lea rcx, user32_dll
    call rax                ;load user32.dll

    lea rdx, msgbox_func
    lea rcx, user32_dll
    call lookup_api         ;get address of MessageBoxA

    xor r9, r9              ;MB_OK
    lea r8, title_str       ;caption
    lea rdx, hello_str      ;Hello world
    xor rcx, rcx            ;hWnd (NULL)
    call rax                ;display message box

    kernel32_dll    db  'KERNEL32.DLL', 0
    loadlib_func    db  'LoadLibraryA', 0
    user32_dll      db  'USER32.DLL', 0
    msgbox_func     db  'MessageBoxA', 0
    hello_str       db  'Hello world', 0
    title_str       db  'Message', 0
```

通过以上代码就可以，跳出一个messagebox，没错确实可以的，但是我把代码做了下修改,出问题了（我的代码在goasm下编译，语法有相应修改）

```
    Lea rdx, str_loadlib_func
    Lea rcx, str_kernel32_dll
    Call lookup_api         ;get address of LoadLibraryA,rax=LoadLibraryA
    Mov r15, rax

    Lea rdx, str_create_thread
    Lea rcx, str_kernel32_dll
    Call lookup_api         ;get address of createthread to rax

    str_kernel32_dll: DB 'KERNEL32.DLL', 0H
    str_loadlib_func: DB 'LoadLibraryA', 0H
    str_create_thread: DB 'CreateThread', 0H
```

调用`createthread` api函数时，程序出现异常，退出了~写程序调试一下看问题出在哪里：

```
    Invoke LoadLibrary, Addr str_kernel32_dll
    Invoke GetProcAddress, rax, Addr str_create_thread
    Int 3
    Lea rdx, str_loadlib_func
    Lea rcx, str_kernel32_dll
    Call lookup_api         ;get address of LoadLibraryA,rax=LoadLibraryA
    Mov r15, rax
    Int 3

    Lea rdx, str_create_thread
    Lea rcx, str_kernel32_dll
    Call lookup_api         ;get address of createthread to rax
    Int 3

    Ret

    str_kernel32_dll: DB 'KERNEL32.DLL', 0H
    str_loadlib_func: DB 'LoadLibraryA', 0H
    str_create_thread: DB 'CreateThread', 0H
```

用windbg断住每个`int3`，看正确的`createthread`地址 与我们获取的有什么不同。

这是第一个`int 3`现场

![](https://bbs.kanxue.com/upload/attach/201204/509507_8lbza5mh880aorr.png)

注意`rax`的值是`76fa6210h`,这是`createthread`的正确地址，再看第三个`int 3`中断现场

![](https://bbs.kanxue.com/upload/attach/201204/509507_f1dz528pblkcvcg.png)

`rax`的值是`76f9a6a0h`，与正确值`76fa6210h`不一样，显然是错了，那为什么呢？

windbg跟踪一下`lookup_api`函数，发现问题了，这是调试过程中的一个现场，![](https://bbs.kanxue.com/upload/attach/201204/509507_mevj359ps4yzw9l.png)

注意到这里`lookup_api`函数已经匹配成功了，是字符串“`CreateThread`”，但是我们看看`rdi`指向的地址`77034584h`，是什么呢？图片右下角列出来了“`poolWork...`”，当然，这不重要，重要的是它的前面是“`CreateThread`”，哈哈，问题找到了，代码作者做字符串匹配有个小bug，把“`CreateThreadpoolWork`”当成“`CreateThread`”了，因此获取了错误的函数地址。ok，拿出修改意见，还是goasm风格的：
