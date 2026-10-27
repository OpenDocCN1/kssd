
# [原创]锲而不舍，终破`bt`仓库

## `bt`仓库的算法分析

这个软件的算法其实并不难，只是进行的算法过长，让人看了有点晕晕的感觉

这个软件是由用户名得到注册码的，它首先对用户名进行判断，不足16位的用
`00`填充，大于的话只取前16位，然后把它分成4组，下面用`c1`,`c3`，`c4`，`c5`表示

同时还有3个中间量，`c2`,`c6`,`c7`.

运行软件后用`olldbg`的附加功能加载程序，点注册选项，下断点`bp getdlgitem`

输入假码，`regname:ligyuan`，`regcode:123456789`点击注册,被断下，`Alt+F9`返回

到这里：

```
0042D947  |>mov ecx,eax
0042D949  |>call <jmp.&MFC42.#3874>          //获取用户名
0042D94E  |>lea ecx,dword ptr ss:[ebp-10]
0042D951  |>push ecx
0042D952  |>push 414
0042D957  |>mov ecx,dword ptr ss:[ebp-24]
0042D95A  |>call <jmp.&MFC42.#3092>
0042D95F  |>mov ecx,eax
0042D961  |>call <jmp.&MFC42.#3874>        //获取注册码
0042D966  |>push ecx
0042D967  |>mov ecx,esp
0042D969  |>mov dword ptr ss:[ebp-18],esp
0042D96C  |>lea edx,dword ptr ss:[ebp-10]
0042D96F  |>push edx
0042D970  |>call <jmp.&MFC42.#535>
0042D975  |>mov dword ptr ss:[ebp-28],eax
0042D978  |>mov eax,dword ptr ss:[ebp-28]
0042D97B  |>mov dword ptr ss:[ebp-2C],eax
0042D97E  |>mov byte ptr ss:[ebp-4],2
0042D982  |>push ecx
0042D983  |>mov ecx,esp
0042D985  |>mov dword ptr ss:[ebp-1C],esp
0042D988  |>lea edx,dword ptr ss:[ebp-14]
0042D98B  |>push edx
0042D98C  |>call <jmp.&MFC42.#535>
0042D991  |>mov dword ptr ss:[ebp-30],eax        ; |
0042D994  |>mov ecx,BTLocal.0045C528             ; |
0042D999  |>mov byte ptr ss:[ebp-4],1            ; |这里就是关键call
0042D99D  |>call BTLocal.00436F88                ; \BTLocal.00436F88
0042D9A2  |>mov dword ptr ss:[ebp-34],eax
0042D9A5  |>cmp dword ptr ss:[ebp-34],0
0042D9A9  |>jnz short BTLocal.0042D9EB
```

跟进后很快你就回找到一个处理用户名的`call`，处理完后，在下面见`call`就`F7`，不久
就可以来到这里，具体的就不说了，很简单的，下面的几百行代码都是它的注册码
生成代码，熬了一个晚上才把它搞定。晕！！！！

```
0043273B  /$ >push ebp
0043273C  |. >mov ebp,esp
0043273E  |. >sub esp,20
00432741  |. >mov dword ptr ss:[ebp-20],ecx        ; 一数据地址存栈
00432744  |. >mov eax,dword ptr ss:[ebp+8]         ; 取用户地址
00432747  |. >mov ecx,dword ptr ds:[eax]           ;  取c1
00432749  |. >mov edx,dword ptr ss:[ebp-20]        ;
0043274C  |. >add ecx,dword ptr ds:[edx+4]         ;  与一数据相加
0043274F  |. >mov dword ptr ss:[ebp-4],ecx         ;  存于栈中
00432752  |. >mov eax,dword ptr ss:[ebp+8]
00432755  |. >mov ecx,dword ptr ds:[eax+4]         ;  取c3
00432758  |. >mov edx,dword ptr ss:[ebp-20]
0043275B  |. >add ecx,dword ptr ds:[edx+8]         ;  与一数据相加
0043275E  |. >mov dword ptr ss:[ebp-C],ecx         ;  存于栈中
00432761  |. >mov eax,dword ptr ss:[ebp+8]
00432764  |. >mov ecx,dword ptr ds:[eax+8]         ;  取c4
00432767  |. >mov edx,dword ptr ss:[ebp-20]
0043276A  |. >add ecx,dword ptr ds:[edx+C]         ;  与一数据相加
0043276D  |. >mov dword ptr ss:[ebp-10],ecx        ;  存于栈中
00432770  |. >mov eax,dword ptr ss:[ebp+8]
00432773  |. >mov ecx,dword ptr ds:[eax+C]         ;  取c5
00432776  |. >mov edx,dword ptr ss:[ebp-20]
00432779  |. >add ecx,dword ptr ds:[edx+10]        ;  与一数据相加
0043277C  |. >mov dword ptr ss:[ebp-14],ecx        ;  存于栈中
```

