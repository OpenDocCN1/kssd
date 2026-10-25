
# mIRC v6.16

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-247.htm`](https://www.pediy.com/kssd/pediy07/pediy7-247.htm)

**作者**：`aalloverred` (菜菜一只)  
**工具**：`Ollydbg_fix`  
**软件**：`mIRC v6.16`

**介绍**：
`IRC` (Internet Relay Chat) is a virtual meeting place where people from all over the world can meet and talk; you'll find the whole diversity of human interests, ideas, and issues here, and you'll be able to participate in group discussions on one of the many thousands of `IRC` channels, on hundreds of `IRC` networks, or just talk in private to family or friends, wherever they are in the world.

**闲话**：
呵呵,菜菜在学习,多多指点... ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

**开始**：

开始上网也很有一段时日了,呵呵,那天才碰着个叫 `IRC` (Internet Relay Chat)的东西,聊天用的,好了,试试吧.最牛的 Client 号称是叫 `mIRC`,到了它的官网(`http://www.mirc.com/`)上当了一个最新版本 `6.16`,不赖...可是,要注册,晕,国外的...嘿嘿嘿...操刀...

在 `register` 框中输入用户名 `aalloverred`,注册码 `1234567890`

下断点 `GetWindowText`,`GetDlgItemText`,`MessageBox...`,`Pointh...` 全部失败......菜菜我实在是没辙了,只好下了个 `Apibreak` 插件试试,又经过了一番"众里寻她千百度"的折腾,终于断在了"`SendDlgItemMessage`"上,晕掉!

```
004C7BAA    .   68 E0075A00 push mirc.005A07E0               ; /lParam = 5A07E0
004C7BAF    .   68 E7030000 push 3E7                         ; |wParam = 3E7
004C7BB4    .   6A 0D      push 0D                            ; |Message = WM_GETTEXT
004C7BB6    .   68 83000000 push 83                           ; |ControlID = 83 (131.)
004C7BBB    .   56       push esi                             ; |hWnd
004C7BBC    .   FFD7     call edi                             ; \SendDlgItemMessageA<------------断在这里
004C7BBE    .   68 C80B5A00 push mirc.005A0BC8               ; /lParam = 5A0BC8
004C7BC3    .   68 E7030000 push 3E7                         ; |wParam = 3E7
004C7BC8    .   6A 0D      push 0D                            ; |Message = WM_GETTEXT
004C7BCA    .   68 84000000 push 84                           ; |ControlID = 84 (132.)
004C7BCF    .   56       push esi                             ; |hWnd
004C7BD0    .   FFD7     call edi                             ; \SendDlgItemMessageA
004C7BD2    .   BA C80B5A00 mov edx,mirc.005A0BC8             ;   ASCII "123456790"
004C7BD7    .   B9 E0075A00 mov ecx,mirc.005A07E0             ;   ASCII "aalloverred"
004C7BDC    .   E8 1FFAFFFF call mirc.004C7600     <-----关键 Call?
004C7BE1    .   85C0     test eax,eax                   <-----关键比较?:D
004C7BE3    .   0F84 E50000>je mirc.004C7CCE     <------关键跳转?
004C7BE9    .   C605 3A085A>mov byte ptr ds:[5A083A],0
004C7BF0    .   33C0     xor eax,eax
004C7BF2    >   8A88 E0075A>mov cl,byte ptr ds:[eax+5A07E0]
004C7BF8    .   8888 A89D5A>mov byte ptr ds:[eax+5A9DA8],cl
004C7BFE    .   40       inc eax

.
.
.

004C7C2F    .   C705 0C9E5A>mov dword ptr ds:[5A9E0C],0
004C7C39    .   E8 12480100 call mirc.004DC450
004C7C3E    .   50       push eax                             ; |hMenu
004C7C3F    .   FF15 B43558>call dword ptr ds:[<&USER32.Delete>; \DeleteMenu
004C7C45    .   6A 01      push 1                             ; /Result = 1
004C7C47    .   56       push esi                             ; |hWnd
004C7C48    .   FF15 103658>call dword ptr ds:[<&USER32.EndDia>; \EndDialog

.
.
.

004C7D4B    .   8BD0     mov edx,eax                          ; |
004C7D4D    .   8BCE     mov ecx,esi                          ; |
004C7D4F    .   E8 ACFEFEFF call mirc.004B7C00        ; \mirc.004B7C00<------到这里蹦出了错误框
```

