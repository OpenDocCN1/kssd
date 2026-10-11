
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



### 1. 主程序段反汇编

```assembly
:0051312B 8B8504FEFFFF            mov eax, dword ptr [ebp+FFFFFE04]  
:00513131 8D9508FEFFFF            lea edx, dword ptr [ebp+FFFFFE08]  
:00513137 E8E06EEFFF              call 0040A01C  
:0051313C 8D8508FEFFFF            lea eax, dword ptr [ebp+FFFFFE08]  

* Possible StringData Ref from Code Obj ->"\\key.kc"  

  |  

:00513142 BAD0325100              mov edx, 005132D0  
:00513147 E83C1EEFFF              call 00404F88  
:0051314C 8B9508FEFFFF            mov edx, dword ptr [ebp+FFFFFE08]  
:00513152 8D8528FEFFFF            lea eax, dword ptr [ebp+FFFFFE28]  
:00513158 E817FEEEFF              call 00402F74  
:0051315D 8D8528FEFFFF            lea eax, dword ptr [ebp+FFFFFE28]  
:00513163 E89CFBEEFF              call 00402D04  
:00513168 E813F8EEFF              call 00402980  
:0051316D 8B55F4                  mov edx, dword ptr [ebp-0C]  
:00513170 8D8528FEFFFF            lea eax, dword ptr [ebp+FFFFFE28]  
:00513176 E81922EFFF              call 00405394  
:0051317B E8B406EFFF              call 00403834  
:00513180 E8FBF7EEFF              call 00402980  
:00513185 33C0                    xor eax, eax  
:00513187 5A                      pop edx  
:00513188 59                      pop ecx  
:00513189 59                      pop ecx  
:0051318A 648910                  mov dword ptr fs:[eax], edx  
:0051318D EB0A                    jmp 00513199  
:0051318F E94411EFFF              jmp 004042D8  
:00513194 E86B15EFFF              call 00404704  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0051318D(U)  

|  

:00513199 33C0                    xor eax, eax  
:0051319B 5A                      pop edx  
:0051319C 59                      pop ecx  
:0051319D 59                      pop ecx  
:0051319E 648910                  mov dword ptr fs:[eax], edx  
:005131A1 68BE315100              push 005131BE  

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:005131BC(U)  

|  

:005131A6 8D8528FEFFFF            lea eax, dword ptr [ebp+FFFFFE28]  
:005131AC E83FFFEEFF              call 004030F0        写入识别码计算值  
:005131B1 E8CAF7EEFF              call 00402980  
:005131B6 C3                      ret  
:005131B7 E9D013EFFF              jmp 0040458C  
:005131BC EBE8                    jmp 005131A6  
:005131BE 8B45FC                  mov eax, dword ptr [ebp-04]  
:005131C1 E81A61F4FF              call 004592E0  

* Possible StringData Ref from Code Obj ->"注册成功，谢谢使用！确定后重新启动程序"  

                                      |  

:005131C6 BAD8325100              mov edx, 005132D8  
:005131CB B940000000              mov ecx, 00000040  
:005131D0 E89B50FEFF              call 004F8270  
:005131D5 A130F35500              mov eax, dword ptr [0055F330]  
:005131DA 8B00                    mov eax, dword ptr [eax]  
:005131DC E87B0BF6FF              call 00473D5C  
:005131E1 6A03                    push 00000003  
:005131E3 6A00                    push 00000000  
:005131E5 6A00                    push 00000000  
:005131E7 8D9500FEFFFF            lea edx, dword ptr [ebp+FFFFFE00]  
:005131ED A130F35500              mov eax, dword ptr [0055F330]  
:005131F2 8B00                    mov eax, dword ptr [eax]  
:005131F4 E84F10F6FF              call 00474248  
:005131F9 8B8500FEFFFF            mov eax, dword ptr [ebp+FFFFFE00]  
:005131FF E8741FEFFF              call 00405178  
:00513204 50                      push eax  

* Possible StringData Ref from Code Obj ->"open"  

                                      |  

:00513205 6800335100              push 00513300  
:0051320A 8B45FC                  mov eax, dword ptr [ebp-04]  
:0051320D E8CE60F4FF              call 004592E0  
:00513212 50                      push eax  

* Reference To: shell32.ShellExecuteA, Ord:0171h  

                                      |  

:00513213 E82CE9F2FF              Call 00441B44  
:00513218 EB17                    jmp 00513231  

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:0051309E(C), :005130D7(C), :005130F7(C)  

|  

:0051321A 8B45FC                  mov eax, dword ptr [ebp-04]  
:0051321D E8BE60F4FF              call 004592E0  

* Possible StringData Ref from Code Obj ->"注册码不正确"  

                                      |  

:00513222 BAAC325100              mov edx, 005132AC  
:00513227 B930000000              mov ecx, 00000030  
:0051322C E83F50FEFF              call 004F8270  
```

