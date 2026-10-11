
# Aqua 3D Screen Saver v1.5

> 原文：[`www.pediy.com/kssd/pediy03/forum814.htm`](https://www.pediy.com/kssd/pediy03/forum814.htm)

软件下载：`http://www.digimindsoft.com/aqua3d.exe`

软件大小：`552KB`

问题：用找到的注册码可以注册成功，但屏幕保护出来大约 10 秒钟后，还是出现“未注册”提示框，让按“空格”键输入注册码。请高手看看，哪里有限制？

### Aqua 3D Screen Saver v1.5 注册码算法

用户名：任意

注册码格式：`xxxx-xxxx-xxxx-xxxx` (共 19 位，由 0-F 的数字组成)

#### 一、关键代码

```
:0040C0A8 E8A38EFFFF                call 00404F50 ;判断注册码格式及位数是否为 13H 位，第 5 位是否为'-'
:0040C0AD 83C414                   add esp, 00000014
:0040C0B0 84C0                    test al, al  ; 若符合 xxxx-xxxxxxxxxxxxxx 格式，则 al=01，
:0040C0B2 0F8493000000            je 0040C14B  ; 不跳
:0040C0B8 8B45E8                  mov eax, dword ptr [ebp-18]  ; ax=1-4 位
:0040C0BB 8B55E4                  mov edx, dword ptr [ebp-1C]  ; dx=6-9 位
:0040C0BE 8B4DE0                  mov ecx, dword ptr [ebp-20]  ; cx=11-14 位
:0040C0C1 057E340000              add eax, 0000347E
:0040C0C6 33C2                    xor eax, edx
:0040C0C8 33C1                    xor eax, ecx
:0040C0CA 35CD540000              xor eax, 000054CD
:0040C0CF 663945EE                cmp word ptr [ebp-12], ax    ; [ebp-12]=16-19 位
:0040C0D3 7576                    jne 0040C14B                 ; 相等即可注册成功
:0040C0D5 A1F0454400              mov eax, dword ptr [004445F0]
:0040C0DA 8945E8                  mov dword ptr [ebp-18], eax
:0040C0DD 8945EC                  mov dword ptr [ebp-14], eax

* Possible Reference to String Resource ID=59145: "Registration"
|
:0040C0E0 6809E70000              push 0000E709
:0040C0E5 8D4DE8                  lea ecx, dword ptr [ebp-18]
:0040C0E8 C645FC02                mov [ebp-04], 02
:0040C0EC E8540A0200              call 0042CB45

* Possible Reference to String Resource ID=59147: "Thanks for support!"
|
:0040C0F1 680BE70000              push 0000E70B
:0040C0F6 8D4DEC                  lea ecx, dword ptr [ebp-14]
:0040C0F9 E8470A0200              call 0042CB45
:0040C0FE 85F6                    test esi, esi
:0040C100 7504                    jne 0040C106
:0040C102 33C0                    xor eax, eax
:0040C104 EB03                    jmp 0040C109
```

#### 二、判断注册码是否为 13H(19)位

```
:00404FB3 83F913                  cmp ecx, 00000013
:00404FB6 7425                   je 00404FDD
```

#### 三、判断注册码第 5 位是否为'-'号

```
:00404FDD 807E042D                cmp byte ptr [esi+04], 2D
:00404FE1 7425                   je 00405008
```

#### 四、注册码算法

`(((1-4 位 + 347E) xor (6-9 位)) xor (11-14 位)) xor 54CD ==16-19 位`

#### 五、一组可用的注册码

User name: 任意

Reg. Code: `1111-1111-1111-1142`

#### 六、注册信息保存在注册表中

```
[HKEY_USERS\.DEFAULT\Software\Digital Minds Software\AQUA3D\Aqua3D]

"UserName"="abcdef"
"RegCode"="1111-1111-1111-1142"
```

`404cc7` 改 `jne` 为 `je`

你试试？呵呵......

`4dd466d0e1e0cc123afbe504da2462ca`
