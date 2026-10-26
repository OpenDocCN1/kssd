
可以看出是与注册过程是相同的。不同是在下面，它验证了前4--9位：

```
004D1E31    .  8D45 C8        lea eax,[dword ss:ebp-38]

004D1E34    .  50             push eax                        ;  注册码!

004D1E35    .  A1 B8484D00    mov eax,[dword ds:4D48B8]

004D1E3A    .  8B00           mov eax,[dword ds:eax]          ;  ASCII "h6s015lphi00000448"

004D1E3C    .  B9 09000000    mov ecx,9

004D1E41    .  BA 04000000    mov edx,4

004D1E46    .  E8 6DC0FEFF    call XXXXXXXX.004BDEB8          ;  取4-9位?

004D1E4B    .  8B55 C8        mov edx,[dword ss:ebp-38]

004D1E4E    .  B8 E8604D00    mov eax,XXXXXXXX.004D60E8

004D1E53    .  E8 1C2BF3FF    call XXXXXXXX.00404974          ;  是否为0?

004D1E58    .  33C0           xor eax,eax

004D1E5A    .  55             push ebp

004D1E5B    .  68 7C1E4D00    push XXXXXXXX.004D1E7C

004D1E60    .  64:FF30        push [dword fs:eax]

004D1E63    .  64:8920        mov [dword fs:eax],esp

004D1E66    .  A1 E8604D00    mov eax,[dword ds:4D60E8]       ;  4-9位

004D1E6B    .  E8 E075F3FF    call XXXXXXXX.00409450          ； 验证！

004D1E70    .  8BD8           mov ebx,eax

004D1E72    .  33C0           xor eax,eax
```

这里计算验证出来后,在下面还有个计算比较：

```
004D1E8C    .  E8 E724F3FF    call unpacked.00404378

004D1E91    >  8BC3           mov eax,ebx

004D1E93    .  B9 10270000    mov ecx,2710

004D1E98    .  99             cdq

004D1E99    .  F7F9           idiv ecx                        ;

004D1E9B    .  83F8 0A        cmp eax,0A

004D1E9E    .  74 08          je short unpacked.004D1EA8      ;

004D1EA0    .  A1 B44B4D00    mov eax,[dword ds:4D4BB4]

004D1EA5    .  C600 01        mov [byte ds:eax],1

004D1EA8    >  E8 770AFFFF    call unpacked.004C2924
```

（由于某种原因，我不能说明这里的关系了）

如果你有兴趣看到这，是否也觉得它的注册验证比较有意思?它这个样子可有不少麻烦的，我一共用了7天来搞它的，回头想想，也就知道我为什么是莱鸟了！

祝元旦快乐！![](https://bbs.kanxue.com/view/img/face/27.gif)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