### 4.2 对识别码的计算

#### 主调用入口

```assembly
:00513060 E86363FEFF    call 004F93C8   
```

#### 子程序 `004F93C8`

```assembly
004F93C8  push ebp  
004F93C9  mov ebp,esp  
004F93CB  push ecx  
004F93CC  mov ecx,7  
004F93D1  /push 0  
004F93D3  |push 0  
004F93D5  |dec ecx  
004F93D6  \\jnz short UPNetGet.004F93D1  
004F93D8  push ecx  
004F93D9  xchg dword ptr ss:[ebp-4],ecx  
004F93DC  push ebx  
004F93DD  push esi  
004F93DE  push edi  
004F93DF  mov word ptr ss:[ebp-6],cx  
004F93E3  mov dword ptr ss:[ebp-4],edx  
004F93E6  mov eax,dword ptr ss:[ebp-4]  
004F93E9  call UPNetGet.00405168  
004F93EE  xor eax,eax  
004F93F0  push ebp  
004F93F1  push UPNetGet.004F95DE  
004F93F6  push dword ptr fs:[eax]  
004F93F9  mov dword ptr fs:[eax],esp  
004F93FC  lea eax,dword ptr ss:[ebp-C]  
004F93FF  call UPNetGet.00404CC8  
004F9404  lea eax,dword ptr ss:[ebp-10]  
004F9407  call UPNetGet.00404CC8  
004F940C  mov eax,dword ptr ss:[ebp-4]  
004F940F  call UPNetGet.00404F80  
004F9414  mov esi,eax  
004F9416  dec esi  
004F9417  test esi,esi  
004F9419  jl short UPNetGet.004F9463  
004F941B  inc esi  
004F941C  xor edi,edi  
004F941E  /mov eax,dword ptr ss:[ebp-4]  
004F9421  |mov bl,byte ptr ds:[eax+edi-1]  
004F9425  |mov eax,ebx  
004F9427  |add al,0BF  
004F9429  |sub al,1A  
004F942B  |jnb short UPNetGet.004F9447  
004F942D  |lea eax,dword ptr ss:[ebp-14]  
004F9430  |mov edx,dword ptr ss:[ebp-4]  
004F9433  |mov edx,ebx  
004F9435  |call UPNetGet.00404EA8  
004F943A  |mov edx,dword ptr ss:[ebp-14]  
004F943D  |lea eax,dword ptr ss:[ebp-C]  
004F9440  |call UPNetGet.00404F88  
004F9445  |jmp short UPNetGet.004F945F  
004F9447  |lea eax,dword ptr ss:[ebp-18]  
004F944A  |mov edx,dword ptr ss:[ebp-4]  
004F944D  |mov edx,ebx  
004F944F  |call UPNetGet.00404EA8  
004F9454  |mov edx,dword ptr ss:[ebp-18]  
004F9457  |lea eax,dword ptr ss:[ebp-10]  
004F945A  |call UPNetGet.00404F88  
004F945F  |inc edi  
004F9460  |dec esi  
004F9461  \\jnz short UPNetGet.004F941E  
004F9463  mov ax,word ptr ss:[ebp-6]  
004F9467  sub ax,1  
004F946B  jb short UPNetGet.004F9486  
004F946D  je short UPNetGet.004F94D7  
004F946F  dec ax  
004F9472  je UPNetGet.004F9528  
004F9478  dec ax  
004F947B  je UPNetGet.004F956B  
004F9481  jmp UPNetGet.004F95AE  
004F9486  lea edx,dword ptr ss:[ebp-1C]  
004F9489  mov eax,dword ptr ss:[ebp-C]  
004F948C  call UPNetGet.004093A8  
004F9491  mov ecx,dword ptr ss:[ebp-1C]  
004F9494  lea eax,dword ptr ss:[ebp-C]  
004F9497  mov edx,UPNetGet.004F95F8          ;  ASCII "EN"  
004F949C  call UPNetGet.00404FCC  
004F94A1  lea eax,dword ptr ss:[ebp-24]  
004F94A4  push eax  
004F94A5  xor ecx,ecx  
004F94A7  mov edx,dword ptr ss:[ebp-C]  
004F94AA  mov eax,dword ptr ss:[ebp-4]  
004F94AD  call UPNetGet.004C315C  
004F94B2  mov eax,dword ptr ss:[ebp-24]  
004F94B5  lea ecx,dword ptr ss:[ebp-20]  
004F94B8  mov edx,1C  
004F94BD  call UPNetGet.0044C3AC  
004F94C2  mov ecx,dword ptr ss:[ebp-20]  
004F94C5  mov eax,dword ptr ss:[ebp+8]  
004F94C8  mov edx,UPNetGet.004F9604          ;  ASCII "EN-"  
004F94CD  call UPNetGet.00404FCC  
004F94D2  jmp UPNetGet.004F95BB  
004F94D7  lea edx,dword ptr ss:[ebp-28]  
004F94DA  mov eax,dword ptr ss:[ebp-C]  
004F94DD  call UPNetGet.004093E4  
004F94E2  mov ecx,dword ptr ss:[ebp-28]  
004F94E5  lea eax,dword ptr ss:[ebp-C]  
004F94E8  mov edx,UPNetGet.004F9610          ;  ASCII "pr"  
004F94ED  call UPNetGet.00404FCC  
```



