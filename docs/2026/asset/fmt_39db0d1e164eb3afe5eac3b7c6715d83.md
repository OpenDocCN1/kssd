
这个字符串可以是可执行模块的[绝对路径](https://bbs.kanxue.com/elink@932K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2b7V1u0Q4x3U0f1&6c8q4)9J5y4f1f1#2i4K6t1#2b7f1k6Q4x3U0g2n7z5g2)9J5y4f1f1^5i4K6t1#2b7U0N6Q4x3U0g2m8c8W2)9J5y4f1f1#2i4K6t1#2b7V1g2Q4x3U0f1^5y4l9`.`.)，也可以是[相对路径](https://bbs.kanxue.com/elink@426K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2z5f1u0Q4x3U0g2n7z5q4)9J5y4f1f1#2i4K6t1#2b7f1k6Q4x3U0g2n7z5g2)9J5y4f1f1^5i4K6t1#2b7U0N6Q4x3U0g2m8c8W2)9J5y4f1f1#2i4K6t1#2b7V1g2Q4x3U0f1^5y4l9`.`.)，在后一种情况下，函数使用当前[驱动器](https://bbs.kanxue.com/elink@2d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1&6i4K6t1#2b7e0W2Q4x3U0g2n7x3g2)9J5y4f1f1#2i4K6t1#2z5p5q4Q4x3U0g2m8z5q4)9J5y4f1f1#2i4K6t1#2z5e0W2Q4x3U0g2m8z5l9`.`.)和目录建立可执行模块的路径。

这个参数可以被设为`NULL`，在这种情况下，可执行模块的名字必须处于`lpCommandLine`参数最前面并由[空格符](https://bbs.kanxue.com/elink@d01K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2b7e0W2Q4x3U0g2n7b7g2)9J5y4f1f1$3i4K6t1#2b7e0m8Q4x3U0g2n7b7#2)9J5y4f1f1%4i4K6t1#2b7f1y4Q4x3U0g2m8y4R3`.`.)与后面的字符分开。

### `lpCommandLine`

指向一个以`NULL`结尾的字符串，该字符串指定要执行的命令行。

这个参数可以为空，那么函数将使用`lpApplicationName`参数指定的字符串当做要运行的程序的命令行。

如果`lpApplicationName`和`lpCommandLine`参数都不为空，那么`lpApplicationName`参数指定将要被运行的模块，`lpCommandLine`参数指定将被运行的模块的命令行。新运行的进程可以使用[`GetCommandLine`](https://bbs.kanxue.com/elink@ca0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8V1N6W2N6p5y4G2L8h3#2S2L8X3c8x3K9h3&6W2)函数获得整个命令行。C语言程序可以使用`argc`和`argv`参数。

### `lpProcessAttributes`

指向一个`SECURITY_ATTRIBUTES`结构体，这个[结构体](https://bbs.kanxue.com/elink@2f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2b7V1u0Q4x3U0f1&6x3#2)9J5y4f1f1$3i4K6t1#2z5f1g2Q4x3U0f1^5y4q4)9J5y4f1f1@1i4K6t1#2b7V1c8Q4x3U0f1&6x3H3`.`.)决定是否返回的句柄可以被子进程继承。如果`lpProcessAttributes`参数为空（`NULL`），那么句柄不能被继承。

**在Windows NT中**：[`SECURITY_ATTRIBUTES`](https://bbs.kanxue.com/elink@be4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2y4q4b7#2g2d9d9g2c8k6i4K6g2X3b7g2c8f1f1V1W2n7g2g2c8q4f1H3`.`.)结构的`lpSecurityDescriptor`成员指定了新进程的[安全描述符](https://bbs.kanxue.com/elink@f98K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1#2i4K6t1#2b7f1g2Q4x3U0f1^5z5g2)9J5y4f1f1#2i4K6t1#2z5o6g2Q4x3U0g2m8z5q4)9J5y4f1f1$3i4K6t1#2z5p5k6Q4x3U0f1^5c8W2)9J5y4f1f1^5i4K6t1#2b7V1k6Q4x3U0g2n7x3q4)9J5y4f1f1%4i4K6t1#2b7f1y4Q4x3U0g2m8y4R3`.`.)，如果参数为空，新进程使用默认的安全描述符。

### `lpThreadAttributes`

同`lpProcessAttribute`,不过这个参数决定的是线程是否被继承.通常置为`NULL`.

### `bInheritHandles`

指示新进程是否从调用进程处继承了句柄。

如果参数的值为真，调用进程中的每一个可继承的打开句柄都将被子进程继承。被继承的句柄与原进程拥有完全相同的值和访问权限。

### `dwCreationFlags`

指定附加的、用来控制优先类和进程的创建的标志。以下的创建标志可以以除下面列出的方式外的任何方式组合后指定。

⑴值：`CREATE_DEFAULT_ERROR_MODE`

含义：新的进程不继承调用进程的错误模式。`CreateProcess`函数赋予新进程当前的默认错误模式作为替代。应用程序可以调用[`SetErrorMode`](https://bbs.kanxue.com/elink@925K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2y4W2N6p5g2J5M7X3!0J5e0h3!0V1k6b7`.`.)函数设置当前的默认错误模式。

这个标志对于那些运行在没有硬件错误环境下的多线程[外壳程序](https://bbs.kanxue.com/elink@b29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1#2i4K6t1#2b7e0c8Q4x3U0f1&6y4W2)9J5y4f1f1#2i4K6t1#2b7e0y4Q4x3U0g2n7x3#2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7W2)9J5y4f1f1#2i4K6t1#2b7V1q4Q4x3U0f1^5c8R3`.`.)是十分有用的。

对于`CreateProcess`函数，默认的行为是为新进程继承调用者的错误模式。设置这个标志以改变默认的处理方式。

⑵值：`CREATE_NEW_CONSOLE`

含义：新的进程将使用一个新的控制台，而不是继承[父进程](https://bbs.kanxue.com/elink@d9aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2z5o6S2Q4x3U0g2n7y4W2)9J5y4f1f1^5i4K6t1#2b7V1k6Q4x3U0f1&6b7W2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7R3`.`.)的控制台。这个标志不能与`DETACHED_PROCESS`标志一起使用。

⑶值：`CREATE_NEW_PROCESS_GROUP`

含义：新进程将是一个[进程树](https://bbs.kanxue.com/elink@c16K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1^5i4K6t1#2b7V1k6Q4x3U0f1&6b7W2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7W2)9J5y4f1f1$3i4K6t1#2b7e0m8Q4x3U0f1&6x3b7`.`.)的根进程。进程树中的全部进程都是根进程的子进程。新进程树的[用户标识符](https://bbs.kanxue.com/elink@a4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2z5e0c8Q4x3U0g2m8z5q4)9J5y4f1f1$3i4K6t1#2z5o6S2Q4x3U0g2n7y4#2)9J5y4f1f1$3i4K6t1#2b7e0m8Q4x3U0f1^5y4#2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1^5y4W2)9J5y4f1f1%4i4K6t1#2b7f1y4Q4x3U0g2m8y4R3`.`.)与这个进程的标识符是相同的，由`lpProcessInformation`参数返回。进程树经常使用`GenerateConsoleCtrlEvent`函数允许发送`CTRL+C`或`CTRL+BREAK`信号到一组控制台进程。

⑷值：`CREATE_SEPARATE_WOW_VDM`

如果被设置，新进程将会在一个私有的虚拟DOS机（VDM）中运行。另外，默认情况下所有的16位Windows应用程序都会在同一个共享的VDM中以[线程](https://bbs.kanxue.com/elink@000K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2b7V1q4Q4x3U0g2n7c8W2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7R3`.`.)的方式运行。单独运行一个16位程序的优点是一个应用程序的崩溃只会结束这一个VDM的运行；其他那些在不同VDM中运行的程序会继续正常的运行。同样的，在不同VDM中运行的16位Windows应用程序拥有不同的输入队列，这意味着如果一个程序暂时失去响应，在独立的VDM中的应用程序能够继续获得输入。

⑸值：`CREATE_SHARED_WOW_VDM`

如果`WIN.INI`中的Windows段的`DefaultSeparateVDM`选项被设置为真，这个标识使得`CreateProcess`函数越过这个选项并在共享的虚拟DOS机中运行新进程。

⑹值：`CREATE_SUSPENDED`

含义：新进程的[主线程](https://bbs.kanxue.com/elink@9aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1@1i4K6t1#2b7U0S2Q4x3U0g2n7b7W2)9J5y4f1f1%4i4K6t1#2b7V1q4Q4x3U0g2n7c8W2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7R3`.`.)会以暂停的状态被创建，直到调用[`ResumeThread`](https://bbs.kanxue.com/elink@695K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2u0W2M7%4g2E0k6g2c8Z5M7X3g2S2k6l9`.`.)函数被调用时才运行。

⑺值：`CREATE_UNICODE_ENVIRONMENT`

含义：如果被设置，由`lpEnvironment`参数指定的环境块使用Unicode[字符](https://bbs.kanxue.com/elink@279K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1#2i4K6t1#2b7f1c8Q4x3U0f1&6y4#2)9J5y4f1f1%4i4K6t1#2b7f1y4Q4x3U0g2m8y4R3`.`.)，如果为空，环境块使用ANSI字符。

⑻值：`DEBUG_PROCESS`

含义：如果这个标志被设置，调用进程将被当做一个[调试程序](https://bbs.kanxue.com/elink@341K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1^5i4K6t1#2b7U0m8Q4x3U0f1^5x3#2)9J5y4f1f1^5i4K6t1#2b7f1k6Q4x3U0f1&6y4g2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7W2)9J5y4f1f1#2i4K6t1#2b7V1q4Q4x3U0f1^5c8R3`.`.)，并且新进程会被当做被调试的进程。系统把被调试程序发生的所有调试事件通知给调试器。

如果你使用这个标志创建进程，只有调用进程（调用`CreateProcess`函数的进程）可以调用[`WaitForDebugEvent`](https://bbs.kanxue.com/elink@63dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2N6S2K9i4c8r3L8%4u0p5k6h3u0#2k6@1g2$3k6h3&6@1)函数。

⑼值：`DEBUG_ONLY_THIS_PROCESS`

含义：如果此标志没有被设置且调用进程正在被调试，新进程将成为调试调用进程的调试器的另一个调试对象。如果调用进程没有被调试，有关调试的行为就不会产生。

⑽值：`DETACHED_PROCESS`

含义：对于控制台进程，新进程没有访问[父进程](https://bbs.kanxue.com/elink@231K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8W2)9J5y4f1f1%4i4K6t1#2z5o6S2Q4x3U0g2n7y4W2)9J5y4f1f1^5i4K6t1#2b7V1k6Q4x3U0f1&6b7W2)9J5y4f1f1%4i4K6t1#2b7e0S2Q4x3U0f1^5b7R3`.`.)控制台的权限。新进程可以通过[`AllocConsole`](https://bbs.kanxue.com/elink@fbbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1j5h3W2C8k6g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3K9i4c8W2L8g2)9J5c8V1q4D9L8r3!0U0b7$3!0F1M7$3!0D9k6b7`.`.)函数自己创建一个新的控制台。这个标志不可以与`CREATE_NEW_CONSOLE`标志一起使用。

〔11〕值：`CREATE_NO_WINDOW`

含义：系统不为新进程创建CUI窗口，使用该标志可以创建不含窗口的CUI程序。

### `dwCreationFlags`参数
