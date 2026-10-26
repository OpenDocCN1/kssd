
### 代码片段 1

`01004338` `|.  83FF 09` `|cmp edi,9` `;  i < 9`

`0100433B` `|.^ 72 D9` `\jb short freecell.01004316`

```c
for (i = 1; i < 9; i++)
{
  for (j = 1; j < 9; j++)
  {
    if (i == j)
    {
      continue;
    }
    if(IsCanMove(aryCards[i], aryCards[j]))
    {
      nFlag++;
    }
  }
}
```
// 同上，只不过是牌面与牌面之间的互相比较看是否能移动，有能移动的`nFlag++`

`0100433D` `|.  837D FC 00` `cmp dword ptr ss:[ebp-4],0` `;  如果标志位为0 说明游戏over`

`01004341` `|.  5F` `pop edi`

`01004342` `|.  76 3D` `jbe short freecell.01004381`

```c
if (nFlag <= 0)
{
  goto OVER;
}
```
 // 如果从一开始的 `0100424F` 处，到这里`nFlag(ebp - 4)`都没有任何赋值，说明游戏死定了！，直接跳到Dialog显示over

`01004344` `|.  837D FC 01` `cmp dword ptr ss:[ebp-4],1`

`01004348` `|.  75 79` `jnz short freecell.010043C3`

```c
if (nFlag != 1)
{
  return;
}
```
 // 比较是否等于1，不等于就走人，说明游戏死不了，为什么？其实这又要说到`nFlag(ebp - 4)`这个值的含义了，其实它是记录了可以移牌的个数，如果是0个，那就是over，如果是1个，说明暂时还死不了，不过这个时候有玩过空挡的人可能知道，在即将over的时候，你的游戏窗口会闪几下，也基本上是由这个标志来决定的。

后面的部分我没有分析，因为也没什么可分析的了，其中`1008378` 与 `10079A0`至今不知道是做什么的，也懒得管了！

`0100434A` `|.  833D 78830001 0>cmp dword ptr ds:[1008378],0`

`01004351` `|.  C705 A0790001 0>mov dword ptr ds:[10079A0],4`

`0100435B` `|.  74 0B` `je short freecell.01004368`

`0100435D` `|.  6A 02` `push 2` `; /TimerID = 2`

`0100435F` `|.  FF75 08` `push dword ptr ss:[ebp+8]` `; |hWnd`

`01004362` `|.  FF15 F0100001` `call dword ptr ds:[<&USER32.KillTimer>]` `; \KillTimer`

`01004368` `|>` `6A 00` `push 0` `; /Timerproc = NULL`

`0100436A` `|.  68 90010000` `push 190` `; |Timeout = 400. ms`

`0100436F` `|.  6A 02` `push 2` `; |TimerID = 2`

`01004371` `|.  FF75 08` `push dword ptr ss:[ebp+8]` `; |hWnd`

`01004374` `|.  FF15 E8100001` `call dword ptr ds:[<&USER32.SetTimer>]` `; \SetTimer`

`0100437A` `|.  A3 78830001` `mov dword ptr ds:[1008378],eax`

`0100437F` `|.  EB 42` `jmp short freecell.010043C3`

OVER;

`01004381` `|>` `33DB` `xor ebx,ebx`

`01004383` `|>` `6A 01` `push 1` `; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING`

`01004385` `|.  6A 73` `push 73` `; |ItemID = 73 (115.)`

`01004387` `|.  FF75 08` `push dword ptr ss:[ebp+8]` `; |/hWnd`

`0100438A` `|.  891D 84790001` `mov dword ptr ds:[1007984],ebx` `; ||`

`01004390` `|.  FF15 34110001` `call dword ptr ds:[<&USER32.GetMenu>]` `; |\GetMenu`

`01004396` `|.  50` `push eax` `; |hMenu`

`01004397` `|.  FF15 30110001` `call dword ptr ds:[<&USER32.EnableMenuIt>` `; \EnableMenuItem`

`0100439D` `|.  53` `push ebx` `; /lParam`

`0100439E` `|.  68 A5250001` `push freecell.010025A5` `; |DlgProc = freecell.010025A5`

`010043A3` `|.  FF75 08` `push dword ptr ss:[ebp+8]` `; |hOwner`

`010043A6` `|.  68 38150001` `push freecell.01001538` `; |pTemplate = "YouLose"`

`010043AB` `|.  FF35 60780001` `push dword ptr ds:[1007860]` `; |hInst = 01000000`

`010043B1` `|.  FF15 2C110001` `call dword ptr ds:[<&USER32.DialogBoxPar>` `; \DialogBoxParamW`

`010043B7` `|.  891D 4C830001` `mov dword ptr ds:[100834C],ebx`

`010043BD` `|.  891D 30710001` `mov dword ptr ds:[1007130],ebx`

`010043C3` `|>` `5E` `pop esi`

`010043C4` `|.  5B` `pop ebx`

`010043C5` `|.  C9` `leave`

`010043C6` `\.  C2 0400` `retn 4`

好了，现在我们回来看一下关键的`IsCanMove`

名字是我随便起的，怎么方便怎么着来，其原型为：

```c
int IsCanMove(int paiA, int paiB);
```

功能：计算`paiB`是否可以移动到`paiA`，

在这里我先介绍一下大概的思路：空挡接龙中是否可以移动判断，是由两个条件决定的：其一，牌按照从大到小顺序排列，并且递减量为1，其二，黑红花色顺次相反。只有符合这两个条件才能满足移动的要求（这点我是看反汇编看了半天才看明白的，哎~，早知道多玩几盘游戏咯。。。）

