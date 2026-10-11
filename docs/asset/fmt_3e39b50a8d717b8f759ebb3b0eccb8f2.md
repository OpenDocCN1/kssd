
## 004201D1 函数分析

`push eax`

`push dword ptr [ebp+14]` ----> `LParam`

`push dword ptr [ebp+10]` ----> `WParam`

`push dword ptr [ebp+C]` ------> `Message`

`mov edx, esp` ----> 把当前堆栈地址 `esp` 传给 `edx` 好奇怪为什么这么做请看下一个函数

`mov eax, [ecx+4]` ----> 窗口对象指针 `[ecx+4]` 给 `eax`

`call [ecx]` ----> `TWinControl.MainWndProc(var Message: TMessage);`

`add esp, 0C` -----> `Object Pascal` 经典 `register` 调用约定

`pop eax` ----> 返回值放到 `eax`

`pop ebp`

`retn 10`

`Controls.pas.6234`: `TWinControl.MainWndProc(var Message: TMessage);`

```
TMessage = packed record
    Msg: Cardinal;
    case Integer of
        0: (
        WParam: Longint;
        LParam: Longint;
        Result: Longint);
        1: (
        WParamLo: Word;
        WParamHi: Word;
        LParamLo: Word;
        LParamHi: Word;
        ResultLo: Word;
        ResultHi: Word);
end;
```

这个函数中加了两个 `SEH` 异常处理句柄然后跳到各个窗体类重载的 `WndProc`

## 00458BCC 函数分析

`push ebp`

`mov ebp, esp`

`push ecx`

`push ebx`

`push esi`

`push edi`

`mov [ebp-4], eax` ----> `TfrmMain` 对象地址,这里跟踪的是 `TfrmMain` 父窗口类对象

`xor eax, eax`

`push ebp`

`push 00458C24` ------ > 新 `SEH` 处理句柄

`push dword ptr fs:[eax]`

`mov fs:[eax], esp` -----> `SEH` 构造完毕

`xor eax, eax`

`push ebp`

`push 00458C13` ----> 新 `SEH` 处理句柄

`push dword ptr fs:[eax]`

`mov fs:[eax], esp` ---> 新 `SEH` 构造完毕

`mov ebx, [ebp-4]` ---> `TfrmMain` 类对象地址给 `ebx`

`mov eax, [ebx+3C]` -> 又把 `TfrmMain` 类对象地址 `[epx+3C]` 传给 `eax`,下边与很多这样相同的约定.

`call [ebx+38]` ----> `TCustomForm.WndProc(var Message: TMessage)`

`xor eax, eax`

`pop edx`

`pop ecx`

`pop ecx`

`mov fs:[eax], edx` ------------ > 这就是上边函数传递过来的 `edx` 堆栈指针,原来是构造 `SEH`

`push 00458C1A`

`call 004534DC` ---> `Controls.pas.6239:FreeDeviceContexts`

`call 00426D88` ------> `Controls.pas.6240:FreeMemoryContexts`

`retn`

## `TCustomForm.WndProc` 函数分析

`TCustomForm.WndProc(var Message: TMessage)`

没有可处理的消息,把消息分派给父类 `TWinControl.WndProc(var Message: TMessage);`

`eax` 传对象指针,`edx` 传数据消息指针

`mov edx, ebx` -------> `Message` 参数

`mov eax, [ebp-4]` ---> `[ebp-4]` 把 `TfrmMain` 父窗口对象地址给 `eax` 往下传参数

`call 00458E24` --------> `TWinControl.WndProc(var Message: TMessage);`

`pop edi`

`pop esi`

`pop ebx`

`mov esp, ebp`

`pop ebp`

`retn`

`Controls.pas.6309`: `TWinControl.WndProc(var Message: TMessage);`

没有可处理的消息,把消息分派给父类 `TControl.WndProc(TControl;TMessage;TMessage);`

`eax` 传对象指针,`edx` 传数据消息指针

`mov edx, esi` ------> `Message`

`mov eax, ebx` --------> 把 `TfrmMain` 父窗口对象指针给 `eax` 往下传参数

`call 00455D94` ----> `TControl.WndProc(TControl;TMessage;TMessage);`

`add esp, 10`

`pop edi`

`pop esi`

`pop ebx`

`retn`

## `TControl.WndProc` 函数分析

`TControl.WndProc(TControl;TMessage;TMessage);`

同样也没有可处理的消息,把消息再分派给父类 `TObject.Dispatch(var Message);`

