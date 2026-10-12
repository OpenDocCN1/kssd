
# Windows 红心大战随机发牌程序分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-333.htm`](https://www.pediy.com/kssd/pediy09/pediy09-333.htm)

> Windows 红心大战有 52 张牌，每次开局时，程序会自动洗牌和发牌。
>
> 按照我们一般的想法，随机发牌无非就是随机产生 `0..51` 个随机数，代表 52 张牌。然后分发给四个玩家。当随机产生 `0...51` 之间的随机数时，可能会产生相同的随机数，此时，就要将这种情况去除，并重新产生一个随机数，直到完全产生 `0...51` 之间所有的数值，这种方式下，调用随机函数的次数是不固定的，而且肯定大于 52 次。
>
> 研究了一下红心大战，发现这个程序并不是这样洗牌的，它的洗牌方式有点巧妙，只需调用 52 次随机函数，就可以完全随机地整理出一幅牌来。这里面相关的代码并不多，只有四十来行，却花了我一个晚上，才看明白其中的道理，高兴之余，赶快拿出来与我一样的菜鸟分享。失误之处，请高手指正。
>
> 在 `OD` 中跟踪红心大战，很容易跟踪到随机发牌的地方，这里就不多说了。确定了函数地址后，再用 `IDA` 定位到这一段程序，如下：

```assembly
.text:01007F89 private: void __thiscall CMainWindow::Shuffle(void) proc near
.text:01007F89                  ; CODE XREF: OnNewGame+10Fp
.text:01007F89                  ; CMainWindow::DispatchCards(void):loc_1008966p
.text:01007F89                  mov     eax, offset unknown_libname_40 ; MFC4.2 release library by ForEver[RCT]
.text:01007F89                  ; __ehhandler$?ExecCommand@?$CHtmlEditCtrlBase@VCHtmlEditView@@@@QBEJPBU_GUID@@JJPAUtagVARIANT@@1@Z
.text:01007F8E                  call    __EH_prolog
.text:01007F8E

.text:01007F93                  sub     esp, 104h
.text:01007F99                  push    ebx
.text:01007F9A                  push    esi
.text:01007F9B                  push    edi
.text:01007F9C                  push    34h
.text:01007F9E                  mov     esi, ecx
.text:01007FA0                  xor     eax, eax
.text:01007FA2                  pop     ecx                 ; ecx=34h (52 张牌)
.text:01007FA2

.text:01007FA3 在内存中产生一幅牌
.text:01007FA3

.text:01007FA3 loc_1007FA3:                     ; CODE XREF: CMainWindow::Shuffle(void)+24j
.text:01007FA3                  mov     [ebp+eax*4-110h], eax
.text:01007FAA                  inc     eax
.text:01007FAB                  cmp     eax, ecx
.text:01007FAD                  jl      short loc_1007FA3
.text:01007FAD

.text:01007FAF                  and     dword ptr [ebp-14h], 0     ; [ebp-14h] 已发牌数，置初值 0
.text:01007FB3                  mov     [ebp-18h], ecx            ; [ebp-18h]:末发牌数，置初值:52

.text:01007FB3

.text:01007FB6 发牌
.text:01007FB6

.text:01007FB6 locNextCard:                     ; CODE XREF: CMainWindow::Shuffle(void)+8Aj
.text:01007FB6                  call    ds:___imp__rand
.text:01007FBC                  cdq
.text:01007FBD                  idiv    dword ptr [ebp-18h]       ; 随机数/末发牌数 (余数 edx 就是要抽取的牌的位置)

.text:01007FC0                  mov     eax, [ebp-14h]            ; eax=当前已发牌数

.text:01007FC3                  push    0Dh
.text:01007FC5                  pop     edi                     ; edi=13
.text:01007FC6                  push    4
.text:01007FC8                  pop     ebx
.text:01007FC9                  mov     ecx, edx                  ; 要抽取的牌的位置
.text:01007FCB                  cdq
.text:01007FCC                  idiv    edi                     ; edi=13, eax 就只能是 0,1,2,3, edx 只能是：0...12
.text:01007FCE                  sub     eax, [esi+140h]           ; [esi+140h]=0, eax 得到当前是给哪一位发牌，只可能是 0,1,2,3
.text:01007FD4                  mov     edi, edx                  ; edi 只能是 0...12
.text:01007FD6                  add     eax, 4                    ; eax 只能是 4,5,6,7
.text:01007FD9                  cdq
.text:01007FDA                  idiv    ebx                     ; ebx=4, edx 就只能为 0,1,2,3 分别代表四个玩家

.text:01007FDC                  lea     eax, [ebp+ecx*4-110h]
.text:01007FE3                  mov     ebx, [eax]                ; ebx=[ebp+ecx*4-110h] 这里面就是内存中的那副牌
.text:01007FE3                  ; 抽出这张牌，传给 ebx
.text:01007FE5                  shl     edi, 4                ; 玩家的当前牌和第一张牌偏移地址(相邻两张牌的地址相隔 10h)
.text:01007FE8                  lea     ecx, [esi+edx*4+130h]      ; 要发往玩家牌当前牌的地址
.text:01007FEF                  mov     edx, [ecx]
.text:01007FF1                  mov     [edi+edx+1Ch], ebx      ; 发牌给某一个玩家
.text:01007FF5                  mov     ecx, [ecx]
.text:01007FF7                  xor     ebx, ebx
.text:01007FF9                  dec     dword ptr [ebp-18h]      ; 统计末发牌数
.text:01007FFC                  inc     dword ptr [ebp-14h]      ; 统计已发牌数
.text:01007FFF                  cmp     dword ptr [ebp-14h], 34h              ; 已发牌数达到 52 张?
.text:01008003                  mov     [edi+ecx+28h], ebx
.text:01008007                  mov     ecx, [ebp-18h]
.text:0100800A                  mov     ecx, [ebp+ecx*4-110h]
.text:01008011                  mov     [eax], ecx                ; 将最后一张牌插入当前抽出牌的位置上来
.text:01008013                  jl      short locNextCard
.text:01008013

.text:01008015                  cmp     dword ptr [esi+144h], 3
.text:0100801C                  jz      short loc_100806A
```

