
### 00522BC1

`00522BC1     FFD0                call eax                ; dumped\_.0055B1D8//进入`

`00522BC3     803D 48335600 00    cmp byte ptr ds:[563348],0`

`00522BCA     74 43               je short dumped\_.00522C0F`

`00522BA1-00522BA7`处的循环是使`0055B1D8`处的12个字节值各自-1：

```
0055B1D8  5C 59 8C 41 03 8C 01 00 31 90 01 00 E4 8D 40 00  //原先值
0055B1D8  5B 58 8B 40 02 8B 00 FF 30 8F 00 FF E3 8D 40 00  //改变值
```

进入`00522BC1 call eax ;  dumped\_.0055B1D8`看看：

```
0055B1D8     5B                  pop ebx
0055B1D9     58                  pop eax
0055B1DA     8B40 02             mov eax,dword ptr ds:[eax+2]
0055B1DD     8B00                mov eax,dword ptr ds:[eax]
0055B1DF     FF30                push dword ptr ds:[eax]
0055B1E1     8F00                pop dword ptr ds:[eax]//[eax]=[77E53C49]=0868146A  异常！![icon_sad.gif](https://www.kanxue.com/images/smiles/icon_sad.gif)
0055B1E3     FFE3                jmp ebx
```

――――――――――――――――――――――――

### 2、0052487D 处     这里和上面的一样

```
00524855     BB 0D000000         mov ebx,0D
0052485A     B8 D8B15500         mov eax,dumped\_.0055B1D8//[eax]=0055B1D8
0052485F     FE08                dec byte ptr ds:[eax]
00524861     40                  inc eax
00524862     4B                  dec ebx//EBX=D
00524863     75 FA               jnz short dumped\_.0052485F//循环
00524865     33D2                xor edx,edx
00524867     55                  push ebp
00524868     68 D54B5200         push dumped\_.00524BD5
0052486D     64:FF32             push dword ptr fs:[edx]
00524870     64:8922             mov dword ptr fs:[edx],esp
00524873     68 08544000         push <jmp.&kernel32.ExitThread>
00524878     B8 D8B15500         mov eax,dumped\_.0055B1D8
0052487D     FFD0                call eax  ; dumped\_.0055B1D8//这里也使0055B1E1处发生了异常！
```

`0052485F-00524863`处的循环是使`0055B1D8`处的12个字节值各自-1：

```
0055B1D8  5C 59 8C 41 03 8C 01 00 31 90 01 00 E4 8D 40 00  //原先值
0055B1D8  5B 58 8B 40 02 8B 00 FF 30 8F 00 FF E3 8D 40 00  //改变值
```

――――――――――――――――――――――――

### 3、解决方法

`0055B1E1     8F00                pop dword ptr ds:[eax]//异常处`

偶偷了一下懒 ![icon_smile.gif](https://www.kanxue.com/images/smiles/icon_smile.gif)   使程序把上面的异常处NOP掉！

```
0055B1E1  90 01  //原先值
0055B1E1  91 91  //修改值
```

改了`0055B1E1`处的2个字节，呵呵，终于运行正常了！ ![icon_smile.gif](https://www.kanxue.com/images/smiles/icon_smile.gif)

―――――――――――――――――――――――――――――――――

```
         ,     \_/

        /| \_.-~/            \_     ,        青春都一饷

       ( /~   /              ~-.\_ |

       `\  \_/                   ~ )          忍把浮名

   \_-~~~-.)  )\_\_/;;,.          \_  //'

  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂

 `~ \_( ,\_..-- (     ,;'' /    ~--   /.\_`

  /~~//'   /' `~         ) /--..\_, )\_  `~

  "  `~"  "      `"      /~'`    `\~~

                         "     "   "~'  ""

            Cracked By： jingulong

            整      理： fly

                2004-01-15  17:20
```
