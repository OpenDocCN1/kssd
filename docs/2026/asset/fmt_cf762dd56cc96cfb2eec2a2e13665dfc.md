
```
  function EncryStrHex(Str, Key: String): String;
  var
  StrResult, TempResult, Temp: String;
  I: Integer;
  begin
  TempResult := EncryStr(Str, Key);
  StrResult := '';
  for I := 0 to Length(TempResult) - 1 do
  begin
  Temp := Format('%x', [Ord(TempResult[I + 1])]);
  if Length(Temp) = 1 then Temp := '0' + Temp;
  StrResult := StrResult + Temp;
  end;
  Result := StrResult;
  end;

  function DecryStrHex(StrHex, Key: String): String;
  function HexToInt(Hex: String): Integer;
  var
  I, Res: Integer;
  ch: Char;
  begin
  Res := 0;
  for I := 0 to Length(Hex) - 1 do
  begin
  ch := Hex[I + 1];
  if (ch >= '0') and (ch <= '9') then
  Res := Res \* 16 + Ord(ch) - Ord('0')
  else if (ch >= 'A') and (ch <= 'F') then
  Res := Res \* 16 + Ord(ch) - Ord('A') + 10
  else if (ch >= 'a') and (ch <= 'f') then
  Res := Res \* 16 + Ord(ch) - Ord('a') + 10
  else raise Exception.Create('Error: not a Hex String');
  end;
  Result := Res;
  end;
  var
  Str, Temp: String;
  I: Integer;
  begin
  Str := '';
  for I := 0 to Length(StrHex) div 2 - 1 do
  begin
  Temp := Copy(StrHex, I \* 2 + 1, 2);
  Str := Str + Chr(HexToInt(Temp));
  end;
  Result := DecryStr(Str, Key);
  end;
  end.
```

---

主程序部分：

```
  uses
  des;
  procedure TForm1.Button1Click(Sender: TObject);
   var
   diskid,nameok,regsn:string;
  begin
   diskid:=copy(edit1.Text,1,8);
   nameok:=copy(edit2.Text,1,8);
   regsn:=des.EncryStrHex(diskid,'WAVE990');
   regsn:=copy(regsn,1,9);
   regsn:=des.EncryStrHex(nameok,regsn);
   regsn:=copy(regsn,1,9);
   regsn:=des.EncryStrHex(regsn,'WAVE990');
   edit3.Text:=regsn;
  end;
```

---

【版权声明】: 转载请注明作者并保持文章的完整, 谢谢!

2007年05月29日 17:39:57

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
