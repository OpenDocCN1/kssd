
# 第二题算法－－拨开乌云见月明

> 原文：[`www.pediy.com/kssd/pediy10/74160.html`](https://www.pediy.com/kssd/pediy10/74160.html)

怎么找算法位置的过程就略过了，算法会先计算用户名是否为 a-y 这 25 个小写字母，长度是否为 12。然后用二重循环比较了一下大小，并统计了各字符的大小顺序，保存到数组 `tbNO[]` 中。这里会要求用户名中不能有重复的字符。再对注册码进行了转换，要求为 0－3 这四种数字，长度不小于 53。

然后进入到注册码变换过程。我花了 N 长时间把这个过程用 VB 描述了一下：

代码:
```
```



```vb
Private Function CheckKey(UN As String, SN As String) As String
    Dim i As Long, j As Long, k As Long, l As Long
    Dim nRowCount As Long, n108 As Long, n10C As Long, n110 As Long, n114 As Long, n118 As Long, n11C As Long, n120 As Long, n124 As Long
    Dim tbTemp40(MAXLEN - 1) As Long
    Dim tbTemp100(100) As Long
    
    
    Dim tbRet(MAXLEN * 2) As Byte '返回字符串
    Dim tbSN(53) As Long '注册码
    Dim tbNO(MAXLEN - 1) As Long '用户名各字符的大小顺序
    
    
    '----init table------
    '这一段是我添加的，作用是对用户名和注册码进行转换。
    For i = 1 To Len(UN)
        For j = 1 To Len(UN)
            If Mid$(UN, i, 1) > Mid$(UN, j, 1) Then
                tbNO(i - 1) = tbNO(i - 1) + 1
            End If
        Next
    Next
    
    
    For i = 1 To Len(SN)
        tbSN(i - 1) = Val(Mid$(SN, i, 1))
        If i > 53 Then Exit For
    Next
    '--------------------
    
    '4022AC:
    i = 0
    j = 0
    k = 0
    l = 0
    
    '4022C8:
    Do Until i >= MAXLEN
        tbTemp40(i) = &H1E
        i = i + 1
    Loop
    
    nRowCount = 0
    n108 = 0
    
    If tbSN(0) > 0 Then
        nRowCount = MAXLEN \ tbSN(0)
    End If
    
    n10C = 0
    
    '40231B:
    Do Until n108 >= 2
        
        If n10C > 0 Then
            tbTemp40(tbNO(n10C - 1)) = &H1E
        End If
        
        n10C = 0
        
        '402350:
        Do Until n10C >= MAXLEN
            
            If n10C > 0 Then
                tbTemp40(tbNO(n10C - 1)) = &H1E
            End If
            
            If n108 = 0 Then
                tbTemp40(tbNO(n10C)) = &H28
                n10C = n10C + 1
            End If
            
            If n108 = 1 Then
                tbTemp40(tbNO(n10C)) = &H14
                n10C = n10C + 1
            End If
            
            '4023D3:
            k = 0
            For i = 0 To tbSN(0) - 1
                For j = 0 To nRowCount - 1
                    tbTemp100(i * 4 + j) = tbTemp40(k)
                    k = k + 1
                Next
            Next
            
            '402448:
            n110 = 0
            n114 = 0
            For i = 0 To nRowCount - 1
                n110 = tbTemp100(tbSN(1) * 4 + i) + n110
                n114 = tbTemp100(tbSN(2) * 4 + i) + n114
            Next
            
            '4024BD:
            If n110 = n114 Then '==============================================================================================
                If (tbTemp100(tbSN(3) * 4 + tbSN(4))) = (tbTemp100(tbSN(5) * 4 + tbSN(6))) Then
                    
                    If (tbTemp100(tbSN(7) * 4 + tbSN(8))) = (tbTemp100(tbSN(9) * 4 + tbSN(10))) Then
                        tbRet(l) = &H6C '"l"
                        l = l + 1
                    Else
                        tbRet(l) = &H6B '"k"
                        l = l + 1
                    End If
                    
                Else
                    
                    If (tbTemp100(tbSN(7) * 4 + tbSN(8))) = (tbTemp100(tbSN(9) * 4 + tbSN(10))) Then
                        tbRet(l) = &H6A '"j"
                        l = l + 1
                    Else
                        tbRet(l) = &H69 '"i"
                        l = l + 1
                    End If
                    
                End If
            End If
            
            '4025B9:
            If n110 > n114 Then '==============================================================================================
                i = 0
                n118 = 0
                n11C = 0
                
                tbTemp100(tbSN(1) * 4 + tbSN(11)) = tbTemp100(tbSN(12) * 4 + tbSN(13))
                tbTemp100(tbSN(1) * 4 + tbSN(14)) = tbTemp100(tbSN(15) * 4 + tbSN(16))
                tbTemp100(tbSN(1) * 4 + tbSN(17)) = tbTemp100(tbSN(18) * 4 + tbSN(19))
                tbTemp100(tbSN(2) * 4 + tbSN(20)) = tbTemp100(tbSN(21) * 4 + tbSN(22))
                tbTemp100(tbSN(2) * 4 + tbSN(23)) = tbTemp100(tbSN(24) * 4 + tbSN(25))
                tbTemp100(tbSN(2) * 4 + tbSN(26)) = tbTemp100(tbSN(27) * 4 + tbSN(28))
                
                '402712:
                For i = 0 To nRowCount - 1
                    n118 = tbTemp100(tbSN(1) * 4 + i) + n118
                    n11C = tbTemp100(tbSN(2) * 4 + i) + n11C
                Next
                
                If n118 > n11C Then '--------------------------------------------------------------------------
                    If (tbTemp100(tbSN(29) * 4 + tbSN(30))) = (tbTemp100(tbSN(31) * 4 + tbSN(32))) Then
                        tbRet(l) = &H65 '"e"
                        l = l + 1
                    Else
                        tbRet(l) = &H61 '"a"
                        l = l + 1
                    End If
                End If
                
                '4027D7:
                If n118 < n11C Then '--------------------------------------------------------------------------
                    If (tbTemp100(tbSN(33) * 4 + tbSN(34))) > (tbTemp100(tbSN(35) * 4 + tbSN(36))) Then
                        tbRet(l) = &H67 '"g"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(33) * 4 + tbSN(34))) < (tbTemp100(tbSN(35) * 4 + tbSN(36))) Then
                        tbRet(l) = &H66 '"f"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(33) * 4 + tbSN(34))) = (tbTemp100(tbSN(35) * 4 + tbSN(36))) Then
                        tbRet(l) = &H68 '"h"
                        l = l + 1
                    End If
                End If
                
                '4028DF:
                If n118 = n11C Then '--------------------------------------------------------------------------
                    k = 0
                    For i = 0 To tbSN(0) - 1
                        For j = 0 To nRowCount - 1
                            tbTemp100(i * 4 + j) = tbTemp40(k)
                            k = k + 1
                        Next
                    Next
                    
                    '402966:
                    If (tbTemp100(tbSN(37) * 4 + tbSN(38))) > (tbTemp100(tbSN(39) * 4 + tbSN(40))) Then
                        tbRet(l) = &H62 '"b"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(37) * 4 + tbSN(38))) < (tbTemp100(tbSN(39) * 4 + tbSN(40))) Then
                        tbRet(l) = &H63 '"c"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(37) * 4 + tbSN(38))) = (tbTemp100(tbSN(39) * 4 + tbSN(40))) Then
                        tbRet(l) = &H64 '"d"
                        l = l + 1
                    End If
                End If
            End If
            
            '402A5C:
            If n110 < n114 Then '==============================================================================================
                i = 0
                n120 = 0
                n124 = 0
                
                tbTemp100(tbSN(1) * 4 + tbSN(11)) = tbTemp100(tbSN(12) * 4 + tbSN(13))
                tbTemp100(tbSN(1) * 4 + tbSN(14)) = tbTemp100(tbSN(15) * 4 + tbSN(16))
                tbTemp100(tbSN(1) * 4 + tbSN(17)) = tbTemp100(tbSN(18) * 4 + tbSN(19))
                tbTemp100(tbSN(2) * 4 + tbSN(20)) = tbTemp100(tbSN(21) * 4 + tbSN(22))
                tbTemp100(tbSN(2) * 4 + tbSN(23)) = tbTemp100(tbSN(24) * 4 + tbSN(25))
                tbTemp100(tbSN(2) * 4 + tbSN(26)) = tbTemp100(tbSN(27) * 4 + tbSN(28))
                
                '402BB5:
                For i = 0 To nRowCount - 1
                    n120 = tbTemp100(tbSN(1) * 4 + i) + n120
                    n124 = tbTemp100(tbSN(2) * 4 + i) + n124
                Next
                
                '402C0F:
                If n120 > n124 Then '--------------------------------------------------------------------------
                    If (tbTemp100(tbSN(41) * 4 + tbSN(42))) > (tbTemp100(tbSN(43) * 4 + tbSN(44))) Then
                        tbRet(l) = &H66 '"f"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(41) * 4 + tbSN(42))) < (tbTemp100(tbSN(43) * 4 + tbSN(44))) Then
                        tbRet(l) = &H67 '"g"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(41) * 4 + tbSN(42))) < (tbTemp100(tbSN(43) * 4 + tbSN(44))) Then
                        tbRet(l) = &H68 '"h"
                        l = l + 1
                    End If
                End If
                
                '402D17:
                If n120 < n124 Then '--------------------------------------------------------------------------
                    If (tbTemp100(tbSN(45) * 4 + tbSN(46))) = (tbTemp100(tbSN(47) * 4 + tbSN(48))) Then
                        tbRet(l) = &H65 '"e"
                        l = l + 1
                    Else
                        tbRet(l) = &H61 '"a"
                        l = l + 1
                    End If
                End If
                
                '402D8B:
                If n120 = n124 Then '--------------------------------------------------------------------------
                    k = 0
                    For i = 0 To tbSN(0) - 1
                        For j = 0 To nRowCount - 1
                            tbTemp100(i * 4 + j) = tbTemp40(k)
                            k = k + 1
                        Next
                    Next
                    
                    '402E12:
                    If (tbTemp100(tbSN(49) * 4 + tbSN(50))) > (tbTemp100(tbSN(51) * 4 + tbSN(52))) Then
                        tbRet(l) = &H63 '"c"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(49) * 4 + tbSN(50))) < (tbTemp100(tbSN(51) * 4 + tbSN(52))) Then
                        tbRet(l) = &H62 '"b"
                        l = l + 1
                    End If
                    
                    If (tbTemp100(tbSN(49) * 4 + tbSN(50))) = (tbTemp100(tbSN(51) * 4 + tbSN(52))) Then
                        tbRet(l) = &H64 '"d"
                        l = l + 1
                    End If
                End If
            End If
            
        Loop
        
        n108 = n108 + 1
    Loop
    
    CheckKey = StrConv(tbRet, vbUnicode)
End Function
```



这时候可以发现有个地方的比较重复了，字符`g`和`h`计算了二次。

别看这段代码这么长，每次循环只会用到很小一个分支，只输出一个字符。

我盯着这段代码看了一整天（功力有限啊），才算是明白大概意思是生成一个`3*4`填充为`0x1E=30`的表格，每次循环把表格中的用户名各字符的大小顺序位变换成`0x28=40`或`0x14=20`，再判断某些位置的规则，符合其中某条时就将输入一个字符。最后循环`24`次后生成一个`24`字节的字符串返回。

返回后会对生成的`24`字节字符串分二组再次统计各字符的大小顺序，当这二组顺序值与用户名的顺序值`tbNO[]`都相同时则注册成功。

观察规律可以发现，如果把注册码某些位按规则设置，这个算法是恒成立的！（前提是把其中那个重复的判断修改掉）

于是输入用户名为`abcdefghijkl`，构造注册码使输入的字符串为`abcdefghijklabcdefghijkl`，我最后得到的注册码为`30120212220111212313121222323000101020102010200010102`，这就是这个程序的通用注册码。

### 对这个注册码简单解释一下

第`1`位表示表格的行数，按程序算法中使用的数据看必须为`3*4`的表格，因此行数必须为`3`。

第`2`，`3`位表示统计每行和值时使用的行号。当输出字符为`i`,`j`,`k`,`l`时表格第`3`行值会被更换，但统计和值时要求这二行的和值相同，因此这二个行号必须为第一行与第二行，即这二位的值为`0`，`1`。

后面部分按`2`位分组，每组都指定这个`3*4`表格的一个行列坐标点。

测试用户名`abcdefghijkl`和`abcghijkldef`等，注册码都用`30120212220111212313121222323000101020102010200010102`，全部通过。

到这里这个`CM`应该算是结束了，但是由于算法中修改了一个重复的比较，导致用户名又出现变数，当执行到那个比较时，可能会一次输出`2`个字符，也可能一个字符也没有。比如当使用`abcdefgijklh`时就会注册失败。

调试发现，当执行到那个重复的比较时，如果输入为`g`，输出则为`gh`；输入为`h`时则没有输出。因此用户名中的`h`必须紧跟在`g`之后，注册才能通过。

这是使用`a-l`这`12`个字符时的要求。如果扩展到所有合法字符，则要求变为：使用的`12`个字符中，大小顺序排第`8`的字符必须紧跟在大小顺序排第`7`的字符之后。

### 再总结一下对用户名的限制

1. 长度必须为`12`位。
2. 必须为`a-y`之间的这`25`个小写字母。
3. 用户名中不能有重复字符。
4. 所使用的`12`个字符中，大小顺序排第`8`的字符必须紧跟在大小顺序排第`7`的字符之后。

### 给出 NameGen 的 VB 代码

```
Private Function NameGen() As String
Dim i As Long, j As Long, iChar As Long
Dim i7 As Long, i8 As Long
Dim s As String, sTemp As String
Dim tbNO(MAXLEN - 1) As Long

    Randomize Timer
    
    '随机生成 12 位不重复字符
    For i = 0 To MAXLEN - 1
        iChar = 0
        Do Until iChar > 0
            iChar = Int(Rnd * 25) + 97
            For j = 0 To i - 1
                If iChar = Asc(Mid$(s, j + 1, 1)) Then
                    iChar = 0
                End If
            Next
        Loop
        
        s = s & Chr(iChar)
    Next
    
    '统计各字符的大小顺序
    For i = 1 To MAXLEN
        For j = 1 To MAXLEN
            If Mid$(s, i, 1) > Mid$(s, j, 1) Then
                tbNO(i - 1) = tbNO(i - 1) + 1
            End If
        Next
    Next
    
    '找到排第 7 和第 8 的字符位置
    For i = 0 To MAXLEN - 1
        If tbNO(i) = 6 Then i7 = i
        If tbNO(i) = 7 Then i8 = i
    Next
    
    If i7 + 1 <> i8 Then '当第 8 字符不在第 7 字符后时
        sTemp = Mid$(s, i8 + 1, 1) '取出第 8 字符
        s = Mid$(s, 1, i8) & Mid$(s, i8 + 2) '组合除第 8 字符的字符串
        If i7 > i8 Then i7 = i7 - 1 '调整第 7 字符的位置
        s = Mid$(s, 1, i7 + 1) & sTemp & Mid$(s, i7 + 2) '把第 8 字符插入到第 7 字符后
    End If
    
    NameGen = s
End Function
```

PS：看这个`CM`的标题为：拨开乌云见月明，恍然大悟：原来就是指用户名与注册码都被乌云遮住了，注册码为固定值，用户名才是真正的注册码！
