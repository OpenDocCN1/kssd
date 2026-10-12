
# 视窗锁王 ver4.8

> 原文：[`www.pediy.com/kssd/pediy03/forum1137.htm`](https://www.pediy.com/kssd/pediy03/forum1137.htm)

萍”来看看，视窗锁王 ver4.8 彻底完蛋了。破解原来如此简单。只改两字节.哈哈哈.......

用 `w32dasm` 打开 `discal.vxd`，找到如下代码:

```
:000001A0 83EC10                sub esp, 00000010
:000001A3 56                    push esi
:000001A4 33F6                  xor esi, esi
:000001A6 56                    push esi
:000001A7 6A02                  push 00000002
:000001A9 893500000000          mov dword ptr [00000000], esi
:000001AF 893500000000          mov dword ptr [00000000], esi
:000001B5 E862090000            call 00000B1C
:000001BA 85C0                  test eax, eax
:000001BC 0F85F5000000          jne 000002B7
:000001C2 8D442404              lea eax, dword ptr [esp+04]
:000001C6 50                    push eax
:000001C7 6800000000            push 00000000
:000001CC 6802000080            push 80000002
:000001D1 E83E090000            call 00000B14
:000001D6 83C40C                add esp, 0000000C
:000001D9 85C0                  test eax, eax
:000001DB 0F85D6000000          jne 000002B7
:000001E1 8B442404              mov eax, dword ptr [esp+04]
:000001E5 8D4C2410              lea ecx, dword ptr [esp+10]
:000001E9 8D542408              lea edx, dword ptr [esp+08]
:000001ED 51                    push ecx
:000001EE 52                    push edx
:000001EF 56                    push esi
:000001F0 56                    push esi
:000001F1 6800000000            push 00000000
:000001F6 50                    push eax
:000001F7 E810090000            call 00000B0C
:000001FC 83C418                add esp, 00000018
:000001FF 85C0                  test eax, eax
:00000201 0F85A3000000          jne 000002AA
:00000207 8B442408              mov eax, dword ptr [esp+08]
:0000020B 8D4C2410              lea ecx, dword ptr [esp+10]
:0000020F 8D54240C              lea edx, dword ptr [esp+0C]
:00000213 3578EFCDAB            xor eax, ABCDEF78
:00000218 51                    push ecx
:00000219 52                    push edx
:0000021A 89442410              mov dword ptr [esp+10], eax
:0000021E 8B44240C              mov eax, dword ptr [esp+0C]
:00000222 56                    push esi
:00000223 56                    push esi
:00000224 6800000000            push 00000000
:00000229 50                    push eax
:0000022A E8DD080000            call 00000B0C
:0000022F 83C418                add esp, 00000018
:00000232 85C0                  test eax, eax
:00000234 7574                  jne 000002AA
:00000236 8B4C240C              mov ecx, dword ptr [esp+0C]
:0000023A 33D2                  xor edx, edx
:0000023C 81F1780AE305          xor ecx, 05E30A78
:00000242 BEDF150000            mov esi, 000015DF
:00000247 8BC1                  mov eax, ecx
:00000249 F7F6                  div esi
:0000024B BE56000000            mov esi, 00000056
:00000250 81C278563412          add edx, 12345678
:00000256 23CA                  and ecx, edx
:00000258 33D2                  xor edx, edx
:0000025A 8BC1                  mov eax, ecx
:0000025C F7F6                  div esi
:0000025E 8BC1                  mov eax, ecx
:00000260 8BF2                  mov esi, edx
:00000262 33D2                  xor edx, edx
:00000264 03F1                  add esi, ecx
:00000266 B9BB230000            mov ecx, 000023BB
:0000026B F7F1                  div ecx
:0000026D 8B442408              mov eax, dword ptr [esp+08]
:00000271 81C2AF010000          add edx, 000001AF
:00000277 33F2                  xor esi, edx
:00000279 81EE05270000          sub esi, 00002705
:0000027F 3BC6                  cmp eax, esi
:00000281 8974240C              mov dword ptr [esp+0C], esi
:00000285 7523                  jne 000002AA                <--nop 掉
:00000287 8B542404              mov edx, dword ptr [esp+04]
:0000028B C7050000000001000000  mov dword ptr [00000000], 00000001
:00000295 52                    push edx
:00000296 E869080000            call 00000B04
:0000029B 83C404                add esp, 00000004
:0000029E E89D000000            call 00000340
:000002A3 5E                    pop esi
:000002A4 83C410                add esp, 00000010
:000002A7 C20800                ret 0008
```

只要把 `00000285` 行 nop 掉，全搞定了。既 `7523` 改为 `9090` 既可。

现在注册码可任意输入了.不过，改该文件前，重起 `windows`，按 `F8` 键，单步装载，不让 `discal.vxd` 装入.

说明:`000001bc` 和 `000001d1` 两处是查找系统中是否装有 `SICE` 和 `TRW`.

## 第二

在 `smith.vxd` 中还有防 `SICE` 和 `TRW` 的暗桩，如：

```
:00002761 55                    push ebp
:00002762 8BEC                  mov ebp, esp
:00002764 83EC24                sub esp, 00000024
:00002767 8B450C                mov eax, dword ptr [ebp+0C]
:0000276A 56                    push esi
:0000276B 8BF1                  mov esi, ecx
:0000276D C745F84F000000        mov [ebp-08], 0000004F
:00002774 83662000              and dword ptr [esi+20], 00000000
:00002778 894624                mov dword ptr [esi+24], eax
:0000277B 8B4508                mov eax, dword ptr [ebp+08]
:0000277E C70600000000          mov dword ptr [esi], 00000000
:00002784 89461C                mov dword ptr [esi+1C], eax
:00002787 8D45DC                lea eax, dword ptr [ebp-24]
:0000278A 50                    push eax
:0000278B 6A41                  push 00000041
:0000278D E84A100000            call 000037DC
:00002792 817DF886F30000        cmp dword ptr [ebp-08], 0000F386      <----此处防 SICE
:00002799 750F                  jne 000027AA
:0000279B 6A01                  push 00000001
:0000279D E8F20F0000            call 00003794
:000027A2 50                    push eax
:000027A3 6A00                  push 00000000
:000027A5 E8B6100000            call 00003860
:000027AA 8BC6                  mov eax, esi
:000027AC 5E                    pop esi
:000027AD C9                    leave
:000027AE C20800                ret 0008
```

但是这样一改，仍然还有防 `SICE` 和 `TRW` 的代码，有兴趣的朋友可继续查找。

当然，如你系统中未装载 `SICE` 和 `TRW`，则不需修改以上防 `SICE` 和 `TRW` 处的代码.

作者也真是的，如你购买了正确的注册码，系统中仍有 `SICE` 的话，仍不会注册成功。用心险恶!!!

有兴趣的朋友，可根据以上代码和 `locksmith.exe` 中的一段代码写出它的注册机 `4dd466d0e1e0cc123afbe504da2462ca`
