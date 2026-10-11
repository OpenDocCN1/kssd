
## TWinControl.WMCommand 消息分派

`00470BEE` `. 8BC3` `mov eax, ebx` ------> TfrmMain 对象地址给 eax

`00470BF0` `. E8 0B8BFEFF` `call 00459700` ----------------> TWinControl.WMCommand(var Message: TWMCommand);

`00470BF5` `> 5E` `pop esi`

`00470BF6` `. 5B` `pop ebx`

`00470BF7` `. C3` `retn`

## TWinControl.WMCommand 内部逻辑

`TWinControl.WMCommand(var Message: TWMCommand);`

`00459700` `/$ 53` `push ebx`

`00459701` `|. 56` `push esi`

`00459702` `|. 8BDA` `mov ebx, edx`

`00459704` `|. 8BF0` `mov esi, eax`

`00459706` `|. 8BD3` `mov edx, ebx` ----> 消息 Message 地址 ebx 传给 edx

`00459708` `|. 8B43 08` `mov eax, [ebx+8]` ---> [ebx+8] TMessage 的句柄 Ctl 字段，给 eax 传参数约定

`0045970B` `|. E8 80F9FFFF` `call 00459090` -----> DoControlMsg(ControlHandle: HWnd; var Message): Boolean;

`00459710` `|. 84C0` `test al, al` --------> 返回 boolean 变量给 al，al=1 成功，al=0 失败

`00459712` `|. 75 09` `jnz short 0045971D` ---> 判断 al 跳

`00459714` `|. 8BD3` `mov edx, ebx` ---> 传 Message 给 edx

`00459716` `|. 8BC6` `mov eax, esi` ---> 传 TfrmMain 对象给 eax

`00459718` `|. 8B08` `mov ecx, [eax]` ----> TfrmMain 对象地址给 ecx

`0045971A` `|. FF51 F0` `call [ecx-10]` ------> 还记得它！是哪个虚函数么？在 TObject.Dispatch(var Message) 自己找！

`0045971D` `|> 5E` `pop esi`

`0045971E` `|. 5B` `pop ebx`

`0045971F` `\. C3` `retn`

从上边给出的 VCL `WM_Command` 消息分派路线看出，`TfrmMain` 父窗体没有处理这个消息，只是把它继续传递给下一个消息通道函数。

## DoControlMsg 实现

`DoControlMsg(ControlHandle: HWnd; var Message): Boolean;`

`00459090` `/$ 53` `push ebx`

`00459091` `|. 56` `push esi`

`00459092` `|. 57` `push edi`

`00459093` `|. 8BF2` `mov esi, edx` ----> edx=Message 是传进来的参数把消息又给了 esi

`00459095` `|. 33DB` `xor ebx, ebx` ---> 注意 FindControl 函数的参数是由 eax 传进来的，eax 在上一个函数中已经传进来了

`00459097` `|. E8 A488FFFF` `call 00451940` --------> FindControl(Handle: HWnd): TWinControl;

`0045909C` `|. 8BF8` `mov edi, eax` ----> 返回 TWinControl 类对象在 eax 中，传给 edi。注意我们这里找到这个控件是预知的 Tbutton 类

`0045909E` `|. 85FF` `test edi, edi` ----> 判断 TWinControl 类对象是否为空

`004590A0` `|. 74 1B` `je short 004590BD` 空就跳

`004590A2` `|. 8B46 08` `mov eax, [esi+8]` ---> [esi] 是 TWMCommand 对象 Ctl 域

`004590A5` `|. 50` `push eax`

`004590A6` `|. 8B4E 04` `mov ecx, [esi+4]` ---> [esi] 是 TWMCommand 对象中 ItemID 和 NotifyCode 域

`004590A9` `|. 8B16` `mov edx, [esi]` ---> [esi] 是 TWMCommand 对象中 Msg 域

`004590AB` `|. 81C2 00BC0000` `add edx, 0BC00` ---> Window Notify Message 加上 BC00 就是 VCL FrameWork 内部约定的控件通知消息

