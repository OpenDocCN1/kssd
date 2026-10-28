
另外，我自己实现了`GetModuleFileName`（我自己实现的函数的函数名，下面的括号中，如果没有特殊说明，都是指我自己实现的函数的函数名,函数名：`GetDllModuleFileName`）

`LoadLibrary`(`OtherLoadLibrary`)

`GetModuleHandle`(`GetDllModuleHandle`)

`FreeLibrary`(`OtherFreeLibrary`)

`OpenProcess`(`OpenProcessByProcessId`)

`OpenThread`(`OpenThreadByThreadId`)

`GetProcAddress`(`GetFunctionAddressByDllExprot`,这个函数比原函数多了一个参数，这个参数返回了，被寻找函数的条目在导出表中的地址，借助修改这个地址就可以进行导出表HOOK)

在`FileSystem`工程中使用了网上收集的WINDOWS代码

实现了部分WINDOWS文件操作函数，文件名为以`FileSystem`为前缀加上WINDOWS原函数的函数名！可代替WINDOWS部分函数！大家HOOK这些WINDOWS函数时，可直接调用工程中对应已经实现的函数，不必调用原函数！

被程序在WINDOWS 7 X64中实验通过，编译环境为VS2010+WDK 7,要是在别的版本中出现如没有在NTDLL中找到某函数的错误的话，请尝试动态调用，否则请删除对对应函数的调用，或者在这里将错误反馈给我，嘿嘿!有些结构和函数在VS中没有声明的，如遇到没有声明的函数和结构，请到目录下的`winternl.h`文件红复制，要是还没有的，请到MSDN，或者是WDK中复制！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [BPHookFunction.rar](javascript:void(0);)

  （525.95kb，229次下载）
