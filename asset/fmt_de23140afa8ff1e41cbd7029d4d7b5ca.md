
```
000134B4: 5C                        pop esp
000134B5: 52                        push edx
000134B6: 45                        inc ebp
000134B7: 47                        inc edi
000134B8: 56                        push esi
000134B9: 58                        pop eax
000134BA: 44                        inc esp
000134BB: 005C5C2E                  add byte ptr [esp+ebx*2+2E],bl
000134BF: 5C                        pop esp
000134C0: 4E                        dec esi
000134C1: 54                        push esp
000134C2: 49                        dec ecx
000134C3: 43                        inc ebx
000134C4: 45                        inc ebp
000134C5: 005C5C2E                  add byte ptr [esp+ebx*2+2E],bl
000134C9: 5C                        pop esp
000134CA: 53                        push ebx
000134CB: 49                        dec ecx
000134CC: 43                        inc ebx
000134CD: 45                        inc ebp
000134CE: 005C5C2E                  add byte ptr [esp+ebx*2+2E],bl
000134D2: 5C                        pop esp
000134D3: 56                        push esi
000134D4: 4B                        dec ebx
000134D5: 45                        inc ebp
000134D6: 59                        pop ecx
000134D7: 50                        push eax
000134D8: 52                        push edx
000134D9: 4F                        dec edi
000134DA: 44                        inc esp
000134DB: 005C5C2E                  add byte ptr [esp+ebx*2+2E],bl
000134DF: 5C                        pop esp
000134E0: 46                        inc esi
000134E1: 49                        dec ecx
000134E2: 4C                        dec esp
000134E3: 45                        inc ebp
000134E4: 56                        push esi
000134E5: 58                        pop eax
000134E6: 44                        inc esp
000134E7: 0000                      add byte ptr [eax],al
```

**非代码区**

**判断检测的结果**

```
000134E9: 0083F8FF7406              add byte ptr [ebx+0674FFF8],al   <---跳转，不跳就死定了
```

**退出程序**

```
000134EF: FF956CBF4000              call dword ptr [ebp+0040BF6C]    <---ExitProcess
```

**这是一段反跟踪的代码**

```
000134F5: 60                        pushad                <---跳到此处
000134F6: BE53BD4000               mov esi,0040BD53
000134FB: 03F5                      add esi,ebp
000134FD: 0F010E                   sidt dword ptr [esi]      <---获取 IDTR 内容
00013500: 8B7602                   mov esi,dword ptr [esi+02]     <---获取 IDT 表的基地址
00013503: 668B4608                 mov ax,word ptr [esi+08]       <---取 int 1 的低位偏移
00013507: 66052302                 add ax,0223            <---计算新值
0001350B: 66894608                 mov word ptr [esi+08],ax       <---保存 int 1 新偏移值
0001350F: 668B460E                 mov ax,word ptr [esi+0E]       <---取 int 1 的高位偏移
00013513: 668B5E06                 mov bx,word ptr [esi+06]       <---取 int 0 的高位偏移
00013517: 663BC3                   cmp ax,bx               <---是否相等
0001351A: 740E                     jz 0001352A             <---相等则跳转
0001351C: 668B4608                 mov ax,word ptr [esi+08]       <---取 int 1 的低位偏移
00013520: 668B5E18                 mov bx,word ptr [esi+18]       <---取 int 3 的低位偏移
00013524: FA                       cli
00013525: 663BC3                   cmp ax,bx               <---判断是否相等
00013528: 7400                     jz 0001352A             <---相等则跳转
0001352A: 668B4608                 mov ax,word ptr [esi+08]       <---取 int 1 的低位偏移
0001352E: 668B5E0E                 mov bx,word ptr [esi+0E]       <---取 int 1 的高位偏移
00013532: 66898567BE4000          mov word ptr [ebp+0040BE67],ax   <---保存
00013539: 66899D69BE4000          mov word ptr [ebp+0040BE69],bx   <---保存
00013540: 668B4618                 mov ax,word ptr [esi+18]       <---取 int 3 的低位偏移
00013544: 668B5E1E                 mov bx,word ptr [esi+1E]       <---取 int 3 的高位偏移
00013548: 6689856BBE4000          mov word ptr [ebp+0040BE6B],ax   <---保存
0001354F: 66899D6DBE4000          mov word ptr [ebp+0040BE6D],bx   <---保存
```

**保存 `ebp` 的值**

```
00013556: 89AD65BD4000            mov dword ptr [ebp+0040BD65],ebp
```

**保存 `ReadProcessMemory` 的首地址**

```
0001355C: 8B85C7BB4000            mov eax,dword ptr [ebp+0040BBC7]   <---取 ReadProcessMemory 的首地址
00013562: 89856FBE4000            mov dword ptr [ebp+0040BE6F],eax   <---保存
```

**这里是一段反跟踪代码**

```
00013568: E8A9140000              call 00014A16              <---这里又是一段 SIDT 代码
```

**开始分析注册情况**

```
0001356D: BB63BD4000              mov ebx,0040BD63
00013572: 03DD                    add ebx,ebp
00013574: 53                      push ebx
00013575: 6A64                    push 00000064
00013577: 6A63                    push 00000063
00013579: 6A00                    push 00000000
0001357B: FF9508BC4000           call dword ptr [ebp+0040BC08]    <---SetTimer
00013581: 61                      popad
00013582: 6A00                    push 00000000
00013584: FF95C3BB4000           call dword ptr [ebp+0040BBC3]     <---GetModuleHandle
0001358A: 89855FBD4000           mov dword ptr [ebp+0040BD5F],eax
```

**判断是否用的是注册版的幻影加密**

```
00013590: 80BDC8B2400001         cmp byte ptr [ebp+0040B2C8],01    <---是否注册标志
00013597: 7435                   jz 000135CE             <---相等，注册版则跳走
```

**用系统时间随机显示未注册的 NAG 窗口**

```
00013599: BEE1B94000              mov esi,0040B9E1
0001359E: 03F5                    add esi,ebp
000135A0: 56                      push esi
000135A1: FF959FBB4000           call dword ptr [ebp+0040BB9F]     <---GetSystemTime
000135A7: 668B85EFB94000         mov ax,word ptr [ebp+0040B9EF]
000135AE: 240F                    and al,0F
000135B0: 3C02                    cmp al,02
000135B2: 731A                    jnb 000135CE            <---结果随机，跳转
000135B4: B8BEB94000             mov eax,0040B9BE           <---到此，将显示 NAG 窗口
000135B9: 03C5                    add eax,ebp
000135BB: BB11B24000             mov ebx,0040B211
000135C0: 03DD                    add ebx,ebp
000135C2: 6A30                    push 00000030
000135C4: 50                      push eax
000135C5: 53                      push ebx
000135C6: 6A00                    push 00000000
```

**调用 `MessageBoxA` 显示 NAG 窗口**

```
000135C8: FF9500BC4000           call dword ptr [ebp+0040BC00]     <---MessageBoxA
```

**跳到此处**

**开始处理用户加密的信息**

```
000135CE: E862060000              call 00013C35              <---从注册表获取信息
000135D3: 663D0100               cmp ax,0001
000135D7: 7416                   jz 000135EF     <---成功就跳走
```

**没注意此段内容**

```
000135D9: BE60B14000              mov esi,0040B160
000135DE: 03F5                    add esi,ebp
000135E0: 668B8589B44000         mov ax,word ptr [ebp+0040B489]
000135E7: 668906                 mov word ptr [esi],ax
000135EA: E802070000             call 00013CF1
```