下面这段是对上面变换后的用户名进行处理的一个阶段，分8个部分，每部分执行的都是类似
操作，取一组用户名比如`c1`，循环右移`0x8`后得到`c2`，然后取`c1`的低8位值，由它在表中得到
一数据，把它与下一组（这里是`c3`）异或，并替换`c3`，取`c1`的低8位值，由它在表中得到一
数据与`c3`异或替换`c3`；`c1`循环右移`0x10`替换`c2`，`c1`循环右移`0x18`替换`c1`；取`c2`低8位，得一表
中数据并与再下一组相加并替换（这里是`c4`）；取`c1`的低8位，得一数据与最后一组异或并替疑惑
换（这里是`c5`）;只有少许不同。

```
0043277F  |. >mov eax,dword ptr ss:[ebp-4]         ;  取c1
00432782  |. >ror eax,8                            ;  循环右移0x8
00432785  |. >mov dword ptr ss:[ebp-8],eax         ;  存为c2
00432788  |. >mov ecx,dword ptr ss:[ebp-4]         ;  取c1
0043278B  |. >and ecx,0FF                          ;  取低8位
00432791  |. >mov edx,dword ptr ss:[ebp-C]         ;  取c3
00432794  |. >xor edx,dword ptr ds:[ecx*4+45B3C0]  ;  用一数据与edx异或
0043279B  |. >mov dword ptr ss:[ebp-C],edx         ;  替换原数据
0043279E  |. >mov eax,dword ptr ss:[ebp-8]         ;  取c2
004327A1  |. >and eax,0FF                          ;  取低2位
004327A6  |. >mov ecx,dword ptr ss:[ebp-C]         ;  取c3
004327A9  |. >add ecx,dword ptr ds:[eax*4+45B7C0]  ;  与表中一数据加
004327B0  |. >mov dword ptr ss:[ebp-C],ecx         ;  替换c3
004327B3  |. >mov edx,dword ptr ss:[ebp-4]         ;  取c1
004327B6  |. >ror edx,10                           ;  循环右移0x10
004327B9  |. >mov dword ptr ss:[ebp-8],edx         ;  替换c2
004327BC  |. >mov eax,dword ptr ss:[ebp-4]         ;  取c1
004327BF  |. >ror eax,18                           ;  循环右移0x18
004327C2  |. >mov dword ptr ss:[ebp-4],eax         ;  替换c1
004327C5  |. >mov ecx,dword ptr ss:[ebp-8]         ;  取c2
004327C8  |. >and ecx,0FF                          ;  取低8位
004327CE  |. >mov edx,dword ptr ss:[ebp-10]        ;  取c4
004327D1  |. >add edx,dword ptr ds:[ecx*4+45B3C0]  ;  与表中一数据相加
004327D8  |. >mov dword ptr ss:[ebp-10],edx        ;  替换c4
004327DB  |. >mov eax,dword ptr ss:[ebp-4]         ;  取c1
004327DE  |. >and eax,0FF                          ;  取低8位
004327E3  |. >mov ecx,dword ptr ss:[ebp-14]        ;  取c5
004327E6  |. >xor ecx,dword ptr ds:[eax*4+45B7C0]  ;  与表中一数据异或
004327ED  |. >mov dword ptr ss:[ebp-14],ecx        ;  替换c5
004327F0  |. >mov edx,dword ptr ss:[ebp-4]         ;  取c1
004327F3  |. >add edx,dword ptr ss:[ebp-14]        ;  与c5相加
004327F6  |. >mov dword ptr ss:[ebp-4],edx         ;  替换c1
```