# 4.3 程序调用分析

## 汇编指令序列 1

```asm
004F94F2  lea eax, dword ptr ss:[ebp-30]  
004F94F5  push eax  
004F94F6  xor ecx, ecx  
004F94F8  mov edx, dword ptr ss:[ebp-C]  
004F94FB  mov eax, dword ptr ss:[ebp-4]  
004F94FE  call UPNetGet.004C315C  
004F9503  mov eax, dword ptr ss:[ebp-30]  
004F9506  lea ecx, dword ptr ss:[ebp-2C]  
004F9509  mov edx, 1C  
004F950E  call UPNetGet.0044C3AC  
004F9513  mov ecx, dword ptr ss:[ebp-2C]  
004F9516  mov eax, dword ptr ss:[ebp+8]  
004F9519  mov edx, UPNetGet.004F961C        ;  ASCII "pr-"  
004F951E  call UPNetGet.00404FCC  
004F9523  jmp UPNetGet.004F95BB  
004F9528  lea eax, dword ptr ss:[ebp-C]  
004F952B  mov ecx, dword ptr ss:[ebp-10]  
004F952E  mov edx, UPNetGet.004F9628        ;  ASCII "pe"  
004F9533  call UPNetGet.00404FCC  
004F9538  lea eax, dword ptr ss:[ebp-38]  
004F953B  push eax  
004F953C  xor ecx, ecx  
004F953E  mov edx, dword ptr ss:[ebp-C]  
004F9541  mov eax, dword ptr ss:[ebp-4]  
004F9544  call UPNetGet.004C315C  
004F9549  mov eax, dword ptr ss:[ebp-38]  
004F954C  lea ecx, dword ptr ss:[ebp-34]  
004F954F  mov edx, 1C  
004F9554  call UPNetGet.0044C3AC  
004F9559  mov ecx, dword ptr ss:[ebp-34]  
004F955C  mov eax, dword ptr ss:[ebp+8]  
004F955F  mov edx, UPNetGet.004F9634        ;  ASCII "pe-"  
004F9564  call UPNetGet.00404FCC  
004F9569  jmp short UPNetGet.004F95BB  
004F956B  lea eax, dword ptr ss:[ebp-C]  
004F956E  mov ecx, dword ptr ss:[ebp-10]  
004F9571  mov edx, UPNetGet.004F9640        ;  ASCII "ex"  
004F9576  call UPNetGet.00404FCC  
004F957B  lea eax, dword ptr ss:[ebp-40]  
004F957E  push eax  
004F957F  xor ecx, ecx  
004F9581  mov edx, dword ptr ss:[ebp-C]  
004F9584  mov eax, dword ptr ss:[ebp-4]  
004F9587  call UPNetGet.004C315C  
004F958C  mov eax, dword ptr ss:[ebp-40]  
004F958F  lea ecx, dword ptr ss:[ebp-3C]  
004F9592  mov edx, 1C  
004F9597  call UPNetGet.0044C3AC  
004F959C  mov ecx, dword ptr ss:[ebp-3C]  
004F959F  mov eax, dword ptr ss:[ebp+8]  
004F95A2  mov edx, UPNetGet.004F964C        ;  ASCII "ex-"  
004F95A7  call UPNetGet.00404FCC  
004F95AC  jmp short UPNetGet.004F95BB  
004F95AE  mov eax, dword ptr ss:[ebp+8]  
004F95B1  mov edx, UPNetGet.004F9658        ;  ASCII "NoneReg"  
004F95B6  call UPNetGet.00404D1C  
004F95BB  xor eax, eax  
004F95BD  pop edx  
004F95BE  pop ecx  
004F95BF  pop ecx  
004F95C0  mov dword ptr fs:[eax], edx  
004F95C3  push UPNetGet.004F95E5  
004F95C8  lea eax, dword ptr ss:[ebp-40]  
004F95CB  mov edx, 0E  
004F95D0  call UPNetGet.00404CEC  
004F95D5  lea eax, dword ptr ss:[ebp-4]  
004F95D8  call UPNetGet.00404CC8  
004F95DD  retn  
004F95DE  jmp UPNetGet.0040458C  
004F95E3  jmp short UPNetGet.004F95C8  
004F95E5  pop edi  
004F95E6  pop esi  
004F95E7  pop ebx  
004F95E8  mov esp, ebp  
004F95EA  pop ebp  
004F95EB  retn 4  
```

