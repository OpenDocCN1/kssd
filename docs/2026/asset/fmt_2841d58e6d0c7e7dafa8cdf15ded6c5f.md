
## 反汇编片段

```
:005068D4 BA01000000              mov edx,
00000001
```
```
:005068D9 E82ADBEFFF              call 00404408
```
```
:005068DE 8B45A0
mov eax, dword ptr [ebp-60]
```
```
:005068E1 50
  push eax
```
```
:005068E2 8D4598
lea eax, dword ptr [ebp-68]
```
```
:005068E5 50
  push eax
```
```
:005068E6 8B45F8
mov eax, dword ptr [ebp-08]
```
```
:005068E9 E812D9EFFF              call 00404200
```
```
:005068EE 8BD0
  mov edx, eax
```
```
:005068F0 83EA00
sub edx, 00000000
```
```
:005068F3 B901000000              mov ecx,
00000001
```
```
:005068F8 8B45F8
mov eax, dword ptr [ebp-08]
```
```
:005068FB E808DBEFFF              call 00404408
```
```
:00506900 8B5598
mov edx, dword ptr [ebp-68]
```
```
:00506903 58
  pop eax
```
```
:00506904 E807DAEFFF              call 00404310
```
`//序列号倒数第 1 位等于 "M" 吗`

```
:00506909 7430
  je 0050693B　　//不等则错
```
`Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`
```
|:005067BB(C), :00506810(C), :00506865(C), :005068B8(C)
```
`//就是上面的四处验证`

```
:0050690B 6A10
  push 00000010
```
`Possible StringData Ref from Code Obj ->"错误"`
```
:0050690D B9A06A5000              mov ecx,
00506AA0
```
`Possible StringData Ref from Code Obj ->"  注 册 号 错 误!  "`
```
:00506912 BAA86A5000              mov edx,
00506AA8
```
```
:00506917 A188FD5100              mov eax,
dword ptr [0051FD88]
```
```
:0050691C 8B00
  mov eax, dword ptr [eax]
```
```
:0050691E E861E5F4FF              call 00454E84
```
```
:00506923 A1F40A5200              mov eax,
dword ptr [00520AF4]
```
```
:00506928 8B8010030000            mov eax, dword
ptr [eax+00000310]
```
```
:0050692E 8B10
  mov edx, dword ptr [eax]
```
```
:00506930 FF92B0000000            call dword ptr
[edx+000000B0]
```
```
:00506936 E9A7000000              jmp 005069E2
```

## 算法总结

　　算法总结：为了叙述的方便，我用 `S1,S2,S3...S16` 分别指代注册码的 1 - 16 位。

    序列号第 1 位＝`S1`

    序列号第 3 位＝`S6`

    序列号第 5 位＝`S11`

    序列号第 7 位＝`S16`

序列号倒数第 4 位＝`S4`

序列号倒数第 3 位＝`S7`

序列号倒数第 2 位＝`S10`

序列号倒数第 1 位＝`S13`

　　其余的可为任意字符或数字。

　　这么简单的算法，写注册机应该非常容易吧，留给大家练练手吧。

    如有错误，请多多批评指正。
