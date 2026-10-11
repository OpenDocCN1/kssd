
# 有关花指令的一点小研究(三)

> 原文：[`www.pediy.com/kssd/pediy10/62138.html`](https://www.pediy.com/kssd/pediy10/62138.html)

闲来无事，再研究下花指令，呵呵。

请看下面的花指令调用：（`OD` 下 `ctrl+A` 分析结束后结果）

代码:

```
0040112D > $  B8 05000000             mov     eax, 5  ;start
00401132   .  E8 D61E0000             call    0040300D
00401137   .  85C0                    test    eax, eax
00401139   .  74 00                   je      short 0040113B
0040113B   >  E8 6A00E831             call    322811AA
00401140   ?  0000                    add     byte ptr [eax], al
00401142   ?  00A3 30314000           add     byte ptr [ebx+403130], ah

;...
00403000    0000                      add     byte ptr [eax], al
00403002    0083 042401E8             add     byte ptr [ebx+E8012404], al
00403008    0000                      add     byte ptr [eax], al
0040300A    0000                      add     byte ptr [eax], al
0040300C    C3                        retn
0040300D    E8 F1FFFFFF              call    00403003
00403012    E8 010424E8              call    E8643418
00403017    0000                      add     byte ptr [eax], al
00403019    0000                      add     byte ptr [eax], al
0040301B    58                        pop     eax
0040301C    E9 FFFFFFFF              jmp     00403020
00403021    E0 00                     loopdne short 00403023
```

源代码如下：

代码:

```
;----花指令为什么写在数据段？为了避免 OD 分析，因为 OD 只自动分析代码段。---
;----为此，那些 label 跳转无法在数据段写，只好写机器码了----
;----所有的跳转都是相对偏移---
.data
  db  00,00,00    ;三个 0 混淆后面指令
  add  dword ptr [esp],1  ;把返回地址+1
  db  0E8h,0,0,0,0    ;call  $+5
  retn        ;这条指令被调用两次，一次 call,一次自身
myjmp  proc
  db  0E8h,0F1h,0FFh,0FFh,0FFh  ;call 上面
  db  0E8h      ;混淆指令
  add [esp],eax      ;把需要的偏移加上去
  db  0E8h,0,0,0,0    ;call  $+5
  pop  eax      ;取出返回地址，会有两次经过
  db  0E9h,0FFh,0FFh,0FFh,0FFh,0E0h  ;**就在这里了，两条指令合并
myjmp  endp

.code
start:
  mov  eax,offset @F - $ - 10  ;@@指向正常指令
  call  myjmp
  test  eax,eax
  jz  $+2
  db  0E8h
@@:
  invoke  GetModuleHandle,NULL
```

总结：

1. 一处阴险的地方在于 `retn` 会被连续调用两次，一个 `call`，一次 `retn` 返回到自身执行一次。
2. 一处阴险的地方在于把花指令写在数据段，避免了 `OD` 分析。
3. 一处阴险的地方在于最后三条指令都会被过两遍，跳晕人。
4. 一处很阴险的地方在于 `jmp $+4` 与 `jmp eax` 两条指令合并，无论是 `OD` 还是 `IDA`，只能分析其中一条指令，看到的只是 `jmp $+4`。嘿嘿。够阴险吧。

思考：

发现了其实真正的 `jmp $+X` 是短跳，所以只需要 `jmp $+1` 就可以了，那最后的机器可以改成 `db 0EBh,0FFh,0E0h` 了。。。随便啦。

由于写在数据段，所以，以后看看在 `myjmp` 里加上 `SMC`，嘿嘿。

娱乐研究，不足之处请指正，谢谢。欢迎拍砖。
