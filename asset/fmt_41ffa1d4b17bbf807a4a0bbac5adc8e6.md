
```asm
00403A86  . |83C1 01          |add ecx,1  
00403A89  > |8A58 09          |mov bl,byte ptr ds:[eax+9]  
00403A8C  . |3A1D ADB14400     |cmp bl,byte ptr ds:[44B1AD]  
00403A92  . |75 03             |jnz short dumped_.00403A97  
00403A94  . |83C1 01          |add ecx,1  
00403A97  > |83F9 0A          |cmp ecx,0A  
00403A9A  . |0F85 AA000000    |jnz dumped_.00403B4A  
00403AA0  . |B9 90909090      |mov ecx,90909090  
00403AA5  . |8908             |mov dword ptr ds:[eax],ecx                     ;  将上面的东东用 90 填充  
00403AA7  . |8948 04          |mov dword ptr ds:[eax+4],ecx  
00403AAA  . |66:8948 08       |mov word ptr ds:[eax+8],cx  
00403AAE  . |83C0 0A          |add eax,0A  
00403AB1  . |E9 94000000      |jmp dumped_.00403B4A  
00403AB6  > |8A18             |mov bl,byte ptr ds:[eax]                       ;  第一个字节赋给 bl  
00403AB8  . |33C9             |xor ecx,ecx  
00403ABA  . |3A1D B4B14400    |cmp bl,byte ptr ds:[44B1B4]                   ;  比较是否是 eb  
00403AC0  . |75 05             |jnz short dumped_.00403AC7  
00403AC2  . |B9 01000000      |mov ecx,1  
00403AC7  > |8A58 01          |mov bl,byte ptr ds:[eax+1]                   ;  比较下一个字节  
00403ACA  . |3A1D B5B14400    |cmp bl,byte ptr ds:[44B1B5]                   ;  是否是 08  
00403AD0  . |75 03             |jnz short dumped_.00403AD5  
00403AD2  . |83C1 01          |add ecx,1  
00403AD5  > |8A58 02          |mov bl,byte ptr ds:[eax+2]  
00403AD8  . |3A1D B6B14400    |cmp bl,byte ptr ds:[44B1B6]  
00403ADE  . |75 03             |jnz short dumped_.00403AE3  
00403AE0  . |83C1 01          |add ecx,1  
00403AE3  > |8A58 03          |mov bl,byte ptr ds:[eax+3]  
00403AE6  . |3A1D B7B14400    |cmp bl,byte ptr ds:[44B1B7]  
00403AEC  . |75 03             |jnz short dumped_.00403AF1  
00403AEE  . |83C1 01          |add ecx,1  
00403AF1  > |8A58 04          |mov bl,byte ptr ds:[eax+4]  
00403AF4  . |3A1D B8B14400    |cmp bl,byte ptr ds:[44B1B8]  
00403AFA  . |75 03             |jnz short dumped_.00403AFF  
00403AFC  . |83C1 01          |add ecx,1  
00403AFF  > |8A58 05          |mov bl,byte ptr ds:[eax+5]  
00403B02  . |3A1D B9B14400    |cmp bl,byte ptr ds:[44B1B9]  
00403B08  . |75 03             |jnz short dumped_.00403B0D  
00403B0A  . |83C1 01          |add ecx,1  
00403B0D  > |8A58 06          |mov bl,byte ptr ds:[eax+6]  
00403B10  . |3A1D BAB14400    |cmp bl,byte ptr ds:[44B1BA]  
00403B16  . |75 03             |jnz short dumped_.00403B1B  
00403B18  . |83C1 01          |add ecx,1  
00403B1B  > |8A58 07          |mov bl,byte ptr ds:[eax+7]  
00403B1E  . |3A1D BBB14400    |cmp bl,byte ptr ds:[44B1BB]  
00403B24  . |75 03             |jnz short dumped_.00403B29  
00403B26  . |83C1 01          |add ecx,1  
00403B29  > |8A58 08          |mov bl,byte ptr ds:[eax+8]  
00403B2C  . |3A1D BCB14400    |cmp bl,byte ptr ds:[44B1BC]  
00403B32  . |75 03             |jnz short dumped_.00403B37  
00403B34  . |83C1 01          |add ecx,1  
00403B37  > |8A58 09          |mov bl,byte ptr ds:[eax+9]  
00403B3A  . |3A1D BDB14400    |cmp bl,byte ptr ds:[44B1BD]  
00403B40  . |75 03             |jnz short dumped_.00403B45  
00403B42  . |83C1 01          |add ecx,1  
00403B45  > |83F9 0A          |cmp ecx,0A                                     ;  比较是否全部匹配  
00403B48  . |74 15             |je short dumped_.00403B5F  
00403B4A  > |8000 0A          |add byte ptr ds:[eax],0A                     ;  ****** 来到这里   
00403B4D  . |83C6 01          |add esi,1  
00403B50  . |83C0 01          |add eax,1  
00403B53  . |3BF7             |cmp esi,edi  
00403B55  . |^.0F82 46FEFFFF    \jb dumped_.004039A1  
00403B5B  .   5B               pop ebx  
00403B5C  .   5E               pop esi  
00403B5D  .   5F               pop edi  
00403B5E  .   C3               retn  
00403B5F  >  \B9 90909090      mov ecx,90909090                             ;  匹配的话就将其 nop  
00403B64  .   8908             mov dword ptr ds:[eax],ecx  
00403B66  .   8948 04          mov dword ptr ds:[eax+4],ecx  
00403B69  .   66:8948 08       mov word ptr ds:[eax+8],cx  
00403B6D  >   5B               pop ebx  
00403B6E  >   5E               pop esi  
00403B6F  >   5F               pop edi  
00403B70  \\>   C3               retn  
```

