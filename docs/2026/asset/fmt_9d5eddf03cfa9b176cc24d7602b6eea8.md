
`0045878A`     `83C7 0A`              `add edi,0A`

`0045878D`     `EB 20`                `jmp short SoftDefe.004587AF`

如果这上面的3个地方跳转的话，则提示 时间错误 等等，然后`OVER`!

――――――――――――――――――――――――

## 二、寻找OEP、避开Stolen Code

`OK`，偶们开始快速寻找`OEP`  `:-D`  程序接下来要干的“坏事”是`Stolen Code`，`:-)` 恰恰是`Stolen Code`使其轻易暴露了`OEP`，让偶们省了很多麻烦去寻觅`OEP`

`Ctrl+F`在整个区段搜索命令：`mov byte ptr ss:[esp+1C],dl`   找到下面的地方！

在`00456A9E`（或其下的代码）处下 硬件执行 断点！切记：不要直接用`F2`下断！会`OVER`的！`F9`运行，顺利断在`00456A9E`处。

让偶们看看 寄存器 窗口，现在的`EDI=00405248`  猜猜看，这个是什么？这个就是`OEP`啦！ `:-)`

程序会把`OEP`处的14个字节改掉！前8个字节改为`00 00 00 00 00 00 00 00`

偶们现在要做的就是让`SoftDefender`别给原来的程序动手脚！`Come On`!

