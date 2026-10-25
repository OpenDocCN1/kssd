
# 断点失效的简单分析

> 原文：[`www.pediy.com/kssd/pediy04/kanxue311.htm`](https://www.pediy.com/kssd/pediy04/kanxue311.htm)

`VCL` 中标准的单行编辑控件 `TEdit` 和多行编辑控件 `TMemo` 在类树中的派生关系如下：

`TObject` -> `TPersistent` -> `TComponent` -> `TControl` -> `TWinControl` -> `TCustomEdit` -> `TEdit`

`TObject` -> `TPersistent` -> `TComponent` -> `TControl` -> `TWinControl` -> `TCustomEdit` -> `TCustomMemo` -> `TMemo`

以 `TEdit` 为例，来看一下它是如何获取控件中的字符串的，把这个过程（也就是 `VCL` 内部处理消息的过程）搞清楚了，就能够知道该设什么样的断点来监视这个获得字符串的过程，从而找到我们输入的注册码在内存中的存放位置，进而可以通过 `BPM`/`BPR` 断点来跟踪对注册码的处理过程。

`TEdit` 用来存放字符串的属性是 `Text`，该属性继承自 `TControl`。读取该属性的函数是私有函数 `GetText()`。

（以下代码来自 `VCL` 的源码 `stdctrls.pas`、`control.pas`、`system.pas`、`sysutils.pas`）

```pascal
//----------------------------------------------------------------------------------------------------
TWndMethod = procedure(var Message: TMessage) of object;

TControl = class(TComponent)
    private
        FWindowProc: TWndMethod;
        FText:       PChar;
        function     GetText: TCaption;
        procedure    SetText(const Value: TCaption);
        ......
    protected
        property  Text: TCaption read GetText write SetText;
        procedure WndProc(var Message: TMessage); virtual;
        ......
    public
        function  GetTextBuf(Buffer: PChar; BufSize: Integer): Integer;
        function  GetTextLen: Integer;
        procedure SetTextBuf(Buffer: PChar);
        function  Perform(Msg: Cardinal; WParam, LParam: Longint): Longint;
        property  WindowProc: TWndMethod read FWindowProc write FWindowProc;
        ......
end;
//----------------------------------------------------------------------------------------------------
```

`GetText()` 函数及相关函数的定义如下。可见，`TControl` 最终是以 `WM_GETTEXTLENGTH` 和 `WM_GETTEXT` 消息为参数直接调 `WndProc()` 的。

```pascal
//----------------------------------------------------------------------------------------------------
function TControl.GetText: TCaption;
var
  Len: Integer;
begin
  Len := GetTextLen;
  SetString(Result, PChar(nil), Len);
  if Len <> 0 then GetTextBuf(Pointer(Result), Len + 1);
end;

function TControl.GetTextLen: Integer;
begin
  Result := Perform(WM_GETTEXTLENGTH, 0, 0);
end;

function TControl.GetTextBuf(Buffer: PChar; BufSize: Integer): Integer;
begin
  Result := Perform(WM_GETTEXT, BufSize, Longint(Buffer));
end;

procedure TControl.SetTextBuf(Buffer: PChar);
begin
  Perform(WM_SETTEXT, 0, Longint(Buffer));
  Perform(CM_TEXTCHANGED, 0, 0);
end;

procedure TControl.SetText(const Value: TCaption);
begin
  if GetText <> Value then SetTextBuf(PChar(Value));
end;

function TControl.Perform(Msg: Cardinal; WParam, LParam: Longint): Longint;
var
  Message: TMessage;
begin
  Message.Msg := Msg;
  Message.WParam := WParam;
  Message.LParam := LParam;
  Message.Result := 0;
  if Self <> nil then WindowProc(Message);
  Result := Message.Result;
end;

constructor TControl.Create(AOwner: TComponent);
begin
  inherited Create(AOwner);
  FWindowProc := WndProc;
  ......
end;
//----------------------------------------------------------------------------------------------------
```

`WndProc()` 是个 `virtual` 函数。`TEdit` 和 `TCustomEdit` 未重载这个函数，而 `TWinControl` 重载了这个函数，但它并未处理 `WM_GETTEXT` 消息，而是直接把该消息传递给了父类的函数 `TControl.WndProc()`。

```pascal
//----------------------------------------------------------------------------------------------------
procedure TWinControl.WndProc(var Message: TMessage);
var
  Form: TCustomForm;
  KeyState: TKeyboardState;
  WheelMsg: TCMMouseWheel;
begin
  ......
  inherited WndProc(Message);
end;
//----------------------------------------------------------------------------------------------------
```

`TControl.WndProc()` 直接用 `Dispatch()` 来分发该消息。

```pascal
//----------------------------------------------------------------------------------------------------
procedure TControl.WndProc(var Message: TMessage);
var
  Form: TCustomForm;
begin
  ......
  Dispatch(Message);
end;
//----------------------------------------------------------------------------------------------------------
```

由于 `WM_GETTEXT` 在 `TEdit` 中没有对应的专门的处理函数，所以 `Dispatch()` 最终会调用 `TEdit.DefaultHandler()`，这个方法是从 `TCustomEdit` 继承来的，如此层层向上往父类分发，直到 `TControl.DefaultHandler()` 对 `WM_GETTEXT` 消息进行了处理，它只是简单地把 `TControl` 的私有变量 `FText` 拷贝到软件提供的缓冲区中，所用到的函数只有 `StrLen()`、`StrLCopy()`。

```pascal
//----------------------------------------------------------------------------------------------------------
procedure TControl.DefaultHandler(var Message);
var
  P: PChar;
begin
  with TMessage(Message) do
    case Msg of
      WM_GETTEXT:
        begin
          if FText <> nil then P := FText else P := '';
          Result := StrLen(StrLCopy(PChar(LParam), P, WParam - 1));
        end;
      WM_GETTEXTLENGTH:
        if FText = nil then Result := 0 else Result := StrLen(FText);
      WM_SETTEXT:
        begin
          P := StrNew(PChar(LParam));
          StrDispose(FText);
          FText := P;
          SendDockNotification(Msg, WParam, LParam);
        end;
    end;
end;
//----------------------------------------------------------------------------------------------------------
```

看一下 `sysutils.pas` 中的 `StrLen()`、`StrLCopy()` 函数的实现，是用汇编写的，没有调用任何 API 函数。这就说明从进入 `TControl.GetText()` 开始，一直到 `WM_GETTEXT` 成功返回为止，没有调用过任何 Win32 API 函数。所以常用的 `GetDlgItemTextA`、`GetWindowTextA` 断点不生效是当然的。

```asm
//----------------------------------------------------------------------------------------------------------
function StrLen(const Str: PChar): Cardinal; assembler;
asm
        MOV    EDX,EDI
        MOV    EDI,EAX
        MOV    ECX,0FFFFFFFFH
        XOR    AL,AL
        REPNE  SCASB
        MOV    EAX,0FFFFFFFEH
        SUB    EAX,ECX
        MOV    EDI,EDX
end;

function StrLCopy(Dest: PChar; const Source: PChar; MaxLen: Cardinal): PChar; assembler;
asm
        PUSH    EDI
        PUSH    ESI
        PUSH    EBX
        MOV    ESI,EAX
        MOV    EDI,EDX
        MOV    EBX,ECX
        XOR    AL,AL
        TEST    ECX,ECX
        JZ      @@1
        REPNE  SCASB
        JNE    @@1
        INC    ECX
@@1:    SUB    EBX,ECX
        MOV    EDI,ESI
        MOV    ESI,EDX
        MOV    EDX,EDI
        MOV    ECX,EBX
        SHR    ECX,2
        REP    MOVSD
        MOV    ECX,EBX
        AND    ECX,3
        REP    MOVSB
        STOSB
        MOV    EAX,EDX
        POP    EBX
        POP    ESI
        POP    EDI
end;
//-------------------------------------------------------------------------------------------------------
```

那么如何才能在 `VCL` 将用户输入的字符串拷贝到软件的缓冲区中时使 `SoftICE` 弹出来呢？一种办法是先在软件的内存中搜索 `StrLCopy()` 这个函数的机器码（即其 Signature），找到之后在其入口处设个断点即可。实际上，你也可以在 `TControl.DefaultHandler()`、`TControl.GetText()` 等函数的入口处设断点，只要 `VCL` 在获取 `TEdit.Text` 的过程中调用了该函数，当然调用的次数越少越好。

另外一种方法就是跟踪 `FText` 何时被赋值，实际上就在上面的 `WM_SETTEXT` 处。

`blowfish` `4dd466d0e1e0cc123afbe504da2462ca`
