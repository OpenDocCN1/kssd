
# [原创]Enigma Protector脱壳

## 文章信息
【文章标题】: `Enigma Protector`脱壳
【文章作者】: `Smoke`
【编写语言】: `delphi7`
【操作平台】: `windows xp sp3`
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

`Enigma Protector`脱壳

`OllyDBG`插件 `PhantOm`

勾住protect `DRx` 选项 来方便下断点

设置完毕后重新载入`OllyDBG`

```
0041A3DA >  55              push    ebp
0041A3DB    8BEC            mov     ebp, esp
0041A3DD    83C4 F0         add     esp, -10
0041A3E0    B8 00104000     mov     eax, 00401000
0041A3E5    E8 01000000     call    0041A3EB
0041A3EA    9A 83C4108B E55>call    far 5DE5:8B10C483
0041A3F1  - E9 A2910900     jmp     004B3598
0041A3F6  ^ E0 D9           loopdne short 0041A3D1
```

用二次断点

`Ctrl+G` 输入 `VirtualAlloc`

```
7C809AE1 > 8BFF mov edi, edi
7C809AE3 55 push ebp
7C809AE4 8BEC mov ebp, esp
7C809AE6 FF75 14 push dword ptr [ebp+14]
7C809AE9 FF75 10 push dword ptr [ebp+10]
7C809AEC FF75 0C push dword ptr [ebp+C]
7C809AEF FF75 08 push dword ptr [ebp+8]
7C809AF2 6A FF push -1
7C809AF4 E8 09000000 call VirtualAllocEx //F2下断 F9运行 断下之后在到数据窗口 到401000 处下硬件写入断点 找解码处
7C809AF9 5D pop ebp
```

下面应该就是解码过程了

```
005502AF FC cld
005502B0 B2 80 mov dl, 80
005502B2 8A06 mov al, byte ptr [esi]
005502B4 46 inc esi
005502B5 8807 mov byte ptr [edi], al
005502B7 47 inc edi ; UnpackMe.00401000
005502B8 00D2 add dl, dl
005502BA 75 05 jnz short 005502C1
005502BC 8A16 mov dl, byte ptr [esi]
005502BE 46 inc esi
005502BF 10D2 adc dl, dl
005502C1 ^ 73 EF jnb short 005502B2
```

往下拉 来到

```
005503C8 F3:A4 rep movs byte ptr es:[edi], byte ptr>
005503CA 5E pop esi
005503CB ^ E9 E8FEFFFF jmp 005502B8
005503D0 5D pop ebp
005503D1 2B7D 40 sub edi, dword ptr [ebp+40]
005503D4 897C24 1C mov dword ptr [esp+1C], edi
005503D8 61 popad
005503D9 5D pop ebp
005503DA C3 retn //F2下断 F9运行 发现数据窗口已经解码完了
```

接下来就来找`OEP`

跟随到`00401000` 搜索特征码 `A3??????00A1??????00A3??????0033C0A3??????0033C0A3??????00` (`delphi`程序的第一个`call`)

来到以下地方

```
00405BC8 53 push ebx 第一个call
00405BC9 8BD8 mov ebx, eax
00405BCB 33C0 xor eax, eax
00405BCD A3 9CD04400 mov dword ptr [44D09C], eax
00405BD2 6A 00 push 0
00405BD4 E8 FF4A0000 call 0040A6D8 第一个函数
00405BD9 A3 64F64400 mov dword ptr [44F664], eax
00405BDE A1 64F64400 mov eax, dword ptr [44F664]
00405BE3 A3 A8D04400 mov dword ptr [44D0A8], eax
00405BE8 33C0 xor eax, eax
00405BEA A3 ACD04400 mov dword ptr [44D0AC], eax
00405BEF 33C0 xor eax, eax
00405BF1 A3 B0D04400 mov dword ptr [44D0B0], eax
```

`ALT+M`在

Memory map, 条目 22

地址=`00401000`

大小=`0004C000` (311296.)

属主=`UnpackMe 00400000`

区段=

包含=`SFX`,代码

类型=`Imag 01001002`

访问=`R`

初始访问=`RWE`

下内存写入断点 `F9`运行

删除内存断点 来到

