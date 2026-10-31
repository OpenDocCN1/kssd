
# [原创]热血江湖V40004版喊话`CALL`(Delphi2010源码)

热血江湖最新V40004版的喊话`CALL`，找了好长时间也无法找到，只找到了逐字符发送的`CALL`。由于我是在DELPHI2010版下写的代码，所以字符串做了些转换，2010版以下就不用转换直接用了。有时间我会慢慢找其他`CALL`，有兴趣请到`58bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4c8S2L8Y4y4G2L8#2)9J5k6h3y4F1i4@1f1$3i4K6W2r3i4@1p5#2i4@1f1%4i4K6W2o6i4K6S2n7`。

```delphi
procedure TfrmMain.Button1Click(Sender: TObject);

var
  neirong: PAnsiChar;
  I: Integer;

  // 发送字符的CALL
  procedure SendByte(b: Byte);
  var
    Address: Pointer;
  begin
    Address := Pointer($0043C380);
    asm
      pushad
      xor   eax, eax
      mov   al, 
      mov   edi, $00020001
      mov   esi, dword ptr [$1700910]
      mov   ecx, esi
      push  edi
      push  eax
      call  Address
      popad
    end;
  end;

begin
  // 将Unicode字符串转换成PAnsiChar型,并在最后加个回车符
  neirong := PAnsiChar(AnsiString(Edit3.Text)+#$0D);
  // 逐一发送字符
  for I := 0 to Length(neirong)-1 do
    SendByte(Byte(neirong[i]));
end;
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
