
# NoodleCrypt V2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6380.htm`](https://www.pediy.com/kssd/pediy06/pediy6380.htm)

NoodleCrypt V2.0 脱壳——Win98 的 Notepad

下载地址：`http://protools.anticrack.de/files/packers/noodlecrypt.zip`

软件大小：18 KB

**【软件简介】**：Win9x compatible. Anti ProcDump full dump features. Anti API hooks. Code and import sections encryption. 应该是属于 Crypters/Protectors 类型的壳啦。

**【作者声明】**：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

**【调试环境】**：Win98、Ollydbg1.09、PEiD、LordPE、ImportREC 1.6

### 【脱壳过程】

用 Ollydbg 手动脱壳，老规矩：载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

注：以下没有说明皆用 F8 走，呵呵。

```
0040D000     EB 01                 jmp short NOTEPAD.0040D003
                                  =====>进入 OD 后断在这！
0040D002     9A E8760000 00EB      call far EB00:000076E8
0040D003     E8 76000000           call NOTEPAD.0040D07E
0040D008     EB 01                 jmp short NOTEPAD.0040D00B
0040D00B     E8 65000000           call NOTEPAD.0040D075
0040D010     EB 01                 jmp short NOTEPAD.0040D013
0040D013     E8 7D000000           call NOTEPAD.0040D095
0040D018     EB 01                 jmp short NOTEPAD.0040D01B
0040D01B     E8 55000000           call NOTEPAD.0040D075
0040D020     EB 01                 jmp short NOTEPAD.0040D023
0040D023     E8 43040000           call NOTEPAD.0040D46B
0040D028     EB 01                 jmp short NOTEPAD.0040D02B
0040D02B     E8 E1000000           call NOTEPAD.0040D111
                                  =====>F7 进入
```

```
0040D111    /EB 01                 jmp short NOTEPAD.0040D114
0040D114     E8 5CFFFFFF           call NOTEPAD.0040D075
0040D119     EB 01                 jmp short NOTEPAD.0040D11C
0040D11C     8BB7 10170000         mov esi,dword ptr ds:[edi+1710]
0040D122     EB 01                 jmp short NOTEPAD.0040D125
0040D125     81C6 00100000         add esi,1000
0040D12B     EB 01                 jmp short NOTEPAD.0040D12E
0040D12E     8B87 08170000         mov eax,dword ptr ds:[edi+1708]
0040D134     0387 10170000         add eax,dword ptr ds:[edi+1710]
0040D13A     EB 01                 jmp short NOTEPAD.0040D13D
0040D13D     2D 00100000           sub eax,1000
0040D142     8BF8                 mov edi,eax
0040D144     EB 01                 jmp short NOTEPAD.0040D147
0040D147     B9 00040000           mov ecx,400
0040D14C     EB 01                 jmp short NOTEPAD.0040D14F
0040D14F     F3:A5                 rep movs dword ptr es:[edi],dword ptr ds:[esi]
0040D151     E8 1FFFFFFF           call NOTEPAD.0040D075
0040D156     EB 01                 jmp short NOTEPAD.0040D159
0040D159     8BF7                 mov esi,edi
0040D15B     EB 01                 jmp short NOTEPAD.0040D15E
0040D15E     8B87 10170000         mov eax,dword ptr ds:[edi+1710]
0040D164     EB 01                 jmp short NOTEPAD.0040D167
0040D167     8BF8                 mov edi,eax
0040D169     EB 01                 jmp short NOTEPAD.0040D16C
0040D16C     81C7 00100000         add edi,1000
0040D172     EB 01                 jmp short NOTEPAD.0040D175
0040D175     0F6ECF               movd mm1,edi
0040D178     B9 00040000           mov ecx,400
0040D17D     F3:A5                 rep movs dword ptr es:[edi],dword ptr ds:[esi]
0040D17F     EB 01                 jmp short NOTEPAD.0040D182
0040D182     05 30100000           add eax,1030
0040D187     EB 01                 jmp short NOTEPAD.0040D18A
0040D18A     50                   push eax
0040D18B     C3                   retn
                                  =====>返回到 00401030
```

```
00401030    /EB 01                 jmp short NOTEPAD.00401033
00401033     E8 3D000000           call NOTEPAD.00401075
00401038     EB 01                 jmp short NOTEPAD.0040103B
0040103B     E8 EB010000           call NOTEPAD.0040122B
00401040     EB 01                 jmp short NOTEPAD.00401043
00401043     E8 2C040000           call NOTEPAD.00401474
00401048     EB 01                 jmp short NOTEPAD.0040104B
0040104B     E8 25000000           call NOTEPAD.00401075
00401050     EB 01                 jmp short NOTEPAD.00401053
00401053     E8 02040000           call NOTEPAD.0040145A
00401058     EB 01                 jmp short NOTEPAD.0040105B
0040105B     E8 19070000           call NOTEPAD.00401779
00401060     EB 01                 jmp short NOTEPAD.00401063
00401063     E8 9C000000           call NOTEPAD.00401104
                                  =====>F7 进入
00401068     EB 01                 jmp short NOTEPAD.0040106B
```

```
00401104     E8 6CFFFFFF           call NOTEPAD.00401075
00401109     83C0 68               add eax,68
0040110C     EB 01                 jmp short NOTEPAD.0040110F
0040110F     50                   push eax
00401110     C3                   retn
                                  =====>返回到 0040D068
```

```
0040D068    /EB 01                 jmp short NOTEPAD.0040D06B
0040D06B     E8 9C060000           call NOTEPAD.0040D70C
                                  =====>F7 进入
```

