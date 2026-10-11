
### 逆向分析：循环逻辑与汇编对照

> `0100430E | . 83FE 10 | cmp esi,10 ; i < 10H`
> `01004311 | .^ 72 D8 | \jb short freecell.010042EB`

对应的 C 语言伪代码逻辑如下：

```
for (i = 0; i < 4; i++)
{
  unsigned int nRecycle = pBaseAddress[i];
  for (j = 1; j < 9; j++)
  {
    // 010042F9 | . E8 90FAFFFF | |call freecell.01003D8E
    if(IsCanMove(nRecycle, aryCards[j])) // 关键算法，后面有详解
    {
      nFlag++;
    }
  }
}
```

这部分是将中转站中的每张牌分别与牌面中的最下面的牌分别用 `IsCanMove` 做以下比较，如果可以移动那就将 `nFlag` 标志 `++`。

其中 `IsCanMove` 下面还要用，把这段分析完我们再来看 `IsCanMove` 的实现，看流程先........

### 逆向分析：牌面互相比较逻辑

> `01004313 | . 33FF | xor edi,edi ; 下面是用来计算最下面的 8 张牌是否可以互相放`
> `01004315 | . 47 | inc edi ; int i = 1`
> `01004316 | > 33F6 | /xor esi,esi`
> `01004318 | . 46 | |inc esi ; int j = 1`
> `01004319 | > 3BF7 | |/cmp esi,edi`
> `0100431B | . 74 14 | ||je short freecell.01004331`
> `0100431D | . FF74B5 D8 | ||push dword ptr ss:[ebp+esi*4-28]`
> `01004321 | . FF74BD D8 | ||push dword ptr ss:[ebp+edi*4-28]`
> `01004325 | . E8 64FAFFFF | ||call freecell.01003D8E`
> `0100432A | . 85C0 | ||test eax,eax`
> `0100432C | . 74 03 | ||je short freecell.01004331`
> `0100432E | . FF45 FC | ||inc dword ptr ss:[ebp-4]`
> `01004331 | > 46 | ||inc esi ; j++`
> `01004332 | . 83FE 09 | ||cmp esi,9 ; j < 9`
> `01004335 | .^ 72 E2 | |\jb short freecell.01004319`
> `01004337 | . 47 | |inc edi ; i++`
> `01004338 | . 83FF 09 | |cmp edi,9 ; i < 9`
> `0100433B | .^ 72 D9 | \jb short freecell.01004316`

对应的 C 语言伪代码逻辑如下：

