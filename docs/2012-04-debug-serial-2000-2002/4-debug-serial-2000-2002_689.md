
# 天网防火墙个人版 2.0.2.98

> 原文：[`www.pediy.com/kssd/pediy03/forum619.htm`](https://www.pediy.com/kssd/pediy03/forum619.htm)

天网防火墙个人版 2.0.2.98(beta) 的破解，参考 TAE! 的破解。

程序：电脑报合订本 2000 配套光盘。

目标：找出注册码。

工具：`TRW2K` & `W32DASM`

如有雷同，纯属巧合。

---

## 1、首先

*   Possible StringData Ref from Data Obj -> " 注册成功！ 谢谢您的注册。"

```
:0040AFBE BA11004D00              mov edx, 004D0011
:0040AFC3 8D45DC                  lea eax, dword ptr [ebp-24]
:0040AFC6 E821DF0B00              call 004C8EEC
:0040AFCB FF461C                  inc [esi+1C]
:0040AFCE 8B00                    mov eax, dword ptr [eax]
:0040AFD0 E8A7930500              call 0046437C
:0040AFD5 FF4E1C                  dec [esi+1C]
:0040AFD8 8D45DC                  lea eax, dword ptr [ebp-24]
:0040AFDB BA02000000              mov edx, 00000002
:0040AFE0 E823E00B00              call 004C9008
:0040AFE5 8BC3                    mov eax, ebx
:0040AFE7 E898E80400              call 00459884
:0040AFEC EB2D                    jmp 0040B01B
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:0040AE92`(C) => 注意！看看！

```
:0040AFEE 66C746103800            mov [esi+10], 0038
```

*   Possible StringData Ref from Data Obj -> "密码和用户名不对，请注意检查大小写。"

```
:0040AFF4 BA2C004D00              mov edx, 004D002C
```

---

## 2、往上看看这里

```
:0040AE51 E8C6AEFFFF              call 00405D1C ==> 此 CALL 重要！进入！
```

*   Reference To: `snfw._frmMain`

```
:0040AE56 8B15D8864D00            mov edx, dword ptr [004D86D8]
:0040AE5C 8B12                    mov edx, dword ptr [edx]
:0040AE5E 888205030000            mov byte ptr [edx+00000305], al ==> 注意
:0040AE64 FF4E1C                  dec [esi+1C]
:0040AE67 8D45F8                  lea eax, dword ptr [ebp-08]
:0040AE6A BA02000000              mov edx, 00000002
:0040AE6F E894E10B00              call 004C9008
:0040AE74 FF4E1C                  dec [esi+1C]
:0040AE77 8D45FC                  lea eax, dword ptr [ebp-04]
:0040AE7A BA02000000              mov edx, 00000002
:0040AE7F E884E10B00              call 004C9008
:0040AE84 A1D8864D00              mov eax, dword ptr [004D86D8]
:0040AE89 8B38                    mov edi, dword ptr [eax]
:0040AE8B 80BF0503000000          cmp byte ptr [edi+00000305], 00 ==> 注意
:0040AE92 0F8456010000            je 0040AFEE ==> 注意！
```

---

## 3、进入 call 00405D1C 看看

*   Referenced by a CALL at Addresses:
    *   `:004060B8`, `:0040AE51` => 可返回 `004060B8` 看看是 TAE! 关键处。

```
:00405D1C 55                      push ebp
:00405D1D 8BEC                    mov ebp, esp
:00405D1F 81C474FFFFFF            add esp, FFFFFF74
:00405D25 B81CDA4C00              mov eax, 004CDA1C
......
......
......
......
:00405EB5 8D45EC                  lea eax, dword ptr [ebp-14]
:00405EB8 5A                      pop edx
:00405EB9 E82E320C00              call 004C90EC ==> 关键 CALL 进入！
:00405EBE 50                      push eax
:00405EBF FF4E1C                  dec [esi+1C]
:00405EC2 8D45E8                  lea eax, dword ptr [ebp-18]
:00405EC5 BA02000000              mov edx, 00000002
:00405ECA E839310C00              call 004C9008
:00405ECF FF4E1C                  dec [esi+1C]
:00405ED2 8D45EC                  lea eax, dword ptr [ebp-14]
:00405ED5 BA02000000              mov edx, 00000002
:00405EDA E829310C00              call 004C9008
:00405EDF FF4E1C                  dec [esi+1C]
:00405EE2 8D45F0                  lea eax, dword ptr [ebp-10]
:00405EE5 BA02000000              mov edx, 00000002
:00405EEA E819310C00              call 004C9008
:00405EEF FF4E1C                  dec [esi+1C]
:00405EF2 8D45F4                  lea eax, dword ptr [ebp-0C]
:00405EF5 BA02000000              mov edx, 00000002
:00405EFA E809310C00              call 004C9008
:00405EFF FF4E1C                  dec [esi+1C]
:00405F02 8D45F8                  lea eax, dword ptr [ebp-08]
:00405F05 BA02000000              mov edx, 00000002
:00405F0A E8F9300C00              call 004C9008
:00405F0F FF4E1C                  dec [esi+1C]
:00405F12 8D45FC                  lea eax, dword ptr [ebp-04]
:00405F15 BA02000000              mov edx, 00000002
:00405F1A E8E9300C00              call 004C9008
:00405F1F 58                      pop eax
:00405F20 8B0E                    mov ecx, dword ptr [esi]
:00405F22 64890D00000000          mov dword ptr fs:[00000000], ecx
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:00405DFE`(U)

```
:00405F29 5E                      pop esi
:00405F2A 5B                      pop ebx
:00405F2B 8BE5                    mov esp, ebp
:00405F2D 5D                      pop ebp
:00405F2E C3                      ret
```

---

## 4、进入 call 004C90EC 看看

*   Referenced by a CALL at Addresses:
    *   `:00405B00`, `:00405B3E`, `:00405EB9`, `:004064B3`, `:00407F5B`, `:00408059`, `:004083C5`

```
:004C90EC 55                      push ebp
:004C90ED 8BEC                    mov ebp, esp
:004C90EF 53                      push ebx
:004C90F0 8B00                    mov eax, dword ptr [eax] ==> 看看这里 cool code
:004C90F2 8B12                    mov edx, dword ptr [edx] ==> 看看这里 your bad code
:004C90F4 E8E7E8FDFF              call 004A79E0   ==> 看看里面
:004C90F9 0F94C0                  sete al          ==> 看看
:004C90FC 83E001                  and eax, 00000001 ==> 看
:004C90FF 5B                      pop ebx
:004C9100 5D                      pop ebp
:004C9101 C3                      ret
```

---

## 5、结论

`UserName=zest`
`RegisterKey=9ED4`

```
/zest/
2001.2  4dd466d0e1e0cc123afbe504da2462ca
```
