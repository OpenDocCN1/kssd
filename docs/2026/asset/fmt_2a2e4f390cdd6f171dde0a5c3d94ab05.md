
对其他弹出的窗体，比如Tools菜单的 References... 和 Running Scripts...等也有作用。。。
 
所以因为被`nop`掉，这些窗体在创建之后都没显示，然后直接就被隐藏了！等有时间再完善吧。。。
 
终于写完了，要过年回家了！！！
 
(改的地方很少，就不上传改后的文件啦！)
 
## 以下是新修改的
 
23:33 2012-01-27
 
### 【新增使用工具】: `LordPE`
 
上回说到在取消掉注册窗体的显示的同时，所有其他的弹出窗体也不能显示了，今天终于解决了这个问题！
 
我的思路是这样的：
 
判断窗体的标题是否以`Register`打头，是的话就不显示，不是的话就显示它！
 
上回修改的是这里：
 
```
0047E40A   >  57            push    edi;push进的参数（nop掉！）
0047E40B   .  8BCE          mov     ecx, esi
0047E40D   .  E8 38400000   call    0048244A;显示窗体的函数，跟进去发现，不断的在PeekMessage。（nop掉！）
0047E412   .  33FF          xor     edi, edi
```
 
根据我的思路，必须找一个地方写代码，上面这小块儿地方显然不够！不如从这个地方跳到另一个自己写的代码的地方，然后跳回到`0047E412`！
 
来找区块间隙吧！（因为新增区块我不会:P）
 
用`LordPE`查看区块`.text`的`voffset`为`1000`，`vsize`为`27bf0c`，而`RSize`为`27c000`。
 
找到了，在这里：`0067cf0c`到`0067cfff`。
 
注意：因为取窗体标题的`GetWindowTextA`用到了缓冲`buffer`，所以我们写代码所在区块应当有可写的属性，用`LordPE`轻松改之！
 
改后的：
 
```
0047E40A     /E9 11EB1F00   jmp     0067CF20
0047E40F     |90            nop
0047E410     |90            nop
0047E411     |90            nop
0047E412   . |33FF          xor     edi, edi
```
 
---
 
下面是判断窗体标题并决定是否显示的代码：
 
```
0067CF0C      52            push    edx                              ;  db 'Register',0,0
0067CF0D      65:67:6973 74>imul    esi, dword ptr gs:[bp+di+74], 72>
0067CF16      0000          add     byte ptr [eax], al               ;  buffer,  10 bytes(0)
0067CF18      0000          add     byte ptr [eax], al               ;  2
0067CF1A      0000          add     byte ptr [eax], al               ;  4
0067CF1C      0000          add     byte ptr [eax], al               ;  6
0067CF1E      0000          add     byte ptr [eax], al               ;  8
0067CF20      6A 0A         push    0A;前面是定义“常量”与缓冲buffer，而从这句开始才是真正的代码
0067CF22      68 16CF6700   push    0067CF16                         ;  Buffer 地址
0067CF27      FF76 20       push    dword ptr [esi+20]               ;  窗体句柄
0067CF2A      E8 FC516B77   call    USER32.GetWindowTextA
0067CF2F      53            push    ebx;保存原来的ebx
0067CF30      BB 0CCF6700   mov     ebx, 0067CF0C                    ;  ASCII "Register"
0067CF35      66:B9 0800    mov     cx, 8
0067CF39      8A03          mov     al, byte ptr [ebx]
0067CF3B      3A43 0A       cmp     al, byte ptr [ebx+A]
0067CF3E      75 0A         jnz     short 0067CF4A                   ;  不相等则表示不是注册窗体，显示它
0067CF40      43            inc     ebx
0067CF41      67:E2 F5      loopw   short 0067CF39
0067CF44      5B            pop     ebx
0067CF45    ^ E9 C814E0FF   jmp     0047E412                         ;  是注册窗体！不显示它！
0067CF4A      5B            pop     ebx
0067CF4B      57            push    edi
0067CF4C      8BCE          mov     ecx, esi
0067CF4E      E8 F754E0FF   call    0048244A;调用显示窗体的函数
0067CF53    ^ E9 BA14E0FF   jmp     0047E412
```
 
全文完。
 
[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
 
[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
