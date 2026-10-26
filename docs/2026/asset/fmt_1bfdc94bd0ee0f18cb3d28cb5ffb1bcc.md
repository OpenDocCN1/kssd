
```vb
txt1_0 = Hex(Asc(Mid(txt1, i, 1)))

txt1_1 = txt1_1 & txt1_0

Next

For i = 1 To Len(txt1_1) Step 2

txt1_3 = Mid(txt1_1, i, 2)

txt1_2 = txt1_3 & txt1_2

Next

If Len(txt1_1) > 8 Then

int1_1 = &H5DFE

int1_2 = &HE4A4

n = 0

For m = 1 To Len(txt1_2) Step 2

int2 = Mid(txt1_2, Len(txt1_2) - 6 - m, 8)

int2_1 = Left(int2, 4)

int2_2 = Right(int2, 4)

int1_1 = int1_1 Xor ("&H" & int2_1)

int1_2 = int1_2 Xor ("&H" & int2_2)

n = n + 1

If n >= (Len(txt1_2) / 2 - 4) Then

Exit For

End If

Next

int1_2 = Right(Hex(int1_2), 4)

txt2 = Hex(int1_1) & int1_2

For i = 1 To Len(txt2)

        Select Case Mid(txt2, Len(txt2) - i + 1, 1)

            Case "0": B = B + 16 ^ (i - 1) * 0

            Case "1": B = B + 16 ^ (i - 1) * 1

            Case "2": B = B + 16 ^ (i - 1) * 2

            Case "3": B = B + 16 ^ (i - 1) * 3

            Case "4": B = B + 16 ^ (i - 1) * 4

            Case "5": B = B + 16 ^ (i - 1) * 5

            Case "6": B = B + 16 ^ (i - 1) * 6

            Case "7": B = B + 16 ^ (i - 1) * 7

            Case "8": B = B + 16 ^ (i - 1) * 8

            Case "9": B = B + 16 ^ (i - 1) * 9

            Case "A": B = B + 16 ^ (i - 1) * 10

            Case "B": B = B + 16 ^ (i - 1) * 11

            Case "C": B = B + 16 ^ (i - 1) * 12

            Case "D": B = B + 16 ^ (i - 1) * 13

            Case "E": B = B + 16 ^ (i - 1) * 14

            Case "F": B = B + 16 ^ (i - 1) * 15

        End Select

    Next i

Text2.Text = B

Else

Text2.Text = "请输入一个位数大于五位的英文名或大于两位的中文名！"

End If

End Sub
```

------------------------------------------------------------------------

【版权声明】本文只是出于学习和交流目的，请勿用于商业用途，否则后果自负。软件版权归作者所有。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
