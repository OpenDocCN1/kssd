
```asm
0043D684      <  66:05 0100               add ax,1                         ;  +1
0043D688      .  66:B9 1A00               mov cx,1A
0043D68C      .  0F80 05010000            jo ExcelToE.0043D797
0043D692      .  66:99                    cwd
0043D694      .  66:F7F9                  idiv cx                          ;  mod/1a
0043D697      .  66:83C2 41               add dx,41                        ;  +41
0043D69B      <  8B45 DC                  mov eax,dword ptr ss:[ebp-24]
0043D69E      .  C785 2CFFFFFF 08000000   mov dword ptr ss:[ebp-D4],8
0043D6A8      .  0F80 E9000000            jo ExcelToE.0043D797
0043D6AE      .  0FBFCA                   movsx ecx,dx                     ;   得到真正的ascii
```

共9位临时码

逐位取出临时码的每一位，当作数字，乘以位数，判断跟`1e`的关系。

大于的话，计算结果`mod 1a +41`

否则 计算结果`mod a +30`

得到注册码的ASCII

## 算法描述

```vb
For i = 1 To Len(user)
    userAscii = Asc(Mid(user, i, 1)) + userAscii
Next

sn = userAscii * Len(user)
sn = CStr(sn) & CStr(Len(user) ^ 3 + Len(code) ^ 3)
code = CLng(sn) + 69813 + 20030207

For i = 1 To Len(code)
    If CInt(Mid(code, i, 1)) * i < 30 Then '&H1e
        sn = sn + Chr((CInt(Mid(code, i, 1)) * i + 1) Mod 26 + 65) '&H1a hh41
    Else
        sn = sn + Chr((CInt(Mid(code, i, 1)) * i + 1) Mod 10 + 48) ' &H31
    End If
Next
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