```
0040D70C     0F7EF8               movd eax,mm7
0040D70F     EB 01                 jmp short NOTEPAD.0040D712
0040D712     8BF8                 mov edi,eax
0040D714     E8 40000000           call NOTEPAD.0040D759
                                  =====>F7 进入
```

```
0040D759     97                   xchg eax,edi
0040D75A     EB 01                 jmp short NOTEPAD.0040D75D
0040D75D     0F6EF0               movd mm6,eax
0040D760     55                   push ebp
0040D761     EB 01                 jmp short NOTEPAD.0040D764
0040D764     E8 0CF9FFFF           call NOTEPAD.0040D075
0040D769     EB 01                 jmp short NOTEPAD.0040D76C
0040D76C     E8 1BFAFFFF           call NOTEPAD.0040D18C
                                  =====>F7 进入
```

```
0040D18C    /EB 01                 jmp short NOTEPAD.0040D18F
0040D18F     E8 E1FEFFFF           call NOTEPAD.0040D075
0040D194     8BB7 10170000         mov esi,dword ptr ds:[edi+1710]
0040D19A     EB 01                 jmp short NOTEPAD.0040D19D
0040D19D     81C6 00100000         add esi,1000
0040D1A3     EB 01                 jmp short NOTEPAD.0040D1A6
0040D1A6     8B87 08170000         mov eax,dword ptr ds:[edi+1708]
0040D1AC     0387 10170000         add eax,dword ptr ds:[edi+1710]
0040D1B2     EB 01                 jmp short NOTEPAD.0040D1B5
0040D1B5     2D 00100000           sub eax,1000
0040D1BA     8BF8                 mov edi,eax
0040D1BC     EB 01                 jmp short NOTEPAD.0040D1BF
0040D1BF     87F7                 xchg edi,esi
0040D1C1     B9 00040000           mov ecx,400
0040D1C6     EB 01                 jmp short NOTEPAD.0040D1C9
0040D1C9     F3:A5                 rep movs dword ptr es:[edi],dword ptr ds:[esi]
0040D1CB     E8 A5FEFFFF           call NOTEPAD.0040D075
0040D1D0     8B97 00170000         mov edx,dword ptr ds:[edi+1700]
0040D1D6     EB 01                 jmp short NOTEPAD.0040D1D9
0040D1D9     0397 10170000         add edx,dword ptr ds:[edi+1710]
0040D1DF     EB 01                 jmp short NOTEPAD.0040D1E2
0040D1E2     8B8F 04170000         mov ecx,dword ptr ds:[edi+1704]
0040D1E8     C00A 07               ror byte ptr ds:[edx],7
0040D1EB     EB 01                 jmp short NOTEPAD.0040D1EE
0040D1EE     8002 04               add byte ptr ds:[edx],4
0040D1F1     EB 01                 jmp short NOTEPAD.0040D1F4
0040D1F4     C002 04               rol byte ptr ds:[edx],4
0040D1F7     EB 01                 jmp short NOTEPAD.0040D1FA
```



```
0040D1FA     42                     inc edx   

0040D1FB     EB 01                 jmp short NOTEPAD.0040D1FE   

0040D1FE     ^E2 E8                loopd short NOTEPAD.0040D1E8   

                                       ==== >F4 下去跳出 LOOP   

0040D200     8B87 0C170000         mov eax,dword ptr ds:\[edi+170C\]   

                                       ==== >EAX\=000010CC   

0040D206     EB 01                 jmp short NOTEPAD.0040D209   

0040D209     0387 10170000         add eax,dword ptr ds:\[edi+1710\]   

                                       ==== >EAX\=000010CC + 00400000=004010CC   这就是 OEP 值   

0040D20F     50                    push eax   

0040D210     C3                    retn   

                                       ==== >飞向光明之巅！
```

———————————————————————

```
004010CC     55                    push ebp   

                                       ==== >在这儿用 LordPE 完全 DUMP 这个进程   

004010CD     8BEC                  mov ebp,esp   

004010CF     83EC 44              sub esp,44   

004010D2     56                    push esi   

004010D3     FF15 E4634000        call dword ptr ds:\[4063E4\]   

004010D9     8BF0                 mov esi,eax   

004010DB     8A00                 mov al,byte ptr ds:\[eax\]   

004010DD     3C 22                cmp al,22   

004010DF     75 1B                jnz short NOTEPAD.004010FC
```

———————————————————————

运行`ImportREC 1.6`，选择这个进程。把`OEP`改为`000010CC`，点`IT AutoSearch`，点“Get Import”，`FixDump`。用`LordPE`清除`.Ncryo`和`.De-vir`两个区段，然后重建`PE`。OK，正常运行！`60K`->`41.3K`。

—————————————————————————————————

```
         ,     \_/   
       /| \_.-~/            \_     ,        青春都一饷
      ( /~   /               ~-.\_ |
      \`\\   \_/                   ~ )          忍把浮名
   _-~~~-.)  )\_\_/;;,.            \_  //'    
  /'\_,    --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂
 \`~ \_( ,\_..-- (      ,;'' /    ~--   /.\_\`   
  /~~//'   /' \`~             ) /--..\_, )\_  \`~    
  "  \`~"  "       \`"       /~'\`    \`\\~~   


             "     "   "~'  ""
```

`Cracked By 巢水工作坊——fly [OCN][FCG]`

`2003-10-15  01:20`

`4dd466d0e1e0cc123afbe504da2462ca`
