
```
004BFAF4  |.  0F94C0        sete    al
004BFAF7  |.  83E0 7F       and     eax, 7F
004BFAFA  |.  0145 F8       add     dword ptr [ebp-8], eax
004BFAFD  |.  837D F8 07    cmp     dword ptr [ebp-8], 7          ; 如果前面 7 位数字验证成功，则 [ebp-8]=7
004BFB01  |.  0F9445 FF     sete    byte ptr [ebp-1]              ; 真则置[ebp-1]=1
004BFB05  |>  8A45 FF       mov     al, byte ptr [ebp-1]          ; 标志位 al 的值取决于[ebp-1]的值
004BFB08  |.  8BE5          mov     esp, ebp
004BFB0A  |.  5D            pop     ebp
```

```
004BFB05       mov     al, byte ptr [ebp-1] 修改为
               mov     al, 1 即可完美爆破。
```

### 六、总结

此程序算法简单，捏了个软柿子，呵呵。不过方便我等菜鸟学习。总结下注册算法部分，后面程序还对注册码进行了加密，放置在注册表 `HKEY_LOCAL_MACHINE\SOFTWARE\Access Manager` 不进行讨论。

1. 注册码必须是 14 位；
2. 注册码必须满足：
   - 第 1 位是”2”，第 3 位是“”，第 4 位是“1”，第 5 位是“9”，第 8 位是“0”，第 9 位是“5”，第 11 位是“3”。
   - 其它为任意数字或字符，提供一组可用注册码：`2@21956052367a`

---
