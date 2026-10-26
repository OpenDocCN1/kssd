
```assembly
00494534   |>  FF75 EC                 push dword ptr ss:[ebp-14]           ; /Arg2
00494537   |.  FF75 E8                 push dword ptr ss:[ebp-18]           ; |Arg1
0049453A   |.  8BD7                    mov edx,edi                          ; |
0049453C   |.  8BC3                    mov eax,ebx                          ; |
0049453E   |.  E8 A554F7FF             call TimeHelp.004099E8               ; \TimeHelp.004099E8
00494543   |.  8B07                    mov eax,dword ptr ds:[edi]
00494545   |.  E8 7209F7FF             call TimeHelp.00404EBC
0049454A   |.  8BC8                    mov ecx,eax
0049454C   |.  2B4E 70                 sub ecx,dword ptr ds:[esi+70]
0049454F   |.  8B56 70                 mov edx,dword ptr ds:[esi+70]
00494552   |.  42                      inc edx
00494553   |.  8BC7                    mov eax,edi
00494555   |.  E8 FA0BF7FF             call TimeHelp.00405154
0049455A   |>  33C0                    xor eax,eax
0049455C   |.  5A                      pop edx
0049455D   |.  59                      pop ecx
0049455E   |.  59                      pop ecx
0049455F   |.  64:8910                 mov dword ptr fs:[eax],edx
00494562   |.  68 87454900             push TimeHelp.00494587
00494567   |>  8D45 CC                 lea eax,dword ptr ss:[ebp-34]
0049456A   |.  E8 9506F7FF             call TimeHelp.00404C04
0049456F   |.  8D45 F4                 lea eax,dword ptr ss:[ebp-C]
00494572   |.  E8 8D06F7FF             call TimeHelp.00404C04
00494577   |.  8D45 FC                 lea eax,dword ptr ss:[ebp-4]
0049457A   |.  E8 8506F7FF             call TimeHelp.00404C04
0049457F   \.  C3                      retn
```

---

## Crack总结

作者的算法很简单，我不知到里面的浮点运算是干什么用的。

1. 将机器码转为16进制字符串，记为`M1`。
2. 在`M1`前加`'aB'`，后加`'Cd'`，即`'aB'＋M1＋'Cd'`，记为`M2`。
3. 循环，从`M2`后面开始取字符`Char`。
4. 将常数`$587C5F33`除以`Char`的ASCII值，取余数，余数转化为10进制字符串。
5. 循环取字符到`length(M2)-6`结束。
6. 将余数10进制字符串转换为16进制字符串即为注册码。

贴出注册机 delphi 源码：

```
Procedure TForm1.btn1Click(Sender: TObject);
Const
  Num: Integer = $587C5F33;
Var
  MnoStr, SerailNo: String;
  i: Integer;
Begin
  MnoStr := 'aB' + IntToHex(StrToInt64(edt1.Text), 1) + 'Cd';
  For i := 0 To Length(MnoStr) - 6 Do
    SerailNo := SerailNo + FormatFloat('0', Num Mod Ord(MnoStr[Length(MnoStr) - i]));
  edt2.Text := IntToHex(StrToInt64(SerailNo), 1);
End;
```

### 内存注册机

- 中断地址：`004949F1`
- 中断次数：`1`
- 第一字节：`8B`
- 指令长度：`3`
- 注册码：内存方式 寄存器 `EAX`

大家可以验证一下：

- 机器码：`1072429567`
- 注册码：`278114D14C6B`

第2次写破文，请大侠勿见笑。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
