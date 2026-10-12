
# 算法分析实战篇和应用篇之提高(三)关键变量引发的思考--- 一些高手误解的 crackme\_by\_topmint

> 原文：[`www.pediy.com/kssd/pediy10/64605.html`](https://www.pediy.com/kssd/pediy10/64605.html)

【文章标题】: 算法分析实战篇和应用篇之提高(三)关键变量引发的思考--- 一些高手误解的 crackme\_by\_topmint

【文章作者】: NBA2005

【作者邮箱】: stockfox1699@sohu.com

【软件名称】: crackme\_by\_topmint

【下载地址】: `http://bbs.pediy.com/showthread.php?t=46209`

【使用工具】: OD

【操作平台】: WIN XP

【软件介绍】: [PEDIY Crackme 竞赛 2007] [第一回] 第 16 队 topmint

摔碎牙琴焦尾寒,
子期不在向谁弹？
春风满面皆朋友,
欲觅知音难上难。

算法分析的三个基本问题：

1. 注册判断的关键位置在哪？具体又分为两部分：
   - A. 注册成功与失败的关键位置。
   - B. 注册判断的关键 CALL。
   - 判断方法：确定关键变量，按照就近原则顺藤摸瓜。
2. 注册判断的关键变量是谁？
3. 注册判断的程序流程。具体又分为两部分：
   - A. 注册正确的程序流程。
   - B. 注册正确的程序流程所要求的一些条件。

其中，关键的变量没有详细地说明。特补此篇。

## 第一节 简单的 CRACK ME

五一期间,我帮朋友做裁判,无聊时看到了这个简单的 CRACK ME。一做之下，这个 CRACK ME 却让我大吃一惊。这是一个不起眼的 CRACK ME，有明确的注册提示，很容易找到注册判断的关键位置：

```
004017ED    .  E8 4EFEFFFF   call    00401640            ;  关键 CALL
004017F2    .  A1 B0AA4000   mov     eax, dword ptr [40AAB0] ;  用户名的 ASCII 值累加和+1
004017F7    .  8B0D 5CAB4000 mov     ecx, dword ptr [40AB5C] ;  1B80
004017FD    .  3BC1          cmp     eax, ecx
004017FF    .  74 31         je      short 00401832
```

粗粗地读了一遍，将该注册判断的主程序段的与注册有关功能概括如下（按注册流程顺序）：

1. 用户名必须小于 255：
   ```
   00401697    .  8BD8 mov     ebx, eax
   00401699    .  81FB>cmp     ebx, 0FF
   0040169F    .  76 3>jbe     short 004016D3
   ```
2. KEY 不能为空：
   ```
   004016E4    .  8BE8 mov     ebp, eax
   004016E6    .  85ED test    ebp, ebp
   004016E8    .  75 2>jnz     short 00401710
   ```
3. 用户名必须是数字或字母：
   ```
   00401750    > /8A44>mov     al, byte ptr [esp+ecx+10]
   00401754    . |3C 3>cmp      al, 30
   00401756    . |7C 0>jl       short 0040175C
   00401758    . |3C 3>cmp      al, 39
   0040175A    . |7E 1>jle      short 00401774
   0040175C    > |3C 4>cmp      al, 41
   0040175E    . |7C 0>jl       short 00401764
   00401760    . |3C 5>cmp      al, 5A
   00401762    . |7E 1>jle      short 00401774
   00401764    > |3C 6>cmp      al, 61
   00401766    . |0F8C>jl       00401824
   0040176C    . |3C 7>cmp      al, 7A
   0040176E    . |0F8F>jg       00401824
   ```
4. KEY 的形式判断：`XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX`
   ```
   00401792    > \83FD>cmp      ebp, 27
   0040179B    .  0F85>jnz      00401855
   ```

   七个“-”的判断：
   ```
   0040174C    .  85DB test     ebx, ebx
   0040174E    .  7E 2>jle      short 0040177E
   004017A1    .  8A0D>mov      cl, byte ptr [40AA08]
   004017A7    .  B0 2>mov      al, 2D
   004017A9    .  3AC8 cmp      cl, al
   004017AB    .  0F85>jnz      00401855
   004017B1    .  3805>cmp      byte ptr [40AA0D], al
   004017B7    .  0F85>jnz      00401855
   004017BD    .  3805>cmp      byte ptr [40AA12], al
   004017C3    .  0F85>jnz      00401855
   004017C9    .  3805>cmp      byte ptr [40AA17], al
   004017CF    .  0F85>jnz      00401855
   004017D5    .  3805>cmp      byte ptr [40AA1C], al
   004017DB    .  75 7>jnz      short 00401855
   004017DD    .  3805>cmp      byte ptr [40AA21], al
   004017E3    .  75 7>jnz      short 00401855
   004017E5    .  3805>cmp      byte ptr [40AA26], al
   004017EB    .  75 6>jnz      short 00401855
   ```
5. 用户名的 ASCII 值累加和：
   ```
   00401774    > \0FBE>movsx   eax, al
   00401777    .  03D0 add      edx, eax
   00401779    .  41   inc      ecx
   0040177A    .  3BCB cmp      ecx, ebx
   0040177C    . ^ 7C D>jl      short 00401750
   0040177E    >  42   inc      edx
   0040177F    .  81FA>cmp      edx, 1B80
   00401785    .  7E 0>jle      short 00401792
   ```
6. 关键注册判断处：
   ```
   004017ED    .  E8 4EFEFFFF   call    00401640            ;  关键 CALL
   004017F2    .  A1 B0AA4000   mov     eax, dword ptr [40AAB0] ;  用户名的 ASCII 值累加和+1
   004017F7    .  8B0D 5CAB4000 mov     ecx, dword ptr [40AB5C] ;  1B80
   004017FD    .  3BC1          cmp     eax, ecx
   004017FF    .  74 31         je      short 00401832
   ```

这很简单嘛，我利用 CRACK ME 自身得出一组用户名：

用户名：`ZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZZ1NH`

KEY 随手输了一组。

```
0040177E    >  42   inc      edx      此处设断
```

## 第二节 第一个关键变量 [40AB5C] 的含义

这么简单？我有点不敢相信。现在伪注册成功可多了。就象上篇的例子，20-24 位不是数字也会出注册成功的提示，但不会将注册信息写入注册表，下次启动软件时读取注册信息判断却是未注册版。站在 CRACK ME 的作者角度考虑，肯定会对用户名的重复现象做限制，不然就真的太简单了。

出于这种考虑，我试验性地第二次注册，惊奇地发现居然失败了。果然，事情并不象看上去那么简单。看来里面还有目前我尚未发现的暗桩。

但是，从何处着手呢？初学者常常面临这样的困惑。这种情况通常有两种思路。一是透过现象看本质。通过你发现的异常处着手，逆向追踪问题的根源。二是经常与 CRACK ME 的作者换位思考。如果我是 CRACK ME 的作者，我会怎样处理？实践和思考是破解最有威力的武器。实践是为了发现问题，思考是为了更有效率地解决问题。发现问题越多，你掌握的知识就越多、越扎实。那些所谓的天分的天才，无非就是喜欢问为什么罢了。正是换位思考，我运气地识破了作者的伪注册成功。

伪注册成功的现象是第一次注册会成功，但第二次和第三次......都是失败的。这种情况按常理推断:程序中存在着一种变量，它的两个不同的值控制着注册成功和伪注册成功。首先，我们从现象着手，即从正确和错误提示入手。仔细阅读注册判断的主程序，发现只有这一处判断决定了成功和失败的注册：

```
004017ED    .  E8 4EFEFFFF   call    00401640            ;  关键 CALL
004017F2    .  A1 B0AA4000   mov     eax, dword ptr [40AAB0] ;  用户名的 ASCII 值累加和+1
004017F7    .  8B0D 5CAB4000 mov     ecx, dword ptr [40AB5C] ;  1B80
004017FD    .  3BC1          cmp     eax, ecx
004017FF    .  74 31         je      short 00401832
```

这次不同的是，多次实验，我发现了一个现象：`[40AB5C]`有两个值 `1B80` 和零，这两个不同的值控制着注册成功和伪注册成功。这是我发现的第一个关键变量。

在辛勤地实验中，我运气地发现了可以重复注册成功的一组 KEY：

NAME: `NBA2005QQIIJADKLFAGALKDSJFLKADGADHDJLKADFJGPOEJJQERTLKDFJALJFDKLGAQIJISAJFALJSDIGFJQIEJGIOQJEJIMHA`
key: `1234-5678-1234-5678-1234-5678-1234-5678`

一组可以成功注册的例子可以节省大量的时间和精力。

## 第三节 关键变量 [40AB5C] 的赋值处

关键变量 `[40AB5C]` 的两个值是如何赋值的？这是常识了，写入断点。发现了两处赋值所在：

1. 赋值为零：
   ```
   00401000   /$  57   push    edi
   00401001   |.  B9 1>mov     ecx, 11
   00401006   |.  33C0 xor     eax, eax
   00401008   |.  BF B>mov     edi, 0040AAB4
   0040100D   |.  F3:A>rep     stos dword ptr es:[edi]
   0040100F   |.  B9 1>mov     ecx, 11
   00401014   |.  BF 6>mov     edi, 0040AA6C
   00401019   |.  F3:A>rep     stos dword ptr es:[edi]
   0040101B   |.  B9 1>mov     ecx, 19
   00401020   |.  BF F>mov     edi, 0040AAF8            ;  42e7-7062-6c58-5275-35b1-9294-6ca4-d1a6
   00401025   |.  F3:A>rep     stos dword ptr es:[edi]
   00401027   |.  A3 5>mov     dword ptr [40AB5C], eax    赋值所在
   0040102C   |.  5F   pop     edi
   0040102D   \.  C3   retn
   ```
2. 赋值为 `1B80`：
   ```
   00401030   /$  81EC>sub     esp, 0A4
   00401036   |.  A1 5>mov     eax, dword ptr [40AB5C]
   0040103B   |.  40   inc     eax
   0040103C   |.  A3 5>mov     dword ptr [40AB5C], eax   赋值所在
   00401041   |.  33C0 xor     eax, eax
   00401043   |.  8944>mov     dword ptr [esp+4], eax
   00401047   |.  33C9 xor     ecx, ecx
   00401049   |.  8944>mov     dword ptr [esp+8], eax
   0040104D   |.  8944>mov     dword ptr [esp+C], eax
   00401051   |.  8944>mov     dword ptr [esp+10], eax
   00401055   |.  8844>mov     byte ptr [esp+14], al
   00401059   |.  B8 6>mov     eax, 0040AA6C
   0040105E   |>  8A10 /mov    dl, byte ptr [eax]
   00401060   |.  83C0>|add    eax, 4
   00401063   |.  80C2>|add    dl, 41
   00401066   |.  8854>|mov    byte ptr [esp+ecx+4], dl
   0040106A   |.  41   |inc     ecx
   0040106B   |.  3D A>|cmp     eax, 0040AAAC
   00401070   |.^ 7E E>\jle    short 0040105E
   00401072   |.  53   push    ebx
   00401073   |.  55   push    ebp
   00401074   |.  56   push    esi
   00401075   |.  8D44>lea     eax, dword ptr [esp+58]
   00401079   |.  57   push    edi
   0040107A   |.  50   push    eax
   0040107B   |.  E8 3>call    004019B0
   00401080   |.  8D4C>lea     ecx, dword ptr [esp+18]
   00401084   |.  6A 1>push    11
   00401086   |.  8D54>lea     edx, dword ptr [esp+64]
   0040108A   |.  51   push    ecx
   0040108B   |.  52   push    edx
   0040108C   |.  E8 4>call    004019E0
   00401091   |.  8D44>lea     eax, dword ptr [esp+38]
   00401095   |.  8D4C>lea     ecx, dword ptr [esp+6C]
   00401099   |.  50   push    eax
   0040109A   |.  51   push    ecx
   ```



## 逆向分析：注册机逻辑与关键变量控制

### 汇编代码段：`00401000` - `00401150`

```
0040109B   |.  E8 D>call    00402470                ; 内有 4019E0 下的 MD5 算法 401AB0  
004010A0   |.  83C4>add     esp, 18  
004010A3   |.  33F6 xor     esi, esi  
004010A5   |.  8D7C>lea     edi, dword ptr [esp+38]  
004010A9   |>  33D2 /xor    edx, edx  
004010AB   |.  8A54>|mov    dl, byte ptr [esp+esi+28]  
004010AF   |.  52   |push   edx  
004010B0   |.  68 3>|push    00408030                ; %02x  
004010B5   |.  6A 0>|push   2  
004010B7   |.  57   |push    edi  
004010B8   |.  E8 3>|call    004024F0  
004010BD   |.  83C4>|add     esp, 10  
004010C0   |.  46   |inc     esi  
004010C1   |.  83C7>|add     edi, 2  
004010C4   |.  83FE>|cmp     esi, 10  
004010C7   |.^ 7C E>\\jl      short 004010A9  
004010C9   |.  BA 0>mov     edx, 4  
004010CE   |.  8D74>lea     esi, dword ptr [esp+38]  
004010D2   |.  33C0 xor     eax, eax  
004010D4   |.  33ED xor     ebp, ebp  
004010D6   |.  B9 1>mov     ecx, 19  
004010DB   |.  BF F>mov     edi, 0040AAF8            ; 42e7-7062-6c58-5275-35b1-9294-6ca4-d1a6  
004010E0   |.  2BF2 sub     esi, edx  
004010E2   |.  C644>mov     byte ptr [esp+58], 0  
004010E7   |.  F3:A>rep     stos dword ptr es:[edi]  
004010E9   |.  8974>mov     dword ptr [esp+10], esi  
004010ED   |>  8D42>/lea    eax, dword ptr [edx-4]  
004010F0   |.  3BC2 |cmp     eax, edx  
004010F2   |.  7D 2>|jge      short 00401117  
004010F4   |.  B8 0>|mov     eax, 4  
004010F9   |.  8DBD>|lea     edi, dword ptr [ebp+40AAF8]  
004010FF   |.  8BC8 |mov     ecx, eax  
00401101   |.  03F2 |add     esi, edx  
00401103   |.  8BD9 |mov     ebx, ecx  
00401105   |.  C1E9>|shr     ecx, 2  
00401108   |.  F3:A>|rep     movs dword ptr es:[edi], dword ptr [esi]  
0040110A   |.  8BCB |mov     ecx, ebx  
0040110C   |.  83E1>|and     ecx, 3  
0040110F   |.  03E8 |add     ebp, eax  
00401111   |.  F3:A>|rep     movs byte ptr es:[edi], byte ptr [esi]  
00401113   |.  8B74>|mov     esi, dword ptr [esp+10]  
00401117   |>  83FA>|cmp     edx, 20  
0040111A   |.  7D 0>|jge      short 00401124  
0040111C   |.  C685>|mov     byte ptr [ebp+40AAF8], 2D  
00401123   |.  45   |inc      ebp  
00401124   |>  83C2>|add     edx, 4  
00401127   |.  83FA>|cmp     edx, 24  
0040112A   |.^ 7C C>\\jl      short 004010ED  
0040112C   |.  68 0>push    0040AA04                ; /1234-5678-1234-5678-1234-5678-1234-5678  
00401131   |.  68 F>push    0040AAF8                ; |42e7-7062-6c58-5275-35b1-9294-6ca4-d1a6  
00401136   |.  FF15>call    dword ptr [<&KERNEL32.lstrcmpA>] ; \\(initial cpu selection)  
0040113C   |.  F7D8 neg     eax  
0040113E   |.  1BC0 sbb     eax, eax  
00401140   |.  5F   pop      edi  
00401141   |.  40   inc      eax  
00401142   |.  5E   pop      esi  
00401143   |.  5D   pop      ebp  
00401144   |.  A3 6>mov     dword ptr [40AB60], eax  
00401149   |.  5B   pop      ebx  
0040114A   |.  81C4>add     esp, 0A4  
00401150   \\.  C3   retn  
```

#### 第四节 控制关键变量`40AB5C`不同赋值的判断处

仔细地阅读这两个程序段，从中并没有发现控制两个结果的所在。似乎又陷入了僵局，程序是在何处根据什么条件判断，给出这两种不同的数值呢？

从编程的常识入手，是很常规的手段。从编程的角度考虑，这两个不同的结果代表了一个控制条件主干的两个分支。换言之，这两个程序段的同一源头就是我们要找的地方。

*   `00401000` 的调用来自 `00401640`
*   `00401030` 的调用来自 `004015C3`

不是同一个地方啊？等等，`00401640` 好熟悉啊？啊，想起来了：

```
004017ED    .  E8 4EFEFFFF   call    00401640              ; 关键 CALL  
004017F2    .  A1 B0AA4000   mov     eax, dword ptr [40AAB0] ; 用户名的 ASCII 值累加和+1  
004017F7    .  8B0D 5CAB4000 mov     ecx, dword ptr [40AB5C] ; 1B80  
004017FD    .  3BC1          cmp     eax, ecx  
004017FF    .  74 31        je      short 00401832  
```

呵呵，绕了半天，又回到了起点，不同的是对程序流程的理解加深了。在关键 CALL 处，作者控制着关键变量`40AB5C`的不同赋值，从而控制真正的注册成功条件。进入 `00401640`：

```
00401640   /$  E8 B>call    00401000  
00401645   |.  6A 0>push    1  
00401647   |.  E8 6>call    004015B0  
0040164C   |.  59   pop     ecx  
0040164D   \\.  C3   retn  
```

其中 `00401647 |. E8 6>call 004015B0` 结合下句 `00401030` 的调用来自 `004015C3`，看来，控制关键变量`40AB5C`不同赋值的判断处在 `004015B0` 处：

```
004015B0   /$  A1 6>mov     eax, dword ptr [40AB60]  
004015B5   |.  57   push   edi  
004015B6   |.  85C0 test   eax, eax  
004015B8   |.  75 7>jnz     short 00401637  
004015BA   |.  8B7C>mov     edi, dword ptr [esp+8]         
004015BE   |.  83FF>cmp     edi, 10  
004015C1   |.  7E 0>jle     short 004015CA  
004015C3   |.  E8 6>call    00401030  
004015C8   |.  5F   pop     edi  
004015C9   |.  C3   retn  
004015CA   |>  8B04>mov     eax, dword ptr [edi*4+40AA6C]  
004015D1   |.  85C0 test   eax, eax  
004015D3   |.  75 5>jnz     short 00401625  
004015D5   |.  53   push    ebx  
004015D6   |.  56   push    esi  
004015D7   |.  BB 0>mov     ebx, 1  
004015DC   |.  BE B>mov     esi, 0040AAB8  
004015E1   |>  833E>/cmp     dword ptr [esi], 0  
004015E4   |.  75 2>|jnz     short 00401615  
004015E6   |.  57   |push    edi  
004015E7   |.  891C>|mov     dword ptr [edi*4+40AA6C], ebx  
004015EE   |.  C706>|mov     dword ptr [esi], 1  
004015F4   |.  E8 5>|call    00401250  
004015F9   |.  83C4>|add     esp, 4  
004015FC   |.  85C0 |test    eax, eax  
004015FE   |.  74 0>|je      short 0040160C  
00401600   |.  8D47>|lea     eax, dword ptr [edi+1]  
00401603   |.  50   |push    eax  
00401604   |.  E8 A>|call    004015B0  
00401609   |.  83C4>|add     esp, 4  
0040160C   |>  57   |push    edi  
0040160D   |.  E8 4>|call    00401160  
00401612   |.  83C4>|add     esp, 4  
00401615   |>  83C6>|add     esi, 4  
00401618   |.  43   |inc     ebx  
00401619   |.  81FE>|cmp     esi, 0040AAF4  
0040161F   |.^ 7E C>\\jle     short 004015E1  
00401621   |.  5E   pop      esi  
00401622   |.  5B   pop      ebx  
00401623   |.  5F   pop      edi  
00401624   |.  C3   retn  
00401625   |>  8D4F>lea     ecx, dword ptr [edi+1]  
00401628   |.  51   push    ecx  
00401629   |.  E8 8>call    004015B0  
0040162E   |.  57   push    edi  
0040162F   |.  E8 2>call    00401160  
00401634   |.  83C4>add     esp, 8  
00401637   |>  5F   pop      edi  
00401638   \\.  C3   retn  
```

#### 第五节 第二个关键变量`40AB60`的含义

认真地阅读此段，我认为能控制关键变量`40AB5C`的不同赋值的核心条件如下：

```
004015B0   /$  A1 6>mov     eax, dword ptr [40AB60]  
004015B5   |.  57   push   edi  
004015B6   |.  85C0 test   eax, eax  
004015B8   |.  75 7>jnz     short 00401637  
004015BA   |.  8B7C>mov     edi, dword ptr [esp+8]         ;  EDI=1 初始值  
004015BE   |.  83FF>cmp     edi, 10  
004015C1   |.  7E 0>jle     short 004015CA  
004015C3   |.  E8 6>call    00401030  
004015C8   |.  5F   pop      edi  
```

这里出现了本软件的第二个关键变量`40AB60`。通过反复实验（上提到，我运气地得到可以重复注册成功的一组注册信息），我终于理解了程序的主要注册流程：

**前提**---用户名的 ASCII 值累加和必须是（`1B80-1`）

1.  第二个关键变量`40AB60`为零时，不会绕过 `00401030`，从而第一个关键变量`40AB5C`得到重复地记数，累加到 `1B80`，结果是可以重复注册成功。
    ```
    0040103B   |.  40   inc      eax  
    0040103C   |.  A3 5>mov     dword ptr [40AB5C], eax  
    ```
2.  第二个关键变量`40AB60`为 1 时，就会只执行 `00401000` 段，绕过 `00401030`，从而第一个关键变量`40AB5C`仅赋值为零，结果是仅注册成功一次。
    ```
    00401027   |.  A3 5>mov     dword ptr [40AB5C], eax  
    ```

现在要了解的是，关键变量`40AB60`又是在程序什么地方赋值的呢？常识啊，写入断点：

```
0040112C   |.  68 0>push    0040AA04                ; /1234-5678-1234-5678-1234-5678-1234-5678  
00401131   |.  68 F>push    0040AAF8                ; |42e7-7062-6c58-5275-35b1-9294-6ca4-d1a6  
00401136   |.  FF15>call    dword ptr [<&KERNEL32.lstrcmpA>] ; \\(initial cpu selection)  
0040113C   |.  F7D8 neg     eax  
0040113E   |.  1BC0 sbb     eax, eax  
00401140   |.  5F   pop      edi  
00401141   |.  40   inc      eax  
00401142   |.  5E   pop      esi  
00401143   |.  5D   pop      ebp  
00401144   |.  A3 6>mov     dword ptr [40AB60], eax    赋值处  
00401149   |.  5B   pop      ebx  
0040114A   |.  81C4>add     esp, 0A4  
00401150   \\.  C3   retn  
```

哈哈，又回到了第二个起点。明码比较！然而，正是简单的明码比较，让一些高手轻敌上当，飞蛾扑火。第二个关键变量`40AB60`的取值的具体含义，与一些高手的理解截然相反。



第二个关键变量 `[40AB60]`，根据动态多次调试后，正确的结论居然是：

1. 程序经过计算得出的 1B80 组 KEY，必须都不同于你输入的 KEY，才能重复注册成功。
2. 你输入的 KEY，只要有一次和程序经过计算得出的 1B80 组 KEY 中的任何一组相同，就只能注册成功一次。

我一度也掉入了陷阱近两个小时，错误理解为必须相同才能成功。庆幸地是，我停下了调试，对自己的多次试验进行了认真地思考，注意到第二个关键变量 `[40AB60]` 的取值与第一个关键变量 `[40AB5C]` 赋值为 1B80 功能的先前判断的矛盾。多次的失败，让我对第二个关键变量 `[40AB60]` 的值为 1 就会成功的习惯性思维产生了怀疑。一试之下，恍如隔世。

明白了吗？高手的误区在于对 00401030 程序段的功能理解不完整。它不仅仅具有判断注册成功的功能，同时还具有使第一个关键变量 `[40AB5C]` 赋值为 1B80 的功能。其中隐含了用户名的 ASCII 值累加和必须是（1B80-1）的隐性判断。这个判断条件必须熟悉完整的注册流程和两个关键变量的具体取值含义，才能推断得出。作者并没有如我猜想的那样，对用户名的重复现象进行限制。所以，我说“我运气地识破了作者的伪注册成功。”不管怎样，这个简单的 CRACK ME 堪称伪注册成功的典范。在此，对作者表示我的敬意：

于平淡中见真功夫，  
于简单中巧设陷阱，  
于谈笑间玩弄高手，  
于误解中虚怀若谷。

`crackme_by_topmint`

### 第六节 实践和思考是破解最有威力的武器

More practice, More thinking.

给出两组可混用的：

NAME: `NBA2005QQIIJADKLFAGALKDSJFLKADGADHDJLKADFJGPOEJJQERTLKDFJALJFDKLGAQIJISAJFALJSDIGFJQIEJGIOQJEJIMHA`  
NAME: `NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBA2005NBAQTHA`

key: `1234-5678-1234-5678-1234-5678-1234-5678`  
key: `0000-0000-0000-0000-0000-0000-0000-0000`

通过这个例子，希望增加你对关键变量理解的感性认识。同时，本例子想要说明的是：实践和思考是破解最有威力的武器。实践是为了发现问题，思考是为了更有效率地解决问题。发现问题越多，你掌握的知识就越多、越扎实。那些所谓的有天分的天才，无非就是喜欢问为什么罢了。

这里，我就有一个疑问：我是如何运气地蒙对了一组重复注册成功的 KEY 呢？当时，我只是在利用程序算出符合条件的用户名，随意输入的一组 KEY 居然蒙对了。难道真是巧合？站在 CRACK ME 作者的角度，我更愿理解为是一个巨大的 BUG。现在，我用心地推敲此 CRACK ME 的设计思想，终于明白了问题的所在：

```
0040112C   |.  68 0>push    0040AA04                           
00401131   |.  68 F>push    0040AAF8                             
00401136   |.  FF15>call    dword ptr [<&KERNEL32.lstrcmpA>]    
```

这是此 CRACK ME 作者的得意之处：程序经过计算得出的 1B80 组 KEY，必须都不同于你输入的 KEY，才能重复注册成功。这一标新立异的反常规设计，在玩弄了别的高手的同时，也玩弄了作者自己。这意味着：

破解者有 1B80 次的机会中圈套，  
但是破解者有 1B80 次以外的数不胜数的机会蒙对 KEY。  
两相对比，蒙对 KEY 的几率远远大于中圈套的几率。  
其中，破解者最容易采用的连续字符或相同的字符，使蒙对的几率在实践中相当于 90%（MD5 又是罪魁祸首）。前提是你必须让用户名的 ASCII 值累加和必须是（1B80-1）。

举例：

1234-5678-9012-3456-7890-1234-5678-9012  
ABCD-EFGH-IJKL-MNOP-QRST-UVWX-YZAB-CDEF  
1212-3434-5656-7878-1212-3434-5656-7878  
0000-1111-2222-3333-4444-5555-6666-7777  
......

我就是好习惯地输入了 `1234-5678-1234-5678-1234-5678-1234-5678`，加上开始就运气地理解为“必须让用户名的 ASCII 值累加和必须是（1B80-1）”，蒙对了 KEY。站在 CRACK ME 作者的角度，如果在开始时限制这些有规律的 KEY，无疑会引起破解者的警惕。那么，怎么样巧妙地弥补这一巨大的设计思想上的缺陷呢？用户名的 ASCII 值累加和必须是（1B80-1）的隐性判断条件，是作者精心构思弥补漏洞的神来之笔。这个隐性判断条件，贯穿了此 CRACK ME 的设计思想始终。同时，在下面构思了第二个看似简单的小陷阱：

```
0040177E    >  42   inc     edx
0040177F    .  81FA>cmp     edx, 1B80                       
00401785    .  7E 0>jle     short 00401792
004017F2    .  A1 B0AA4000  mov     eax, dword ptr [40AAB0] ;  用户名的 ASCII 值累加和+1
004017F7    .  8B0D 5CAB4000 mov    ecx, dword ptr [40AB5C] ;  1B80
004017FD    .  3BC1           cmp     eax, ecx
004017FF    .  74 31          je      short 00401832
```

此处与下面遥相呼应，环环相扣，巧妙地对 BUG 进行了一定程度的弥补：

```
0040112C   |.  68 0>push    0040AA04                           
00401131   |.  68 F>push    0040AAF8                             
00401136   |.  FF15>call    dword ptr [<&KERNEL32.lstrcmpA>]    
```

呵呵，真的有高手理解为用户名的 ASCII 值累加和是随意的。让人欲说还休，却道作者好奸猾！

于平淡中见真功夫，  
于简单中巧设陷阱，  
于谈笑间玩弄高手，  
于误解中虚怀若谷。

`crackme_by_topmint`

### 第七节 高手的误区

后记：这两日无意中发现，这个简单的 CRACK ME 居然是看雪论坛上的往年竞赛题目。而作者精心设计的明码比较和 1B80，让许多高手飞蛾扑火。其实，这个 CRACK ME 对于高手来说很简单。如果要怪，就怪作者的玩心太重，专门和高手过不去。毛主席教导我们：在战略上要藐视敌人，在战术上要重视敌人！这是无数先烈用鲜血和生命证明的铁律！！！

我看了看论坛上当时竞赛时的帖子，心中不禁产生了一个疑问：一些高手除了轻敌外，还有没有别的原因呢？我知道，现在大多数的高手都喜欢用 IDA。于是我试图用 IDA 来破解此软件。结果发现，如果不进行思考，单单依赖 IDA 的流程图功能，也不容易掌握程序的注册流程。下面的伪代码，如果不细心看，很容易产生误解：

```
while ( v4 < 36 );
result = lstrcmpA(String1, String) == 0;
dword_40AB60 = result;
return result;
```

我想，上面这段伪代码应该是当时部分高手掉入陷阱的原因之一。粗看之下，很容易误解为必须相等才能通过的常规设计思路。但是注意：后面的关键变量的值是零。个人的理解，IDA 在这个软件的优势不大。反正我后来试试用 IDA 破解时，先前清晰的破解思路反而混乱了。静下心来思考了一下，感觉这个软件的破解关键是两个变量的不同赋值。而动态调试对于变量的理解远远胜于静态调试。那么，我是不是可以这样理解：忽视动态调试的高手很容易上当？

附 `sub_4015B0` 段伪代码：

```c
int __cdecl sub_4015B0(signed int a1)
{
  int result; // eax@1
  signed int v2; // edi@2
  int v3; // ebx@5
  signed int v4; // esi@5
  result = dword_40AB60;
  if ( !dword_40AB60 )
  {
    v2 = a1;
    if ( a1 <= 16 )
    {
      result = dword_40AA6C[a1];
      if ( result )
      {
        sub_4015B0(a1 + 1);
        result = sub_401160(v2);
      }
      else
      {
        v3 = 1;
        v4 = (signed int)&unk_40AAB8;
        do
        {
          if ( !*(_DWORD *)v4 )
          {
            dword_40AA6C[v2] = v3;
            *(_DWORD *)v4 = 1;
            if ( sub_401250(v2) )
              sub_4015B0(v2 + 1);
            result = sub_401160(v2);
          }
          v4 += 4;
          ++v3;
        }
        while ( v4 <= (signed int)&unk_40AAF4 );
      }
    }
    else
    {
      result = sub_401030();
    }
  }
  return result;
}
```

附 `sub_401030` 段伪代码：

```c
int __cdecl sub_401030()
{
  int *v0; // eax@1
  int v1; // ecx@1
  char *v2; // edi@3
  signed int v3; // esi@3
  int v4; // edx@5
  int v5; // ebp@5
  char *v6; // esi@5
  int result; // eax@11
  char v8; // dl@2
  int v9; // esi@7
  void *v10; // edi@7
  int v11; // [sp+4h] [bp-A0h]@1
  int v12; // [sp+8h] [bp-9Ch]@1
  int v13; // [sp+Ch] [bp-98h]@1
  int v14; // [sp+10h] [bp-94h]@1
  char v15; // [sp+14h] [bp-90h]@1
  char v16; // [sp+4Ch] [bp-58h]@3
  _BYTE v17[16]; // [sp+18h] [bp-8Ch]@3
  char Dest; // [sp+28h] [bp-7Ch]@3
  int v19; // [sp+24h] [bp-80h]@5
  char v20; // [sp+48h] [bp-5Ch]@5
  char *v21; // [sp+0h] [bp-A4h]@5
  ++dword_40AB5C;
  v11 = 0;
  v1 = 0;
  v12 = 0;
  v13 = 0;
  v14 = 0;
  v15 = 0;
  v0 = dword_40AA6C;
  do
  {
    v8 = *(_BYTE *)v0;
    ++v0;
    *((_BYTE *)&v11 + v1++) = v8 + 65;
  }
  while ( (signed int)v0 <= (signed int)&dword_40AAAC );
  sub_4019B0(&v16);
  sub_4019E0(&v16, &v11, 17);
  sub_402470(&v16, v17);
  v3 = 0;
  v2 = &Dest;
  do
  {
    _snprintf(v2, 2u, "%02X", v17[v3++]);
    v2 += 2;
  }
  while ( v3 < 16 );
  v4 = 4;
  v5 = 0;
  v6 = (char *)&v19;
  v20 = 0;
  memset(String1, 0, 0x64u);
  v21 = (char *)&v19;
  do
  {
    if ( v4 - 4 < v4 )
    {
      v9 = (int)&v6[v4];
      memcpy(&String1[v5], (const void *)v9, 4u);
      v10 = &String1[v5 + 4];
      v5 += 4;
      memcpy(v10, (const void *)(v9 + 4), 0);
      v6 = v21;
    }
    if ( v4 < 32 )
      String1[v5++] = 45;
    v4 += 4;
  }
  while ( v4 < 36 );
  result = lstrcmpA(String1, String) == 0;
  dword_40AB60 = result;
  return result;
}
```

### 第八节 不识源代码真面目

1B80 的值是如何巧妙地算出的呢？答案就在上述两段伪代码中，其中作者对 MD5 的使用恰如其分。这让我本能地放弃了进一步追踪其原理的兴趣。我凭直觉相信，作者是根据这两段的代码算出了固定值 1B80 后，才将它做为比较的判断条件的：

```
0040177E   >  \42           inc     edx
0040177F   .  81FA 801B0000 cmp     edx, 1B80          ;  1B80 ?
00401785   .  7E 0B        jle     short 00401792
00401787   >  C1FA 02      sar     edx, 2
0040178A   .  81FA 801B0000 cmp     edx, 1B80
00401790   .^ 7F F5        jg      short 00401787
```

最后，要交代一下用户名的 ASCII 累加和不是 1B80 值时，第二个关键变量 `[40AB60]` 的值较复杂，除了 0 和 1 外，有时为 4，有时为 8。由于本文的核心是关键变量和破解的思路，对于注册错误的程序流程没有进一步深入研究，毕竟我不是 CRACK ME 作者。希望该 CRACK ME 作者有兴趣的话，谈谈创作的思路，让我也见识一下 1B80 的隐含判断条件的源代码。

摔碎牙琴焦尾寒，  
子期不在向谁弹？  
春风满面皆朋友，  
欲觅知音难上难。

这个 CRACK ME 的作者，有没有可能是一位绝世美女呢？我的脑中，又冒出了一个疑问？

【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2008 年 05 月 10 日 上午 08:05:27
