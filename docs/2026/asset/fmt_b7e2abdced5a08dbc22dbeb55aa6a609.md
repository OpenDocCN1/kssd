
```
.rdata:100C4288 48 54 0A 10                             dd offset ?OnSetActive@CPropertyPage@@UAEHXZ ; CPropertyPage::OnSetActive(void)

.rdata:100C5A30 48 54 0A 10                             dd offset ?OnSetActive@CPropertyPage@@UAEHXZ ; CPropertyPage::OnSetActive(void)

.rdata:100C5C28 48 54 0A 10                             dd offset ?OnSetActive@CPropertyPage@@UAEHXZ ; CPropertyPage::OnSetActive(void)

.rdata:100C6F00 48 54 0A 10                             dd offset ?OnSetActive@CPropertyPage@@UAEHXZ ; CPropertyPage::OnSetActive(void)

.rdata:100C86D0 48 54 0A 10                             dd offset ?OnSetActive@CPropertyPage@@UAEHXZ ; CPropertyPage::OnSetActive(void)

.rdata:100C9708 48 54 0A 10                             dd offset ?OnSetActive@CPropertyPage@@UAEHXZ ; CPropertyPage::OnSetActive(void) <- used in Project Settings
```

找`PropertyPage`时候, 发现实际上有六个派生类, 为了不涉及无辜, 还是只`hook`一个吧, 经过6次测试, 最后一个就是工程选项的`PropertyPage`回调的地方.

![](https://bbs.kanxue.com/upload/attach/201212/670_7yfpj2xdkylpbp2.png)

注入后的样子, `Microsoft Reference Sans Serif`可耻的消灭了.

另外顺便一提, 我们是处理的没有用栈传递参数的回调函数, 所以可以直接再次`call`, 压入的地址不会破坏原来的参数列表, 做完我们的事情, 最后末尾调用父类函数的情况, 可以直接`jmp`, 栈顶保留的调用方的返回地址会被父类函数使用.

不过如果有需要一开始就调用父类函数, 还需要额外使用堆栈的情况, 要么就是一个一个`push [esp+4]`, 因为每次`push`后`esp`都会自增所以重复参数数量-1次即可, 不过这样就等于增加了一次栈的深度.

还有个办法, 就是这样
```
        MOV DWORD PTR[ESP], offset label_magicret

        JMP [gfPrjMgrGui_FooPropSheetCtor] // will return to next line

label_magicret:
```
这样`jmp`后, 父类函数的`RETN`取到的就是我们的地址, 实际返回的是`label_magicret`下面这行, 而这下一行的堆栈就和返回调用处地址时候一样的布局了, 可以继续执行我们的代码, 在末尾再直接跳回去原来的返回地址.

注入证实可用以后, 就开始做资源替换版的. 用`Restrator`把资源从语言文件里解出来, 加到工程中.

编译后, 替换原文件, 运行成功, 不过关闭时候, 出现一个`MFC`处理`Terminate`中的异常. 经过反复测试, 是`hook`代码地方出的问题.

在获取`ProjectManagerGui.dll`的地址以便补丁`vft`时候, 一开始是用的`GetModuleHandle`不带路径, 然后复制了一份带路径的是`LoadLibrary`, 这样这个`ProjectManagerGui.dll`的生命周期要出问题.

改成`GetModuleHandle`带完整路径后, 变成另外的异常, 读写异常.

因为一开始设计的是注射, 所以注射和取消注射时候, `ProjectManagerGui.dll`都在内存中, 而程序终止时候就不一样了. 再次修改拆除`hook`的函数, 判断是程序终止, 就跳过拆除`hook`.

世界终于清静了~~~

附带的`idb`和工程, 对应最新的`IAR EWARM 6.40`版的, 可以用来做为类似的重载虚函数参考.

其中`Debug/Release`模式都是输出`hook`用的`dll`, `shipping`模式是编译包含资源的替换`dll`, 同时还去掉了日志, 增加了`pushad`.

为了避免在多线程调用时候, `dll`入口函数反复调用不停地补丁和恢复, 还调用了`DisableThreadLibraryCalls`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [image001.png](javascript:void(0);)
  （22.43kb，4次下载）

- [image002.png](javascript:void(0);)
  （41.00kb，8次下载）

- [image003.png](javascript:void(0);)
  （22.06kb，10次下载）

- [image004.png](javascript:void(0);)
  （40.39kb，4次下载）

- [ProjectManagerGui_idb_pediy.rar](javascript:void(0);)
  （2.56MB，23次下载）

- [IARFont_src_pediy.rar](javascript:void(0);)
  （54.67kb，28次下载）

- [LibSpyMod.rar](javascript:void(0);)
  （30.13kb，30次下载）
