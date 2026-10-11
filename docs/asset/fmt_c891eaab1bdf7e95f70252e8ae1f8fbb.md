
# [原创]－网络信息采集大师 v2.6

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-746.htm`](https://www.pediy.com/kssd/pediy07/pediy7-746.htm)

## 1. 试运行

找到相关注册项目，提示：

版本：个人版，企业版...

识别码：`WD-WCAD12522066`

注册码：`87654321`

确认后提示：注册码不正确。

## 2. 查壳脱壳

`JDPack 1.x` / `JDProtect 0.9` -> `TLZJ18 Software`

`OD` 载入：

```
0060F000   pushad
0060F001   call NetGet.0060F006
```

一路下来，狂多的 `pushad` 和 `popad`，看的头晕，而且 n 多 `call` 进去也是 `pushad` 和 `popad`。

采用 esp 定律，从入口 `0060F000` 按 `F8` 一下到下一条 `0060F001`，然后查看寄存器 `esp` 值为：

`esp`：`0012FFA4`，在数据窗口 `hr 0012FFA4`，下硬件访问断点，然后 `F9` 运行，断在：

```
0060F415   push eax         ; NetGet.00559B64
0060F416   retn             （retn 可用作 call 或 jmp）
```

看到了吧，`00559B64`，离 `0060F415` 好远啊，然后 `F8` 运行 `retn` 退到：

```
00559B64   push ebp
```

呵呵，`OD` 现成的 `dump`，保存。

`PEID`：`Borland Delphi 6.0 - 7.0`

试运行，ok。

## 3. 根据“注册码不正确”下断：`MessageBoxA`

```
0012F1A8   004F827D   /CALL 到 MessageBoxA 来自 UPNetGet.004F8278
0012F1AC   001E04FE   |hOwner = 001E04FE ('注册',class='TregForm')
0012F1B0   005132AC   |Text = "注册码不正确"
0012F1B4   004F8280   |Title = "NetGet"
0012F1B8   00000030   \\Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
```

返回程序，再返回程序来到：`0051322C call UPNetGet.004F8270`

向上查看，代码如下：

```
005131C1   call UPNetGet.004592E0
005131C6   mov edx,UPNetGet.005132D8
  005132D0  5C 6B 65 79 2E 6B 63 00  \\key.kc.
  005132D8  D7 A2 B2 E1 B3 C9 B9 A6  注册成功
  005132E0  A3 AC D0 BB D0 BB CA B9  ，谢谢使
  005132E8  D3 C3 A3 A1 C8 B7 B6 A8  用！确定
  005132F0  BA F3 D6 D8 D0 C2 C6 F4  后重新启
  005132F8  B6 AF B3 CC D0 F2 00 00  动程序..

005131CB   mov ecx,40
005131D0   call UPNetGet.004F8270    估计是注册成功
005131D5   mov eax,dword ptr ds:[55F330]
005131DA   mov eax,dword ptr ds:[eax]
005131DC   call UPNetGet.00473D5C
005131E1   push 3
005131E3   push 0
005131E5   push 0
005131E7   lea edx,dword ptr ss:[ebp-200]
005131ED   mov eax,dword ptr ds:[55F330]
005131F2   mov eax,dword ptr ds:[eax]
005131F4   call UPNetGet.00474248
005131F9   mov eax,dword ptr ss:[ebp-200]
005131FF   call UPNetGet.00405178
00513204   push eax
00513205   push UPNetGet.00513300                  ;  ASCII "open"
0051320A   mov eax,dword ptr ss:[ebp-4]
0051320D   call UPNetGet.004592E0
00513212   push eax                                 ; |hWnd
00513213   call UPNetGet.00441B44                   ; \\ShellExecuteA
00513218   jmp short UPNetGet.00513231
0051321A   mov eax,dword ptr ss:[ebp-4]
0051321D   call UPNetGet.004592E0
00513222   mov edx,UPNetGet.005132AC
  005132AC  D7 A2 B2 E1 C2 EB B2 BB  注册码不
  005132B4  D5 FD C8 B7 00 00 00 00  正确....
00513227   mov ecx,30
0051322C   call UPNetGet.004F8270    “注册码不正确”
```

小结：从该段程序大概看出，注册成功后会产生一个文件（可能为 `key.kc`）。接下来代码很乱，不好跟踪，暂停。

## 4. `w32dasm` 串参考：“注册码不正确”

有 2 个地方，都列下来：

### 4.1 主线

```
:00512FEC E883F9F3FF           call 00452974    取识别码
:00512FF1 8B8524FEFFFF         mov eax, dword ptr [ebp+FFFFFE24]
:00512FF7 E8841FEFFF           call 00404F80    识别码长度
:00512FFC 83F806               cmp eax, 00000006
:00512FFF 7D1C                 jge 0051301D
:00513001 8B45FC               mov eax, dword ptr [ebp-04]
:00513004 E8D762F4FF           call 004592E0

* Possible StringData Ref from Code Obj ->"注册码不正确"    >>>这个好像没什么用
                        |
