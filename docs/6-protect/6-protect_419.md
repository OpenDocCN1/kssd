

# 检测本进程有无硬件断点的程序

> 原文：[`www.pediy.com/kssd/pediy12/134757.html`](https://www.pediy.com/kssd/pediy12/134757.html)

代码:

```
var
  TestVar: Integer;

function NtGetContextThread(ThreadHandle: THANDLE; Context: PCONTEXT): Integer; stdcall; external 'ntdll' name 'NtGetContextThread';

function HBExists: Boolean;
var
  C: TContext;
begin
  C.ContextFlags:=Windows.CONTEXT\_DEBUG\_REGISTERS;
  NtGetContextThread(GetCurrentThread, @C);
  Result:=(C.Dr0<>0) or (C.Dr1<>0) or (C.Dr2<>0) or (C.Dr3<>0) or (C.Dr6<>0) or (C.Dr7<>0);
end;

procedure TXForm.BtTestVarClick(Sender: TObject);
begin
  Inc(TestVar);
end;

procedure TXForm.TimerTimer(Sender: TObject);
begin
  MmShow.Text:='TestVar 地址: '+IntToHex(Cardinal(@TestVar), 8)+#13#10+'TestVar 数值: '+IntToStr(TestVar);
  if HBExists then
    EtState.Text:='硬件断点 有！'
  else
    EtState.Text:='硬件断点 无...'
end;

```