## 调用指令

在程序 `00513097` 处，指令 `E8B856FEFF` 调用函数 `004F8754`。

## 注册码校验逻辑

将输入的注册码与下列内置的字符串比较，若相等则程序继续，否则注册错误。

```asm
004F8786  mov edx, UPNetGet.004F8AC0        ;  ASCII "EN-D1D9673F0A6C4A08A597BA3282CB"  
004F8793  mov edx, UPNetGet.004F8AE8        ;  ASCII "pr-4721D414F70B5ACDE4BC8B63D05B"  
004F87A0  mov edx, UPNetGet.004F8B10        ;  ASCII "pe-E5168778342086A114405B138529"  
004F87AD  mov edx, UPNetGet.004F8B38        ;  ASCII "pe-7DAE3433BD7D102F93AB2FB98944"  
004F87BA  mov edx, UPNetGet.004F8B60        ;  ASCII "EN-BABF0099063E9362F16E7DDA3265"  
004F87C7  mov edx, UPNetGet.004F8B88        ;  ASCII "pe-AF24A6F8AAD69CCA956C168B705F"  
004F87D4  mov edx, UPNetGet.004F8BB0        ;  ASCII "ex-33F8430B2B150255E115BB41D808"  
004F87E1  mov edx, UPNetGet.004F8BD8        ;  ASCII "ex-7093DFD5D13EF78B7C569053F8A8"  
004F87EE  mov edx, UPNetGet.004F8C00        ;  ASCII "ex-ED1DA86A88B16EBC55B70758AE8F"  
004F87FB  mov edx, UPNetGet.004F8C28        ;  ASCII "EN-655F3971F512305E73D0DDB1DB4A"  
004F8808  mov edx, UPNetGet.004F8C50        ;  ASCII "EN-C1C27E3061AF6E57AF8966DE8C1E"  
004F8815  mov edx, UPNetGet.004F8C78        ;  ASCII "pe-A0FCF4BE1B68EE0718531A59BE41"  
004F8822  mov edx, UPNetGet.004F8CA0        ;  ASCII "pe-7F9AD364C665B37A302111748B0F"  
004F882F  mov edx, UPNetGet.004F8CC8        ;  ASCII "pe-1C0303AB77B7A4B9A6690987C5AF"  
004F883C  mov edx, UPNetGet.004F8CF0        ;  ASCII "pr-587ED034328E75DE5B22C703C859"  
004F8849  mov edx, UPNetGet.004F8D18        ;  ASCII "pe-F6EF8235BFA130FE560DBB85746A"  
004F8856  mov edx, UPNetGet.004F8D40        ;  ASCII "EN-3EFA729C76F0EF0578191E221EFA"  
004F8863  mov edx, UPNetGet.004F8D68        ;  ASCII "EN-D0BF945CB4E303BBE2765438C22E"  
004F8870  mov edx, UPNetGet.004F8D90        ;  ASCII "EN-5EAD3D30B8D113604C229A2F852C"  
004F887D  mov edx, UPNetGet.004F8DB8        ;  ASCII "ex-D202693E59B810CE6AF026864345"  
004F888A  mov edx, UPNetGet.004F8DE0        ;  ASCII "ex-78C12A1F987AD72FF021DA6CFFBA"  
004F8897  mov edx, UPNetGet.004F8E08        ;  ASCII "EN-6C6DF3AB83A652C30CF2D9806FBB"  
004F88A4  mov edx, UPNetGet.004F8E30        ;  ASCII "pr-74F363811310B8284D3E99F35837"  
004F88B1  mov edx, UPNetGet.004F8E58        ;  ASCII "pe-46FA89DF1CF47B1E05C943BEA218"  
004F88BE  mov edx, UPNetGet.004F8E80        ;  ASCII "EN-6A3F1726740C84C4E44B9FBFF5AC"  
004F88CB  mov edx, UPNetGet.004F8EA8        ;  ASCII "EN-685B7C1B91DD1AD26D36AEAA8F1B"  
004F88D8  mov edx, UPNetGet.004F8ED0        ;  ASCII "pe-78B6D0BAD93AA6D998C894A7BD8B"  
004F88E5  mov edx, UPNetGet.004F8EF8        ;  ASCII "EN-63E9D81888D61C411F2C453C061A"  
004F88F2  mov edx, UPNetGet.004F8F20        ;  ASCII "ex-6C3E1A8EA7B1EE6A370C5B39A168"  
004F88FF  mov edx, UPNetGet.004F8F48        ;  ASCII "EN-7C5AD967A26C2E92F2FD96CDA8CA"  
004F890C  mov edx, UPNetGet.004F8F70        ;  ASCII "EN-2E863D911FB55BA700E1AB6E4D33"  
004F8919  mov edx, UPNetGet.004F8F98        ;  ASCII "EN-71B93F8FB025406EF9681A1ABAAA"  
004F8926  mov edx, UPNetGet.004F8FC0        ;  ASCII "ex-75B3CAA328EE9F91217285038AED"  
004F8933  mov edx, UPNetGet.004F8FE8        ;  ASCII "ex-336931F283D761D3F0AB833FF7B8"  
004F8940  mov edx, UPNetGet.004F9010        ;  ASCII "ex-C6E6069B9FBF6AB1A6C0121D6D7C"  
004F894D  mov edx, UPNetGet.004F9038        ;  ASCII "ex-1E8354E83118422E84BA31322306"  
004F895A  mov edx, UPNetGet.004F9060        ;  ASCII "pe-1F26A3B4C0E1A4F5976560FB6F3C"  
004F8967  mov edx, UPNetGet.004F9088        ;  ASCII "pe-ACABA6CCC8A681484E9D0A3C842F"  
004F8974  mov edx, UPNetGet.004F90B0        ;  ASCII "pe-7B767DC5149C42534334D5529378"  
004F8981  mov edx, UPNetGet.004F90D8        ;  ASCII "pr-04C5AA23CFDAEC5357F4283E9071"  
004F898E  mov edx, UPNetGet.004F9100        ;  ASCII "pr-19ABBF2B6BDCC6A612D538336A7E"  
004F899B  mov edx, UPNetGet.004F9128        ;  ASCII "ex-9A6D966BD9C342C5F1E22F5C6262"  
004F89A8  mov edx, UPNetGet.004F9150        ;  ASCII "EN-62A2461FDF00A36723C0747AA037"  
004F89B5  mov edx, UPNetGet.004F9178        ;  ASCII "EN-AFB901625AF858C64F59AC801BC7"  
004F89C2  mov edx, UPNetGet.004F91A0        ;  ASCII "EN-062019DC00DF10278EEBB39F09ED"  
004F89CF  mov edx, UPNetGet.004F91C8        ;  ASCII "pr-FD64576C94A38E223884481B62E7"  
004F89DC  mov edx, UPNetGet.004F91F0        ;  ASCII "pe-D830B6BFEA4CDC6B01EA4070B873"  
004F89E9  mov edx, UPNetGet.004F9218        ;  ASCII "ex-DB3361889A3A85DB00EE7A0EDF41"  
004F89F6  mov edx, UPNetGet.004F9240        ;  ASCII "ex-E624BFD9F641886D316B5B595205"  
004F8A03  mov edx, UPNetGet.004F9268        ;  ASCII "pe-E41AA909F57DD61B01D04FB6236C"  
004F8A10  mov edx, UPNetGet.004F9290        ;  ASCII "pe-81F4742BD88BA22407ECFDAC3EA5"  
004F8A1D  mov edx, UPNetGet.004F92B8        ;  ASCII "pe-58DBD86C0335E86495FC042106F2"  
004F8A2A  mov edx, UPNetGet.004F92E0        ;  ASCII "pe-AB40462CC3BB7E1DF3B279ADC91C"  
004F8A37  mov edx, UPNetGet.004F9308        ;  ASCII "EN-1CEF455F6A9586B90217AD353B45"  
004F8A44  mov edx, UPNetGet.004F9330        ;  ASCII "pe-29E2E51767B58D5F5DABCC227224"  
```



