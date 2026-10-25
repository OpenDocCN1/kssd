
# 1stclass3000 和 infopower3000

> 原文：[`www.pediy.com/kssd/pediy04/kanxue398.htm`](https://www.pediy.com/kssd/pediy04/kanxue398.htm)

`delphi`控件`1stclass3000`和`infopower3000`的口令`pj`

两个软件的主页为:`http://www.woll2woll.com`

## 1STCLASS3000 RELEASE PRO

工具:`W32asm8.93`

用`W32asm`将`1stclass3000 pro.exe`反编译后并加载调试，直至弹出

口令错误的提示窗口，然后切换到`w32asm`的领空,按`w32asm`的自动跟踪

按钮，等一会儿再一次弹出口令的输入窗口，此时再切换到`w32asm`的领空

，按`[F7:step into]`按钮,然后再一次回到`1stclass3000`的口令输入窗口,

按`[ok]`按键，程序自动回到`w32asm`的领空，并停在下面的位置：

```
:10013BEC call GLC92B5.1001739A
:10013BF1 push eax
:10013BF2 push 00000603
:10013BF7 push dword ptr [100227DC]
:10013BFD call USER32.DialogBoxParamA
:10013C03 cmp eax, 00000002  <<-------当前 EIP 的位置
:10013C06 je 10013C36
:10013C08 push dword ptr [10022694]
:10013C0E push edi
:10013C0F call GLC92B5.10011B3B
:10013C14 push dword ptr [10022694]
:10013C1A call GLC92B5.10013C3B  <<-----关键的口令校验函数[跟入]
:10013C1F add esp, 0000000C
:10013C22 test eax, eax
:10013C24 jne 10013B7C
:10013C2A and dword ptr [100227FC], FFFFFFFE
:10013C31 xor eax, eax
:10013C33 pop edi
:10013C34 pop esi
:10013C35 ret
:10013C36 push 00000001
:10013C38 pop eax
.....进入 10013C1A 的 call 后，按一会儿 F10,来到这里：
:10013CC6 cmp byte ptr [eax], bl
:10013CC8 je 10013D01
:10013CCA cmp byte ptr [esi+edi], bl
:10013CCD je 10013D01
:10013CCF mov al, byte ptr [eax+esi]
:10013CD2 not al   <<----取反
:10013CD4 movzx eax, al
:10013CD7 push eax
:10013CD8 call GLC2332.100180BF
:10013CDD mov ebp, eax
:10013CDF movsx eax, byte ptr [esi+edi]
:10013CE3 push eax
:10013CE4 call GLC2332.100180BF
:10013CE9 pop ecx
:10013CEA cmp ebp, eax    <<---逐字节地比较
:10013CEC pop ecx
:10013CED jne 10013CFC    <<---不相等就出错
:10013CEF mov eax, dword ptr [100224D4]
:10013CF4 inc esi
:10013CF5 cmp byte ptr [eax+esi], bl
:10013CF8 jne 10013CCA
:10013CFA jmp GLC2332.10013D01
:10013CFC mov eax, dword ptr [100224D4]
:10013D01 cmp byte ptr [eax+esi], bl
:10013D04 jne 10013D0F
:10013D06 cmp byte ptr [esi+edi], bl
:10013D09 jne 10013D0F
:10013D0B xor eax, eax
:10013D0D jmp GLC2332.10013D12
:10013D0F push 00000001
:10013D11 pop eax
:10013D12 pop edi
:10013D13 pop esi
:10013D14 pop ebp
:10013D15 pop ebx
:10013D16 ret
```

看明白了密码的校验过程,我们可以在`10013CD2`处设断点,拦下后可见到

以下的数据:

| 地址 | 原码 | 取反 | 字符 |
| :--- | :--- | :--- | :--- |
| `[eax+00000000]` | `ce` | `.31` | `'1'` |
| `[eax+00000001]` | `ac` | `.53` | `'S'` |
| `[eax+00000002]` | `ab` | `.54` | `'T'` |
| `[eax+00000003]` | `cc` | `.33` | `'3'` |
| `[eax+00000004]` | `cf` | `.30` | `'0'` |
| `[eax+00000005]` | `cf` | `.30` | `'0'` |
| `[eax+00000006]` | `cf` | `.30` | `'0'` |
| `[eax+00000007]` | `c7` | `.38` | `'8'` |
| `[eax+00000008]` | `cd` | `.32` | `'2'` |
| `[eax+00000009]` | `cb` | `.34` | `'4'` |
| `[eax+0000000A]` | `c8` | `.37` | `'7'` |
| `[eax+0000000B]` | `cd` | `.32` | `'2'` |
| `[eax+0000000C]` | `ca` | `.35` | `'5'` |
| `[eax+0000000D]` | `ad` | `.52` | `'R'` |
| `[eax+0000000E]` | `b4` | `.4b` | `'K'` |
| `[eax+0000000F]` | `00` | | |

`1STCLASS3000 RELEASE PRO`

故此得到的注册码为：`1ST3000824725RK`

## INFORPOWER3000 RELEASE PRO

以同样的跟踪过程可以得到`INFORPOWER3000`的注册数据如下：

| 地址 | 原码 | 取反 | 字符 |
| :--- | :--- | :--- | :--- |
| `[eax+00000000]` | `b6` | `.49` | `'I'` |
| `[eax+00000001]` | `af` | `.50` | `'P'` |
| `[eax+00000002]` | `cc` | `.33` | `'3'` |
| `[eax+00000003]` | `c9` | `.36` | `'6'` |
| `[eax+00000004]` | `ca` | `.35` | `'5'` |
| `[eax+00000005]` | `cd` | `.32` | `'2'` |
| `[eax+00000006]` | `c8` | `.37` | `'7'` |
| `[eax+00000007]` | `c6` | `.39` | `'9'` |
| `[eax+00000008]` | `cd` | `.32` | `'2'` |
| `[eax+00000009]` | `cf` | `.30` | `'0'` |
| `[eax+0000000A]` | `ce` | `.31` | `'1'` |
| `[eax+0000000B]` | `c6` | `.39` | `'9'` |
| `[eax+0000000C]` | `cd` | `.32` | `'2'` |
| `[eax+0000000D]` | `00` | | |

`INFORPOWER 3000 RELEASE PRO`

故此得到的注册码为：`IP36527920192`

`4dd466d0e1e0cc123afbe504da2462ca`