现在我们就应该思考一下了既然原程序没有出现异常而本程序出现异常，说明加壳的程序必定是跳过了这个异常，使用 `ctrl + g` 跳转到 `0040604d` 往上翻两下。

我们在数据窗口中反汇编，发现：

```asm
00406049   .   85C0           test eax,eax                         ;  脱壳之后 eax = 0  
0040604B   .   75 01          jnz short dumped_.0040604E  
0040604D   ?   CC             int3  
```

我们在 `00406049` 处下硬件执行断点，重载修复后的程序，同时打开另一个 `od` 打开加壳的程序。先到达 `oep` 然后 `ctrl + g` 来到 `00406049` 处下硬件执行断点。

`f9` 运行来到 `00406049` 发现加壳的程序 `eax = 1`，而脱壳后的程序 `eax = 0`，一定是哪里影响了这个跳转。现在我们明确了只要跳过 `0040604d` 就可以了。简单的方法就是修改 `0040604b` 处的跳转，但是我们前面跟踪发现 `0040604b` 处的代码已经被 `smc` 过了，对应的字节都加了 `A`。这个我们可以在 `smc` 之前将其给成 `jz` 这样的话就可以跳过了。具体方法如下：

重新加载脱壳后的程序，在数据窗口 `ctrl ＋ g` 来到 `0040604b` 处将 `6b` 改成 `6a`，这样的话 `smc` 之后这里就可以跳过 `int3` 了。改过之后发现 `f9` 运行还有这样的错误发生在 `00406581`。同样的代码，为了防止后面也有类似的代码，我们重新加载程序 `ctrl + B` 搜索 `6B F7 C2`，找到三处 `0040604B`，`0040657E`，`00406842`，将其修改为：`6A F7 C2`。这样的话程序就正常运行了。

程序有校验，我们可以跟踪分析是哪个地方改了 `eax` 的值。经跟踪发现：

```asm
00406044     E8 F7F0FFFF        call QQLog.00405140                 ；是这个 call 改变了 eax 的值我们可以跟进去看看  
00406049     85C0               test eax,eax  
0040604B     75 01              jnz short QQLog.0040604E  
```

我们可以开两个 `od` 然后对比着跟踪，这样很容易发现关键。

```asm
00405140     55                 push ebp  
00405141     8BEC               mov ebp,esp  
00405143     6A FF              push -1  
00405145     68 1B994300        push QQLog.0043991B  
0040514A     64:A1 00000000     mov eax,dword ptr fs:[0]  
00405150     50                 push eax  
00405151     81EC A4010000      sub esp,1A4  
00405157     A1 0CC74400        mov eax,dword ptr ds:[44C70C]  
0040515C     33C5               xor eax,ebp  
0040515E     8945 E4            mov dword ptr ss:[ebp-1C],eax  
00405161     50                 push eax  
00405162     8D45 F4            lea eax,dword ptr ss:[ebp-C]  
00405165     64:A3 00000000     mov dword ptr fs:[0],eax  
0040516B     898D 60FEFFFF      mov dword ptr ss:[ebp-1A0],ecx  
00405171     C785 C8FEFFFF 000000>mov dword ptr ss:[ebp-138],0  
0040517B     C745 EC 00000000   mov dword ptr ss:[ebp-14],0  
00405182     B8 A8514000        mov eax,QQLog.004051A8  
00405187     8985 C8FEFFFF      mov dword ptr ss:[ebp-138],eax  
0040518D     B8 AB534000        mov eax,QQLog.004053AB  
00405192     8945 EC            mov dword ptr ss:[ebp-14],eax  
00405195     8B45 EC            mov eax,dword ptr ss:[ebp-14]  
00405198     50                 push eax  
00405199     8B8D C8FEFFFF      mov ecx,dword ptr ss:[ebp-138]  
0040519F     51                 push ecx  
004051A0     E8 CBE7FFFF        call QQLog.00403970  
004051A5     83C4 08            add esp,8  
004051A8     90                 nop  
004051A9     90                 nop  
004051AA     90                 nop  
004051AB     90                 nop  
004051AC     90                 nop  
004051AD     90                 nop  
004051AE     90                 nop  
004051AF     90                 nop  
004051B0     90                 nop  
004051B1     90                 nop  
004051B2     E8 19EDFFFF        call QQLog.00403ED0  
004051B7     C785 D0FEFFFF 000000>mov dword ptr ss:[ebp-130],0  
004051C1     C745 E8 00000000   mov dword ptr ss:[ebp-18],0  
004051C8     C685 DCFEFFFF 00    mov byte ptr ss:[ebp-124],0  
```