```
00456A9E     885424 1C            mov byte ptr ss:[esp+1C],dl//这里下 硬件执行 断点！:-)
00456AA2     884C24 1D            mov byte ptr ss:[esp+1D],cl
00456AA6     884424 1E            mov byte ptr ss:[esp+1E],al
00456AAA     C64424 1F 6A         mov byte ptr ss:[esp+1F],6A
00456AAF     C64424 20 FF         mov byte ptr ss:[esp+20],0FF
00456AB4     885424 24            mov byte ptr ss:[esp+24],dl
00456AB8     884C24 25            mov byte ptr ss:[esp+25],cl
00456ABC     884424 26            mov byte ptr ss:[esp+26],al
00456AC0     C64424 27 83         mov byte ptr ss:[esp+27],83
00456AC5     C64424 28 C4         mov byte ptr ss:[esp+28],0C4
00456ACA     C64424 29 F4         mov byte ptr ss:[esp+29],0F4
00456ACF     C64424 2A B8         mov byte ptr ss:[esp+2A],0B8
00456AD4     885424 34            mov byte ptr ss:[esp+34],dl
00456AD8     884C24 35            mov byte ptr ss:[esp+35],cl
00456ADC     884424 36            mov byte ptr ss:[esp+36],al
00456AE0     C64424 37 83         mov byte ptr ss:[esp+37],83
00456AE5     C64424 38 C4         mov byte ptr ss:[esp+38],0C4
00456AEA     C64424 39 F4         mov byte ptr ss:[esp+39],0F4
00456AEF     C64424 3A 53         mov byte ptr ss:[esp+3A],53
00456AF4     C64424 3B 56         mov byte ptr ss:[esp+3B],56
00456AF9     C64424 3C 57         mov byte ptr ss:[esp+3C],57
00456AFE     C64424 3D B8         mov byte ptr ss:[esp+3D],0B8
00456B03     885424 2C            mov byte ptr ss:[esp+2C],dl
00456B07     884C24 2D            mov byte ptr ss:[esp+2D],cl
00456B0B     884424 2E            mov byte ptr ss:[esp+2E],al
00456B0F     C64424 2F 83         mov byte ptr ss:[esp+2F],83
00456B14     C64424 30 C4         mov byte ptr ss:[esp+30],0C4
00456B19     C64424 31 F8         mov byte ptr ss:[esp+31],0F8
00456B1E     C64424 32 B8         mov byte ptr ss:[esp+32],0B8
00456B23     885424 40            mov byte ptr ss:[esp+40],dl
00456B27     884C24 41            mov byte ptr ss:[esp+41],cl
00456B2B     884424 42            mov byte ptr ss:[esp+42],al
00456B2F     C64424 43 83         mov byte ptr ss:[esp+43],83
00456B34     C64424 44 C4         mov byte ptr ss:[esp+44],0C4
00456B39     C64424 45 F8         mov byte ptr ss:[esp+45],0F8
00456B3E     C64424 46 53         mov byte ptr ss:[esp+46],53
00456B43     C64424 47 56         mov byte ptr ss:[esp+47],56
00456B48     C64424 48 57         mov byte ptr ss:[esp+48],57
00456B4D     C64424 49 B8         mov byte ptr ss:[esp+49],0B8
00456B52     E8 92270000          call SoftDefe.004592E9
00456B57     50                   push eax
00456B58     E8 C8270000          call SoftDefe.00459325
00456B5D     6A 20                push 20
00456B5F     6A 08                push 8
00456B61     894424 14            mov dword ptr ss:[esp+14],eax
00456B65     E8 7F270000          call SoftDefe.004592E9
00456B6A     50                   push eax
00456B6B     E8 B5270000          call SoftDefe.00459325
00456B70     8BF0                 mov esi,eax
00456B72     6A 05                push 5
00456B74     894424 14            mov dword ptr ss:[esp+14],eax
00456B78     8D4424 18            lea eax,dword ptr ss:[esp+18]
00456B7C     50                   push eax
00456B7D     57                   push edi
00456B7E     E8 ADF4FFFF          call SoftDefe.00456030
00456B83     85C0                 test eax,eax
00456B85     75 5A                jnz short SoftDefe.00456BE1//注意：这里应跳！！◆
00456B87     C606 58              mov byte ptr ds:[esi],58
00456B8A     46                   inc esi
00456B8B     6A 0F                push 0F
00456B8D     57                   push edi
00456B8E     56                   push esi
00456B8F     E8 7DF3FFFF          call SoftDefe.00455F11
00456B94     83C6 0F              add esi,0F
00456B97     6A 0F                push 0F
00456B99     6A 00                push 0
00456B9B     57                   push edi
00456B9C     C606 FF              mov byte ptr ds:[esi],0FF
00456B9F     C646 01 E0           mov byte ptr ds:[esi+1],0E0
00456BA3     E8 97F3FFFF          call SoftDefe.00455F3F     //Stolen Code   :-(
00456BA8     8D47 09              lea eax,dword ptr ds:[edi+9]
00456BAB     8D4C24 0C            lea ecx,dword ptr ss:[esp+C]
00456BAF     6A 04                push 4
00456BB1     51                   push ecx
00456BB2     C600 FF              mov byte ptr ds:[eax],0FF
00456BB5     40                   inc eax
00456BB6     C600 15              mov byte ptr ds:[eax],15
00456BB9     40                   inc eax
00456BBA     50                   push eax
00456BBB     E8 51F3FFFF          call SoftDefe.00455F11
00456BC0     8D5424 10            lea edx,dword ptr ss:[esp+10]
00456BC4     6A 04                push 4
00456BC6     8B4424 10            mov eax,dword ptr ss:[esp+10]
00456BCA     52                   push edx
00456BCB     50                   push eax
00456BCC     E8 40F3FFFF          call SoftDefe.00455F11
00456BD1     8B43 18              mov eax,dword ptr ds:[ebx+18]
00456BD4     5F                   pop edi
00456BD5     83C0 09              add eax,9
00456BD8     5E                   pop esi
00456BD9     8943 18              mov dword ptr ds:[ebx+18],eax
00456BDC     5B                   pop ebx
00456BDD     83C4 38              add esp,38
00456BE0     C3                   retn
00456BE1     8D4C24 1C            lea ecx,dword ptr ss:[esp+1C]
00456BE5     6A 07                push 7
00456BE7     51                   push ecx
00456BE8     57                   push edi
00456BE9     E8 42F4FFFF          call SoftDefe.00456030
00456BEE     85C0                 test eax,eax
00456BF0     0F84 95000000        je SoftDefe.00456C8B
00456BF6     8D5424 24            lea edx,dword ptr ss:[esp+24]
00456BFA     6A 07                push 7
00456BFC     52                   push edx
00456BFD     57                   push edi
00456BFE     E8 2DF4FFFF          call SoftDefe.00456030
00456C03     85C0                 test eax,eax
00456C05     0F84 80000000        je SoftDefe.00456C8B
00456C0B     8D4424 2C            lea eax,dword ptr ss:[esp+2C]
00456C0F     6A 0A                push 0A
00456C11     50                   push eax
00456C12     57                   push edi
00456C13     E8 18F4FFFF          call SoftDefe.00456030
00456C18     85C0                 test eax,eax
00456C1A     74 15                je short SoftDefe.00456C31//注意：改变标志位Z=0 使这里不跳！！◆
00456C1C     8D4C24 38            lea ecx,dword ptr ss:[esp+38]
00456C20     6A 0A                push 0A
00456C22     51                   push ecx
00456C23     57                   push edi
00456C24     E8 07F4FFFF          call SoftDefe.00456030
00456C29     85C0                 test eax,eax
00456C2B     0F85 A8000000        jnz SoftDefe.00456CD9//注意：这里应跳！！◆
00456C31     C606 59              mov byte ptr ds:[esi],59
00456C34     46                   inc esi
00456C35     6A 0E                push 0E
00456C37     57                   push edi
00456C38     56                   push esi
00456C39     E8 D3F2FFFF          call SoftDefe.00455F11
00456C3E     83C6 0E              add esi,0E
00456C41     6A 0E                push 0E
00456C43     6A 00                push 0
00456C45     57                   push edi
00456C46     C606 FF              mov byte ptr ds:[esi],0FF
00456C49     C646 01 E1           mov byte ptr ds:[esi+1],0E1
```
