
# NoteWorthy Composer 1.55b

> 原文：[`www.pediy.com/kssd/pediy01/forum059.htm`](https://www.pediy.com/kssd/pediy01/forum059.htm)

**软件名称:** `NoteWorthy Composer 1.55b`

**软件来源:** 软件世界 2000.2

**软件介绍:** 音乐创造者的最爱,可以直接用五线谱创作音乐

**设断点:** `bpx hmemcpy`

软件被中断后,按 `F12` 22 次(第 23 次出错误画面)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|`:0041E469(U)`

```
|
:0041D68C 55             push ebp
:0041D68D 8BEC           mov ebp, esp
:0041D68F 53             push ebx
:0041D690 8B5D08         mov ebx, dword ptr [ebp+08]
:0041D693 8D4324         lea eax, dword ptr [ebx+24]
:0041D696 50             push eax
:0041D697 53             push ebx
:0041D698 E818FFFFFF     call 0041D5B5
:0041D69D 83C408         add esp, 00000008 <-----程序返回到此
:0041D6A0 8D4324         lea eax, dword ptr [ebx+24]
:0041D6A3 50             push eax
:0041D6A4 E8B9F00300     call 0045C762
:0041D6A9 59             pop ecx
:0041D6AA 85C0           test eax, eax
:0041D6AC 7542           jne 0041D6F0 <-----正常注册由此跳转
```

追入 `0045C762` 的 `CALL` 找到下面的程序

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|`:0045C795(C)`

```
|
:0045C79B C645F000       mov [ebp-10], 00
:0045C79F 8D45F0         lea eax, dword ptr [ebp-10]
:0045C7A2 50             push eax
:0045C7A3 FF7348         push [ebx+48]
:0045C7A6 8D4308         lea eax, dword ptr [ebx+08]
:0045C7A9 50             push eax
:0045C7AA FF7304         push [ebx+04]
:0045C7AD FF33           push dword ptr [ebx]
:0045C7AF E8D5000000     call 0045C889
:0045C7B4 83C414         add esp, 00000014
:0045C7B7 83C34C         add ebx, 0000004C
:0045C7BA 53             push ebx
:0045C7BB 8D45F0         lea eax, dword ptr [ebp-10]
:0045C7BE 50             push eax    <------在此 D EAX 可看到注册码
:0045C7BF E8404EFBFF     call 00411604
:0045C7C4 83C408         add esp, 00000008
:0045C7C7 85C0           test eax, eax
:0045C7C9 0F94C0         sete al
:0045C7CC 83E001         and eax, 00000001
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|`:0045C77C(U), :0045C786(U), :0045C799(U)`

```
|
:0045C7CF 5B             pop ebx
:0045C7D0 8BE5           mov esp, ebp
:0045C7D2 5D             pop ebp
:0045C7D3 C3             ret
```

几组用户名和注册码:

**用户名:** `Killer`
**注册码:** `000-640-492482`

**用户名:** `Cracker`
**注册码:** `000-313-555322`

**用户名:** `Player`
**注册码:** `000-685-858909`

**用户名:** `Begginner`
**注册码:** `000-485-395681`

由上面的注册码可看出第一组 `000` 是固定的,第二组我未找到计算的地方

第三组我找到了计算的地方但该计算很复杂,此处不再详述