```
0055026B 294C0F 01 sub dword ptr [edi+ecx+1], ecx
0055026F 83C1 05 add ecx, 5
00550272 83E8 04 sub eax, 4
00550275 ^ EB D7 jmp short 0055024E
00550277 29540F 02 sub dword ptr [edi+ecx+2], edx
0055027B 83C1 06 add ecx, 6
0055027E 83EA 04 sub edx, 4
00550281 83E8 05 sub eax, 5
00550284 ^ EB C8 jmp short 0055024E
00550286 61 popad
00550287 5D pop ebp
00550288 C2 0800 retn 8
0055028B 90 nop
00550281 83E8 05 sub eax, 5
00550284 ^ EB C8 jmp short 0055024E
00550286 61 popad
00550287 5D pop ebp
00550288 C2 0800 retn 8 //F2下断 F9运行
```

继续来到

```
00405BC8 53 push ebx
00405BC9 8BD8 mov ebx, eax
00405BCB 33C0 xor eax, eax
00405BCD A3 9CD04400 mov dword ptr [44D09C], eax
00405BD2 6A 00 push 0
00405BD4 E8 2BFFFFFF call 00405B04 //F7跟进
00405BD9 A3 64F64400 mov dword ptr [44F664], eax
00405BDE A1 64F64400 mov eax, dword ptr [44F664]
00405BE3 A3 A8D04400 mov dword ptr [44D0A8], eax
```

来到

```
00405B04 FF25 E4014500 jmp dword ptr [4501E4]
00405B0A 8BC0 mov eax, eax
00405B0C FF25 E0014500 jmp dword ptr [4501E0]
00405B12 8BC0 mov eax, eax
00405B14 FF25 DC014500 jmp dword ptr [4501DC]
00405B1A 8BC0 mov eax, eax
00405B1C FF25 D8014500 jmp dword ptr [4501D8]
```

发现已经解密完毕 不再是开始那样

返回`EIP`

搜索特征码 `ctrl+F`

`mov dword ptr ds:[edx+eax*4+4],ecx`

把找到的特征码全部下硬件执行断点

搜索完及下完硬件点段后 直接`F9`

把找到的地方`nop`掉

```
0054C9EA 90 nop
0054C9EB 90 nop
0054C9EC 90 nop
0054C9ED 90 nop
```

删掉`0054C9EA`硬件执行断点

```
0054C9F8 33C0 xor eax, eax //F2下断 F9运行
0054C9FA 5A pop edx
0054C9FB 59 pop ecx
0054C9FC 59 pop ecx
```

撤消`0054C9EA`修改后的代码 并下硬件执行断点

来到`OEP`发现有些函数已经解密了

重新载入`OllyDBG`

硬件执行断点都在

`F9`运行 来到

```
0054C9EA 894C82 04 mov dword ptr [edx+eax*4+4], ecx
0054C9EE 47 inc edi
0054C9EF FF4D D0 dec dword ptr [ebp-30]
```

先`nop`填充掉

```
0054C9EA 90 nop
0054C9EB 90 nop
0054C9EC 90 nop
0054C9ED 90 nop
```

删除硬件断点

```
0054C9F8    33C0            xor     eax, eax  //F2下断 F9运行
```

在`0054C9F8`断下后 撤销`0054C9EA`修改后的代码 重新下硬件执行断点 再`F9`

来到

```
0054BDBD 894C82 04 mov dword ptr [edx+eax*4+4], ecx ; UnpackMe.0054E204
0054BDC1 83C3 04 add ebx, 4
0054BDC4 66:8B3B mov di, word ptr [ebx]
```

`nop`掉

```
0054BDBD 90 nop
0054BDBE 90 nop
0054BDBF 90 nop
0054BDC0 90 nop
```

删除硬件断点 `0054BDBD`

往下拉 来到

```
0054BE30 83F8 FF cmp eax, -1
0054BE33 ^ 0F85 24FDFFFF jnz 0054BB5D
0054BE39 33C0 xor eax, eax //F2下断 F9运行
0054BE3B 5A pop edx
```

断下之后取消断点 撤销修改的地方`0054BDBD` 并下硬件执行断点 再`F9`

来到

```
0054C9EA 894C82 04 mov dword ptr [edx+eax*4+4], ecx
```

继续重复上一次操作

取消掉硬件执行断点 `004EF086`  `0054BC97`

跟随到`00405BC8` 下个硬件执行断点 `F9`运行

来到

