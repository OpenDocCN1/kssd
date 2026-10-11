
### 汇编代码分析

#### 第一部分：内存操作与初始化

```
00425213    66:8B5E 1E      mov     bx, word ptr ds:[esi+1E]  
00425244    66:8985 5434420>mov     word ptr ss:[ebp+423454], ax  
00425250    66:899D 5634420>mov     word ptr ss:[ebp+423456], bx  
00425289    B8 5C3A4200     mov     eax, Try.00423A5C  
00425293    03C5            add     eax, ebp  
004252AC    66:8946 18      mov     word ptr ds:[esi+18], ax  
004252B5    C1E8 10         shr     eax, 10  
004252E5    66:8946 1E      mov     word ptr ds:[esi+1E], ax  
```

; 好在我不是 9x....哈

---

#### 第二部分：解密逻辑（Part 3）

; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 当当当~~~~~ 醒醒啦！！！
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
0042532D    8D85 D05C4000   lea     eax, dword ptr ss:[ebp+405CD0] ; 没弄明白，也许是参数  
00425334    B0 94           mov     al, 94        ; 后边用来 xor 的 key  
00425336    E8 00000000     call    Try.0042533B  
0042533B    5E              pop     esi      ; 取这条信令地址  
0042533C    81C6 A6000000   add     esi, 0A6 ; 加上这段解密代码的长度，指向下一部分，进行 SMC 解密  
00425347    B9 68010000     mov     ecx, 168h        ; 解码长度  
00425356    8A26            mov     ah, byte ptr ds:[esi] ; 读取一个字节  
0042535D    32E0            xor     ah, al  
00425376    F6D4            not     ah  
0042538F    8826            mov     byte ptr ds:[esi], ah ; 解密后存回去  
004253A8    46              inc     esi ; 下一个字节  
004253AE    49              dec     ecx ; 计数器--  
004253B4    83F9 00         cmp     ecx, 0  
004253B7    75 98           jnz     short Try.00425351 ; 循环直到都解密完成  
```

; 精彩片断！！！！！！！！！！

```
004253BE    FEC8            dec     al     ; 换一个 key  
004253C5    BB 4F434E55     mov     ebx, 554E434F ; 不清楚，自己猜吧  
0042540E    B9 68010000     mov     ecx, 168     ; 这一块长度  
0042542A    8A240E          mov     ah, byte ptr ds:[esi+ecx]    ; 读取一个字节  
00425444    88240E          mov     byte ptr ds:[esi+ecx], ah   ; 再写进去，什么名堂？  
0042545E    CC              int     3 ; 别乱动，int 3 解码  
0042545F    E9 E5000000     jmp     Try.00425549     ; 按 F4 ，int3 的向量处理好解码了，过去看看  
                        ; 跳过的好像是下一次 int3 的向量,俺可不敢进去胡闹  
00425464    3D 534E5552     cmp     eax, 52554E53  
00425469    75 24           jnz     short Try.0042548F  
00425482    FFD3            call    ebx  
00425489    CF              iretd  
```

#### 第三部分：重复的 Int3 陷阱循环

以下代码块展示了多次类似的 `int3` 和跳转逻辑，用于调试或混淆。

```
00425576    B9 68010000     mov     ecx, 168 ; TMD,没完没了了  
00425592    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004255AC    88240E          mov     byte ptr ds:[esi+ecx], ah  
004255C6    CC              int3  
004255C7    E9 E5000000     jmp     Try.004256B1     ; F4  
004256DE    B9 68010000     mov     ecx, 168 ; KAO....  
004256FA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425714    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042572E    CC              int3  
0042572F    E9 E5000000     jmp     Try.00425819  
00425846    B9 68010000     mov     ecx, 168 ; ……  
00425862    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042587C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425896    CC              int3  
00425897    E9 E5000000     jmp     Try.00425981  
004259AE    B9 68010000     mov     ecx, 168     ; my heart will go on  
004259CA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004259E4    88240E          mov     byte ptr ds:[esi+ecx], ah  
004259FE    CC              int3  
004259FF    E9 E5000000     jmp     Try.00425AE9  
00425B16    B9 68010000     mov     ecx, 168 ; 没完没了  
00425B32    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425B4C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425B66    CC              int3  
00425B67    E9 E5000000     jmp     Try.00425C51  
00425C7E    B9 68010000     mov     ecx, 168 ; 不见不散  
00425C9A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425CB4    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425CCE    CC              int3  
00425CCF    E9 E5000000     jmp     Try.00425DB9  
```

#### 第四部分：后续 Int3 序列

```
00425C7E    B9 68010000     mov     ecx, 168 ; 有话好好说  
00425C9A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425CB4    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425CCE    CC              int3  
00425CCF    E9 E5000000     jmp     Try.00425DB9  
00425DE6    B9 68010000     mov     ecx, 168 ; 声声慢……  
00425E02    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425E1C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425E36    CC              int3  
00425E37    E9 E5000000     jmp     Try.00425F21  
00425F4E    B9 68010000     mov     ecx, 168  
00425F6A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425F84    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425F9E    CC              int3  
00425F9F    E9 E5000000     jmp     Try.00426089  
004260B6    B9 68010000     mov     ecx, 168  
004260D2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004260EC    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426106    CC              int3  
00426107    E9 E5000000     jmp     Try.004261F1  
0042621E    B9 68010000     mov     ecx, 168  
0042623A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426254    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042626E    CC              int3  
0042626F    E9 E5000000     jmp     Try.00426359  
00426386    B9 68010000     mov     ecx, 168  
004263A2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004263BC    88240E          mov     byte ptr ds:[esi+ecx], ah  
004263D6    CC              int3  
004263D7    E9 E5000000     jmp     Try.004264C1  
004264EE    B9 68010000     mov     ecx, 168  
0042650A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426524    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042653E    CC              int3  
0042653F    E9 E5000000     jmp     Try.00426629  
00426656    B9 68010000     mov     ecx, 168  
00426672    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042668C    88240E          mov     byte ptr ds:[esi+ecx], ah  
004266A6    CC              int3  
004266A7    E9 E5000000     jmp     Try.00426791  
004267BE    B9 68010000     mov     ecx, 168  
004267DA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004267F4    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042680E    CC              int3  
0042680F    E9 E5000000     jmp     Try.004268F9  
00426926    B9 68010000     mov     ecx, 168  
00426942    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042695C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426976    CC              int3  
00426977    E9 E5000000     jmp     Try.00426A61  
00426A8E    B9 68010000     mov     ecx, 168  
00426AAA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426AC4    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426ADE    CC              int3  
00426ADF    E9 E5000000     jmp     Try.00426BC9  
00426BF6    B9 68010000     mov     ecx, 168  
00426C12    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426C2C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426C46    CC              int3  
00426C47    E9 E5000000     jmp     Try.00426D31  
00426D5E    B9 68010000     mov     ecx, 168  
00426D7A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426D94    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426DAE    CC              int3  
00426DAF    E9 E5000000     jmp     Try.00426E99  
```

#### 第五部分：尾部代码

```
00426D5E    B9 68010000     mov     ecx, 168  
00426D7A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
```