```
004F8A51   mov edx,UPNetGet.004F9358          ;  ASCII "pe-26B5C93D620AC37F6721E74C20EA"
004F8A5E   mov edx,UPNetGet.004F9380          ;  ASCII "EN-0A800A43438BEAEED3415216FDDC"
004F8A6B   mov edx,UPNetGet.004F93A8          ;  ASCII "EN-78105BEDAB07A8A455A8BB530B23"
```

### 4.4 小结

a. 对识别码进行计算：
- 企业版：`"EN-E95F79FD79870B62C45292A6AB2C"`；
- 专业版：`"pr-4721D414F70B5ACDE4BC8B63D05B"`；
- 个人版：`"pe-CA0FF709D551D98CB2EC4EACD0B2"`；
- 体验版：`"ex-A504D985613415A8D5F7B3E0209F"`。

b. 输入的注册码与程序内置的字符串（30 几个）比较，如果没有相同的就赋值 `eax=0`，然后跳到注册错误的地方，如果有相同的就继续；

c. 输入的注册码再与识别码计算值比较，如果相同的话就继续；

d. 注册码再经过浮点运算，如果不等的话完完；

e. 然后将注册码写入 `key.kc`。

### 4.5 推测

根据上面的分析，如果修改内置字符串为本机识别码计算值，那么程序到此应该成功注册。