:00513009 BAAC325100           mov edx, 005132AC
:0051300E B930000000           mov ecx, 00000030
:00513013 E85852FEFF           call 004F8270
:00513018 E91E020000           jmp 0051323B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00512FFF(C)
|
:0051301D B201                 mov dl, 01

* Possible StringData Ref from Code Obj ->"Treg 岪"
                        |
:0051301F A1B4814F00           mov eax, dword ptr [004F81B4]
:00513024 E83356FEFF           call 004F865C
:00513029 8945F8               mov dword ptr [ebp-08], eax
:0051302C 8D45F4               lea eax, dword ptr [ebp-0C]
:0051302F 50                   push eax
:00513030 8B45FC               mov eax, dword ptr [ebp-04]
:00513033 8B8010030000         mov eax, dword ptr [eax+00000310]
:00513039 8B10                 mov edx, dword ptr [eax]
:0051303B FF92C8000000         call dword ptr [edx+000000C8]
:00513041 50                   push eax
:00513042 8D9520FEFFFF         lea edx, dword ptr [ebp+FFFFFE20]
:00513048 8B45FC               mov eax, dword ptr [ebp-04]
:0051304B 8B80F0020000         mov eax, dword ptr [eax+000002F0]
:00513051 E81EF9F3FF           call 00452974    取识别码
:00513056 8B9520FEFFFF         mov edx, dword ptr [ebp+FFFFFE20]
:0051305C 8B45F8               mov eax, dword ptr [ebp-08]
:0051305F 59                   pop ecx
:00513060 E86363FEFF           call 004F93C8    识别码的计算
:00513065 66BB3828             mov bx, 2838
:00513069 8D9518FEFFFF         lea edx, dword ptr [ebp+FFFFFE18]
:0051306F 8B45FC               mov eax, dword ptr [ebp-04]
:00513072 8B80F4020000         mov eax, dword ptr [eax+000002F4]
:00513078 E8F7F8F3FF           call 00452974    取注册码
:0051307D 8B8518FEFFFF         mov eax, dword ptr [ebp+FFFFFE18]
:00513083 8D951CFEFFFF         lea edx, dword ptr [ebp+FFFFFE1C]
:00513089 E86A65EFFF           call 004095F8    注册码首尾与 20 比较
:0051308E 8B951CFEFFFF         mov edx, dword ptr [ebp+FFFFFE1C]
:00513094 8B45F8               mov eax, dword ptr [ebp-08]
:00513097 E8B856FEFF           call 004F8754

将输入的注册码与下列内置的字符串比较，字符串详见下面，如果没有相同的，eax=0，否则为 1
:0051309C 84C0                 test al, al
:0051309E 0F8476010000         je 0051321A    跳往注册错误的地方
:005130A4 8D9510FEFFFF         lea edx, dword ptr [ebp+FFFFFE10]
:005130AA 8B45FC               mov eax, dword ptr [ebp-04]
:005130AD 8B80F4020000         mov eax, dword ptr [eax+000002F4]
:005130B3 E8BCF8F3FF           call 00452974    取注册码
:005130B8 8B8510FEFFFF         mov eax, dword ptr [ebp+FFFFFE10]
:005130BE 8D9514FEFFFF         lea edx, dword ptr [ebp+FFFFFE14]
:005130C4 E82F65EFFF           call 004095F8
:005130C9 8B8514FEFFFF         mov eax, dword ptr [ebp+FFFFFE14]  输入的注册码
:005130CF 8B55F4               mov edx, dword ptr [ebp-0C]    识别码计算值
:005130D2 E8ED1FEFFF           call 004050C4        比较是否相等
:005130D7 0F853D010000         jne 0051321A        跳往注册错误的地方
:005130DD 0FB7C3               movzx eax, bx
:005130E0 89850CFEFFFF         mov dword ptr [ebp+FFFFFE0C], eax
:005130E6 DB850CFEFFFF         fild dword ptr [ebp+FFFFFE0C]    装载整数 ebp-1f4 到 st(0)
:005130EC DB2DBC325100         fld tbyte ptr [005132BC]    装载实数 5132BC 到 st(0)
:005130F2 DED9                 fcompp        实数比较，两次出栈
:005130F4 DFE0                 fstsw ax        保存状态字的值到 AX
:005130F6 9E                   sahf          把 AH 内容装入标志寄存器
:005130F7 0F851D010000         jne 0051321A        跳往注册错误的地方
:005130FD 33C0                 xor eax, eax
:005130FF 55                   push ebp
:00513100 68B7315100           push 005131B7
:00513105 64FF30               push dword ptr fs:[eax]
:00513108 648920               mov dword ptr fs:[eax], esp
:0051310B 33C0                 xor eax, eax
:0051310D 55                   push ebp
:0051310E 688F315100           push 0051318F
:00513113 64FF30               push dword ptr fs:[eax]
:00513116 648920               mov dword ptr fs:[eax], esp
:00513119 8D9504FEFFFF         lea edx, dword ptr [ebp+FFFFFE04]
:0051311F A130F35500           mov eax, dword ptr [0055F330]
:00513124 8B00                 mov eax, dword ptr [eax]
:00513126 E81D11F6FF           call 00474248
```
