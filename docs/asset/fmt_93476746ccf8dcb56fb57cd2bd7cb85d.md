
```vb
If tlen >= 10 Then      '只处理用户名前 10 位  

  e = k  

  h = nlen  

Else  

  If h = nlen Then       '如果注册名不满 10 位的处理方式  

  h = 0                 'h=0，一个循环上去，就又成初始值 1 了  

  End If  

End If  

Next e  

Next h  

ebx = 0  

ss = &HDE928F52  

For i = 2 To 9  

eax = B(i) And &HFF  

edx = ss And &HFF  

eax = eax Xor edx  

TMPLEN = Len(Hex(ss))  

edx = CLng("&h" + Mid(Hex(ss), 1, TMPLEN - 2))  

eax = A(eax) Xor edx  

ss = eax  

tmpsum = eax Mod 36  

If tmpsum < 0 Then         '对于 VB 中出现负数的处理  

 tmpsum = 40 + tmpsum  

End If  

tmpmod = tmpsum + 1  

TMPSTR = Mid(mabiao, tmpmod, 1)  

laststr = laststr + TMPSTR  

Next i  

sjs = Int(89 * Rnd + 10)    '注册码前 2 位设定为 10-99 之间的一个任意随机整数  

laststr = sjs + laststr  

Text2.Text = laststr  

End Sub
```

-----------------------------------------------------

### 7、注册信息保存在注册表里：

```ini
[HKEY_LOCAL_MACHINE\SOFTWARE\SDS Software\Setup2GO]

"username"="KuNgBiM"

"regcode"="98WDMRQ2XU"
```

-----------------------------------------------------

==========收工,睡觉咯~~~=========

Cracked By `KuNgBiM[DFCG]`

`2005-02-02 05:54:36`

`4dd466d0e1e0cc123afbe504da2462ca`
