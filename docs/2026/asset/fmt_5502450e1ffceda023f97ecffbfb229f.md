
```
46                      inc esi
:004B533E 83FE05         cmp esi, 00000005
:004B5341 75E0           jne 004B5323
                        ====> 上面进行 4*16 次的循环！
```

呵呵，如此大动干戈的循环取数只是把 `DDcZ22cM19ZWIWWC` 的前 4 位 `DDcZ` 移动到字符串的最后！

循环结束后 `[EAX]` 内存中的值：

```
0074FD8B  32 32 63 4D 31 39 5A 57 49 57 57 43 44 44 63 5A  22cM19ZWIWWCDDcZ
```

```
:004B5343 C645EB00                mov [ebp-15], 00
:004B5347 8D55DB                  lea edx, dword ptr [ebp-25]
                                ====> EDX=22cM19ZWIWWCDDcZ
:004B534A 8B45F0                  mov eax, dword ptr [ebp-10]
:004B534D E8B23CF5FF              call 00409004
                                ====> EAX=22cM19ZWIWWCDDcZ
:004B5352 B301                    mov bl, 01
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004B523B(C), :004B5252(C), :004B5266(C), :004B5274(C), :004B52A6(C)
|

```
:004B5354 8BC3                  mov eax, ebx
:004B5356 5F                    pop edi
:004B5357 5E                    pop esi
:004B5358 5B                    pop ebx
:004B5359 8BE5                  mov esp, ebp
:004B535B 5D                    pop ebp
:004B535C C3                    ret
```

## 算法总结

算法很简单呀。循环取用户名的字符，依次和程序自给的 `k($j3dAd18L;0gfj` 进行异或运算！所得结果若：`>7A` 则和 `7A` 求模，`<30` 则加上 `30`，`<41` 则加上 `7`，`<61` 则加上 `6`。即把结果转换成数字或字母。最后把所得字符串的前 4 位移到末尾，既是注册码了！

## 完美爆破

```
004BCC00
8A45FF                  mov al, byte ptr [ebp-01]

改为：
8B4501                  mov al, 01
```

呵呵，和上面的 `004BCBC2 mov [ebp-01], 01` 处相映成趣！呵呵，让 `AL` 永远为 1，岂有不 OK 的？这样也就不怕网络校验了！

## KeyMake之{57th}内存注册机

- 中断地址：`4BCBB9`
- 中断次数：`1`
- 第一字节：`E8`
- 指令长度：`5`
- 内存方式：`EAX`

注意：用户名至少 5 位！注册码 16 位！

## 注册信息保存

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\Rockboy\eBookmark]
"UserName"="fly01"
"UserData"=hex:32,32,63,4d,31,39,5a,57,49,57,57,43,44,44,63,5a,00
```

## 整理

用户名：`fly01`

注册码：`22cM19ZWIWWCDDcZ`

Cracked By 巢水工作坊——fly【OCN】

2003-4-13   22:22
