
### 代码实现

```pascal
Procedure TForm1.btn3Click(Sender: TObject);

Var

  Name, Email, EStr, SerialNo: String;

  i, Sum: Integer;

Begin

//信息不完整

  Name := edt1.Text;

  Email := edt2.Text;

  If (Length(edt1.Text) = 0) Or (Length(edt2.Text) = 0) Then Exit;

  Sum := 0;

  For i := 1 To Length(Email) Do EStr := EStr + IntToStr(Ord(Email[i]));

  For i := 1 To Length(EStr) Div 9 Do Sum := Sum Xor StrToIntDef(Copy(EStr, (i - 1) * 9 + 1, 9), 0);

  Sum := Sum Xor StrToIntDef(RightStr(EStr, Length(EStr) Mod 9), 0);

  EStr := IntToStr(Sum Xor StrToIntDef(RightStr(Name, Length(Name) - 4), 0));

  For i := 1 To 9 Do

  Begin

    If Length(EStr) >= i Then

    Begin

      If i Mod 2 = 1 Then

        SerialNo := SerialNo + Char(StrToIntDef(EStr[i], 0) + $41)

      Else

        SerialNo := SerialNo + Char($5A - StrToIntDef(EStr[i], 0));

    End

    Else

    Begin

      If i Mod 2 = 1 Then

        SerialNo := SerialNo + 'A'

      Else

        SerialNo := SerialNo + 'Z';

    End;

    If i = 4 Then SerialNo := SerialNo + '-';

  End;

  edt3.Text := SerialNo;

End;
```

用户名：`baby2008`

电子邮箱：`jw6y8@21cn.com`

注册码：`IWBU-AZFZB`

注册信息保存在：`HKEY_CURRENT_USER\Software\AYCD\Option`，删除可重新注册。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