```
004327F9  |. >mov eax,dword ptr ss:[ebp-C]         ;  取c3
004327FC  |. >ror eax,8                            ;  循环右移0x8
004327FF  |. >mov dword ptr ss:[ebp-8],eax         ;  替换c2
00432802  |. >mov ecx,dword ptr ss:[ebp-C]         ;  取c3
00432805  |. >and ecx,0FF                          ;  取低2位
0043280B  |. >mov edx,dword ptr ss:[ebp-10]        ;  取c4
0043280E  |. >xor edx,dword ptr ds:[ecx*4+45B3C0]  ;  与表中一数据异或
00432815  |. >mov dword ptr ss:[ebp-10],edx        ;  替换c4
00432818  |. >mov eax,dword ptr ss:[ebp-8]         ;  取c2
0043281B  |. >and eax,0FF                          ;  取低2位
00432820  |. >mov ecx,dword ptr ss:[ebp-10]        ;  取c4
00432823  |. >add ecx,dword ptr ds:[eax*4+45B7C0]  ;  与表中一数据相加
0043282A  |. >mov dword ptr ss:[ebp-10],ecx        ;  替换c4
0043282D  |. >mov edx,dword ptr ss:[ebp-C]         ;  取c3
00432830  |. >ror edx,10                           ;  取低2位
00432833  |. >mov dword ptr ss:[ebp-8],edx         ;  替换c2
00432836  |. >mov eax,dword ptr ss:[ebp-C]         ;  取c3
00432839  |. >ror eax,18                           ;  循环右移0x18
0043283C  |. >mov dword ptr ss:[ebp-C],eax         ;  替换c3
0043283F  |. >mov ecx,dword ptr ss:[ebp-8]         ;  取c2
00432842  |. >and ecx,0FF                          ;  取低2位
00432848  |. >mov edx,dword ptr ss:[ebp-14]        ;  取c5
0043284B  |. >add edx,dword ptr ds:[ecx*4+45B3C0]  ;  与表中一数据相加
00432852  |. >mov dword ptr ss:[ebp-14],edx        ;  替换c5
00432855  |. >mov eax,dword ptr ss:[ebp-C]         ;  取c3
00432858  |. >and eax,0FF                          ;  取低2位
0043285D  |. >mov ecx,dword ptr ss:[ebp-4]         ;  取c1
00432860  |. >xor ecx,dword ptr ds:[eax*4+45B7C0]  ;  与表中一数据异或
00432867  |. >mov dword ptr ss:[ebp-4],ecx         ;  替换c1
0043286A  |. >mov edx,dword ptr ss:[ebp-C]         ;  取c3
0043286D  |. >add edx,dword ptr ss:[ebp-10]        ;  与表中一数据相加
00432870  |. >mov dword ptr ss:[ebp-C],edx         ;  替换c3
```

```
00432873  |. >mov eax,dword ptr ss:[ebp-10]        ;  取c4
00432876  |. >ror eax,8                            ;  循环右移0x8
00432879  |. >mov dword ptr ss:[ebp-8],eax         ;  替换c2
0043287C  |. >mov ecx,dword ptr ss:[ebp-10]        ;  取c4
0043287F  |. >and ecx,0FF                          ;  取低2位
00432885  |. >mov edx,dword ptr ss:[ebp-14]        ;  取c5
00432888  |. >xor edx,dword ptr ds:[ecx*4+45B3C0]  ;  与表中一数据异或
0043288F  |. >mov dword ptr ss:[ebp-14],edx        ;  替换c5
00432892  |. >mov eax,dword ptr ss:[ebp-8]         ;  取c2
00432895  |. >and eax,0FF                          ;  取低2位
```