又经过一番折腾,我便"臆"然断定 `004C7BDC` 处是关键了(在 `SendDlgItemMessageA` 和 `004C7D4F` 处的错误框之间只有这个 `Call` 后面有紧随其后的比较和跳转),取消以前所有断点,设断 `004C7BDC`,再运行后,`F7` 跟进.

```
004C7600   /$  81EC 080200>sub esp,208
004C7606   |.   56       push esi
004C7607   |.   57       push edi
004C7608   |.   8BF2     mov esi,edx
004C760A   |.   8BF9     mov edi,ecx
004C760C   |.   BA 04010000 mov edx,104
004C7611   |.   8D4C24 08   lea ecx,dword ptr ss:[esp+8]
004C7615   |.   E8 1679F5FF call mirc.0041EF30      <-fill 0,rubbish?
004C761A   |.   BA 04010000 mov edx,104
004C761F   |.   8D8C24 0C01>lea ecx,dword ptr ss:[esp+10C]
004C7626   |.   E8 0579F5FF call mirc.0041EF30      <-again?!
004C762B   |.   57       push edi
004C762C   |.   BA 04010000 mov edx,104
004C7631   |.   8D4C24 0C   lea ecx,dword ptr ss:[esp+C]
004C7635   |.   E8 0678F5FF call mirc.0041EE40      <-与 41EF30 的 call 很像
004C763A   |.   56       push esi
004C763B   |.   BA 04010000 mov edx,104
004C7640   |.   8D8C24 1001>lea ecx,dword ptr ss:[esp+110]
004C7647   |.   E8 F477F5FF call mirc.0041EE40      <-again?!
004C764C   |.   8D9424 0C01>lea edx,dword ptr ss:[esp+10C]
004C7653   |.   8D4C24 08   lea ecx,dword ptr ss:[esp+8]
004C7657   |.   E8 A4FDFFFF call mirc.004C7400      <-----比较 1
004C765C   |.   85C0     test eax,eax
004C765E   |.   74 0E      je short mirc.004C766E
004C7660   |.   5F       pop edi
004C7661   |.   B8 01000000 mov eax,1
004C7666   |.   5E       pop esi
004C7667   |.   81C4 080200>add esp,208
004C766D   |.   C3       retn        <-出口(1)
004C766E   |>   BA 04010000 mov edx,104
004C7673   |.   8D4C24 08   lea ecx,dword ptr ss:[esp+8]
004C7677   |.   E8 B478F5FF call mirc.0041EF30      <-again...fill 0,rubbish?
004C767C   |.   BA 04010000 mov edx,104
004C7681   |.   8D8C24 0C01>lea ecx,dword ptr ss:[esp+10C]
004C7688   |.   E8 A378F5FF call mirc.0041EF30
004C768D   |.   57       push edi
004C768E   |.   BA 04010000 mov edx,104
004C7693   |.   8D4C24 0C   lea ecx,dword ptr ss:[esp+C]
004C7697   |.   E8 A477F5FF call mirc.0041EE40
004C769C   |.   56       push esi
004C769D   |.   BA 04010000 mov edx,104
004C76A2   |.   8D8C24 1001>lea ecx,dword ptr ss:[esp+110]
004C76A9   |.   E8 9277F5FF call mirc.0041EE40
004C76AE   |.   BA 02000000 mov edx,2
004C76B3   |.   8D4C24 08   lea ecx,dword ptr ss:[esp+8]
004C76B7   |.   E8 F41CFBFF call mirc.004793B0
004C76BC   |.   8D9424 0C01>lea edx,dword ptr ss:[esp+10C]
004C76C3   |.   8D4C24 08   lea ecx,dword ptr ss:[esp+8]
004C76C7   |.   E8 34FDFFFF call mirc.004C7400
004C76CC   |.   F7D8     neg eax
004C76CE   |.   1BC0     sbb eax,eax
004C76D0   |.   5F       pop edi
004C76D1   |.   F7D8     neg eax
004C76D3   |.   5E       pop esi
004C76D4   |.   81C4 080200>add esp,208
004C76DA   \.   C3       retn        <--出口(2)
```