----------------------------------------------------------------

程序分析说明：

程序先在内存中按照： 梅 A 方 A 红 A 黑 A 梅 2 方 2 红 2 黑 2 ... 梅 K 方 K 红 K 黑 K 的顺序有规率地产生一幅牌。

内存数据如下：

```
0006F9E4             00 00 00 00  /.`K/.........
0006F9F4  01 00 00 00 02 00 00 00 03 00 00 00 04 00 00 00  ............
0006FA04  05 00 00 00 06 00 00 00 07 00 00 00 08 00 00 00  ............
0006FA14  09 00 00 00 0A 00 00 00 0B 00 00 00 0C 00 00 00  ...............
0006FA24  0D 00 00 00 0E 00 00 00 0F 00 00 00 10 00 00 00  .............
0006FA34  11 00 00 00 12 00 00 00 13 00 00 00 14 00 00 00  ............
0006FA44  15 00 00 00 16 00 00 00 17 00 00 00 18 00 00 00  ............
0006FA54  19 00 00 00 1A 00 00 00 1B 00 00 00 1C 00 00 00  ............
0006FA64  1D 00 00 00 1E 00 00 00 1F 00 00 00 20 00 00 00  ......... ...
0006FA74  21 00 00 00 22 00 00 00 23 00 00 00 24 00 00 00  !..."...#...$...
0006FA84  25 00 00 00 26 00 00 00 27 00 00 00 28 00 00 00  3...&...'...(...
0006FA94  29 00 00 00 2A 00 00 00 2B 00 00 00 2C 00 00 00  )...*...+...,...
0006FAA4  2D 00 00 00 2E 00 00 00 2F 00 00 00 30 00 00 00  -......./...0...
0006FAB4  31 00 00 00 32 00 00 00 33 00 00 00
```

程序将牌按顺序整理好后，然后随机抽牌，分发给四个玩家。

发牌的顺序是先给第一个玩家发 13 张牌，然后给第二个玩家发 13 张牌，再给第三个玩家发 13 张牌，最后给第四个玩家发 13 张牌。

程序中设置一个变量，记住末发牌数，假设为 `n`，每次发牌时，就是在 `0...(n-1)` 的范围内产生一个随机数，这个随机数就是内存中那幅牌中的第几张牌，然后将这张牌抽出，发给玩家后，再将余下牌中的最后一张牌(索引号:`n-1`)放到这张牌的位置上来。

这样，产生的随机数虽然可能相同，那只是表示要抽取的牌的位置是相同的，由于牌已经被更换，所以每次抽取的牌肯定是不一样的。

这种方法的确很巧妙！对于其它的相关编程也有触类旁通的效果。自我感觉从这里受益不少！

---

*最初由 **szdbg**发布* (帖子 385361) Windows 红心大战有 52 张牌，每次开局时，程序会自动洗牌和发牌。

按照我们一般的想法，随机发牌无非就是随机产生 `0..51` 个随机数，代表 52 张牌。然后分发给四个玩家。当随机产生 `0...51` 之间的随机数时，可能会产生相同的随机数，此时，就要将这种情况去除，并重新产生一个随机数，直到……

红心大战的算法倒是蛮高效的。

我本人倒是曾经编写过一些纸牌类型的程序。

最初就是采用 `LZ` 所说的“随机发牌无非就是随机产生 `0..51` 个随机数，代表 52 张牌。然后分发给四个玩家。当随机产生 `0...51` 之间的随机数时，可能会产生相同的随机数，此时，就要将这种情况去除，并重新产生一个随机数，直到……”很快就发现程序效率非常低下，甚至考虑过某些极限情况，也许永远发不完一副牌。

后来采用的链表，但是效率并不理想。

最后就采用和红心大战类似的方法，不过有一点不同，抽走每一张牌后，其后的所有牌前进一个位置，然后 `n-1`。模拟手工抽取。事实上，红心大战的方法效率要更好。

看看 `VC6` 的 `STL` 部分 `random_shuffle()` 算法的源码，原理一模一样：

```cpp
// TEMPLATE FUNCTION random_shuffle
template<class _RI> inline
void random_shuffle(_RI _F, _RI _L)
{
    if (_F != _L)
        _Random_shuffle(_F, _L, _Dist_type(_F));
}