若不是的话，就是取 `key.kc` 的信息然后与真正的注册码比较。

## 5. 爆破

首先修改内置字符串为本机识别码计算值，相应版本有限制；

然后修改 `005130F7 0F851D010000` `jne 0051321A`（浮点运算，不知道怎么搞），`nop` 掉。

然后输入相应版本识别码计算值，跳出注册成功的消息框。

呵呵，再运行程序，注册的地方变虚的，提示谢谢注册，但是左上角的程序名称还是有试用版本的提示，也不知道有没有功能限制。

## 6. 注册文件：`key.kc` 的跟踪

### 6.1 断点：`KERNEL.FindFirstFileA`

`F9` 运行，观察堆栈，得到 `key.kc` 的显示，返回程序：

```
00409E26   push eax                             ; |FileName = "E:\\网络信息采集大师\\key.kc"
00409E27   call <jmp.&kernel32.FindFirstFileA>  ; \FindFirstFileA
00409E2C   cmp eax,-1
00409E2F   je short UPNetGet.00409E65    没有所要文件就跳
```

如果存在注册文件 `key.kc`，那么赋值 `eax` 整数值，如果不存在 `key.kc`，那么赋值 `eax=-1`（也就是 `FFFFFFFF`）。

然后在内存搜索 `key.kc` 的内容，没有，看来这个函数只是确定是否有这么个文件存在而已。

