
```
0101A600    60                 pushad
0101A601    3D 0D020000      cmp     eax, 20D
0101A606    75 7D            jnz     short winmine\_.0101A685
0101A608    C705 00A70101 0>mov     dword ptr [101A700], 0
0101A612    A1 38530001      mov     eax, dword ptr [1005338]
0101A617    3905 00A70101    cmp     dword ptr [101A700], eax
0101A61D    7D 66            jge     short winmine\_.0101A685
0101A61F    C705 04A70101 0>mov     dword ptr [101A704], 0
0101A629    A1 34530001      mov     eax, dword ptr [1005334]
0101A62E    3905 04A70101    cmp     dword ptr [101A704], eax
0101A634    7D 46            jge     short winmine\_.0101A67C
0101A636    A1 00A70101      mov     eax, dword ptr [101A700]
0101A63B    C1E0 05          shl     eax, 5
0101A63E    05 61530001      add     eax, winmine\_.01005361
0101A643    0305 04A70101    add     eax, dword ptr [101A704]
0101A649    8B18             mov     ebx, dword ptr [eax]
0101A64B    C0EB 04          shr     bl, 4
0101A64E    80FB 08          cmp     bl, 8
0101A651    74 20            je      short winmine\_.0101A673
0101A653    A1 00A70101      mov     eax, dword ptr [101A700]
0101A658    83C0 01          add     eax, 1
0101A65B    50               push    eax
0101A65C    A1 04A70101      mov     eax, dword ptr [101A704]
0101A661    83C0 01          add     eax, 1
0101A664    50               push    eax
0101A665    31DB             xor     ebx, ebx
0101A667    43               inc     ebx
0101A668    891D 98570001    mov     dword ptr [1005798], ebx
0101A66E    E8 9589FEFF      call    winmine\_.01003008
0101A673    8305 04A70101 0>add     dword ptr [101A704], 1
0101A67A   ^ EB AD          jmp     short winmine\_.0101A629
0101A67C    8305 00A70101 0>add     dword ptr [101A700], 1
0101A683   ^ EB 8D          jmp     short winmine\_.0101A612
0101A685    61               popad
0101A686    3D 0C020000      cmp     eax, 20C
0101A68B   - E9 6D77FEFF     jmp     winmine\_.01001DFD
```

### 修改消息循环

秒杀功能代码已经写好，最后一步工作就是修改菜单项的消息循环，这步很容易，修改`01001DF8`处的代码为`jmp 0101A600`即可。

### 测试和保存

保存只需要使用`OD`的“保存到文件”功能即可。

现在点击一下秒杀功能看看，发现几个问题：

- 秒杀功能没有错但是我们还需要右键点击一下有雷的棋子
- 如果游戏已经进行了一些操作，比如在无雷的棋子上插上了小旗子秒杀功能就会出问题

这几个问题都很好解决，在我们添加的代码中再加上几个判断以及适当的右键的点击操作就轻松搞定，有兴趣的可以自己实现一下。
