
# 体育彩票随机号码产生器 v3.3

> 原文：[`www.pediy.com/kssd/pediy03/forum745.htm`](https://www.pediy.com/kssd/pediy03/forum745.htm)

工具：`trw1.22`

目的：耍一伙

`load`

注册码：`31415926` （听 `SB` 的没错^\_^)

`Ctrl-N`

`bpx hmemcpy`

`g`

```
:0041E9BB FF15C0324200              Call dword ptr [004232C0]
:0041E9C1 50                        push eax
* Reference To: MSVBVM50.__vbaBoolVarNull, Ord:0000h
                                    |
:0041E9C2 FF1588324200              Call dword ptr [00423288]
:0041E9C8 6685C0                    test ax, ax
:0041E9CB B804000280                mov eax, 80020004
:0041E9D0 B90A000000                mov ecx, 0000000A
:0041E9D5 8985D8FEFFFF              mov dword ptr [ebp+FFFFFED8], eax
:0041E9DB 898DD0FEFFFF              mov dword ptr [ebp+FFFFFED0], ecx
:0041E9E1 8985E8FEFFFF              mov dword ptr [ebp+FFFFFEE8], eax
:0041E9E7 898DE0FEFFFF              mov dword ptr [ebp+FFFFFEE0], ecx
:0041E9ED 0F842C070000              je 0041F11F        -------在此下 r fl z
:0041E9F3 BF08000000                mov edi, 00000008
:0041E9F8 8D9590FEFFFF              lea edx, dword ptr [ebp+FFFFFE90]
:0041E9FE 8D8DF0FEFFFF              lea ecx, dword ptr [ebp+FFFFFEF0]
* Possible StringData Ref from Code Obj ->"鑜孮 xcknx鲖蛻癳袕 L 坉 ko 忲 N0"
                                    |
:0041EA04 C78598FEFFFF4C964000      mov dword ptr [ebp+FFFFFE98], 0040964C
:0041EA0E 89BD90FEFFFF              mov dword ptr [ebp+FFFFFE90], edi
```

不但能自动注册成功，而且会在当前目录下建立 `registed.txt` 文件，内容如下：

```
体育彩票随机号码产生器 v3.3 注册成功
机器码为： 537921887  注册码为： 537933795  4dd466d0e1e0cc123afbe504da2462ca
```
