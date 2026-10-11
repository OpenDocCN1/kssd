
### 汇编代码片段

```
:004954AC E84FF3F6FF             call 00404800

:004954B1 8D4DF0                 lea ecx, dword ptr [ebp-10]

:004954B4 8B55F4                 mov edx, dword ptr [ebp-0C]

:004954B7 8B45F8                 mov eax, dword ptr [ebp-08]

:004954BA E895FBFFFF             call 00495054

:004954BF 8B45EC                 mov eax, dword ptr [ebp-14]

:004954C2 8B55F0                 mov edx, dword ptr [ebp-10]

:004954C5 E822F2F6FF             call 004046EC              ; <=== d edx 得注册码的最后 6 位真码

:004954CA 7504                   jne 004954D0

:004954CC B301                   mov bl, 01

:004954CE EB02                   jmp 004954D2
```

### 分析说明

上述代码段是通过计算 `KGL-` 后填写的 10 位注册码来得出最后 6 位真码。由于水平有限，具体计算逻辑尚不完全清楚 :)。

### 总结

1. **注册码格式**：
   - 注册码总长度为 22 位。
   - 若未输入 22 位字符，注册按键不会亮起。
   - 标准形式为：`KGL-XXXXXX-XXXX-XXXXX`。

2. **破解实例**：
   - 破解时填写的注册码为：`KGL-8FWNL8-FWNL-888888`。
   - 在地址 `004954C5` 处调试 `edx`，得到结果为 `COY75I`。
   - 因此，正确的完整注册码是：`KGL-8FWNL8-FWNL-COY75I`。

3. **结论**：
   - 注册码与姓名无关。
   - 特别感谢好友 `alphakk` 的指点。

祝各位玩得开心!!!

---

fwnl

2002.1.22

长沙

`4dd466d0e1e0cc123afbe504da2462ca`
