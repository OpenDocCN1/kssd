
```asm
004124B1   |.   EB 44             jmp short 004124F7
004124B3   |>   B8 16000000      mov eax,16
004124B8   |.   EB 3D            jmp short 004124F7
004124BA   |>   B8 1D000000      mov eax,1D
004124BF   |.   EB 36            jmp short 004124F7
004124C1   |>   B8 1E000000      mov eax,1E
004124C6   |.   EB 2F            jmp short 004124F7
004124C8   |>   B8 23000000      mov eax,23
004124CD   |.   EB 28            jmp short 004124F7
004124CF   |>   B8 20000000      mov eax,20
004124D4   |.   EB 21            jmp short 004124F7
004124D6   |>   B8 19000000      mov eax,19
004124DB   |.   EB 1A            jmp short 004124F7
004124DD   |>   B8 06000000      mov eax,6
004124E2   |.   EB 13            jmp short 004124F7
004124E4   |>   B8 1F000000      mov eax,1F
004124E9   |.   EB 0C            jmp short 004124F7
004124EB   |>   B8 0A000000      mov eax,0A
004124F0   |.   EB 05            jmp short 004124F7
004124F2   |>   B8 0B000000      mov eax,0B
004124F7   |>   3BD0             cmp edx,eax
004124F9   |.   74 0C            je short 00412507
004124FB   |.   5F               pop edi
004124FC   |.   C645 0F 10       mov byte ptr ss:[ebp+F],10
00412500   |.   5E               pop esi
00412501   |.   5D               pop ebp
00412502   |.   32C0             xor al,al
00412504   |.   5B               pop ebx
00412505   |.   59               pop ecx
00412506   |.   C3               retn
```

这里首先验证注册码位数，然后逐位验证。验证时没有出现真正的注册码，而是通过假码第一位跟第六位运行后得到一个数字，再利用假码第一位查表得到另外一个数字，比较这两个数字，相等则继续，否则注册失败。

注意`mov byte ptr ss:[ebp+F],10`这句，成功了是`mov byte ptr ss:[ebp+F],13`，这里估计是个暗桩。有兴趣下个读断点试试。

给出`vb`的注册机源码，对比程序代码很容易看：

```vb
    str = "1234567890abcdefghijklmnoprstuvwxyz{"
    str1 = "130201072205041000111c0c140e0d080912030f151b1817211a161d1e232019061f0a0b"
    eax = Asc(Mid$(mc, 6, 1))
    edx = Asc(Mid$(mc, 1, 1))
    ecx = eax * 3
    eax = eax + ecx * 4
    ecx = edx
    ecx = ecx * 2 ^ 5
    ecx = ecx + edx
    ecx = ecx * 3
    edx = edx + ecx * 2
    ecx = &H24
    eax = eax + edx
    edx = eax Mod ecx
    For i = 1 To Len(str1) - 1 Step 2
        If Right$("00" & Hex$(edx), 2) = UCase$(Mid$(str1, i, 2)) Then
            Tmp = Mid$(str, i \ 2 + 1, 1)
            Exit For
        End If
    Next
    If Tmp = "" Then Tmp = "{"
```

后面的7位计算方法与之类似。有几个地方要注意：

```asm
004128E8   |.   C645 0C 01       mov byte ptr ss:[ebp+C],1
00412941   |.   C645 0D 01       mov byte ptr ss:[ebp+D],1
0041298E   |.   C645 0F 13       mov byte ptr ss:[ebp+F],13
```

## 破解心得

有了`mfc`的`Lib`文件，`mfc`的函数一目了然，所以很容易看清注册流程。该软件的思路不错，在关键`call`里面设置了其他返回数据，因此不能采取在函数开始直接返回`al`值的方法来爆破，但是还是很容易找到关键点。注册成功后结果保存在注册表内。

关于利用`dll`取得机器码的方法不是很可靠，简单编写一个`dll`就可以一码多用了。

`4dd466d0e1e0cc123afbe504da2462ca`