`004590B1` `|. 8BC7` `mov eax, edi` ---> 把 Tbutton 类对象地址给 eax，现在开始向 Button 控件发送通知代码了，父 TfrmMain 窗体没有处理 Click 事件通知句柄。

`004590B3` `|. E8 10CCFFFF` `call 00455CC8` ------> TControl.Perform(Msg + CN_BASE, WParam, LParam); CN_BASE=BC00

`004590B8` `|. 8946 0C` `mov [esi+C], eax` ---> [esi+C] 是 TWMCommand 对象的 Result 域

`004590BB` `|. B3 01` `mov bl, 1` ---> DoControlMsg 函数返回成功传 bl=1

`004590BD` `|> 8BC3` `mov eax, ebx` ---> 清零

`004590BF` `|. 5F` `pop edi`

`004590C0` `|. 5E` `pop esi`

`004590C1` `|. 5B` `pop ebx`

`004590C2` `\. C3` `retn`

## TControl.Perform 实现

`TControl.Perform(Msg + CN_BASE, WParam, LParam): Longint;`

`00455CC8` `/$ 55` `push ebp`

`00455CC9` `|. 8BEC` `mov ebp, esp`

`00455CCB` `|. 83C4 F0` `add esp, -10`

`00455CCE` `|. 53` `push ebx`

`00455CCF` `|. 8955 F0` `mov [ebp-10], edx` ----> edx=Msg+CN_BASE，edx 是上一函数传进来

`00455CD2` `|. 894D F4` `mov [ebp-C], ecx` ----> Wparam 同上

`00455CD5` `|. 8B55 08` `mov edx, [ebp+8]` ----------> [ebp+8] 也是从那个函数中传进来的，是 TWMCommand 对象 Ctl 域

`00455CD8` `|. 8955 F8` `mov [ebp-8], edx` ----> TWMCommand 对象 Ctl 域给 [ebp-8]

`00455CDB` `|. 33D2` `xor edx, edx`

`00455CDD` `|. 8955 FC` `mov [ebp-4], edx` ----> Message.Result=0

`00455CE0` `|. 85C0` `test eax, eax` -------------> eax 也是上一个函数传进来的 Tbutton 对象指针

`00455CE2` `|. 74 0B` `je short 00455CEF` 空就跳

`00455CE4` `|. 8D55 F0` `lea edx, [ebp-10]` ----> 由上边构造好的消息类 TMessage 地址给 edx 指针，再传给下边的函数

`00455CE7` `|. 8BD8` `mov ebx, eax` ---> Tbutton 对象指针 eax，传给 ebx

`00455CE9` `|. 8B43 3C` `mov eax, [ebx+3C]` ---> [ebx+3c] 与 eax 同指向一个 Tbutton 对象

`00455CEC` `|. FF53 38` `call [ebx+38]` --> 这个虚函数熟悉不？它在 TWinControl.MainWndProc 中出现过。这里出现了多态了应该调用 TButton.WndProc(var Message: TMessage)

`00455CEF` `|> 8B45 FC` `mov eax, [ebp-4]` ---> 返回值又送到 eax

`00455CF2` `|. 5B` `pop ebx`

`00455CF3` `|. 8BE5` `mov esp, ebp`

`00455CF5` `|. 5D` `pop ebp`

`00455CF6` `\. C2 0400` `retn 4`

到这里应该调用 `TButtonControl.WndProc(var Message: TMessage)` 控件通知分派路线了，检查是否 TButton 类有处理 Click 通知事件处理器！上边已经说了 VCL 消息模式分派是固定的模式所以不用分析了。

它应该与上边 `TCustomForm.WndProc(var Message: TMessage)` 分派路线是相同的，但是终点是不同的。这正是多态对象改变了终点，一个 `TfrmMain` 对象，一个 `TButton` 类。

简单给出与上边相同的分派路线：

## 消息分派路径

`TButtonControl.WndProc(var Message: TMessage)`

`0044DD34` `> \ 8BD7` `mov edx, edi` ----> Message 参数传递给 edx

`0044DD36` `. 8BC3` `mov eax, ebx` --------> TButton 对象指针给 eax

`0044DD38` `. E8 E7B00000` `call 00458E24` ---> TWinControl.WndProc(var Message: TMessage);

