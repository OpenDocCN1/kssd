
# 迷你日历 V2.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50425.htm`](https://www.pediy.com/kssd/pediy05/pediy50425.htm)

算法分析——迷你日历 V2.2(`MiniCalendar`)

**基本信息**

*   下载地址： [`www.skycn.com/soft/9249.html`](http://www.skycn.com/soft/9249.html)
*   软件大小： `814 KB`
*   软件语言： 简体中文
*   软件类别： 国产软件 / 共享版 / 时钟日历
*   应用平台： `Win9x` / `NT` / `2000` / `XP`
*   加入时间： `2003-02-07 11:06:35`
*   下载次数： `15122`
*   推荐等级： ***
*   开发商： [`asp2.6to23.com/gar/`](http://asp2.6to23.com/gar/)

### 【软件简介】

迷你日历是集公历农历(`1901-2099` 年)双界面对照查询、记事栏、记事查询、重要记事动态提醒、自定义节日、节日预告、日历打印和多种日历信息(节气、节日、九九、三伏、生肖、时辰等)查询于一身的多功能日历软件。其具体特点简述如下：

1.  **公历、农历双界面**：可按照以公历为主或以农历为主的两种界面显示日历，两种界面可随意切换，在任一界面中均可进行农历和公历的对照查询，这对以农历为主计算日期的人们来说尤为方便。
2.  **记事栏、记事查询**：软件为每一日期安排有记事栏，你可以将日程安排或备忘等写入记事栏；当日久天长记事很多时，可利用记事查询功能方便地找到以往的记事。
3.  **重要记事动态提醒**：可将你认为重要的记事定义为动态提醒记事，这样软件会定时动态地在屏幕上用醒目的游动提醒窗显示重要记事，并伴有铃声及动画提醒。
4.  **界面配置魔术师**：能满足不同网友的个人爱好，可随意改变界面不同区域的色彩、字体、换肤、贴图等等，定制出变化无穷、丰富多采的日历界面。
5.  **自定义节日**：可将你喜欢的节日或你的生日定义到日历当中，使日历更具个性化。
6.  **节日预告**：可分别对各类节日进行预告，这将更加方便你对节日的安排。
7.  **打印日历**：可打印任意尺寸、独具特色的精美日历。

### 【软件限制】

功能限制

### 【作者声明】

小弟初学 Crack，只是感兴趣，没有其它目的。失误之处敬请各大侠赐教！

### 【破解工具】

`TRW2000` 娃娃修改版、`W32Dasm8.93` 黄金版、`Ollydbg1.09`

---

### 【过程】

准备好一杯浓茶、一盒烟、一支笔、一本草稿纸、一颗安静的心……

呵呵，我们开工吧！唉！^-^^-^ 我的水平很低，许多地方表达的有问题，烦请各位老师指教！

*   机器码：`A8F4W303XY`
*   试炼码：`BCDEFGHIJKSTUVWXCDEF`

（试炼码须 `20` 位，且有限制。单是试炼码就不太好填 ^-^）

程序无壳，`Delphi` 编写。反汇编。还好，重要提示都在。

---

```
:004A8F86 E8FD0BFAFF                  call 00449B88
:004A8F8B 8B45F0                      mov eax, dword ptr [ebp-10]
:004A8F8E 8D55F4                      lea edx, dword ptr [ebp-0C]
:004A8F91 E8CAF8FEFF                  call 00498860
:004A8F96 8B45F4                      mov eax, dword ptr [ebp-0C]
:004A8F99 8D55F8                      lea edx, dword ptr [ebp-08]
:004A8F9C E8DB00F6FF                  call 0040907C
:004A8FA1 8B45F8                      mov eax, dword ptr [ebp-08]
:004A8FA4 E897BEF5FF                  call 00404E40
:004A8FA9 83F814                      cmp eax, 00000014
                                      ====>试炼码是否 20 位？
:004A8FAC 741D                       je 004A8FCB
                                      ====>应跳！
:004A8FAE 6A00                       push 00000000
:004A8FB0 B914914A00                 mov ecx, 004A9114
* Possible StringData Ref from Code Obj ->"注册码输入长度非法 ！"
                                      |
:004A8FB5 BA20914A00                 mov edx, 004A9120
:004A8FBA A194314F00                 mov eax, dword ptr [004F3194]
:004A8FBF 8B00                       mov eax, dword ptr [eax]
:004A8FC1 E8360EFCFF                 call 00469DFC
:004A8FC6 E90C010000                 jmp 004A90D7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A8FAC(C)
|
:004A8FCB 33FF                       xor edi, edi
:004A8FCD B801000000                 mov eax, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A8FF9(C)
```

下面是检查输入的注册码是否在 `BCDEFGHIJKSTUVWX` 之中，如不在，则为“非法字符”！

```
:004A8FD2 8B55F8                      mov edx, dword ptr [ebp-08]
:004A8FD5 8A5402FF                   mov dl, byte ptr [edx+eax-01]
:004A8FD9 8BCA                       mov ecx, edx
:004A8FDB 80C1BE                     add cl, BE
:004A8FDE 80E90A                     sub cl, 0A
:004A8FE1 7212                       jb 004A8FF5
:004A8FE3 8B4DF8                     mov ecx, dword ptr [ebp-08]
:004A8FE6 80C2AD                     add dl, AD
:004A8FE9 80EA06                     sub dl, 06
:004A8FEC 7207                       jb 004A8FF5
:004A8FEE BF01000000                 mov edi, 00000001
:004A8FF3 EB06                       jmp 004A8FFB
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004A8FE1(C), :004A8FEC(C)
|
:004A8FF5 40                         inc eax
:004A8FF6 83F815                    cmp eax, 00000015
:004A8FF9 75D7                       jne 004A8FD2
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A8FF3(U)
|
:004A8FFB 4F                         dec edi
:004A8FFC 751D                       jne 004A901B
                                      ====>应跳！
:004A8FFE 6A00                       push 00000000
:004A9000 B914914A00                 mov ecx, 004A9114
* Possible StringData Ref from Code Obj ->"注册码中输入了非法字符 ！"
                                      |
:004A9005 BA38914A00                 mov edx, 004A9138
:004A900A A194314F00                 mov eax, dword ptr [004F3194]
:004A900F 8B00                       mov eax, dword ptr [eax]
:004A9011 E8E60DFCFF                 call 00469DFC
:004A9016 E9BC000000                 jmp 004A90D7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A8FFC(C)
|
:004A901B 8D45EC                     lea eax, dword ptr [ebp-14]
:004A901E E88102FFFF                 call 004992A4
:004A9023 8B45EC                     mov eax, dword ptr [ebp-14]
:004A9026 50                         push eax
:004A9027 8D55E8                     lea edx, dword ptr [ebp-18]
:004A902A 8B45F8                     mov eax, dword ptr [ebp-08]
:004A902D E8B2EEFEFF                 call 00497EE4
                                      ====>关键 CALL！F8 进入！
:004A9032 8B55E8                     mov edx, dword ptr [ebp-18]
                                      ====>D EDX=1BDAC2999555E7B2EBB6
:004A9035 58                         pop eax
                                      ====>D EAX=A8F4W303XY
:004A9036 E849BFF5FF                 call 00404F84
                                      ====>比较 CALL！F8 进入！
:004A903B 756E                       jne 004A90AB
                                      ====>跳则 OVER！
:004A903D 8B45F8                     mov eax, dword ptr [ebp-08]
:004A9040 E807ECFEFF                 call 00497C4C
:004A9045 A1DC304F00                 mov eax, dword ptr [004F30DC]
:004A904A C70001000000               mov dword ptr [eax], 00000001
:004A9050 A1F42E4F00                 mov eax, dword ptr [004F2EF4]
:004A9055 C70001000000               mov dword ptr [eax], 00000001
:004A905B A16C2F4F00                 mov eax, dword ptr [004F2F6C]
:004A9060 C70001000000               mov dword ptr [eax], 00000001
:004A9066 A1842C4F00                 mov eax, dword ptr [004F2C84]
:004A906B C70001000000               mov dword ptr [eax], 00000001
:004A9071 A1A8324F00                 mov eax, dword ptr [004F32A8]
:004A9076 C70001000000               mov dword ptr [eax], 00000001
:004A907C A19C334F00                 mov eax, dword ptr [004F339C]
:004A9081 C70058020000               mov dword ptr [eax], 00000258
:004A9087 6A00                       push 00000000
* Possible StringData Ref from Code Obj ->"注册《迷你日历》提醒 ！！！"
                                      |
:004A9089 B954914A00                 mov ecx, 004A9154
* Possible StringData Ref from Code Obj ->"真诚地感谢您对《迷你日历》开发工作的支持 "
                                      ->"！
请注意：您现在使用的是共享免费版，功能受到"
                                      ->"限制，
只有使用我们发给您的真正共享版进行注册"
                                      ->"，才能得到全部功能。
      "
                                      ->"             祝您注册顺利 ！   
"
                                      |
:004A908E BA70914A00                 mov edx, 004A9170
:004A9093 A194314F00                 mov eax, dword ptr [004F3194]
:004A9098 8B00                       mov eax, dword ptr [eax]
:004A909A E85D0DFCFF                 call 00469DFC
:004A909F 8B55FC                     mov edx, dword ptr [ebp-04]
:004A90A2 8BC6                       mov eax, esi
:004A90A4 E80BFEFFFF                 call 004A8EB4
:004A90A9 EB2C                       jmp 004A90D7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A903B(C)
|
:004A90AB A1DC304F00                 mov eax, dword ptr [004F30DC]
:004A90B0 33D2                       xor edx, edx
:004A90B2 8910                       mov dword ptr [eax], edx
:004A90B4 A19C334F00                 mov eax, dword ptr [004F339C]
:004A90B9 C70006000000               mov dword ptr [eax], 00000006
```
