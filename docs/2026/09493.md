
# Crack之亲历手迹3--Delphi程序中是如何获得一个Edit的Text内容的 (1千字)

Crack之亲历手迹3--Delphi程序中是如何获得一个Edit的Text内容的

----------Crack之亲历手迹2--我的第一个注册机----之续集

实验工具：`IDA Pro`、`Delphi 6`、`SoftICE4.05`

实验目的：发现Delphi编的程序如何获得文本框（Edit）的数据(`Text`)而不用API函数

## 1. 用Delphi编个简单的试验程序

一个Form包含一个Edit、一个Button、代码如下

```pascal
procedure TForm1.Button1Click(Sender: TObject);
begin
  ShowMessage(Edit1.Text);
end;
```

而`Edit.text`好像是调用下面的类似函数（从IDA的反汇编可以看出，不过那叫`TControl::GetText`）

摘录自Delphi6的`source\vcl\controls.pas`

```pascal
function TControl.GetTextBuf(Buffer: PChar; BufSize: Integer): Integer;
begin
  Result := Perform(WM_GETTEXT, BufSize, Longint(Buffer));
end;
```

`Perform(Msg: Cardinal; WParam, LParam: Longint): Longint;`

Responds as if the control received a specified Windows message.

从中我们可以看出Delphi时通过`Perform`函数直接模拟向Edit发送`WM_GETTEXT`消息来获得Text的内容的！

注意这个`WM_GETTEXT`消息发送是模拟的，你用BPX `MSGFUN`时栏不到的，其实是通过Call `WndProc`的方式！

## 2. 用IDA反汇编1中的exe同样可见到

```asm
call    @TControl@GetText
```

```asm
@TControl@GetText proc near
  ...
  call    unknown_libname_169
  ...
```

```asm
unknown_libname_169 proc near
  ...
  call    @Controls@TControl@Perform$qqruiii ; Controls::TControl::Perform(uint,int,int)
  ...
```

## 3. 结论

Delphi通过向Edit发送`WM_GETTEXT`（直接调用`WNDProc`,而没有使用消息函数）消息来获得Text的内容的！

```pascal
GetText(){
  Call WndProc(WM_GETTEXT);
}
```

```
WndProc(Msg){
  DefWndProc(Msg);
}
```

`Text`的自值由`DefWndProc`系统函数给出！

附：看来Borload的那些家伙写出的东西的确与众不同

Email:`y97523@hotmail.com`
