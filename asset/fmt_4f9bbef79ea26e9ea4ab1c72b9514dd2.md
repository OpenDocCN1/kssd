
```
.text:01003084 sub_1003084     proc near                 ; CODE XREF: sub_1003512+6Fp
.text:01003084                               ; sub_1003512+85p ...
.text:01003084
.text:01003084 arg_0           = dword ptr  8
.text:01003084 arg_4           = dword ptr  0Ch
.text:01003084
.text:01003084                 push    ebp
.text:01003085                 mov     ebp, esp
.text:01003087                 push    ebx
.text:01003088                 push    [ebp+arg_4]
.text:0100308B                 xor     ebx, ebx
.text:0100308D                 push    [ebp+arg_0]
.text:01003090                 inc     ebx
.text:01003091                 mov     dword_1005798, ebx
.text:01003097                 call    sub_1003008
.text:0100309C                 cmp     dword_1005798, ebx ; sub_1003008 判断传入的参数所确定的格子周围是否有雷
.text:010030A2                 jz      short loc_1003114
.text:010030A4                 push    esi
.text:010030A5                 push    edi
.text:010030A6
.text:010030A6 loc_10030A6:                         ; CODE XREF: sub_1003084+8Cj
.text:010030A6                 mov     esi, dword_10057C0[ebx*4]
.text:010030AD                 mov     edi, dword_10051A0[ebx*4] ; 获取前一个周围没有雷的格子的坐标
.text:010030B4                 dec     esi
.text:010030B5                 lea     eax, [edi-1]
.text:010030B8                 push    esi                 ; 判断前一个周围没有雷的格子的，
.text:010030B9                 push    eax                 ; 左上角的格子周围雷的分布情况
.text:010030BA                 call    sub_1003008
.text:010030BF                 push    esi
.text:010030C0                 push    edi                 ; 判断前一个周围没有雷的格子的，
.text:010030C1                 call    sub_1003008         ; 正上方的格子周围雷的分布情况
.text:010030C6                 lea     eax, [edi+1]
.text:010030C9                 push    esi
.text:010030CA                 push    eax                 ; 判断前一个周围没有雷的格子的，
.text:010030CB                 mov     [ebp+arg_4], eax ; 右上方的格子周围雷的分布情况
.text:010030CE                 call    sub_1003008
.text:010030D3                 inc     esi
.text:010030D4                 push    esi
.text:010030D5                 lea     eax, [edi-1]
.text:010030D8                 push    eax                 ; 判断前一个周围没有雷的格子的，
.text:010030D9                 call    sub_1003008         ; 正左方的格子周围雷的分布情况
.text:010030DE                 push    esi
.text:010030DF                 push    [ebp+arg_4]     ; 判断前一个周围没有雷的格子的，
.text:010030E2                 call    sub_1003008         ; 正右方的格子周围雷的分布情况
.text:010030E7                 inc     esi
.text:010030E8                 push    esi
.text:010030E9                 lea     eax, [edi-1]
.text:010030EC                 push    eax                 ; 判断前一个周围没有雷的格子的，
.text:010030ED                 call    sub_1003008         ; 正下方的格子周围雷的分布情况
.text:010030F2                 push    esi
.text:010030F3                 push    edi                 ; 判断前一个周围没有雷的格子的，
.text:010030F4                 call    sub_1003008         ; 正下方的格子周围雷的分布情况
.text:010030F9                 push    esi
.text:010030FA                 push    [ebp+arg_4]     ; 判断前一个周围没有雷的格子的，
.text:010030FD                 call    sub_1003008         ; 右下方的格子周围雷的分布情况
.text:01003102                 inc     ebx
.text:01003103                 cmp     ebx, 64h
.text:01003106                 jnz     short loc_100310A ; 判断递归是否结束
.text:01003108                 xor     ebx, ebx
.text:0100310A
.text:0100310A loc_100310A:                         ; CODE XREF: sub_1003084+82j
.text:0100310A                 cmp     ebx, dword_1005798 ; 判断递归是否结束
.text:01003110                 jnz     short loc_10030A6
.text:01003112                 pop     edi
.text:01003113                 pop     esi
.text:01003114
.text:01003114 loc_1003114:                         ; CODE XREF: sub_1003084+1Ej
.text:01003114                 pop     ebx
.text:01003115                 pop     ebp
.text:01003116                 retn    8
.text:01003116 sub_1003084     endp
```