`0044DD3D` `> 5F` `pop edi`

`0044DD3E` `. 5E` `pop esi`

`0044DD3F` `. 5B` `pop ebx`

直接给出来，与上边调用地址都相同。

`TWinControl.WndProc(var Message: TMessage)` --> `TControl.WndProc(TControl;TMessage;TMessage)`

--> `TObject.Dispatch(var Message)` ------> 终点到这里分派路线转了，开始发送到 TButton 事件处理器中。还记得我们上边处理 VCL 通知 Click 代码是多少么：(WM_COMMAND+CN_BASE)

`TButton.CNCommand(var Message: TWMCommand);`

`0044DF5C` `. 56` `push esi`

`0044DF5D` `. 66:837A 06 00` `cmp word ptr [edx+6], 0` ---> [edx+6] 存放 Message 的 NotifyCode，0 数值代表 BN_CLICKED 宏。

`0044DF62` `. 75 09` `jnz short 0044DF6D` ---> 不等就跳

`0044DF64` `. 66:BE EBFF` `mov si, 0FFEB` ----> 注意传给 si 的数值应该是 EB 是个关键的动态方法索引值

`0044DF68` `. E8 EF5BFBFF` `call 00403B5C` -> procedure _CallDynaInst; 这个函数根据上边 si 传进来的索引值寻找动态方法它内部调用实现与 TObject.Dispatch(var Message) 相同。

`0044DF6D` `> 5E` `pop esi`

`0044DF6E` `. C3` `retn`

从上边进入 procedure `_CallDynaInst` 而它内部又调用 `GetDynaMethod(vmt: TClass; selector: Smallint) : Pointer`，这一点与 `TObject.Dispatch(var Message)` 内部雷同，

它们共同约定的寄存器：`eax` 是对象地址，`si` 是动态方法索引值，`[eax+vmtDynamicTable]` 动态方法表，这比 C++ 等其他面向对象多了一个动态函数表格，最终找到句柄然后 jmp 进去。

我们这里最终跳到了 `TButton.Click`，由此看出 `EB` 索引的是 `TButton.Click`，当然 VCL 内部动态方法索引值还有很多，仅供参考用，或许有变动。

`TCustomEdit.CNCommand(var Message: TWMCommand);` `B3`

`BeginAutoDrag;ed`

来到 TButton.Click 动态方法中...

## TButton.Click 实现

`0044DE58` `. 53` `push ebx`

`0044DE59` `. 8BD8` `mov ebx, eax` --------> TButton 对象地址由 eax 传给 ebx

`0044DE5B` `. 8BC3` `mov eax, ebx` ----> eax 函数参数 Self

`0044DE5D` `. E8 02E60100` `call 0046C464` ----> GetParentForm(Self)，查找 TButton 父类窗体。

`0044DE62` `. 85C0` `test eax, eax` ---> 空就跳

`0044DE64` `. 74 0C` `je short 0044DE72`

`0044DE66` `. 8B93 14020000` `mov edx, [ebx+214]` -----> [ebx+214] 是 TButton 对象的 ModalResult 字段结果，给 edx 保存

`0044DE6C` `. 8990 4C020000` `mov [eax+24C], edx` -> [eax+24C] 是 TButton 父窗体 TfrmMain 的 ModalResult，它的数值由上边传进来。这是处理模式对话框的时候，当 Button_Click 事件返回的时候是不是要把控件 ID 给父窗体，以便交互约定调用

`0044DE72` `> 8BC3` `mov eax, ebx` ----> TButton 对象地址给 eax，开始调用 TBuuton 的继承父类方法 TControl.Click

`0044DE74` `. E8 D7810000` `call 00456050` ----> TControl.Click

`0044DE79` `. 5B` `pop ebx`

`0044DE7A` `. C3` `retn`

## TControl.Click

关键的 `TControl.Click`

到这里真相大白了...

## Delphi Form 资源定义示例

```
object Button2: TButton
  Left = 288
  Top = 8
  Width = 75
  Height = 25
  Caption = 'demo'
  TabOrder = 5
```