```
00405BC8 53 push ebx
00405BC9 8BD8 mov ebx, eax
00405BCB 33C0 xor eax, eax
00405BCD A3 9CD04400 mov dword ptr [44D09C], eax
00405BD2 6A 00 push 0
00405BD4 E8 2BFFFFFF call 00405B04 ; jmp 到 kernel32.GetModuleHandleA //F7跟进
00405BD9 A3 64F64400 mov dword ptr [44F664], eax
00405BDE A1 64F64400 mov eax, dword ptr [44F664]
00405AD6 /EB 20 jmp short 00405AF8
00405AD8 |E8 9FB7FFFF call 0040127C ; jmp 到 kernel32.GetThreadLocale
00405ADD |E8 86FEFFFF call 00405968
00405AE2 |A3 BCF54400 mov dword ptr [44F5BC], eax
00405AE7 |EB 0F jmp short 00405AF8
00405AE9 |E8 8EB7FFFF call 0040127C ; jmp 到 kernel32.GetThreadLocale
00405AEE |E8 75FEFFFF call 00405968
00405AF3 |A3 BCF54400 mov dword ptr [44F5BC], eax
00405AF8 \E8 FFB7FFFF call 004012FC ; jmp 到 kernel32.GetCurrentThreadId
00405AFD A3 30F04400 mov dword ptr [44F030], eax
00405B02 C3 retn
00405B03 90 nop
00405B04 - FF25 646FC600 jmp dword ptr [C66F64] ; kernel32.GetModuleHandleA
00405B0A 8BC0 mov eax, eax
00405B0C - FF25 586FC600 jmp dword ptr [C66F58] ; kernel32.LocalAlloc
00405B12 8BC0 mov eax, eax
00405B14 - FF25 4C6FC600 jmp dword ptr [C66F4C] ; kernel32.TlsGetValue
00405B1A 8BC0 mov eax, eax
00405B1C - FF25 406FC600 jmp dword ptr [C66F40] ; kernel32.TlsSetValue
```

看到函数已经解密了

`CPU`窗口跟随到`00C37CA0`

来到

```
00C37C86 0000 add byte ptr [eax], al
00C37C88 0000 add byte ptr [eax], al
00C37C8A 0000 add byte ptr [eax], al
00C37C8C BA 00000055 mov edx, 55000000
00C37C91 8BEC mov ebp, esp
00C37C93 83C4 F0 add esp, -10
00C37C96 B8 B8C84400 mov eax, 44C8B8
00C37C9B E8 28DF7CFF call UnpackMe.00405BC8
00C37CA0 8B05 B8DF4400 mov eax, dword ptr [44DFB8] ; UnpackMe.0044FBB0
00C37CA6 8B40 00 mov eax, dword ptr [eax]
00C37CA9 E8 A23481FF call UnpackMe.0044B150
00C37CAE 8B0D 94E04400 mov ecx, dword ptr [44E094] ; UnpackMe.0044FBD0
00C37C8C BA 00000055 mov edx, 55000000 //nop掉
00C37C90 90 nop //撤销修改代码
```

复制保存

```
push ebp
mov ebp, esp
add esp, -10
mov eax, 44C8B8
call UnpackMe.00405BC8
mov eax, dword ptr [44DFB8]
mov eax, dword ptr [eax]
call UnpackMe.0044B150
mov ecx, dword ptr [44E094]
mov eax, dword ptr [44DFB8]
mov eax, dword ptr [eax]
mov edx, dword ptr [44C6F0]
call UnpackMe.0044B168
mov eax, dword ptr [44DFB8]
mov eax, dword ptr [eax]
call UnpackMe.0044B1E8
call UnpackMe.00403D20
lea eax, dword ptr [eax]
```

返回到`eip`

```
00405BC8 53 push ebx
00405BC9 8BD8 mov ebx, eax
00405BCB 33C0 xor eax, eax
00405BCD A3 9CD04400 mov dword ptr [44D09C], eax
00405BD2 6A 00 push 0
00405BD4 E8 2BFFFFFF call 00405B04 ; jmp 到 kernel32.GetModuleHandleA
00405BD9 A3 64F64400 mov dword ptr [44F664], eax
00405BDE A1 64F66400 mov eax, dword ptr [44F664]
```

搜索 `558BEC33C05568??????0064FF3064892033C05A595964891068??????00C3E9` (`OEP`特征码)

来到

```
0044C890 55 push ebp
0044C891 8BEC mov ebp, esp
0044C893 33C0 xor eax, eax
0044C895 55 push ebp
0044C896 68 AFC84400 push 0044C8AF
0044C89B 64:FF30 push dword ptr fs:[eax]
0044C89E 64:8920 mov dword ptr fs:[eax], esp
```

继续搜索`90C84400`