玩游戏的时候我们知道：当用户点击一个无雷的格子并且当它周围没有雷的时候，周围的 8 个雷会被自动"点击"。通过分析之后我们发现程序使用的是递归算法：`sub_1003008` 函数实现统计功能和修改对应内存数据的功能，然后如果格子周围没有雷就放入数组中，接着分别调用 `sub_1003008` 函数处理该格子四周的格子，最后通过数组的值确定下一个格子的坐标然后递归前面的过程。逻辑是比较清晰的，我们来看看 `sub_1003008` 做了些什么吧：

代码：

```
.text:01003008 sub_1003008     proc near                 ; CODE XREF: sub_1003084+13p
.text:01003008                               ; sub_1003084+36p ...
.text:01003008
.text:01003008 arg_0           = dword ptr  8
.text:01003008 arg_4           = dword ptr  0Ch
.text:01003008
.text:01003008                 push    ebp
.text:01003009                 mov     ebp, esp
.text:0100300B                 push    ebx
.text:0100300C                 mov     ebx, [ebp+arg_0] ; 点击格子的 Y 坐标
.text:0100300F                 push    esi
.text:01003010                 push    edi
.text:01003011                 mov     edi, [ebp+arg_4] ; 点击格子的 X 坐标
.text:01003014                 mov     esi, edi
.text:01003016                 shl     esi, 5
.text:01003019                 add     esi, ebx
.text:0100301B                 movsx   eax, byte_1005340[esi] ; 点击的格子对应的内存数据
.text:01003022                 test    al, 40h
.text:01003024                 jnz     short loc_100307D ; 如果高 29 位为 1 就跳转
.text:01003026                 and     eax, 1Fh        ; 取低 5 位
.text:01003029                 cmp     eax, 10h        ; 判断是否为 0x10
.text:0100302C                 jz      short loc_100307D ; 是的话也跳转
.text:0100302E                 cmp     eax, 0Eh        ; 不是 0x0e 的话跳转
.text:01003031                 jz      short loc_100307D
.text:01003033                 inc     dword_10057A4   ; 确定不是雷的个数增加 1
.text:01003039                 push    edi                 ; 点击的 X 坐标
.text:0100303A                 push    ebx                 ; 点击的 Y 坐标
.text:0100303B                 call    sub_1002F3B     ; 判断该格子的周围 8 个格子有没有雷,并返回雷的个数
.text:01003040                 mov     [ebp+arg_4], eax ; 记录周围的雷的个数
.text:01003043                 push    edi                 ; 点击的格子的 X 坐标
.text:01003044                 or      al, 40h         ; al 等于 0x40+?,?代表周围雷的个数
.text:01003046                 push    ebx                 ; 点击的格子的 Y 坐标
.text:01003047                 mov     byte_1005340[esi], al ; 更新该格子对应的内存数据
.text:0100304D                 call    sub_1002646     ; 图形操作相关
.text:01003052                 cmp     [ebp+arg_4], 0
.text:01003056                 jnz     short loc_100307D ; 判断该地址周围的 8 个格子是否有雷,有雷就跳转
.text:01003058                 mov     eax, dword_1005798
.text:0100305D                 mov     dword_10051A0[eax*4], ebx ; 保存周围没有雷的格子的 Y 坐标
.text:01003064                 mov     dword_10057C0[eax*4], edi ; 保存周围没有雷的格子的 X 坐标
.text:0100306B                 inc     eax
.text:0100306C                 cmp     eax, 64h        ; 数组的最大长度为 0x64
.text:0100306F                 mov     dword_1005798, eax
.text:01003074                 jnz     short loc_100307D
.text:01003076                 and     dword_1005798, 0 ; 清 0
.text:0100307D
.text:0100307D loc_100307D:                         ; CODE XREF: sub_1003008+1Cj
.text:0100307D                               ; sub_1003008+24j ...
.text:0100307D                 pop     edi
.text:0100307E                 pop     esi
.text:0100307F                 pop     ebx
.text:01003080                 pop     ebp
.text:01003081                 retn    8
.text:01003081 sub_1003008     endp
```

首先，判断点击该点的坐标对应的内存数据是否还没有被处理过的（标记等），如果是的话，调用 `sub_1002F3B` 函数返回该格子周围 8 个格子的雷的数量。

如果周围没有雷的话，将该格子的 X,Y 坐标分别保存在数组里面。接着调用和图像操作相关的函数显示出该格子的情况（有雷标记出雷的个数）。

我们看看 `sub_1002F3B` 函数是怎样的一个算法：

代码：
