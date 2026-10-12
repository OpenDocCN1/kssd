
# Cool ASCII Art Maker V1.21

> 原文：[`www.pediy.com/kssd/pediy03/forum913.htm`](https://www.pediy.com/kssd/pediy03/forum913.htm)

如何破解 `Cool ASCII Art Maker V1.21` 注册码

**作者**：`PaulYoung`

**破解时间**：2001.5.3

**软件简介**：图像转文本工具

**下载**：`http://serser.top263.net/products/caa/caa.htm`

**破解工具**：`TRW 2000 V1.03`

---

该软件运行时弹出提示注册框，很不爽，破吧！

1.  填入用户名、注册码，运行 `TRW 2000`，下 `bpx hmemcpy`，`F5` 退出，按 `OK`，中断于 `TRW 2000`。
2.  `BD *`（中断所有断点），`PMODULE`（跳到程序领空），按 `F12`，8 次出错，因此 `BE *`（重建断点），`F5` 退出，再按 `OK`，这次按 7 次。
3.  `F10` 单步跟踪。

```
.
.
.
:00404207 8D45FC          lea eax, dword ptr [ebp-04]
:0040420A BA02000000      mov edx, 00000002
:0040420F E83C100900      call 00495250
:00404214 59              pop ecx
:00404215 84C9            test cl, cl
:00404217 743B            je 00404254   //下 r fl z，改变跳转方向，否则再跟几步就出错，不能跟踪到计算注册码的 CALL
:00404219 66C746101400    mov [esi+10], 0014
* Possible StringData Ref from Data Obj ->"The code is not correct!"  （出错信息）
                  |
:0040421F BAA4854900      mov edx, 004985A4
:00404224 8D45F0          lea eax, dword ptr [ebp-10]
:00404227 E8200F0900      call 0049514C
:0040422C FF461C          inc [esi+1C]
:0040422F 8B00            mov eax, dword ptr [eax]
:00404231 E812740500      call 0045B648
:00404236 FF4E1C          dec [esi+1C]
:00404239 8D45F0          lea eax, dword ptr [ebp-10]
:0040423C BA02000000      mov edx, 00000002
:00404241 E80A100900      call 00495250
:00404246 8B0E            mov ecx, dword ptr [esi]
:00404248 64890D00000000  mov dword ptr fs:[00000000], ecx
:0040424F E909010000      jmp 0040435D
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404217(C)
|
:00404254 33C0            xor eax, eax
:00404256 8945E8          mov dword ptr [ebp-18], eax
:00404259 8D55E8          lea edx, dword ptr [ebp-18]
:0040425C FF461C          inc [esi+1C]
:0040425F 8B83C8020000    mov eax, dword ptr [ebx+000002C8]
:00404265 E8EEC10500      call 00460458      //D EDX ,ALT+↓，看到了吗，是 CODE 耶（全部数字）
:0040426A 8D55E8          lea edx, dword ptr [ebp-18]
:0040426D 8D45EC          lea eax, dword ptr [ebp-14]
:00404270 8B0A            mov ecx, dword ptr [edx]
.
.
.
```

注：用户名不支持中文，为什么？因为我试过呀！！！！:)

***

小弟为庆祝“五一”国际劳动节，特制了一个网页，主要是面向初学破解者，本站坚持原创精神，软件均为本人原创作品，当然也放了不少大师级人马的经典破解教程，非常适合初学者学习。

欢迎光临！！！！

* `Cracker` 初体验 *

`http://paulyoung.yeah.net` `4dd466d0e1e0cc123afbe504da2462ca`