`004C7657` 处的 `Call` 看起来有些关键的样子,跟进之后有一大堆 `mov`,之后是下面这样子:

```
:

004C7518    > /8A10     mov dl,byte ptr ds:[eax]  <---eax(和 esi)是存储用户名的地方
004C751A    . |40       inc eax
004C751B    . |84D2     test dl,dl      <---用户名结束?
004C751D    . ^\75 F9    jnz short mirc.004C7518
004C751F    .   2BC6     sub eax,esi     <---得到用户名长度
004C7521    .   3BC1     cmp eax,ecx     <---和 ecx(此时值为 4)比较
004C7523    .   73 0D      jnb short mirc.004C7532    <---大于 4 就跳
004C7525    >   5F       pop edi
004C7526    .   5E       pop esi
004C7527    .   5D       pop ebp
004C7528    .   33C0     xor eax,eax
004C752A    .   5B       pop ebx
004C752B    .   81C4 A40000>add esp,0A4
004C7531    .   C3       retn
004C7532    >   8B5C24 10   mov ebx,dword ptr ss:[esp+10]
004C7536    .   6A 2D      push 2D
004C7538    .   53       push ebx
004C7539    .   E8 228D0A00 call mirc.00570260    <---输入的注册码中有"-"?
004C753E    .   8BF0     mov esi,eax
004C7540    .   83C4 08     add esp,8
004C7543    .   85F6     test esi,esi
004C7545    . ^ 74 DE     je short mirc.004C7525
004C7547    .   53       push ebx
004C7548    .   C606 00     mov byte ptr ds:[esi],0
004C754B    .   E8 F38F0A00 call mirc.00570543    <---将输入的注册码转换成数值(前半部分)
004C7550    .   83C4 04     add esp,4
004C7553    .   C606 2D     mov byte ptr ds:[esi],2D
004C7556    .   46       inc esi
004C7557    .   8BE8     mov ebp,eax
004C7559    .   803E 00     cmp byte ptr ds:[esi],0
```



```
004C755C    .  ^ 74 C7       je short mirc.004C7525
004C755E    .    56         push esi
004C755F    .  E8 DF8F0A00 call mirc.00570543    ; <-- 将输入的注册码转换成数值(后半部分)
004C7564    .  8BCF        mov ecx, edi
004C7566    .  83C4 04     add esp, 4
004C7569    .  894424 10   mov dword ptr ss:[esp+10], eax
004C756D    .  8D51 01     lea edx, dword ptr ds:[ecx+1]
004C7570    >  8A01        mov al, byte ptr ds:[ecx] ; <-- 又开始计算用户名长度
004C7572    .  41          inc ecx
004C7573    .  84C0        test al, al
004C7575    .  ^ 75 F9     jnz short mirc.004C7570
004C7577    .  2BCA        sub ecx, edx ; <-- 得到长度
004C7579    .  BE 03000000 mov esi, 3 ; <-- 从用户名第 3 位开始
004C757E    .  33D2        xor edx, edx
004C7580    .  33DB        xor ebx, ebx
004C7582    .  3BCE        cmp ecx, esi
004C7584    .  7E 22       jle short mirc.004C75A8
004C7586    .  EB 08       jmp short mirc.004C7590
004C7588    .  8DA424 0000> lea esp, dword ptr ss:[esp]
004C758F    .  90          nop
004C7590    >  0FB6043E    movzx eax, byte ptr ds:[esi+edi] ; <-- 开始计算
004C7594    .  0FAF4494 14 imul eax, dword ptr ss:[esp+edx*4+14] ; <-- 用户名字符与对应常数值相乘
004C7599    .  03D8        add ebx, eax ; <-- 将每次结果循环加入
004C759B    .  42          inc edx
004C759C    .  83FA 26     cmp edx, 26
004C759F    .  7E 02       jle short mirc.004C75A3
004C75A1    .  33D2        xor edx, edx
004C75A3    >  46          inc esi
004C75A4    .  3BF1        cmp esi, ecx
004C75A6    .  ^ 7C E8     jl short mirc.004C7590
004C75A8    >  3BEB        cmp ebp, ebx ; <-- 此处 ebp=4d2 (十进制 1234), ebx=260d
004C75AA    .  ^ 0F85 75FFFF> jnz mirc.004C7525 ; <-- 跳了就死...
004C75B0    .  BE 03000000 mov esi, 3 ; <-- 又从第三位开始
004C75B5    .  33D2        xor edx, edx
004C75B7    .  33DB        xor ebx, ebx
004C75B9    .  3BCE        cmp ecx, esi
004C75BB    .  7E 23       jle short mirc.004C75E0
004C75BD    .  8D49 00     lea ecx, dword ptr ds:[ecx]
004C75C0    >  0FB6443E FF movzx eax, byte ptr ds:[esi+edi-1] ; <-- 取前一个字符
004C75C5    .  0FB62C3E    movzx ebp, byte ptr ds:[esi+edi] ; <--
004C75C9    .  0FAFC5      imul eax, ebp ; <-- 本字符与前一个字符相乘
004C75CC    .  0FAF4494 14 imul eax, dword ptr ss:[esp+edx*4+14]
004C75D1    .  03D8        add ebx, eax
004C75D3    .  42          inc edx
004C75D4    .  83FA 26     cmp edx, 26
004C75D7    .  7E 02       jle short mirc.004C75DB
004C75D9    .  33D2        xor edx, edx
004C75DB    >  46          inc esi
004C75DC    .  3BF1        cmp esi, ecx
004C75DE    .  ^ 7C E0     jl short mirc.004C75C0
004C75E0    >  8B4C24 10   mov ecx, dword ptr ss:[esp+10]
004C75E4    .  5F          pop edi
004C75E5    .  5E          pop esi
004C75E6    .  33C0        xor eax, eax
004C75E8    .  3BCB        cmp ecx, ebx ; <-- ecx=162e (10 进制 5678), ebx=00103F5C
004C75EA    .  5D          pop ebp
004C75EB    .  0F94C0      sete al ; <-- al!=1, 出去就死...
004C75EE    .  5B          pop ebx
004C75EF    .  81C4 A40000> add esp, 0A4
004C75F5    .  C3          retn
```

