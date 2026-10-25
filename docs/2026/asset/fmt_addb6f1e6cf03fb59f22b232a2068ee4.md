
# 虚函数调用分析

`00401205`   `call`        `dword ptr [edx]`

`00401207`   `cmp`         `esi,esp`

`00401209`   `call`        `__chkesp` (`004092c0`)

`49:`           `p[2]->display();`

`0040120E`   `mov`         `eax,dword ptr [ebp-1Ch]`

`00401211`   `mov`         `edx,dword ptr [eax]`

`00401213`   `mov`         `esi,esp`

`00401215`   `mov`         `ecx,dword ptr [ebp-1Ch]`

`00401218`   `call`        `dword ptr [edx]`

`0040121A`   `cmp`         `esi,esp`

`0040121C`   `call`        `__chkesp` (`004092c0`)

`51:`           `return 0;`

`00401221`   `xor`         `eax,eax`

这里就是虚函数调用,他们因为虚表不同,所以输出的结果 调用的函数是不同的.

先看看

`47:`           `p[0]->display();`

```
004011E8   mov         eax,dword ptr [ebp-24h]
004011EB   mov         edx,dword ptr [eax]
004011ED   mov         esi,esp
004011EF   mov         ecx,dword ptr [ebp-24h]
004011F2   call        dword ptr [edx]
004011F4   cmp         esi,esp
004011F6   call        __chkesp (004092c0)
```

把第一个对象的`THIS`指针给`EAX`,

`mov         edx,dword ptr [eax]`

`THIS`指针开始的4个字节取内容.给`EDX`, 此时`EDX`就是`VPTR`,也就是`VTABLE`的地址

`mov         ecx,dword ptr [ebp-24h]`

把`THIS`指针在给`ECX`

`call        dword ptr [edx]`

每个类的虚函数在虚表中的位置就靠它在类中声明的位置而定,没个虚函数的位置就做为了`VTABLE`中的`ID`,因为测试代码为每个类只定义了一个`VIRTUAL FUNCTION` 所以这里就`CALL`了`EDX`指向的`VTABLE`开始的4字节的那个为一一个`VIRTUAL`函数地址

跟踪一下内存

`0012FF78`  `1C 20 43 00`

跟踪`[eax]`      `EDX = 0043201C`

跟踪`[edx]`     `0043201C`         `5F 10 40 00`

跟踪     `0040105F`   `jmp`         `a::display` (`004012b0`)

于是调用了`A`类的`DISPLAY()`函数

后边的两个调用一样会利用`VTABLE`来调用虚函数.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
