
# 凌云邮神

> 原文：[`www.pediy.com/kssd/pediy04/kanxue221.htm`](https://www.pediy.com/kssd/pediy04/kanxue221.htm)

## 『凌云邮神』注册码破解

（非明码比较的哦 ^\_^）

**作者**：`PaulYoung`（属于 `China Cracking Group`）

**软件**：凌云邮神（`http://fang_public.myetang.com/Dewnload/LYMail.zip` 267 KB）

**简介**：支持国内绝大多数邮件服务器，支持附件发送，可发送无限个附件；支持 `SMTP` 身份验证；速度奇快。可以设置优先级。本软件区别于其他邮件群发软件的最大特点就是在收信地址正确的情况下只须发送次邮件的时间即可发完全部邮件，极大地缩短了上网时间。

**加密**：一机一码，非明码比较

**工具**：`SoftICE 4.05 334`

**日期**：`2001.11.05`

---

最近对爆破和明码比较的软件完全失去破解的兴趣了，内存注册机也玩到腻了，专心搞一些非明码破解和加强对加密算法的研究。下面这个软件，是一个非明码比较的软件，比较简单，细心一点不难破解。如果哪位朋友有一些非明码比较的软件，欢迎向我推荐，最好几百 KB 就好了，还有，`DOS` 和 `VB` 就免了，小生怕怕 :) 呵……忘了告诉大家怎么联系我，`My E-mail: paulyoung@21cn.com`，`QQ:65827779`，不要寄病毒来哦！

为了节约时间，相信大家也非常容易就找到它的关键所在，另外为了叙述，下面的数字均是十进制。

算法验证就在下面了……

\* Referenced by a CALL at Addresses:

|:0046F40F , :0046F915 , :004704BF , :00470961 // WOO...四处地方验证注册码

```
:0046F208 55                        push ebp
:0046F209 8BEC                      mov ebp, esp
:0046F20B 33C9                      xor ecx, ecx
:0046F20D 51                        push ecx
:0046F20E 51                        push ecx
:0046F20F 51                        push ecx
:0046F210 51                        push ecx
:0046F211 51                        push ecx
:0046F212 51                        push ecx
:0046F213 51                        push ecx
:0046F214 53                        push ebx
:0046F215 33C0                      xor eax, eax
:0046F217 55                        push ebp
:0046F218 68F8F24600                push 0046F2F8
:0046F21D 64FF30                    push dword ptr fs:[eax]
:0046F220 648920                    mov dword ptr fs:[eax], esp
:0046F223 C645FF00                  mov [ebp-01], 00
:0046F227 8D45F8                    lea eax, dword ptr [ebp-08]
:0046F22A E811FEFFFF                call 0046F040
:0046F22F B201                      mov dl, 01
:0046F231 A160F84400                mov eax, dword ptr [0044F860]
:0046F236 E82507FEFF                call 0044F960
:0046F23B 8BD8                      mov ebx, eax
:0046F23D BA02000080                mov edx, 80000002
:0046F242 8BC3                      mov eax, ebx
:0046F244 E8B707FEFF                call 0044FA00
:0046F249 8D55EC                    lea edx, dword ptr [ebp-14]
:0046F24C A16C364700                mov eax, dword ptr [0047366C]
:0046F251 8B00                      mov eax, dword ptr [eax]
:0046F253 E8D8A2FDFF                call 00449530
:0046F258 8B4DEC                    mov ecx, dword ptr [ebp-14]
:0046F25B 8D45F0                    lea eax, dword ptr [ebp-10]

* Possible StringData Ref from Code Obj ->"SOFTWARE\"
                                |
:0046F25E BA10F34600                mov edx, 0046F310
:0046F263 E8004BF9FF                call 00403D68
:0046F268 8B55F0                    mov edx, dword ptr [ebp-10]
:0046F26B 8BC3                      mov eax, ebx
:0046F26D E8820CFEFF                call 0044FEF4
:0046F272 84C0                      test al, al
:0046F274 743A                      je 0046F2B0
:0046F276 8D55E4                    lea edx, dword ptr [ebp-1C]
:0046F279 A16C364700                mov eax, dword ptr [0047366C]
:0046F27E 8B00                      mov eax, dword ptr [eax]
:0046F280 E8ABA2FDFF                call 00449530
:0046F285 8B4DE4                    mov ecx, dword ptr [ebp-1C]
:0046F288 8D45E8                    lea eax, dword ptr [ebp-18]

* Possible StringData Ref from Code Obj ->"SOFTWARE\"
                                |
:0046F28B BA10F34600                mov edx, 0046F310
:0046F290 E8D34AF9FF                call 00403D68
:0046F295 8B55E8                    mov edx, dword ptr [ebp-18]
:0046F298 B101                      mov cl, 01
:0046F29A 8BC3                      mov eax, ebx
:0046F29C E89F08FEFF                call 0044FB40
:0046F2A1 8D4DF4                    lea ecx, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"RegSN"
                                |
:0046F2A4 BA24F34600                mov edx, 0046F324
:0046F2A9 8BC3                      mov eax, ebx
:0046F2AB E8580AFEFF                call 0044FD08

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0046F274(C)
|
:0046F2B0 8BC3                      mov eax, ebx
:0046F2B2 E81907FEFF                call 0044F9D0
:0046F2B7 837DF400                  cmp dword ptr [ebp-0C], 00000000
:0046F2BB 7420                      je 0046F2DD
:0046F2BD 8B45F8                    mov eax, dword ptr [ebp-08]
:0046F2C0 E88BFEFFFF                call 0046F150
:0046F2C5 8BD8                      mov ebx, eax        //注意，eax 的值变为 2547 了，并保存到 ebx
:0046F2C7 8B45F4                    mov eax, dword ptr [ebp-0C]
:0046F2CA E89990F9FF                call 00408368
:0046F2CF 03D8                      add ebx, eax        //来到这里，eax 就是你输入的注册码，ebx 的值为  2547 ，eax + ebx ，结果保存到 ebx
:0046F2D1 81FBBAAA9404              cmp ebx, 0494AABA     //ebx 与 0494AABA 的值比较
:0046F2D7 7504                      jne 0046F2DD          //不等则跳
:0046F2D9 C645FF01                  mov [ebp-01], 01      //不跳你就成功了！

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0046F2BB(C), :0046F2D7(C)
|
:0046F2DD 33C0                      xor eax, eax      //注册码不正确，跳到此 eax 清 0，完蛋！
:0046F2DF 5A                        pop edx
:0046F2E0 59                        pop ecx
:0046F2E1 59                        pop ecx
:0046F2E2 648910                    mov dword ptr fs:[eax], edx
:0046F2E5 68FFF24600                push 0046F2FF
```