呵呵,先判断长度,我的用户名长度大于 `4`,怪不得没从 `4C7531` 这个出口出去呢(众人晕!).

继续向下,先 `F8` 粗略的跟了一遍后,我跟进了几乎每一个 `call`,好多我都看不懂是干什么的...汗 ![](img/d8960f6c7afa20ba0ca27920887f4540.jpg)

比如过了 `570260` 处的 `call` 后,程序从 `004C7545` 处跳了回去,然后又出现了错误框,所以我跟到其中,

来到了自认为是关键的这里:

```
00570298   |> /8B0A        /mov ecx,dword ptr ds:[edx]        ; ; 输入的注册码前 4 位
0057029A   |. |BF FFFFE7E  |mov edi,7EFEFEFF
0057029F   |. |8BC1        |mov eax,ecx                       ; eax=ecx="1234"
005702A1   |. |8BF7        |mov esi,edi                       ; esi=edi=7efefeff(7efefeff xor 81010100=ffffffff)
005702A3   |. |33CB        |xor ecx,ebx                       ; ebx="----"
005702A5   |. |03F0        |add esi,eax
005702A7   |. |03F9        |add edi,ecx
005702A9   |. |83F1 FF     |xor ecx,FFFFFFFF
005702AC   |. |83F0 FF     |xor eax,FFFFFFFF
005702AF   |. |33CF        |xor ecx,edi
005702B1   |. |33C6        |xor eax,esi
005702B3   |. |83C2 04     |add edx,4
005702B6   |. |81E1 000101>|and ecx,81010100
005702BC   |. |75 1C       |jnz short mirc.005702DA
005702BE   |. |25 00010181 |and eax,81010100
005702C3   |.^ 74 D3       |je short mirc.00570298
005702C5   |. |25 00010101 |and eax,1010100
005702CA      |75 08       jnz short mirc.005702D4            ; bad jmp
005702CC   |. |81E6 000000>|and esi,80000000
005702D2     ^ 75 C4       jnz short mirc.00570298
```

看到它与 `"----"` 发生了一堆复杂的关系,我又 `"臆"` 然断定用户名中必须有 `"-"` 了,(蒙的呀!晕!)

再来,这次改为 `"1234-5678"` 果然没有再跳回去.

我又跟进了 `570543` 处的 `Call`,到了这里:

