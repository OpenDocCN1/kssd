
# 菜鸟也做注册机(一)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-816.htm`](https://www.pediy.com/kssd/pediy07/pediy7-816.htm)

【作者声明】：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!

【破解工具】：`Ollydbg1.10`、`Hiew6.81`

破解是一项有意思又很枯燥的工作，有意思是当你破解了软件并且做出注册机的时候(那个美啊，如果有 MM 正场效果更佳），枯燥是当你奋斗 N 小时后仍焦头烂额一无所获时(千万别让 MM 看到）!本人系正宗菜鸟，为鼓舞广大菜鸟的破解兴趣，把我的学习过程整理一下贴出来，（并不是标题所注的一二三四系列型的噢)期望对其他和我一样的菜鸟能有所帮助，让我们共同进步！

既然是第一篇，当然得找个巨简单的下手，在看雪老大的加密解密第二版的附加光盘中有（什么？你小子没钱买看雪老大的书,欠 K 啊,%@#^!~......,算了,下次给 MM 少买点零食省点钱买书!）,不过考虑大家方便我还是把源文件也传上来了!无壳，`Dephi` 编写.

考虑是给和我一样的菜鸟看的,把算法稍微分析一下吧!用 `OD` 打开源文件 `echap303.exe`,随意输入用户名:`sharpair` 和假码:`12345678` 搜索字符参考,定位到以下代码:

```
004417D2   lea edx,dword ptr ss:[ebp-4]             ;  下面两行是初始化参数,为下面的 Call 用
004417D5   mov eax,dword ptr ds:[ebx+2C8]
004417DB   call echap303.004231A4                   ;  这个 Call 是取用户输入的假码
004417E0   mov eax,dword ptr ss:[ebp-4]             ;  结果在 EAX 中
004417E3   push eax                                  ;  压入堆栈
004417E4   lea edx,dword ptr ss:[ebp-C]             ;  同上，不说了
004417E7   mov eax,dword ptr ds:[ebx+2C4]
004417ED   call echap303.004231A4                   ;  与上面的 Call 一样，不用我说了吧，这次取用户名
004417F2   mov eax,dword ptr ss:[ebp-C]
004417F5   lea edx,dword ptr ss:[ebp-8]
004417F8   call echap303.004416F8                   ;  关键的计算注册码的 Call
004417FD   mov edx,dword ptr ss:[ebp-8]             ;  此处的[ebp-8]中为正确有注册码,记住此处堆栈的地址
00441800   pop eax
00441801   call echap303.00403B44                   ;  不用说，这是个判断的调用了
00441806   jnz short echap303.00441822              ;  关键跳转，爆破改这就可以了
00441808   push 40
0044180A   mov ecx,echap303.0044186C                 ;  ASCII "U made it"
0044180F   mov edx,echap303.00441878                 ;  ASCII "Right Code"
00441814   mov eax,dword ptr ds:[442C30]
00441819   mov eax,dword ptr ds:[eax]
0044181B   call echap303.0043EEF4
00441820   jmp short echap303.0044183A
00441822   push 10
00441824   mov ecx,echap303.00441884                 ;  ASCII "Error"
00441829   mov edx,echap303.0044188C                 ;  ASCII "Wrong Code"
0044182E   mov eax,dword ptr ds:[442C30]
00441833   mov eax,dword ptr ds:[eax]
00441835   call echap303.0043EEF4
```

我们的目标是做个注册机，当然是跟进 `004417F8` 处的那个关键 `Call` 了

```
004416F8   push ebx
004416F9   push esi
004416FA   push edi
004416FB   add esp,-24
004416FE   mov dword ptr ss:[esp],edx
00441701   mov edi,eax                              ;  EAX 中的用户名备份一份到 EDI 中
00441703   mov ebx,49390305                         ;  下面两句给 edx 和 esi 赋值，还记得刚开始把 ebx 和 esi 压栈吗？
00441708   mov esi,48631220
0044170D   mov eax,edi
0044170F   call echap303.00403A34                  ;  这个 Call 是得到用户名的长度，结果同样在 EAX 中
00441714   test eax,eax
00441716   jle short echap303.00441746
00441718   mov edx,1                                ;  edx 赋 1，看来要循环操作了
0044171D   /xor ecx,ecx                             ;  ecx 清 0
0044171F   |mov cl,byte ptr ds:[edi+edx-1]          ;  果然，edx 作指针用，取用户名第一个字符的 ASCII 码
00441723   |xor ebx,ecx                             ;  与 ebx 或
00441725   |xor esi,ebx                             ;  结果再与 esi 或
00441727   |test bl,1                               ;  测试 bl 为偶或奇数（不知这样理解对不对）
0044172A   |je short echap303.0044173B             ;  为偶则跳
0044172C   |sar ebx,1
0044172E   |jns short echap303.00441733
00441730   |adc ebx,0
00441733   |xor ebx,1200311                        ;  上面计算后若 bl 为奇则此处将 ebx 与 1200311 作或运算
00441739   |jmp short echap303.00441742
0044173B   |sar ebx,1                               ;  ebx 再右移一位，即除 2
0044173D   |jns short echap303.00441742            ;  为负则跳，否则进入下一轮循环
0044173F   |adc ebx,0
00441742   |inc edx                                 ;  edx 加 1，取下一字符
00441743   |dec eax                                 ;  计数器减 1，判断是否取完所有字符
00441744   \jnz short echap303.0044171D
00441746   mov eax,dword ptr ss:[esp]               ;  我这里循环结束后 ebx=0056B90C
00441749   push eax                                 ; /Arg1
0044174A   mov eax,ebx                              ; |果然要用到上面的计算结果参与运算
0044174C   and eax,0FFFF                            ; |取 eax 的低 4 位,eax=0xB90C
00441751   mov dword ptr ss:[esp+8],eax             ; |结果存入堆栈[esp+8]中
00441755   mov byte ptr ss:[esp+C],0                ; |
0044175A   shr ebx,10                               ; |呵呵，又对 ebx 作运算了,ebx=0056
0044175D   mov dword ptr ss:[esp+10],ebx            ; |结果存入堆栈[esp+10]中,注意这里的 dword
00441761   mov byte ptr ss:[esp+14],0               ; |
00441766   mov eax,esi                              ; |又要用 esi 的值了，虽然上面没提，可也别忘了
00441768   and eax,0FFFF                            ; |以上操作同上，不再啰嗦了,eax=3C33
0044176D   mov dword ptr ss:[esp+18],eax            ; |
00441771   mov byte ptr ss:[esp+1C],0               ; |这一句从上看下来不知有何意义
00441776   shr esi,10                               ; |esi=3816
00441779   mov dword ptr ss:[esp+20],esi            ; |
0044177D   mov byte ptr ss:[esp+24],0               ; |
00441782   lea edx,dword ptr ss:[esp+8]             ; |这个堆栈中是什么还记得吗，往上看！
00441786   mov ecx,3                                ; |
0044178B   mov eax,echap303.004417A4                ; |ASCII "%.4x-%.4x-%.4x-%.4x"
00441790   call echap303.00408004                  ; \上面那一行不用说是格式化字符串的，而这个 Call 才是最后生成注册码的
00441795   add esp,24
00441798   pop edi
00441799   pop esi
0044179A   pop ebx
0044179B   retn
```

不入虎穴，焉得虎子，再跟入 `00441790` 处的 `Call`!（没经验！那种格式一般都是格式化串，一般不会再有算法的!老鸟在旁边看不过去了：）,老大别怪,菜鸟都是这样的,下次不会了）

我们用粗跟踪一下就发现没必要再进入上面的那一个 `Call` 了，只是一个连接字串的 `Call`，即把上面得到的结果按连接起来生成格式化字串样式的注册码`B90C-0056-3C33-3816`!

OK!算法是不是很容易，那写个注册机吧！可是我编程很菜啊，而且是不是太麻烦了（懒人的通病）......原程序中不是有完整的注册算法吗，能不能利用原程序直接生成注册机呢？让我们看看！

还记得吗，任意填入一注册码后会跳出一个错误的窗口(呵呵，窗口显现代码省了,还是懒人的通病)，我们可以利用原有代码，只要把显示 `Wrong Code` 的显示参数改为上面程序中生成的注册码不就是一个现成的注册机吗!我们知道在 `004417FD` 处的堆栈 `ss:[ebp-8]`中为正确的注册码，经过 `00441801` 的那个 `Call` 后发现 `ebp` 的值并未变化,OK,可以直接利用上面的地址`[ebp-8]`了，再往下看,`00441829` 处压入`"Wrong Code"`，可以在此处将此句在 `OD` 中直接改为 `mov edx,dword ptr ss:[ebp-8]`,注意要选中“使用 nop 填充"(补充一点，这样操作的前提是新的语句长度不能大于原语句，要是大于怎么办呢，嘿嘿，后面会碰到的，继续关注吧！：），记住此时的机器码：`8B55F89090`，然后用任一十六进制文件编辑器（我是用 `WDasm10.0` 的)打开源文件，将 `00441829` 处的机器码改为 `8B55F89090`,再另存为一个文件，呵呵，注册机诞生了！如果想更拉风一点可以把显示 `Error` 的信息改为你的大名或其他什么的，这里就不多说了，不然被斑竹说我灌水就惨了！

OK!第一篇到此结束,做注册机也不是太难嘛!别忘形哟,要学的还多呢.......

欢迎发信到 `sharpair@163.com` 交流!

4dd466d0e1e0cc123afbe504da2462ca
