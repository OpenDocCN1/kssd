
0050158A   |.  E8 5535F0FF           call Unpacked.00404AE4           ;  求长度

0050158F   |.  8BC8                  mov ecx,eax

00501591   |.  2B4E 60               sub ecx,dword ptr ds:[esi+60]    ;  长度-12(要删除的字符数)

00501594   |.  8B56 60               mov edx,dword ptr ds:[esi+60]    ;  12

00501597   |.  42                    inc edx                          ;  (起始位置12+1=13)

00501598   |.  8BC7                  mov eax,edi

0050159A   |.  E8 DD37F0FF           call Unpacked.00404D7C           ;  System.@LStrDelete;删第除12位后的字符

0050159F   |>  33C0                  xor eax,eax

005015A1   |.  5A                    pop edx

005015A2   |.  59                    pop ecx

005015A3   |.  59                    pop ecx

005015A4   |.  64:8910               mov dword ptr fs:[eax],edx

005015A7   |.  68 CC155000           push Unpacked.005015CC

005015AC   |>  8D45 E4               lea eax,[local.7]

005015AF   |.  E8 7832F0FF           call Unpacked.0040482C

005015B4   |.  8D45 F4               lea eax,[local.3]

005015B7   |.  E8 7032F0FF           call Unpacked.0040482C

005015BC   |.  8D45 FC               lea eax,[local.1]

005015BF   |.  E8 6832F0FF           call Unpacked.0040482C

005015C4   \.  C3                    retn

005015C5    .^ E9 4A2CF0FF           jmp Unpacked.00404214

005015CA    .^ EB E0                 jmp short Unpacked.005015AC

005015CC    .  5F                    pop edi

005015CD    .  5E                    pop esi

005015CE    .  5B                    pop ebx

005015CF    .  8BE5                  mov esp,ebp

005015D1    .  5D                    pop ebp

005015D2    .  C3                    retn

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－

### 【算法总结】

算法比较简单，怪不得很多教程都以它的前期版本为教程，注册算法是：

1、用户名`Name`要求:  3<用户名长度<25

2、`Name`后面开始取字符，

3、用常数`$33FAB9386 / Name[i]`的ASCII 取余数，并转换成10进制字符串

4、连接上述的10进制字符串，记为`Serail`

5、取`Name[i]`字符到<用户名长度-6结束

6、将`Serail`转换成`int64`,并以12位16进制字符串输出,如果长度超过12位，截取12后的所有字符即为注册码。

贴出注册机`delphi`源码：

```Procedure TForm1.btn1Click(Sender: TObject);

Var

  Name, SerailNo: String;

  i: Integer;

Begin

  Name := edt1.Text;

  If (Length(Name) > 25) Or (Length(Name) < 3) Then

  Begin

    edt2.Clear;

    Exit;

  End;

  For i := Length(Name) Downto 1 Do

  Begin

    If i < Length(Name) - 6 Then

      Break

    Else

      SerailNo := SerailNo + FormatFloat('0', $33FAB9386 Mod Ord(Name[i]));

  End;

  edt2.Text := LeftStr(IntToHex(StrToInt64(SerailNo), 12), 12);

End;
```

大家可以验证一下：

用户名：`Baby2008`

注册码：`3176382C48A6`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
