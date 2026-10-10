

# 热血江湖 V40004 版喊话 CALL(Delphi2010 源码)

> 原文：[`www.pediy.com/kssd/pediy11/106522.html`](https://www.pediy.com/kssd/pediy11/106522.html)

热血江湖最新 V40004 版的喊话 CALL,找了好长时间也无法找到,只找到了逐字符发送的 CALL,由于我是在 DELPHI2010 版下写的代码,所以字符串做了些转换,2010 版以下就不用转换直接用了.有时间我会慢慢找其他 CALL,有兴趣请到 www.tansoo.cn 查看  
  
procedure TfrmMain.Button1Click(Sender: TObject);      
var     
  neirong: PAnsiChar;      
  I: Integer;      
     
  // 发送字符的 CALL      
  procedure SendByte(b:Byte);      
  var     
    Address: Pointer;      
  begin     
    Address := Pointer($0043C380);      
    asm     
      pushad      
      xor   eax, eax      
      mov   al, \[b\]      
      mov   edi, $00020001     
      mov   esi, dword ptr \[$1700910\]      
      mov   ecx, esi      
      push  edi      
      push  eax      
      call  Address      
      popad      
    end;      
  end;      
begin     
  // 将 Unicode 字符串转换成 PAnsiChar 型,并在最后加个回车符      
  neirong := PAnsiChar(AnsiString(Edit3.Text)+#$0D);      
  // 逐一发送字符      
  for I := 0 to Length(neirong)-1 do     
    SendByte(Byte(neirong\[i\]));      
end;