我们重点分析 `0046F2D1` 这处关键的比较：

`ebx = 输入的注册码 + 2547` 的和，而 `0494AABA` 的值是 `76851898`，两者相等则成功，那么，那真正的注册码就是 `76851898 - 2547 = 76849351` :)

我的软件序号是：`1D6D-17E5`，注册码是：`76849351`，填入后，“注册完成”！再重启，WA...软件的注册按钮也不见了，成功了！！！！

注册码跟公司名和电话号码无关。

可惜小弟水平有限，汇编学得不精，尚未发现软件序号跟注册码之间的关系，`2547` 这个值是怎么来的呢？？恳请大侠出手，指点小弟一二，不胜感激。

## 『凌云邮神』之算法分析篇

**作者**：`PaulYoung`（属于 `China Cracking Group`）

**软件**：凌云邮神（`http://fang_public.myetang.com/Dewnload/LYMail.zip` 267 KB）

**简介**：支持国内绝大多数邮件服务器，支持附件发送，可发送无限个附件；支持 `SMTP` 身份验证；速度奇快。可以设置优先级。本软件区别于其他邮件群发软件的最大特点就是在收信地址正确的情况下只须发送次邮件的时间即可发完全部邮件，极大地缩短了上网时间。

**加密**：一机一码，非明码比较

**工具**：`SoftICE 4.05 334`

**日期**：`2001.11.05`

---

原文再续书接上一回，上回在注册码破解篇我们说到……（怎么象个说书人？:) ）

```
:0046F2C0 E88BFEFFFF                call 0046F150
:0046F2C5 8BD8                      mov ebx, eax        //注意，eax 的值变为 2547 了，并保存到 ebx
:0046F2C7 8B45F4                    mov eax, dword ptr [ebp-0C]
:0046F2CA E89990F9FF                call 00408368
:0046F2CF 03D8                      add ebx, eax        //来到这里，eax 就是你输入的注册码，ebx 的值为  2547 ，eax + ebx ，结果保存到 ebx
:0046F2D1 81FBBAAA9404              cmp ebx, 0494AABA     //ebx 与 0494AABA 的值比较
:0046F2D7 7504                      jne 0046F2DD          //不等则跳
:0046F2D9 C645FF01                  mov [ebp-01], 01      //不跳你就成功了！
```

很明显，`0046F2C0` 有重大的作案嫌疑，嘻……因为一到 `0046F2C5` `eax` 的值变成 `2547` 了，当然，这是我的序列号计算出的，你的当然不是了。我们就进入去看看，看看 `2547` 这个数是怎么来的，呵……还有，我试过在不同的电脑上，`0494AABA` 的值都是固定的。

各位看官，请进……

\* Referenced by a CALL at Address:

|:0046F2C0

```
:0046F150 55                        push ebp
:0046F151 8BEC                      mov ebp, esp
:0046F153 83C4D8                    add esp, FFFFFFD8
:0046F156 53                        push ebx
:0046F157 56                        push esi
:0046F158 57                        push edi
:0046F159 8945FC                    mov dword ptr [ebp-04], eax  //将参数保存到变量[ebp-04]
```



