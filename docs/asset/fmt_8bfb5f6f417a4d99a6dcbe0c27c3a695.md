
# EmEditor v3 Version 3.09

> 原文：[`www.pediy.com/kssd/pediy03/forum514.htm`](https://www.pediy.com/kssd/pediy03/forum514.htm)

EmEditor v3 Version 3.09 汉化版注册码算法分析

软件名称：EmEditor v3 Version 3.09

下载地址：`http://www.inhua.com/EmEditor.htm`

软件说明：简单好用的文本编辑器，支持多种配置，自定义颜色、字体、工具栏、快捷键设置，可以调整行距，避免中文排列过于紧密，具有选择文本列块的功能（按 `ALT` 键拖动鼠标），并允许无限撤消、重做，总之功能多多，使用方便，是替代记事本的最佳编辑器。

调试工具：`SoftICE 4.05`、`W32dsm89`

调试平台：`Win2K`

作者：`chn-boy`

日期：2001-1-9

## 第一步

下中断：`bpx GetWindowTextW do "d esp\->8; p ret; "`，然后输入你的 `Virtual Code`，点注册。我输入的是：`9999 8888 7777 6666`

程序拦断在如下代码段处：

```
:004173CD 53                          push ebx
:004173CE 8D86F2030000                lea eax, dword ptr [esi+000003F2]
:004173D4 53                          push ebx
:004173D5 50                          push eax
:004173D6 FF7508                      push [ebp+08]
:004173D9 FF1584334300                Call dword ptr [00433384]  / GetWindowTextW()
:004173DF 668907                      mov word ptr [edi], ax     / 中断拦断在此，四个输入框，你将会被拦断四次
                                    / 每次将十进制值返回给 ax，如 9999 = 0x270F, 那么 AX = 270F
                                    / 为了表达方便，我分别用①、②、③、④来表示
:004173E2 46                          inc esi
:004173E3 47                          inc edi
:004173E4 47                          inc edi
:004173E5 83FE04                      cmp esi, 00000004          / 是不是有了四个窗口输入？
:004173E8 7CE3                        jl 004173CD
:004173EA 8D45F4                      lea eax, dword ptr [ebp-0C] / 将得到的四个值连续放到 eax 偏移处
                                    / 此处，用 d eax，可以查看到你输入的四个十进制值
:004173ED 50                          push eax
:004173EE E8F5FeFFFF                  call 004172E8              / 核心处，判断算法，查看输入的注册码是否正确？
:004173F3 3BC3                        cmp eax, ebx               / 如果 eax = ebx ，那么输入正确
:004173F5 5F                          pop edi
:004173F6 742A                        je 00417422
:004173F8 33C9                        xor ecx, ecx
:004173FA 83F802                      cmp eax, 00000002
:004173FD 0F95C1                      setne cl
```

## 第二步

`Call 004172EB` 代码为：

```
:004172E8 56                          push esi
:004172E9 8B742408                    mov esi, dword ptr [esp+08]
:004172ED 57                          push edi
:004172EE 6A0A                        push 0000000A
:004172F0 0FB706                      movzx eax, word ptr [esi]  / ①
:004172F3 99                          cdq
:004172F4 59                          pop ecx                      / ecx = 0x0A = 10
:004172F5 F7F9                        idiv ecx
:004172F7 3DAB000000                   cmp eax, 000000AB           / ①÷10 是否等于 AB，也就是看①是否为 0x6AE=1710
:004172FC 7405                        je 00417303                 / 是，则正确，也就是说①必须为 1710
:004172FE 6A01                        push 000000001
:00417300 58                          pop eax
:00417301 EB15                        jmp 00417318
:00417303 668B7E06                    mov di, word ptr [esi+06]   / ④
:00417307 56                          push esi
:00417308 E834FFFFFF                  call 00417241               / 此 Call 为：用①、②、③进行一系列运算，得到一个结果，假设为⑤
                                    / 最后结果返回到 [esi+06]
:0041730D 85C0                        test eax, eax
:0041730F 7507                        jne 00417318
:00417311 663B7E06                    cmp di, word ptr [esi+06]   / 看看④是否等于⑤？相等，则注册成功
:00417315 0F95C0                      setne al
:00417318 5F                          pop edi
:00417319 5E                          pop esi
:0041731A C20400                      ret 0004
```

## 第三步

`Call 00417241` 代码为：

```
:00417241 56                          push esi
:00417242 8B742408                    mov esi, dword ptr [esp+08]
:00417246 668B4602                    mov ax, word ptr [esi+02]   / ax = ②
:0041724A 6683660600                  and word ptr [esi+06], 0000
:0041724F 663D0F27                    cmp ax, 270F
:00417253 0F8788000000                 ja 004172E1
:00417259 668B4E04                    mov cx, word ptr [esi+04]   / cx = ③
:0041725D 6681F90F27                   cmp cx, 270F
:00417262 777D                        ja 004172E1
:00417264 6685C0                       test ax, ax
:00417267 7474                        je 004172DD
:00417269 663DAE08                     cmp ax, 08AE                / 0x08AE = 2222
:0041726D 746E                        je 004172DD
:0041726F 663D2E16                     cmp ax, 162E                / 0x162E = 5678 ，第二个窗口中不能输入 2222 或 5678
:00417273 7468                        je 004172DD
:00417275 53                          push ebx
:00417276 55                          push ebp
:00417277 57                          push edi
:00417278 6A0A                        push 0000000A
:0041727A 0FB7F9                       movzx edi, cx               / edi = ③
:0041727D 0FB70E                       movzx ecx, word ptr [esi]   / ecx = ①
:00417280 0FB7C0                       movzx eax, ax               / eax = ②
:00417283 89442418                     mov dword ptr [esp+18], eax
:00417287 8BC1                         mov eax, ecx                / eax = ③
:00417289 99                           cdq
:0041728A 5B                           pop ebx
:0041728B F7FB                         idiv ebx                    / eax = ③÷10
:0041728D 6A64                         push 00000064
:0041728F 5D                           pop ebp
:00417290 55                           push ebp
:00417291 8BD8                         mov ebx, eax                / ebx = ③÷10
:00417293 8BC7                         mov eax, edi
:00417295 99                           cdq
:00417296 F7FD                         idiv ebp                    / eax = ③÷100
:00417298 8B542418                     mov edx, dword ptr [esp+18] / edx = ②
:0041729C 03D3                         add edx, ebx
:0041729E 03C2                         add eax, edx
:004172A0 03C7                         add eax, edi                / eax = ②+③+③÷10+③÷100
:004172A2 5F                           pop edi
:004172A3 99                           cdq
:004172A4 F7FF                         idiv edi                    / eax = (②+③+③÷10+③÷100)÷100 的商
                                    / edx 为余数
:004172A6 8B442414                     mov eax, dword ptr [esp+14]
:004172AA 55                           push ebp
:004172AB 5B                           pop ebx
:004172AC 55                           push ebp
:004172AD 8B3C954C7F4300               mov edi, dword ptr [4*edx+00437F4C] / 用余数查表，表可以用 d 00437F4C 查看
。。。。。。    下面的算法分析同前，不再一一列出    。。。。。。
:004172B4 99                           cdq
:004172B5 6BFF64                       imul edi, 00000064
:004172B8 F7FB                         idiv ebx
:004172BA 8BD8                         mov ebx, eax
:004172BC 8BC1                         mov eax, ecx
:004172BE 99                           cdq
:004172BF F7FD                         idiv ebp
:004172C1 03CB                         add ecx, ebx
:004172C3 03C1                         add eax, ecx
:004172C5 59                           pop ecx
:004172C6 99                           cdq
:004172C7 F7F9                         idiv ecx
:004172C9 8B04954C7F4300               mov eax, dword ptr [4*edx+00437F4C]
:004172D0 03F8                         add edi, eax
:004172D2 33C0                         xor eax, eax
:004172D4 66897E06                     mov word ptr [esi+06], di   / 将用①、②、③的值算出的值赋给 [esi+06]
                                    / 此处得到的 di 就是⑤，在 SoftICE 环境下用?di
                                    / 得到的十进制的值就是你要输入的第 4 个窗口的数字号码
:004172D8 5F                           pop edi
:004172D9 5D                           pop ebp
:004172DA 5B                           pop ebx
:004172DB EB07                         jmp 004172E4
:004172DD 6A02                         push 000000002
:004172DF EB02                         jmp 004172E3
:004172E1 6A01                         push 000000001
:004172E3 58                           pop eax
```