template<class _RI, class _Pd> inline
void _Random_shuffle(_RI _F, _RI _L, _Pd *)
{
    const int _RBITS = 15;
    const int _RMAX = (1U << _RBITS) - 1;
    _RI _X = _F;
    for (_Pd _D = 1; ++_X != _L; ++_D)
    {
        unsigned long _Rm = _RMAX;
        unsigned long _Rn = rand() & _RMAX;
    }
}
```



```
> for (; _Rm < _D && _Rm != ~0UL;
>   _Rm = _Rm << _RBITS | _RMAX)
>   _Rn = _Rn << _RBITS | _RMAX;
>   iter_swap(_X, _F + _Pd(_Rn % _D)); }}

> template<class _RI, class _Pf> inline
>   void random_shuffle(_RI _F, _RI _L, _Pf& _R)
>   {if (_F != _L)
>     _Random_shuffle(_F, _L, _R, _Dist_type(_F)); }

> template<class _RI, class _Pf, class _Pd> inline
>   void _Random_shuffle(_RI _F, _RI _L, _Pf& _R, _Pd *)
>   {_RI _X = _F;
>   for (_Pd _D = 1; ++_X != _L; ++_D)
>     iter_swap(_X, _F + _Pd(_R(_D))); }
```

没有细读楼主的代码,不知理解的对不?

```pascal
var
   buf,R:array [0..51]  of byte; //R 用于存取出来的牌,BUF 是待发牌
   i,j,K:integer;

begin
  randomize;
  for i:=51 downto 0 do //将每种牌进行排号,放到 BUF 中的
  begin
      j:=random(i+1);
      R[i]:=Buf[j];
      for K:=j to i-1 do  //I-1 防出
        Buf[k]:=buf[k+1];   //szdbg 提了建议了,俺改进一下.
  end;
  showmessage('R 就是发完的结果,可以 13 个一组每个人')
end;
```

```pascal
var
   buf,R:array [0..51]  of byte; //R 用于存取出来的牌,BUF 是待发牌
   i,j,K:integer;

begin
  randomize;
  for i:=51 downto 0 do //将每种牌进行排号,放到 BUF 中的
  begin
      j:=random(i+1);
      R[i]:=Buf[j];
      if i<>j then buf[j]:=buf[i];  //改进之处,看看行不?
  end;
  showmessage('R 就是发完的结果,可以 13 个一组每个人')
end;
```

引用:

* * *

*最初由 `nig` 发布* (帖子 385724)

```pascal
for K:=j to i-1 do  //I-1 防出
    Buf[k]:=buf[k+1];
```

* * *

|  |
| --- |

呵呵,你这和 `seya` 所说的方式是一样的,效率和红心大战比起来,稍差一点，红心大战每次在抽出一张牌后，只需移动一张牌，你这里需要移动多张牌...

```c
int player[4][13];  //4 个玩家 每个玩家有 13 张牌
  int poker[52];//={0,1,2,3,4,...};  扑克牌，从 0 到 51
  int n;
  int rest;     //剩余的扑克牌
  srand(time(NULL));  //随机初始化
  for( int k=0;k<52;k++ )poker[k]=k;  //扑克牌，从 0 到 51
  for( int i=0;i<4;i++ )
    for( int j=0;j<13;j++ )
    {
        rest=52-i*13-j;
        n=rand()%rest;         //随机出一个位置，这个位置不能大于剩余牌数
        player[i][j]=poker[n];            //某位玩家得到这个位置牌
        poker[n]=poker[rest-1];   //把最后的那张牌替换出过的位置牌
    }
```

不知的 C 语言算法对不对！[](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") [](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

`4dd466d0e1e0cc123afbe504da2462ca`