### 6.2 断点：`KERNEL.CreateFileA`

```
00402E40   push 0                              ; /hTemplateFile = NULL
00402E42   push 80                             ; |Attributes = NORMAL
00402E47   push ecx                            ; |Mode = OPEN_EXISTING
00402E48   push 0                              ; |pSecurity = NULL
00402E4A   push edx                            ; |ShareMode = FILE_SHARE_READ
00402E4B   push eax                            ; |Access = GENERIC_READ
00402E4C   lea eax,dword ptr ds:[esi+48]      ; |
00402E4F   push eax                            ; |FileName= "E:\\网络信息采集大师\\key.kc"
00402E50   call <jmp.&kernel32.CreateFileA>   ; \CreateFileA
00402E55   cmp eax,-1        eax=300
00402E58   je UPNetGet.00402F66
```

如果存在注册文件 `key.kc`，那么赋值 `eax` 非零整数值，如果不存在，那么赋值 `eax=-1`（也就是 `FFFFFFFF`）。

返回主程序：

```
004F96E2   call UPNetGet.00402CF8    打开 key.kc 文件
004F96E7   call UPNetGet.00402980    
004F96EC   lea edx,dword ptr ss:[ebp-18]
004F96EF   lea eax,dword ptr ss:[ebp-1EC]
004F96F5   call UPNetGet.00403428    读取 key.kc 内容－注册码，保存于 ss:[ebp-18]
...
004F97AF   lea ecx,dword ptr ss:[ebp-1F4]
004F97B5   mov edx,2
004F97BA   mov eax,dword ptr ss:[ebp-18]    注册码赋值给 eax
004F97BD   call UPNetGet.0044C390    取得注册码的前两位－EN
004F97C2   mov eax,dword ptr ss:[ebp-1F4]
004F97C8   mov edx,UPNetGet.004F98E0          ;  ASCII "EN"
004F97CD   call UPNetGet.004050C4    注册码前两位与‘EN’比较
004F97D2   jnz short UPNetGet.004F97DA    不同就跳过，继续比较
004F97D4   mov word ptr ss:[ebp-1E],0    ‘相同值’给 ss:[ebp-1E]＝0
004F97DA   lea ecx,dword ptr ss:[ebp-1F8]
004F97E0   mov edx,2
004F97E5   mov eax,dword ptr ss:[ebp-18]
004F97E8   call UPNetGet.0044C390    取得注册码的前两位－EN
004F97ED   mov eax,dword ptr ss:[ebp-1F8]
004F97F3   mov edx,UPNetGet.004F98EC          ;  ASCII "pr"
004F97F8   call UPNetGet.004050C4    注册码前两位与‘pr’比较
004F97FD   jnz short UPNetGet.004F9805    不同就跳过，继续比较
004F97FF   mov word ptr ss:[ebp-1E],1    ‘相同值’给 ss:[ebp-1E]＝1
004F9805   lea ecx,dword ptr ss:[ebp-1FC]
004F980B   mov edx,2
004F9810   mov eax,dword ptr ss:[ebp-18]
004F9813   call UPNetGet.0044C390
004F9818   mov eax,dword ptr ss:[ebp-1FC]
004F981E   mov edx,UPNetGet.004F98F8          ;  ASCII "pe"
004F9823   call UPNetGet.004050C4
004F9828   jnz short UPNetGet.004F9830
004F982A   mov word ptr ss:[ebp-1E],2
004F9830   lea ecx,dword ptr ss:[ebp-200]
004F9836   mov edx,2
004F983B   mov eax,dword ptr ss:[ebp-18]
004F983E   call UPNetGet.0044C390
004F9843   mov eax,dword ptr ss:[ebp-200]
004F9849   mov edx,UPNetGet.004F9904          ;  ASCII "ex"
004F984E   call UPNetGet.004050C4
004F9853   jnz short UPNetGet.004F985B
004F9855   mov word ptr ss:[ebp-1E],3
004F985B   lea eax,dword ptr ss:[ebp-14]
004F985E   push eax
004F985F   mov cx,word ptr ss:[ebp-1E]    将‘相同值’赋给 cx（这个关系到识别码计算值）
004F9863   mov edx,dword ptr ss:[ebp-10]    堆栈 ss:[0012FDC0]= (ASCII "WD-WCAD12522066")
004F9866   mov eax,dword ptr ss:[ebp-4]
004F9869   call UPNetGet.004F93C8    识别码的计算（见上面的 4.2 分析），值保存于 ss:[ebp-14]
004F986E   mov eax,dword ptr ss:[ebp-14]    识别码计算值＝ss:[ebp-14]
004F9871   mov edx,dword ptr ss:[ebp-18]    注册码＝ ss:[ebp-18]
004F9874   call UPNetGet.004050C4    识别码计算值与注册码比较
004F9879   jnz short UPNetGet.004F9891    不等就跳走，over
004F987B   mov edx,dword ptr ss:[ebp-18]
004F987E   mov eax,dword ptr ss:[ebp-4]
004F9881   call UPNetGet.004F8754    注册码与内置字符串（见上面 4.3 所列）比较
004F9886   test al,al        没有相同的就赋值 eax=0，有就为 1
004F9888   je short UPNetGet.004F9891    没有相同的就跳走，over
```