约定与前几个消息分派函数一样,都是 `eax` 传对象指针,`edx` 传数据消息指针

`mov edx, ebx` --------> 消息指针 `edx` 存放 `Message` 地址

`mov eax, esi` ----> 对象地址给 `eax`

`mov ecx, [eax]` ------> 虚拟表首地址给 `ecx`

`call [ecx-14]` -------> 虚函数:`TObject.Dispatch(var Message);`

`pop edi`

`pop esi`

`pop ebx`

`mov esp, ebp`

`pop ebp`

## `TObject.Dispatch` 函数分析

`TObject.Dispatch(var Message);`

`TObject.Dispatch` 负责所有的事件方法(在 `delphi IDE` 对象属性面版中的所有事件方法)的分派和动态方法的调用,它是对虚拟方法调用的一种内存资源优化.

`push esi`

`mov si, [edx]` -----> 从 `edx` 消息指针中取走消息给 `si`

`or si, si`

`je short 00403BDC` ----> `si` 如果小于 0 就跳

`cmp si, 0C000`

`jnb short 00403BDC` ----> `si` 如果大于 `C00` 就跳默认消息处理方式

`push eax` ----------------> 保存对象指针

`mov eax, [eax]` ----> 虚函数表给 `eax`

`call 00403B2C` ------> `GetDynaMethod` 索引动态函数指针表

`pop eax` ----------------> 对象指针放到 `eax`

`je short 00403BDC` ----> 如果相等标志为 1 就说明没有相关事件处理方法就跳否则就不跳.

`mov ecx, esi`

`pop esi`

`jmp ecx` ------------> 找到事件句柄,跳到消息处理事件方法

`pop esi`

`mov ecx, [eax]` ----> 对象虚拟表首地址给 `ecx`

`jmp [ecx-10]` -----------> `TControl.DefaultHandler(var Message);`

`retn`

以上消息分派模式是 `VCL` 固定不变,变的是继承对象的多态转换...以上分派模式不光是针对 `WM_COMMAND` 消息,而且还是 `WM_Paint` 等标准 `Window` 分派模式,同样也是控件与父窗体通信的消息分派模式例如 `Win` 标准控件消息(`WM_COMMAND`)和公共控件消息(`WM_NOTIFY`).

由于偶触发了单击 `Buttou` 事件而发出 `Click` 通知给父窗口 `TfrmMain` 类的父窗口,`TfrmMain` 类的父窗口响应 `WM_COMMAND` 消息,看看 `TfrmMain` 类父窗口是否重载处理了 `WM_COMMAND` 事件句柄,当然这也不一定的.之所以分析这么仔细不是做作,因为 `Dede` 分析的 `RCDATA` 节中没有看到任何的 `ButtonClick` 事件名,想必有可能做了某些处理.

继续分析...

## `TCustomForm.WMCommand` 函数分析

`TCustomForm.WMCommand(var Message: TWMCommand)`

注意这个动态事件函数是从父类 `TObject.Dispatch(var Message)` 分派来的,与前面的消息分派函数一样都是 `eax` 传对象指针,`edx` 传数据消息指针

```
TWMCommand = packed record
    Msg: Cardinal;
    ItemID: Word;
    NotifyCode: Word;
    Ctl: HWND;
    Result: Longint;
end;
```

以次检测窗口句柄,和菜单对象

`push ebx`

`push esi`

`mov esi, edx` ----> 消息 `Message` 对象地址给 `esi`.

`mov ebx, eax` --------> `TfrmMain` 类对象指针由 `eax` 传给 `ebx`

`cmp dword ptr [esi+8], 0` ---> `Message.Ctl`

`jnz short 00470BEC` ------> 对象窗口句柄为 0 就跳

`cmp dword ptr [ebx+248], 0` ---> `[ebx+248]` 定位窗口 `TMenu` 对象

`je short 00470BEC` ------> `TMenu` 为空,跳.

`mov dx, [esi+4]` ---> `Message.ItemID` 菜单 ID 号给 `dx`

`mov eax, [ebx+248]` ---> 菜单 `TMenu` 类对象 `[ebx+248]` 传给 `eax`

`call 00468920` ----> `Menu.DispatchCommand(ItemID)`,菜单处理此消息.

`test al, al` ---> 返回 `boolean` 变量给 `al`,`al=1` 成功,`al=0` 失败

`jnz short 00470BF5` ------> 判断 `al` 跳

`mov edx, esi` ----> 消息 `TMessage` 对象给 `edx`