```
:0046F15C 8B45FC                 mov eax, dword ptr [ebp-04]

:0046F15F E86C4DF9FF             call 00403ED0

:0046F164 33C0                   xor eax, eax

:0046F166 55                     push ebp

:0046F167 68F2F14600             push 0046F1F2

:0046F16C 64FF30                 push dword ptr fs:[eax]

:0046F16F 648920                 mov dword ptr fs:[eax], esp

:0046F172 33C0                   xor eax, eax

:0046F174 8945F8                 mov dword ptr [ebp-08], eax  //变量[ebp-08]赋初值为 0

:0046F177 8D45E3                 lea eax, dword ptr [ebp-1D]

:0046F17A 8B55FC                 mov edx, dword ptr [ebp-04]

:0046F17D E88E95F9FF             call 00408710

:0046F182 8B45FC                 mov eax, dword ptr [ebp-04]

:0046F185 E8924BF9FF             call 00403D1C                   //取序列号长度

:0046F18A 8BF8                   mov edi, eax                   //edi  记录序列号长度

:0046F18C 85FF                   test edi, edi

:0046F18E 7E49                   jle 0046F1D9                   //当然不能为空

:0046F190 BB01000000             mov ebx, 00000001

:0046F195 8D75E3                 lea esi, dword ptr [ebp-1D]   //esi 指向序列号的字符

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046F1D7(C)

|

:0046F198 895DDC                 mov dword ptr [ebp-24], ebx

:0046F19B DB45DC                 fild dword ptr [ebp-24]         //浮点运算，被除数进栈

:0046F19E D83504F24600           fdiv dword ptr [0046F204]       //除以一个数

:0046F1A4 E87B37F9FF             call 00402924                   //结果转换成整数

:0046F1A9 895DD8                 mov dword ptr [ebp-28], ebx

:0046F1AC DB45D8                 fild dword ptr [ebp-28]         //浮点运算，被除数进栈

:0046F1AF D83504F24600           fdiv dword ptr [0046F204]       //除以一个数

:0046F1B5 DED9                   fcompp                           //比较两个浮点数

:0046F1B7 DFE0                   fstsw ax                           //将状态字写入 ax

:0046F1B9 9E                     sahf                               //状态字放入 FLAG 寄存器

:0046F1BA 750D                   jne 0046F1C9                       //是否相等？

　　看到这里，可能有人会问，这是啥呀？其实，上面从 `0046F198` - `0046F1BA` 是验证你取序列号的位数是奇数还是偶数，奇数则跳转，`EBX` 记录你所取序列号的位数。不信？你试试！

　　继续……

　　①当所取序列号的位数为偶数时，会来到这里……

:0046F1BC 33C0                   xor eax, eax

:0046F1BE 8A06                   mov al, byte ptr [esi]           //取 ESI 指向的字符到 EAX 中

:0046F1C0 F7EB                   imul ebx                           //和 EBX 相乘

:0046F1C2 03C3                   add eax, ebx                       //加 EBX

:0046F1C4 0145F8                 add dword ptr [ebp-08], eax       //加到变量[ebp-08]

:0046F1C7 EB0B                   jmp 0046F1D4                      //无条件跳转

　　②当所取序列号位数为奇数时，会跳到这里……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046F1BA(C)

|

:0046F1C9 33C0                   xor eax, eax

:0046F1CB 8A06                   mov al, byte ptr [esi]           //取 ESI 指向的字符到 EAX

:0046F1CD F7EB                   imul ebx                         //和 EBX 相乘

:0046F1CF 2BC3                   sub eax, ebx                     //减去 EBX

:0046F1D1 0145F8                 add dword ptr [ebp-08], eax      //放到变量[ebp-08]

　　无论所取序列号的位数是奇数还是偶数，每步计算完毕都会来到这里……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046F1C7(U)

|

:0046F1D4 43                      inc ebx      //EBX 递增

:0046F1D5 46                      inc esi      //ESI 递增

:0046F1D6 4F                      dec edi      //EDI 递减

:0046F1D7 75BF                   jne 0046F198 //EDI 为 0，循环结束，就是看你的序列号是否已经取完，未取完则继续循环，依此类推……真烦！！！！！！！

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046F18E(C)

|

:0046F1D9 8B5DF8                 mov ebx, dword ptr [ebp-08]

:0046F1DC 33C0                   xor eax, eax

:0046F1DE 5A                      pop edx

:0046F1DF 59                      pop ecx

:0046F1E0 59                      pop ecx

:0046F1E1 648910                 mov dword ptr fs:[eax], edx

:0046F1E4 68F9F14600             push 0046F1F9

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046F1F7(U)

|

:0046F1E9 8D45FC                 lea eax, dword ptr [ebp-04]

:0046F1EC E8AB48F9FF             call 00403A9C

:0046F1F1 C3                      ret

　　如此这般……经过一番折腾，出来时 `EAX` 值就变成 `2547` 了！

　　唉……也不知对不对，还望大侠们赐教！头昏眼花，不写了！！

`4dd466d0e1e0cc123afbe504da2462ca`
```
