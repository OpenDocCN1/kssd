
### 汇编代码分析

#### 初始检查逻辑

```
  01003DC4   |>  3BD6            cmp edx,esi
  01003DC6   |.  74 07           je short freecell.01003DCF
  01003DC8   |.  83FA 02         cmp edx,2
  01003DCB   |.  74 02           je short freecell.01003DCF
  01003DCD   |.  33F6            xor esi,esi
  01003DCF   |>  33C9            xor ecx,ecx
  01003DD1   |.  3BC6            cmp eax,esi
  01003DD3   |.  0F95C1          setne cl
  01003DD6   |.  8BC1            mov eax,ecx
  01003DD8   |>  5F              pop edi
  01003DD9   |.  5E              pop esi
  01003DDA   \.  C2 0800         retn 8
```

这里我们先定义几个变量：

```
int nShangPaiA, nShangPaiB; // 记录两个牌/4 的商，因为英文水平问题，查金山词霸也没找到商的英文，这里用拼音表示了
int nModPaiA, nModPaiB;     // 记录两个牌/4 的余
```

#### 牌值计算逻辑

```
  01003D8E   /$  8B4424 04       mov eax,dword ptr ss:[esp+4]          ;
  01003D92   |.  56              push esi
  01003D93   |.  57              push edi
  01003D94   |.  99              cdq
  01003D95   |.  6A 04           push 4
  01003D97   |.  59              pop ecx
  01003D98   |.  F7F9            idiv ecx
```

对应逻辑：
`nShangPaiA = paiA / 4;`
`nModPaiA = paiA % 4;`

```
  01003D9A   |.  6A 04           push 4
  01003D9C   |.  5F              pop edi
  01003D9D   |.  8BF0            mov esi,eax                           ;  esi = eax = 商
  01003D9F   |.  8B4424 10       mov eax,dword ptr ss:[esp+10]         ;  int temp 场景的牌 = 场景的牌
  01003DA3   |.  8BCA            mov ecx,edx                           ;  ecx = edx = 余
  01003DA5   |.  99              cdq
  01003DA6   |.  F7FF            idiv edi
```

对应逻辑：
`nShangPaiB = paiB / 4;`
`nModPaiB = paiB % 4;`

状态说明：
// 这个时候 `esi` == `nQuotientOfPaiA`, `ecx` == `nModOfPaiA`
// 这个时候 `eax` == `nQuotientOfPaiB`, `edx` == `nModOfPaiB`

#### 差值比较逻辑

```
  01003DA8   |.  2BC6            sub eax,esi                           ;  场景牌的商 - 中转站的商
```

对应逻辑：
`nShangPaiB -= nShangPaiA;`

```
  01003DAA   |.  33F6            xor esi,esi
  01003DAC   |.  46              inc esi
```

对应逻辑：
`int temp = 1;`

```
  01003DAD   |.  3BC6            cmp eax,esi
  01003DAF   |.  /74 04          je short freecell.01003DB5
  01003DB1   |.  |33C0           xor eax,eax
  01003DB3   |.  |EB 23          jmp short freecell.01003DD8
```

对应逻辑：
```
if (nShangPaiB != i)
{
  nShangPaiB = 0;
  return false;
}
```
// 此处判断是否等于 1，也就是 `paiB - paiA` 是否只差 1，如果差 1 就去比较花色，如果不为 0，那就没戏，直接走人了

#### 花色检查逻辑

```
  01003DB5   |>  \3BCE           cmp ecx,esi
  01003DB7   |.  74 09          je short freecell.01003DC2
  01003DB9   |.  83F9 02        cmp ecx,2
  01003DBC   |.  74 04          je short freecell.01003DC2
  01003DBE   |.  33C0           xor eax,eax
  01003DC0   |.  EB 02          jmp short freecell.01003DC4
  01003DC2   |>  8BC6           mov eax,esi
```

