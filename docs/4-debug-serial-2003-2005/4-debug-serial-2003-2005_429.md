
# 乐透之王 V3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6237.htm`](https://www.pediy.com/kssd/pediy06/pediy6237.htm)

## 彩票分析家-“乐透之王”V3.0

没有加壳,VC++6.0 的作品,反汇编后查找字符串资源,到下面：

*   Reference To: `MFC42.Ordinal:181C`, `Ord:181Ch`

```
|
:00430F72 E8BF100400 Call 00472036
:00430F77 8B8690020000 mov eax, dword ptr [esi+00000290]
:00430F7D 8B8094B10200 mov eax, dword ptr [eax+0002B194]
:00430F83 83F81E cmp eax, 0000001E
:00430F86 0F8E8A000000 jle 00431016--------------------------->跳则说明试用期没有结束（1）
```

*   Possible StringData Ref from Data Obj ->`"试用期已完,"`

```
|
:00430F8C 68D0004A00 push 004A00D0
:00430F91 8D4C2414 lea ecx, dword ptr [esp+14]
```

*   Reference To: `MFC42.Ordinal:0219`, `Ord:0219h`

```
|
:00430F95 E87A0E0400 Call 00471E14
:00430F9A 8B442410 mov eax, dword ptr [esp+10]
:00430F9E 8B542434 mov edx, dword ptr [esp+34]
:00430FA2 C68424A800000006 mov byte ptr [esp+000000A8], 06
:00430FAA 8B48F8 mov ecx, dword ptr [eax-08]
:00430FAD 51 push ecx
:00430FAE 50 push eax
:00430FAF 68D2000000 push 000000D2
:00430FB4 6868010000 push 00000168
:00430FB9 8D4C2444 lea ecx, dword ptr [esp+44]
:00430FBD FF5264 call [edx+64]
:00430FC0 8D4C2410 lea ecx, dword ptr [esp+10]
:00430FC4 889C24A8000000 mov byte ptr [esp+000000A8], bl
```

*   Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

```
|
:00430FCB E8BA0D0400 Call 00471D8A
```

*   Possible StringData Ref from Data Obj ->`"如要继续使用您请注册!"`

```
|
:00430FD0 68B8004A00 push 004A00B8
:00430FD5 8D4C2414 lea ecx, dword ptr [esp+14]
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

上面（1）处更改以后可以无限期使用,但是每次使用都会出现提示框，很烦人。进一步跟踪发现这个软件注册码的比较是明文比较的,比较处如下：

*   Possible StringData Ref from Data Obj ->`"CPFXJLTZW"`

```
|
:00465347 68341C4A00 push 004A1C34
```

*   Reference To: `MFC42.Ordinal:0219`, `Ord:0219h`

```
|
:0046534C E8C3CA0000 Call 00471E14
:00465351 51 push ecx
:00465352 8D542428 lea edx, dword ptr [esp+28]
:00465356 8BCC mov ecx, esp
:00465358 89642444 mov dword ptr [esp+44], esp
:0046535C 52 push edx
:0046535D C68424FC0100000A mov byte ptr [esp+000001FC], 0A
```

*   Reference To: `MFC42.Ordinal:0217`, `Ord:0217h`

```
|
:00465365 E832CA0000 Call 00471D9C----------------------->取得机器码,存在堆栈中
:0046536A 53 push ebx
:0046536B 8D4C244C lea ecx, dword ptr [esp+4C]
:0046536F C68424FC01000006 mov byte ptr [esp+000001FC], 06
:00465377 E804B1FFFF call 00460480
:0046537C 8D442438 lea eax, dword ptr [esp+38]
:00465380 8D4C2440 lea ecx, dword ptr [esp+40]
:00465384 50 push eax
:00465385 E876ACFFFF call 00460000------------------------>生成真注册码，写注册机可以仔细研究一下
:0046538A 8B4C241C mov ecx, dword ptr [esp+1C]---------->假注册码
:0046538E 8B3F mov edi, dword ptr [edi]------------->真注册码
:00465390 51 push ecx\-----------------------------
:00465391 57 push edi\-----------------------------/真假注册码入栈
:00465392 C68424F00100000B mov byte ptr [esp+000001F0], 0B
```

*   Reference To: `MSVCRT._mbscmp`, `Ord:0159h` -------------------------->调用比较函数

```
|
:0046539A FF15507D4800 Call dword ptr [00487D50]
:004653A0 83C408 add esp, 00000008
:004653A3 85C0 test eax, eax\-------------------------->以 EAX 为标志
:004653A5 741E je 004653C5---------------------------->跳则正确
:004653A7 6A10 push 00000010
```

*   Possible StringData Ref from Data Obj ->`"注册"`

```
|
:004653A9 68D81C4A00 push 004A1CD8
```

*   Possible StringData Ref from Data Obj ->`"注册码输入错误, 请重新注册 !"`

```
|
:004653AE 68141C4A00 push 004A1C14-------------------------->(2)
:004653B3 8BCE mov ecx, esi
```

*   Reference To: `MFC42.Ordinal:1080`, `Ord:1080h`

```
|
:004653B5 E8FCCA0000 Call 00471EB6
:004653BA FF86C8000000 inc dword ptr [esi+000000C8]
:004653C0 E9B3000000 jmp 00465478
```

由于是明码比较,而且注册码生成过程太复杂,所以新手还是做个自注册版吧。上面（2）处是出错对话框的提示信息，我们让他自己把注册码显示出来，由于注册码比较完毕以后一直都在 `edi` 中（动态跟踪发现的），所以我们把（2）改为 `push edi` ,即 `57` ,其余的四个字节改 `90`，保存后 ok！

`4dd466d0e1e0cc123afbe504da2462ca`