### 6.3 小结：假设要得到注册版

a. 程序先判断有没有 `key.kc` 注册文件；

b. 然后读取 `key.kc` 的注册码，与识别码（其实就是机器码）的计算值比较，相等就继续；

c. 然后注册码再与内置的字符串（其实就是内置的注册码）比较，有就成功注册。

## 7. 过程总结

### 7.1 程序运行后，输入注册码获得 `key.kc` 文件：

a. 对识别码进行计算：
- 企业版：`"EN-E95F79FD79870B62C45292A6AB2C"`；
- 专业版：`"pr-4721D414F70B5ACDE4BC8B63D05B"`；
- 个人版：`"pe-CA0FF709D551D98CB2EC4EACD0B2"`；
- 体验版：`"ex-A504D985613415A8D5F7B3E0209F"`。

b. 输入的注册码与程序内置的字符串（30 几个，其实就是注册码）比较，如果没有相同的就赋值 `eax=0`，然后跳到注册错误的地方，如果有相同的就继续，这个地方只能爆破或者修改内置字符串（也就是注册码）；

c. 输入的注册码再与识别码计算值比较，如果相同的话就继续；

d. 注册码再经过浮点运算，如果不等的话完完；

e. 最后将注册码写入 `key.kc`

### 7.2 程序再次运行过程中对注册码的验证：

a. 程序先判断有没有 `key.kc` 注册文件；

b. 然后读取 `key.kc` 的注册码，与识别码（其实就是机器码）的计算值比较，相等就继续；

c. 然后注册码再与内置的字符串（其实就是内置的注册码）比较，有就成功注册。

`4dd466d0e1e0cc123afbe504da2462ca`
