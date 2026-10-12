
```
代码:

* * *

0043E0AE   |.  68 14E74>push USBTrace.0049E714
0043E0B3   |.  6A 04    push 4
0043E0B5   |.  8BCD     mov ecx,ebp
0043E0B7   |.  E8 D6E00>call USBTrace.0045C192
0043E0BC   |.  68 14E74>push USBTrace.0049E714
0043E0C1   |.  6A 09    push 9
0043E0C3   |.  8BCD     mov ecx,ebp
0043E0C5   |.  E8 C8E00>call USBTrace.0045C192
0043E0CA   |.  BD 01000>mov ebp,1
0043E0CF   |>  8B4424 2>mov eax,dword ptr ss:\[esp+20\]
0043E0D3   |.  50       push eax
0043E0D4   |.  E8 2E020>call USBTrace.0044E307
0043E0D9   |.  83C4 04  add esp,4
0043E0DC   |>  8D4C24 1>lea ecx,dword ptr ss:\[esp+14\]
0043E0E0   |.  C68424 4>mov byte ptr ss:\[esp+148\],3
0043E0E8   |.  E8 30540>call USBTrace.0046351D
0043E0ED   |.  8D4C24 1>lea ecx,dword ptr ss:\[esp+1C\]
0043E0F1   |.  C68424 4>mov byte ptr ss:\[esp+148\],2
0043E0F9   |.  E8 1F540>call USBTrace.0046351D
0043E0FE   |.  8D4C24 1>lea ecx,dword ptr ss:\[esp+18\]
0043E102   |.  C68424 4>mov byte ptr ss:\[esp+148\],1
0043E10A   |.  E8 0E540>call USBTrace.0046351D
0043E10F   |.  8D4C24 3>lea ecx,dword ptr ss:\[esp+34\]
0043E113   |.  C68424 4>mov byte ptr ss:\[esp+148\],0
0043E11B   |.  E8 FD530>call USBTrace.0046351D
0043E120   |.  8D8C24 5>lea ecx,dword ptr ss:\[esp+150\]
0043E127   |.  C78424 4>mov dword ptr ss:\[esp+148\],-1
0043E132   |.  E8 E6530>call USBTrace.0046351D
0043E137   |.  8B8C24 4>mov ecx,dword ptr ss:\[esp+140\]
0043E13E   |.  5F       pop edi
0043E13F   |.  8BC5     mov eax,ebp
0043E141   |.  5E       pop esi
0043E142   |.  5D       pop ebp
0043E143   |.  5B       pop ebx
0043E144   |.  64:890D >mov dword ptr fs:\[0\],ecx
0043E14B   |.  81C4 3C0>add esp,13C
0043E151   \.  C2 0800  retn 8

* * *

```

一路 `F8` 过来，停到 `retn 8`

看堆栈：

```
0012F338    0043DDA3         返回到 USBTrace.0043DDA3 来自 USBTrace.0043DE80

0012F33C    003F7078         ASCII "winndy"

0012F340    0012F350

0012F344    004B04C0         ASCII "8o?"

0012F348    004B0464         USBTrace.004B0464

0012F34C    004ADDB4         USBTrace.004ADDB4

0012F350    003F7028         ASCII "41O1-8Z50-N43Y"
```

注册码出现了。

```
0043E0B3   |.  6A 04    push 4

0043E0C1   |.  6A 09    push 9
```

注意到两个－在第 4 和第 9 个位置，有兴趣的可以跟进去看看。

`retn 8` 执行后，返回继续看。

### 算法总结

1. 根据版本号确定一个字符串。
2. 根据用户名确定一个字符串。
3. 用 `TlsGetValue` 得到一个字符串，但是没用的。
4. 把字符串连接起来。

字符串的具体产生过程见代码分析，都很简单。

### 感谢

all of you！～

### 文章完成时间、地点

2005.12.28，武汉

### 注册机

win32 asm source

`4dd466d0e1e0cc123afbe504da2462ca`