OK 我们先来看下反汇编：

`01003D8E` `/ $  8B4424 04` `mov eax,dword ptr ss:[esp+4]` `;`

`01003D92` `|.  56` `push esi`

`01003D93` `|.  57` `push edi`

`01003D94` `|.  99` `cdq`

`01003D95` `|.  6A 04` `push 4`

`01003D97` `|.  59` `pop ecx`

`01003D98` `|.  F7F9` `idiv ecx`

`01003D9A` `|.  6A 04` `push 4`

`01003D9C` `|.  5F` `pop edi`

`01003D9D` `|.  8BF0` `mov esi,eax` `;  esi = eax = 商`

`01003D9F` `|.  8B4424 10` `mov eax,dword ptr ss:[esp+10]` `;`

`01003DA3` `|.  8BCA` `mov ecx,edx` `;  ecx = edx = 余`

`01003DA5` `|.  99` `cdq`

`01003DA6` `|.  F7FF` `idiv edi`

`01003DA8` `|.  2BC6` `sub eax,esi` `;`

`01003DAA` `|.  33F6` `xor esi,esi`

`01003DAC` `|.  46` `inc esi`

`01003DAD` `|.  3BC6` `cmp eax,esi`

`01003DAF` `|.  74 04` `je short freecell.01003DB5` `;`

`01003DB1` `|.  33C0` `xor eax,eax` `;`

`01003DB3` `|.  EB 23` `jmp short freecell.01003DD8`

`01003DB5` `|>` `3BCE` `cmp ecx,esi`

`01003DB7` `|.  74 09` `je short freecell.01003DC2`

`01003DB9` `|.  83F9 02` `cmp ecx,2`

`01003DBC` `|.  74 04` `je short freecell.01003DC2`

`01003DBE` `|.  33C0` `xor eax,eax`

`01003DC0` `|.  EB 02` `jmp short freecell.01003DC4`

`01003DC2` `|>` `8BC6` `mov eax,esi`

`01003DC4` `|>` `3BD6` `cmp edx,esi`

`01003DC6` `|.  74 07` `je short freecell.01003DCF`

`01003DC8` `|.  83FA 02` `cmp edx,2`

`01003DCB` `|.  74 02` `je short freecell.01003DCF`

`01003DCD` `|.  33F6` `xor esi,esi`

`01003DCF` `|>` `33C9` `xor ecx,ecx`

`01003DD1` `|.  3BC6` `cmp eax,esi`

`01003DD3` `|.  0F95C1` `setne cl`

`01003DD6` `|.  8BC1` `mov eax,ecx`

`01003DD8` `|>` `5F` `pop edi`

`01003DD9` `|.  5E` `pop esi`

`01003DDA` `\.  C2 0800` `retn 8`

这里我们先定义几个变量

```c
int nShangPaiA, nShangPaiB; // 记录两个牌/4的商，因为英文水平问题，查金山词霸也没找到商的英文，这里用拼音表示了
int nModPaiA, nModPaiB; // 记录两个牌/4的余，
```

`01003D8E` `/ $  8B4424 04` `mov eax,dword ptr ss:[esp+4]` `;`

`01003D92` `|.  56` `push esi`

`01003D93` `|.  57` `push edi`

`01003D94` `|.  99` `cdq`

`01003D95` `|.  6A 04` `push 4`

`01003D97` `|.  59` `pop ecx`

`01003D98` `|.  F7F9` `idiv ecx`

`nShangPaiA = paiA / 4;`

`nModPaiA= paiA % 4;`

`01003D9A` `|.  6A 04` `push 4`

`01003D9C` `|.  5F` `pop edi`

`01003D9D` `|.  8BF0` `mov esi,eax` `;  esi = eax = 商`

`01003D9F` `|.  8B4424 10` `mov eax,dword ptr ss:[esp+10]` `;  int temp场景的牌 = 场景的牌`

`01003DA3` `|.  8BCA` `mov ecx,edx` `;  ecx = edx = 余`

`01003DA5` `|.  99` `cdq`

`01003DA6` `|.  F7FF` `idiv edi`

`nShangPaiB = paiB / 4;`

`nModPaiB= paiB % 4;`

// 这个时候`esi == nQuotientOfPaiA`     `ecx == nModOfPaiA`
// 这个时候`eax == nQuotientOfPaiB`     `edx == nModOfPaiB`

`01003DA8` `|.  2BC6` `sub eax,esi` `;  场景牌的商 - 中转站的商`

`nShangPaiB -= nShangPaiA;`

`01003DAA` `|.  33F6` `xor esi,esi`

`01003DAC` `|.  46` `inc esi`

`int temp = 1;`

`01003DAD` `|.  3BC6` `cmp eax,esi`

`01003DAF` `|. /74 04` `je short freecell.01003DB5`

`01003DB1` `|. |33C0` `xor eax,eax`

`01003DB3` `|. |EB 23` `jmp short freecell.01003DD8`

```c
if (nShangPaiB != i)
{
  nShangPaiB = 0;
  return false;
}
```
 // 此处判断是否等于1，也就是`paiB - paiA`是否只差1，如果差1就去比较花色，如果不为0，那就没戏，直接走人了

`01003DB5` `|> \3BCE` `cmp ecx,esi`

`01003DB7` `|.  74 09` `je short freecell.01003DC2`
