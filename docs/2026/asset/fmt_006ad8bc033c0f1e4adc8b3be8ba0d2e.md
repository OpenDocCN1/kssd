
```asm
  004DAFC8  |.  BB 00030000   mov     ebx, 300
  004DAFCD  |.  E8 51720000   call    004E2223
  004DAFD2  |.  83C4 28       add     esp, 28
  004DAFD5  |>  E9 40000000   jmp     004DB01A
  004DAFDA  |>  837D 08 00    cmp     dword ptr [ebp+8], 0
  004DAFDE  |.  0F84 36000000 je      004DB01A
  004DAFE4  |.  68 04000080   push    80000004
  004DAFE9  |.  6A 00         push    0
  004DAFEB  |.  68 5CA74100   push    0041A75C                         ;  提示
  004DAFF0  |.  68 01030080   push    80000301
  004DAFF5  |.  6A 00         push    0
  004DAFF7  |.  68 40000000   push    40
  004DAFFC  |.  68 04000080   push    80000004
  004DB001  |.  6A 00         push    0
  004DB003  |.  68 C8A74100   push    0041A7C8                         ;  注册失败,请输入完整的注册信息！
  004DB008  |.  68 03000000   push    3
  004DB00D  |.  BB 00030000   mov     ebx, 300
  004DB012  |.  E8 0C720000   call    004E2223
  004DB017  |.  83C4 28       add     esp, 28
  004DB01A  |>  8B5D F8       mov     ebx, dword ptr [ebp-8]
  004DB01D  |.  85DB          test    ebx, ebx
  004DB01F  |.  74 09         je      short 004DB02A
  004DB021  |.  53            push    ebx
  004DB022  |.  E8 F0710000   call    004E2217
  004DB027  |.  83C4 04       add     esp, 4
  004DB02A  |>  8BE5          mov     esp, ebp
  004DB02C  |.  5D            pop     ebp
  004DB02D  \.  C2 0800       retn    8
```

注册算法为: 依次把用户名每个字母转换为ASCII码后再转换为十进制数值串起来即可。

```vb
  Private Sub Text1_Change()
  'Keygen by KuNgBiM
      Dim name As String
      Dim n, m, i As Byte
      Dim sn, tmp  As String
      name = Text1.Text
      n = Len(name)
      For i = 1 To n
          tmp = Mid(name, i, 1)
          m = Asc(tmp)
          sn = sn & m
      Next
      Text2.Text = sn
  End Sub
```

```pascal
  procedure Tkeygen.Text1Change(Sender: TObject);
  var
        name, tmp: AnsiString;
        n, m, sn: Variant;
        i: Char;
  begin
  //{$DEFINE def_Text1_Change}
  {$IF Defined(def_Text1_Change)}
        // Keygen by KuNgBiM
        name := Text1.Text;
        n :=  Length(name);
        for i:=1 to n do begin
                tmp :=  Copy(name, i, 1);
                m :=  Asc(tmp);
                sn := sn+m;
        end;
        Text2.Text := sn;
  {$IFEND} // def_Text1_Change
  end;
  end.
```

```cpp
  void Ckeygen::OnChangeText1()
  {
        // TODO: Add your control notification handler code here
        // Keygen by KuNgBiM
         CString name;
         CComVariant n, m; BYTE i;
         CComVariant sn; CString tmp;
        name = Text(m_Text1);
        n =  name.GetLength();
        for(i=1; i<=n; i++) {
                tmp =  name.Mid(i-1, 1);
                m =  Asc(tmp);
                sn = sn+m;
        }
        m_Text2.SetWindowText(sn);
  }
```

---

### 【经验总结】

新年快乐！没什么好总结的，凑合看吧！

---

### 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年02月17日 PM 02:37:57

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`tut.keygen.src.rar`](javascript:void(0);)（44.09kb，46次下载）
