
# [原创]RLPacK 1.21全保护学习笔记

【文章作者】: [UnPacKcN]XuZhenG

【作者邮箱】: xuzheng1111@126.com

【作者主页】: 6deK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6^5N6i4A6Z5k6h3&6Y4x3e0p5I4x3b7.\`.<br>be7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2\*7i4K6u0W2L8%4y4S2i4K6u0W2M7r3H3.\`.<br>4a9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4S2\*7i4K6u0W2j5X3g2W2i4K6u0W2M7r3H3.\`.

【软件名称】: RLPacK Full Protection 1.21

【下载地址】: 自己去UnPacKcN下载

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!<br>如有人将此用入商业用途，给作者造成损失本人概不负责。

---

## 【详细过程】

上次在壳狂下载了性价比很高的壳哈... RLPacK全保护，既然ASPr TMD 和穿山甲 被壳狂的高手 玩得乱转... 我们菜鸟捡个漏... 捏个软柿子... `OD`载入 `F9`第一次运行，状态栏提示`Terminated`，试了N个调试器隐藏插件和N个`OD`都不行... 那怎么办呢？那只能`Step`往下走了... 大胆得往前走啊，往前走，莫回头~~~~~~~~~~ 不好意思，献丑了，言归正传...

入口如下：

```
01001000 > $ /EB 06         jmp short 复件_cal.01001008
01001002   . |68 2EA80000   push 0A82E
01001007   . |C3            retn
01001008   > \9C            pushfd
01001009   .  60            pushad
0100100A   .  E8 02000000   call 复件_cal.01001011
0100100F   .  33C0          xor eax,eax
01001011   $  8BC4          mov eax,esp
01001013   .  83C0 04       add eax,4
01001016   .  93            xchg eax,ebx
01001017   .  8BE3          mov esp,ebx
01001019   .  8B5B FC       mov ebx,dword ptr ds:[ebx-4]
0100101C   .  81EB 3F904000 sub ebx,40903F
01001022   .  61            popad
01001023   .  9D            popfd
01001024   .- E9 D3B50200   jmp 复件_cal.0102C5FC
```

### 过五关斩六将之 Anti(真的挺邪恶的)被斩

一路狂F7,过`JMP`后到了这里

```
0102C5FC    60              pushad
0102C5FD    E8 00000000     call 复件_cal.0102C602
0102C602    83C4 04         add esp,4
0102C605    8B6C24 FC       mov ebp,dword ptr ss:[esp-4]
0102C609    E8 8B020000     call 复件_cal.0102C899
0102C60E    E8 74240000     call 复件_cal.0102EA87
0102C613    E8 88430000     call 复件_cal.010309A0
0102C618    837C24 28 01    cmp dword ptr ss:[esp+28],1
0102C61D    75 0C           jnz short 复件_cal.0102C62B
0102C61F    8B4424 24       mov eax,dword ptr ss:[esp+24]
0102C623    8985 7E470000   mov dword ptr ss:[ebp+477E],eax
0102C629    EB 0C           jmp short 复件_cal.0102C637
0102C62B    8B85 7A470000   mov eax,dword ptr ss:[ebp+477A]
0102C631    8985 7E470000   mov dword ptr ss:[ebp+477E],eax
0102C637    E8 100D0000     call 复件_cal.0102D34C
0102C63C    E8 6F240000     call 复件_cal.0102EAB0
0102C641    E8 1F230000     call 复件_cal.0102E965                ;Anti1
0102C646    8DB5 AD530000   lea esi,dword ptr ss:[ebp+53AD]
0102C64C    8D9D 17030000   lea ebx,dword ptr ss:[ebp+317]
0102C652    33FF            xor edi,edi
0102C654    E8 3B3D0000     call 复件_cal.01030394                ;检测调试器Call
```

### 关于RLPacK的Anti

#### `Anti1`

这个Anti很容易过哈,很简单，把下面有注释的那一行`Nop`掉就可以...

```
0102E965    60              pushad
0102E966    81BD C2470000 0>cmp dword ptr ss:[ebp+47C2],ABBC680>
0102E970    75 11           jnz short 复件_cal.0102E983
省略一部分代码
0102E9CA    64:8925 0000000>mov dword ptr fs:[0],esp
0102E9D1    33C0            xor eax,eax
0102E9D3    8700            xchg dword ptr ds:[eax],eax         ; Anti-Exchange 0 and ???????? Please Nop it And Step Over it Easily.
0102E9D5    64:8F05 0000000>pop dword ptr fs:[0]
0102E9DC    83C4 04         add esp,4
0102E9DF    61              popad
0102E9E0    C3              retn
```

#### `检测调试器Call`

用了很多复杂的API函数哈，我慢慢解释，开头还有两个小插曲，既然也在Anti里面，我也简单说下...
