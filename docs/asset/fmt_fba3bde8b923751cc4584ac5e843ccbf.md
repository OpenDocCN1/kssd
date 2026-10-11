
```
006AF2E2     F7C3 00000080             test ebx,80000000

006AF2E8     75 04                   jnz short DrvStudy.006AF2EE

006AF2EA     03DA                   add ebx,edx

006AF2EC     43                      inc ebx                        ; 取函数名
006AF2ED     43                      inc ebx                        ; 取函数名

006AF2EE     53                      push ebx

006AF2EF     81E3 FFFFFFFF          and ebx,7FFFFFFF

006AF302     53                      push ebx                       ; 函数名入栈

006AF2F6     FFB5 45050000          push dword ptr ss:[ebp+545]     ; kernel32.dll 句柄入栈

006AF302     FF95 490F0000          call dword ptr ss:[ebp+F49]    ; GetProcAddress

006AF302     85C0                   test eax,eax

006AF304     5B                      pop ebx

006AF305     75 6F                  jnz short DrvStudy.006AF376    ; 跳

取 API 函数地址成功后跳到下面的地方

006AF376     8907                   mov dword ptr ds:[edi],eax     ; API 地址存入 edi 指向地址

006AF378     8385 49050000 04      add dword ptr ss:[ebp+549],4   ; 取下一个 API

006AF37F     E9 32FFFFFF            jmp DrvStudy.006AF2B6          ; 跳

006AF384     8906                   mov dword ptr ds:[esi],eax

006AF386     8946 0C               mov dword ptr ds:[esi+C],eax

006AF389     8946 10               mov dword ptr ds:[esi+10],eax

006AF38C     83C6 14               add esi,14

006AF38F     8B95 22040000         mov edx,dword ptr ss:[ebp+422]

006AF395     E9 EBFEFFFF           jmp DrvStudy.006AF285          ; 跳去恢复下个导入 dll 中的函数
```

这个大循环完成后 `IAT` 就恢复完成了。这个 `IAT` 是没有被破坏的，根据一般经验，壳在运行时都会有这么一个大循环来恢复 `IAT`。

有的是破坏，因此当在一个循环中碰到 `GetModuleHandleA`、`LoadLibraryA`、`GetProcAddress` 时就要小心了，记下它。

有可能 `ImportREC` 找不回 API 函数的时候，需要手工修复 `IAT` 时就要用到它。

循环完后，来到这里：

```
006AF39A     B8 08CF1C00           mov eax,1CCF08                 ; OEP 的 RVA
006AF39F     50                    push eax
006AF3A0     0385 22040000        add eax,dword ptr ss:[ebp+422]   ; OEP 值
006AF3A6     59                    pop ecx
006AF3A7     0BC9                 or ecx,ecx
006AF3A9     8985 A8030000       mov dword ptr ss:[ebp+3A8],eax   ; 修改 [ebp+3A8] 处的代码
006AF3AF     61                    popad
006AF3B0     75 08                jnz short DrvStudy.006AF3BA      ; 跳
006AF3B2     B8 01000000          mov eax,1
006AF3B7     C2 0C00              retn 0C
006AF3BA     68 08CF5C00          push DrvStudy.005CCF08         ; 变形跳转
006AF3BF     C3                    retn                           ; ^_^ 飞向光明之颠
```

这里就是典型的 Delphi 程序的 `OEP` 处代码：

```
005CCF08     55                    push ebp                       ; 停在这里
005CCF09     8BEC                 mov ebp,esp
005CCF0B     83C4 F0              add esp,-10
005CCF0E     53                    push ebx
005CCF0F     B8 80CA5C00          mov eax,DrvStudy.005CCA80
005CCF14     E8 B3A7E3FF          call DrvStudy.004076CC
```

现在就可在 `005CCF08` 处 Dump 程序了。用 OD 的 `OllyDump` 插件，Dump 出程序，启动 `ImportREC`，填入 `OEP` 的 `RVA` 值。

先点 `IATAutoSearch`，再点 `GetImport`，找到的 API 全部有效，点 `FixDump`，修复 `IAT` 指针。

运行程序，提示系统数据错误，这是因为程序有自校验，发现自己被脱壳后拒绝运行。去除程序的自校验我就不说了... 用爆破的方法就能搞定...

### 总结

本文讲述了脱壳的一般方法。新手面对未知壳先不要怕，自己试着跟踪分析一下，多做练习才会有进步。

`4dd466d0e1e0cc123afbe504da2462ca`