对应逻辑：
```
if(nModPaiA != i || nModPaiA != 2)
{
  nShangPaiB = 0; // 说明此牌不是红色
}
else
{
  nShangPaiB = i; // 说明此牌是红色的
}
```
// 这部分是用来比较花色的，先与 1 比较，再与 2 比较，上面我给列出过牌表，可以知道，1 代表方片，2 代表红桃，而两者都是红色牌系，所以我们有：这里是比较花色是否是红色系的，如果是红色系的就赋值 1,否则就是黑色系的，赋值 0

```
  01003DC4   |>  3BD6           cmp edx,esi
  01003DC6   |.  74 07          je short freecell.01003DCF
  01003DC8   |.  83FA 02        cmp edx,2
  01003DCB   |.  74 02          je short freecell.01003DCF
  01003DCD   |.  33F6           xor esi,esi
  01003DCF   |>  33C9           xor ecx,ecx
```

对应逻辑：
```
if(nModPaiB != i || nModPaiB != 2)
{
  i = 0;
}
else
{
  nModPaiA = 0;
}
```
// 用来检测 `PaiB` 的花色

#### 结果返回

```
  01003DD1   |.  3BC6           cmp eax,esi
  01003DD3   |.  0F95C1         setne cl
  01003DD6   |.  8BC1           mov eax,ecx
  01003DD8   |>  5F             pop edi
  01003DD9   |.  5E             pop esi
  01003DDA   \.  C2 0800        retn 8
```

// 这个时候 `nShangPaiB` 里记录了 `paiB` 的花色（`eax`），`nModPaiA` 里记录了 `paiA` 的花色（`ecx`），然后通过 `setne cl` 来返回其 0 或 1 的值，然后外部通过其真值或非真值来做相应处理，到了这里大家应该都很清楚了。

### 总结与讨论

我们应该可以看出整个函数是用 `asm` 直接写的（猜测，如果说的不对，望请高手们指出啊！），写完了感觉是不是没有什么秘密感了，如果自己以后做此类程序的时候，或许可以拿来一用也无妨。

哈，经过这一部分的代码，我们可以证明开文提到的空挡接龙判断“死亡”的方法是有缺陷的，比如牌面中任意 2 列牌的最下方牌移动到 `A` 可以，再从 `A` 移动到 `B` 也可以的话，那么游戏就死不了。

**附言：**

还有一个值得注意的地方就是移动多张牌的操作，之所以不写，倒不是篇幅的问题，而是因为这个操作嵌套了很多 `DC` 操作（这就是为什么移动牌的时候你会看到牌从回收站闪到其他牌下面的原因），其中又嵌套了其他操作，拆分起来比较麻烦，所以我也懒得写了。

再有就是最终牌的问题，我们知道，当最终牌从 `A` - `K` 依次填上去的时候，下面的那张牌就覆盖了，所以我们只需要记录最上面那张牌的 `num` 和花色就 `ok` 了，空挡也是这么做的，地址我忘了多少了，知道即可

给出几个已知的全局变量地址：

*   `1007868`: 用于记录纸牌的高度（固定值）
*   `1008370`: 用于记录纸牌的宽度（固定值）
*   `1007800`: 用于存放剩余牌数
*   `....`

有兴趣各位写个空挡出来 `(*^__^*)` 嘻嘻…… 记得发给我哦。。

### 经验总结

自打偶的处女贴“扫雷”得到老大们认可后，自己的信心倍增，恩，我会继续努力的！再此也特别感谢 `Backer` 老师的深深教导，听说 `Backer` 老师开培训班了，在这里我就祝老师公司越办越好，多多培养新新人才！（哈哈，看着像打广告的样子... 仁者见仁,智者见智吧!）

由于是一气呵成，文中难免有不正确的地方，望大家多批评指教！谢先！

呼。。。一口气写了这么一大堆，看看表快 12 点了，睡觉觉咯！

### 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`2007 年 12 月 12 日 9:40:39`

`4dd466d0e1e0cc123afbe504da2462ca`