```
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

同上，只不过是牌面与牌面之间的互相比较看是否能移动，有能移动的 `nFlag++`。

### 逆向分析：游戏结束判断与死局处理

> `0100433D | . 837D FC 00 | cmp dword ptr ss:[ebp-4],0 ; 如果标志位为 0 说明游戏 over`
> `01004341 | . 5F | pop edi`
> `01004342 | . 76 3D | jbe short freecell.01004381`

对应的 C 语言伪代码逻辑如下：

```
if (nFlag <= 0)
{
  goto OVER;
}
```

如果从一开始的 `0100424F` 处，到这里 `nFlag(ebp - 4)` 都没有任何赋值，说明游戏死定了！，直接跳到 `Dialog` 显示 `over`。

> `01004344 | . 837D FC 01 | cmp dword ptr ss:[ebp-4],1`
> `01004348 | . 75 79 | jnz short freecell.010043C3`

对应的 C 语言伪代码逻辑如下：

```
if (nFlag != 1)
{
  return;
}
```

比较是否等于 1，不等于就走人，说明游戏死不了，为什么？其实这又要说到 `nFlag(ebp - 4)` 这个值的含义了，其实它是记录了可以移牌的个数，如果是 0 个，那就是 `over`，如果是 1 个，说明暂时还死不了，不过这个时候有玩过空挡的人可能知道，在即将 `over` 的时候，你的游戏窗口会闪几下，也基本上是由这个标志来决定的。

后面的部分我没有分析，因为也没什么可分析的了，其中 `1008378` 与 `10079A0` 至今不知道是做什么的，也懒得管了！

> `0100434A | . 833D 78830001 0> cmp dword ptr ds:[1008378],0`
> `01004351 | . C705 A0790001 0> mov dword ptr ds:[10079A0],4`
> `0100435B | . 74 0B | je short freecell.01004368`
> `0100435D | . 6A 02 | push 2 ; /TimerID = 2`
> `0100435F | . FF75 08 | push dword ptr ss:[ebp+8] ; |hWnd`
> `01004362 | . FF15 F0100001 | call dword ptr ds:[<&USER32.KillTimer>] ; \KillTimer`
> `01004368 | > 6A 00 | push 0 ; /Timerproc = NULL`
> `0100436A | . 68 90010000 | push 190 ; |Timeout = 400. ms`
> `0100436F | . 6A 02 | push 2 ; |TimerID = 2`
> `01004371 | . FF75 08 | push dword ptr ss:[ebp+8] ; |hWnd`
> `01004374 | . FF15 E8100001 | call dword ptr ds:[<&USER32.SetTimer>] ; \SetTimer`
> `0100437A | . A3 78830001 | mov dword ptr ds:[1008378],eax`
> `0100437F | . EB 42 | jmp short freecell.010043C3`

### OVER 标签：显示失败对话框

> `OVER;`
> `01004381 | > 33DB | xor ebx,ebx`
> `01004383 | > 6A 01 | push 1 ; /Flags = MF_BYCOMMAND|MF_GRAYED|MF_STRING`
> `01004385 | . 6A 73 | push 73 ; |ItemID = 73 (115.)`
> `01004387 | . FF75 08 | push dword ptr ss:[ebp+8] ; |/hWnd`
> `0100438A | . 891D 84790001 | mov dword ptr ds:[1007984],ebx ; ||`
> `01004390 | . FF15 34110001 | call dword ptr ds:[<&USER32.GetMenu>] ; |\GetMenu`
> `01004396 | . 50 | push eax ; |hMenu`
> `01004397 | . FF15 30110001 | call dword ptr ds:[<&USER32.EnableMenuIt>; \EnableMenuItem`
> `0100439D | . 53 | push ebx ; /lParam`
> `0100439E | . 68 A5250001 | push freecell.010025A5 ; |DlgProc = freecell.010025A5`
> `010043A3 | . FF75 08 | push dword ptr ss:[ebp+8] ; |hOwner`
> `010043A6 | . 68 38150001 | push freecell.01001538 ; |pTemplate = "YouLose"`
> `010043AB | . FF35 60780001 | push dword ptr ds:[1007860] ; |hInst = 01000000`
> `010043B1 | . FF15 2C110001 | call dword ptr ds:[<&USER32.DialogBoxPar>; \DialogBoxParamW`
> `010043B7 | . 891D 4C830001 | mov dword ptr ds:[100834C],ebx`
> `010043BD | . 891D 30710001 | mov dword ptr ds:[1007130],ebx`
> `010043C3 | > 5E | pop esi`
> `010043C4 | . 5B | pop ebx`
> `010043C5 | . C9 | leave`
> `010043C6 | \. C2 0400 | retn 4`

### IsCanMove 函数详解

好了，现在我们回来看一下关键的 `IsCanMove`。

名字是我随便起的，怎么方便怎么着来，其原型为：

```
int IsCanMove(int paiA, int paiB);
```

功能：计算 `paiB` 是否可以移动到 `paiA`。

在这里我先介绍一下大概的思路：空挡接龙中是否可以移动判断，是由两个条件决定的：其一，牌按照从大到小顺序排列，并且递减量为 1，其二，黑红花色顺次相反。只有符合这两个条件才能满足移动的要求（这点我是看反汇编看了半天才看明白的，哎~，早知道多玩几盘游戏咯。。。）

OK 我们先来看下反汇编：

> `01003D8E |/$ 8B4424 04 | mov eax,dword ptr ss:[esp+4] ;`
> `01003D92 | . 56 | push esi`
> `01003D93 | . 57 | push edi`
> `01003D94 | . 99 | cdq`
> `01003D95 | . 6A 04 | push 4`
> `01003D97 | . 59 | pop ecx`
> `01003D98 | . F7F9 | idiv ecx`
> `01003D9A | . 6A 04 | push 4`
> `01003D9C | . 5F | pop edi`
> `01003D9D | . 8BF0 | mov esi,eax ; esi = eax = 商`
> `01003D9F | . 8B4424 10 | mov eax,dword ptr ss:[esp+10] ;`
> `01003DA3 | . 8BCA | mov ecx,edx ; ecx = edx = 余`
> `01003DA5 | . 99 | cdq`
> `01003DA6 | . F7FF | idiv edi`
> `01003DA8 | . 2BC6 | sub eax,esi ;`
> `01003DAA | . 33F6 | xor esi,esi`
> `01003DAC | . 46 | inc esi`
> `01003DAD | . 3BC6 | cmp eax,esi`
> `01003DAF | . 74 04 | je short freecell.01003DB5 ;`
> `01003DB1 | . 33C0 | xor eax,eax ;`
> `01003DB3 | . EB 23 | jmp short freecell.01003DD8`
> `01003DB5 | > 3BCE | cmp ecx,esi`
> `01003DB7 | . 74 09 | je short freecell.01003DC2`
> `01003DB9 | . 83F9 02 | cmp ecx,2`
> `01003DBC | . 74 04 | je short freecell.01003DC2`
> `01003DBE | . 33C0 | xor eax,eax`
> `01003DC0 | . EB 02 | jmp short freecell.01003DC4`
> `01003DC2 | > 8BC6 | mov eax,esi`