```
0057059E   |> /83F9 30    /cmp ecx,30
005705A1   |. |7C 0A      |jl short mirc.005705AD
005705A3   |. |83F9 39    |cmp ecx,39         ; <-- 是否数字?
005705A6   |. |7F 05      |jg short mirc.005705AD
005705A8   |. |83E9 30    |sub ecx,30
005705AB   |. |EB 03      |jmp short mirc.005705B0
005705AD   |> |83C9 FF    |or ecx,FFFFFFFF
005705B0   |> |83F9 FF    |cmp ecx,-1
005705B3   |. |74 0C      |je short mirc.005705C1
005705B5   |. |8D0480     |lea eax,dword ptr ds:[eax+eax*4]
005705B8   |. |8D0441     |lea eax,dword ptr ds:[ecx+eax*2] ; <-- 这两句其实就是 ecx+eax*10
005705BB   |. |0FB60E     |movzx ecx,byte ptr ds:[esi]
005705BE   |. |46         |inc esi
005705BF   |.^ \EB DD     \jmp short mirc.0057059E
```

搞了半天终于搞懂了,原来是把字符串转换成数值的,即 `"1234"->1234`...我太笨了.

继续向下,`004C7594` 处,`[esp+edx*4+14]` 这里放着一堆数,我当成常数处理,又被我蒙对了:D ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

到了 `4C75AA` 就死了,呵呵,所以将前半部分改为 `9741`(`16` 进制 `260d`),再来,成功通过这一关.

到了 `004C75EB` 处,`Ollydbg` 显示条件为假 `FALSE`,然后出去就又完了,所以将后半部分改为

`1064796`(`103F5C`),即为 `9412-1064796`,再来,成功!!:)

白费了,以后的代码我都已经跟过了!嘿嘿!

### 总结算法

注册码格式为:`XXXX-XXXX`(`x` 数量不定)

先将用户名从第 `3` 位开始分别与一些常数相乘之后,总和作为前半部分

再将用户名从第 `3` 位开始分别与前一位相乘再与那些常数相乘之后,总和作为前半部分

### 其它

刚刚学习很多地方不懂是干什么的,好多运气的成分

当然更有许多错误,希望大虾们有空能给修正和指点一下,感激不尽.

最后又写了一个供大家见笑的注册机.

```asm
const1 db   0bh,06h,011h,0ch,0ch,0eh
      db   05h,0ch,010h,0ah, 0bh,06h,0eh,0eh
      db   04h,0bh,06h,0eh,0eh,04h,0bh,09h,0ch,0bh, 0ah,08h,0ah,0ah
      db   010h,08h,04h,06h,0ah,0ch,010h,08h,0ah,04h,010h,0

sHyph db "-", 0
sFormat db "%d", 0

buffer db 512 dup(?)
buffer1 db 512 dup(?)
sUser db 512 dup(?)

IDC_USER equ 1001
IDC_KEY equ 1002
IDC_GETKEY equ 1003

invoke GetDlgItemText, hWnd, IDC_USER, addr sUser, 512
xor eax, eax
xor ebx, ebx
xor edx, edx
xor esi, esi
mov ecx, 3
mov al, byte ptr [sUser+ecx]

loop1:
movzx eax, al
mov dl, byte ptr [const1+ecx-3]
movzx edx, dl
imul eax, edx
add ebx, eax
mov al, byte ptr [sUser+ecx-1] ; latter half of key
movzx eax, al
mov dl, byte ptr [sUser+ecx]
movzx edx, dl
imul eax, edx
xor edx, edx
mov dl, byte ptr [const1+ecx-3]
movzx edx, dl
imul eax, edx
add esi, eax
inc ecx
mov al, byte ptr [sUser+ecx]
cmp al, 0
jnz loop1

invoke wsprintf, addr buffer, addr sFormat, ebx
invoke wsprintf, addr buffer1, addr sFormat, esi
invoke lstrcat, addr buffer, addr sHyph
invoke lstrcat, addr buffer, addr buffer1
popad
; invoke MessageBox, hWnd, addr buffer, addr AppName, MB_OK
invoke SetDlgItemText, hWnd, IDC_KEY, addr buffer
```

`=====================================================================`

`[结束]`

`4dd466d0e1e0cc123afbe504da2462ca`
