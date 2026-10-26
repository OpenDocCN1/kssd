
### 0047E733
```
0047E733    FFD0            call eax                                                                      ;跳向OEP

0047E735    E8 A6C00000     call Allok\_Vi.0048A7E0
```

### 0041F872
```
0041F872    55              push ebp                                                         ;OEP到了DUMP。修复。

0041F873    8BEC            mov ebp,esp

0041F875    6A FF           push -1

0041F877    68 68644200     push Allok\_Vi.00426468

0041F87C    68 E8F94100     push Allok\_Vi.0041F9E8                                                       ; jmp to msvcrt.\_except\_handler3

0041F881    64:A1 00000000  mov eax,dword ptr fs:[0]

0041F887    50              push eax

0041F888    64:8925 0000000>mov dword ptr fs:[0],esp

0041F88F    83EC 68         sub esp,68

0041F892    53              push ebx

0041F893    56              push esi

0041F894    57              push edi

0041F895    8965 E8         mov dword ptr ss:[ebp-18],esp

0041F898    33DB            xor ebx,ebx

0041F89A    895D FC         mov dword ptr ss:[ebp-4],ebx

0041F89D    6A 02           push 2
```

### ④破解

运行脱壳后的程序.

破解相当简单，直接写入用户名，注册码，改下注册标志即可。

没啥技术含量。

【版权声明】原创，转载注名.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
