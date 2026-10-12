
# GIF Movie Gear v3.0.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50581.htm`](https://www.pediy.com/kssd/pediy05/pediy50581.htm)

## GIF Movie Gear v3.0.2 破解小谈（附注册机源码）

最近在老罗的坛子里好多人的签名都换成了 `PL` 的图片。嘿嘿，偶自然也不愿落后，so 便想自己 DIY 一个英俊潇洒玉树临风貌比潘安才比伯虎的挂上去，可是以前买的 `Flash MX D` 版盘怎么也找不到了，于是便到天空那边儿去找做 GIF 动画的软件，再于是便找到了这个软件，据说是使用简单，方便快捷…

结果，呵呵（冷汗…搞了一个上午也没搞出张像样的图片来）。偷鸡不成，怎么也要抓一把米回来吧，反正今天除了没完成的事儿其它的事儿都做完了… ;)

首先当然是先要看一下人家有没有加壳，结果是另人满意的，过程是另人愉快的，答案是 `VC6.0` 的。^\_^具然没加壳…

那还等什么呢？点 `Help` 菜单中的 `Register Now`…。接着就呼出你埋伏已久的调试器吧，我用的是 `TRW2000`。（随便问一句哥们儿，`TRW2000` 呼出是按 `Ctrl+N` 这事儿你知道吧，知道啊。那上海 2010 年办世博会，这事儿你也知道吧，哦，都知道了，那甲 A 前天晚上…当！…好好好，我继续就是了）呼出 `TRW2000` 后下断点 `bpx hmemcpy`（我用的是 98 嘛），然后按 `F5` 返回。接着在 `Name` 处偶输入 `Suunb[CCG]`，在 `Code` 处输入 `20030413`(呵呵，今天的日期 ^\_^) 好的，接下来就按 `OK` 吧。

结果是显而易见的，成功被我军爱国者导弹给拦截掉了。之后就一个 `pmodule` 导弹过去跳出程序的领空，之后在按 `F12` 的时候会发现只按 1 次就提示出错了。

好的，知道这些后，就再来一遍吧。这次在 `pmodule` 返回后就按 `F10` 来单步执行，代码如下:

```
0167:00431972  lea      edx,[esp+c4]               <--会来到这里！

0167:00431979  push     byte +64

0167:0043197b  push     edx

0167:0043197c  push     dword 0450

0167:00431981  push     esi

0167:00431982  call     edi                    <--看上边儿压进去的参数，这个 Call 应该是调用 API 函数 GetDlgItem 来得到注册码输入框的句柄。

0167:00431984  push     eax

0167:00431985  call     ebx                    <--用先前得到的注册码输入框的句柄通过调用 GetWindowTextA 来得到输入的注册码

0167:00431987  lea      eax,[esp+c4]           <--用 d 指令可以知道 esp+c4 处装的是用户输入的注册码，将这个地址装入 eax 中

0167:0043198e  lea      ecx,[esp+60]           <--同样用 d 指令可以知道 esp+60 处装的是用户输入的注册名，装其地址装入 ecx 中

0167:00431992  push     eax                  <--注册码地址压栈

0167:00431993  push     ecx                  <--注册名地址压栈

0167:00431994  call     00431590             <--嘿嘿，整个过程就由这个 CALL 一手操控了

0167:00431999  add      esp,byte +08             <--栈顶还原

0167:0043199c  test     eax,eax                <--测试 eax 中装入的值

0167:0043199e  jz       near 00431a51          <--为 0 就跳到 00431a51 处，跳过去就 Over 了

0167:004319a4  lea      edx,[esp+10]           <--从这里开始程序通过调用 RegCreateKeyExA、RegSetvalueExA 等其它一些注册表相关 API 来向注册表中写入用户的注册信息。也就是 HKEY_CURRENT_USER\Software\gamani\GIFMovieGear\2.0 主键下面的 RegName3 与 RegCode3 子键。当然这是在你输入的注册码正确的前提下，RegName3 中装的是你的注册名，RegCode3 中自然就是你输入的册码了…

0167:004319a8  lea      eax,[esp+0c]

0167:004319ac  push     edx

0167:004319ad  push     eax

0167:004319ae  push     byte +00

0167:004319b0  push     dword 000f003f

0167:004319b5  push     byte +00

0167:004319b7  push     dword 0044ed14

0167:004319bc  push     byte +00

0167:004319be  push     dword 0044b3f8         <--此处可以看一下 0044b3f8，得到 Software\gamani\GIFMovieGear\2.0

0167:004319c3  push     dword 80000001         <--这个值就是 HKEY_CURRENT_USER

0167:004319c8  call     `ADVAPI32!RegCreateKeyExA`   <--API 函数 RegCreateKeyExA

0167:004319ce  lea      edi,[esp+60]

0167:004319d2  or       ecx,byte -01

0167:004319d5  xor      eax,eax

0167:004319d7  mov      edx,[esp+0c]

0167:004319db  repne scasb

0167:004319dd  not      ecx

0167:004319df  mov      ebx,[00448018]

0167:004319e5  push     ecx

0167:004319e6  lea      ecx,[esp+64]

0167:004319ea  push     ecx

0167:004319eb  push     byte +01

0167:004319ed  push     eax

0167:004319ee  push     dword 0044d498         <--0044d498 处就是字符串 RegName3

0167:004319f3  push     edx

0167:004319f4  call     ebx                  <--RegSetvalueExA

0167:004319f6  lea      edi,[esp+c4]

0167:004319fd  or       ecx,byte -01

0167:00431a00  xor      eax,eax

0167:00431a02  repne scasb

0167:00431a04  not      ecx

0167:00431a06  lea      eax,[esp+c4]

0167:00431a0d  push     ecx

0167:00431a0e  mov      ecx,[esp+10]

0167:00431a12  push     eax

0167:00431a13  push     byte +01

0167:00431a15  push     byte +00

0167:00431a17  push     dword 0044d4a4         <--0044d4a4 嘛，就是字符串 RegCode3

0167:00431a1c  push     ecx

0167:00431a1d  call     ebx                  <--RegSetvalueExA

0167:00431a1f  mov      edx,[esp+0c]

0167:00431a23  push     edx

0167:00431a24  call     `ADVAPI32!RegCloseKey`

0167:00431a2a  push     dword 0044d4b0

0167:00431a2f  push     dword 80000002

0167:00431a34  call     `ADVAPI32!RegDeleteKeyA`

0167:00431a3a  push     byte +01

0167:00431a3c  push     esi

0167:00431a3d  call     `USER32!EndDialog`        <--用 EndDialog 来关掉输入注册码的这个对话框

0167:00431a43  pop      edi

0167:00431a44  pop      esi

0167:00431a45  xor      eax,eax

0167:00431a47  pop      ebx

0167:00431a48  add      esp,011c

0167:00431a4e  ret      10                   <--返回

0167:00431a51  push     byte +30             <--在前边儿 0043199c 处如果没通过就会跳到这里，到了这里就一切都玩儿完了 ;)

0167:00431a53  push     dword 9d15

0167:00431a58  push     dword 9d14

0167:00431a5d  push     esi

0167:00431a5e  call     0040ee90

0167:00431a63  add      esp,byte +10

0167:00431a66  push     dword 044f

0167:00431a6b  push     esi

0167:00431a6c  call     edi

0167:00431a6e  push     eax

0167:00431a6f  call     `USER32!SetFocus`

0167:00431a75  pop      edi

0167:00431a76  pop      esi

0167:00431a77  xor      eax,eax

0167:00431a79  pop      ebx

0167:00431a7a  add      esp,011c

0167:00431a80  ret      10
```

HeHe~~很明显吧，`00431994` 处的那个 `CALL` 就是关键的所在嘛。

也就是说如果在 `00431994` 处的那个 `CALL` 中程序确定注册码是正确的话就会将其与用户名一同输入进注册表，以后每次程序启动的时候都会进行比较。But 你输入的是错误的话嘛，程序就懒得去写进注册表了。所以我们尽管可以在 `0043199c` 处的时候用 `r fl z` 指令来欺骗程序，但一点儿用都没有 (众人：那你说它干嘛)

那接着就再来一遍，这次就可以直接用断点 `bpx 00431994` 了。断到后就按 `F8` 跟进去吧:

```
0167:00431590  push     ebx

0167:00431591  push     ebp

0167:00431592  mov      ebp,[esp+10]           <--将注册码的地址装入 ebp 中

0167:00431596  push     esi

0167:00431597  push     edi

0167:00431598  cmp      byte [ebp+00],6d       <--ebp+00 也就是注册码的第一位，用注册码的第一位与 6d 相比(6d 即 m 的 ASCII 码)

0167:0043159c  jnz      near 00431642          <--第一位不是 m 就跳到 00431642 处，跳过去就玩儿完了。呵呵，我输入的是 20030413，在这里就挂掉了，再来一遍，这次把前 4 位改成 mg37，即 mg3720030413

0167:004315a2  cmp      byte [ebp+01],67       <--判断注册码的第 2 位是否为 g

0167:004315a6  jnz      near 00431642

0167:004315ac  cmp      byte [ebp+02],33       <--同理，判断第 3 位是否为 3

0167:004315b0  jnz      near 00431642

0167:004315b6  cmp      byte [ebp+03],37       <--第 4 位是否为 4

0167:004315ba  jnz      near 00431642

0167:004315c0  mov      ebx,0044d4c4

0167:004315c5  mov      edx,[ebx]

0167:004315c7  or       ecx,byte -01

0167:004315ca  mov      edi,edx

0167:004315cc  xor      eax,eax

0167:004315ce  repne scasb

0167:004315d0  not      ecx

0167:004315d2  dec      ecx

0167:004315d3  mov      edi,edx

0167:004315d5  mov      esi,ebp

0167:004315d7  xor      eax,eax

0167:004315d9  repe cmpsb

0167:004315db  jz       00431642

0167:004315dd  add      ebx,byte +04

0167:004315e0  cmp      ebx,0044d4c8

0167:004315e6  jl       004315c5

0167:004315e8  cmp      byte [ebp+04],73       <--比较看注册码的第 5 位是否为 s，ebp 中装的其实是注册码的地址，这点很重要，后边儿会有说明

0167:004315ec  jnz      004315ef             <--不是就跳到 004315ef 处

0167:004315ee  inc      ebp                  <--是的话 ebp+1

0167:004315ef  add      ebp,byte +07         <--ebp 加上 7

0167:004315f2  push     ebp                  <--ebp 入栈

0167:004315f3  call     0043f3c8             <--!!

0167:004315f8  mov      edx,[esp+18]

0167:004315fc  add      esp,byte +04

0167:004315ff  mov      edi,edx

0167:00431601  xor      ecx,ecx              <--ecx 清 0

0167:00431603  mov      dl,[edx]             <--装入注册名中的第一个字符

0167:00431605  mov      esi,0bdf             <--esi 中装入 0bdf，即十进制数 3039

0167:0043160a  test     dl,dl                <--看 dl 中是否为 0

0167:0043160c  jz       00431634

0167:0043160e  movsx    edx,dl               <--霸占整个 edx 寄存器 ;)
```



```asm
0167:00431611  inc      ecx                                <-- ecx 加 1
0167:00431612  imul     edx, ecx                           <-- 用 edx 中此时装入的注册名中的某一位字符乘的 ASCII 码乘以 ecx 中的值
0167:00431615  add      esi, edx                           <-- 用 esi 中的值加上 edx 中的值
0167:00431617  cmp      esi, 17be                          <-- 用 esi 中的值与 17be，即十进制数 6078 比较
0167:0043161d  jng      00431625                           <-- 不大于不跳到 00431625 处
0167:0043161f  sub      esi, 17be                          <-- 大于的话就减去它
0167:00431625  cmp      ecx, byte +0a                       <-- 用 A（也就是 10）与 ecx 中的值比较
0167:00431628  jng      0043162c                           <-- 不大于就跳到 0043162c 处
0167:0043162a  xor      ecx, ecx                           <-- ecx 清 0
0167:0043162c  mov      dl, [edi+01]                       <-- dl 中装入注册名中的下一个字符
0167:0043162f  inc      edi                                 <-- edi 加上 1
0167:00431630  test     dl, dl                             <-- 测试 dl
0167:00431632  jnz      0043160e                           <-- 不为 0 就跳加 0043160e 处继续用下一位来进行计算
0167:00431634  cmp      esi, eax                           <-- 比较 esi 与 eax 中的值，esi 中装入的就是前边儿 0043160e-00431632 处将注册名中各位字符进行计算后的值，而 eax 中的值是通过 004315f3 处的这个 CALL 装进来的，相对于我输入的这个注册名及注册码，它的值是 76CD，也就是十进制数 30413。
0167:00431636  jnz      00431642                           <-- 不相等就跳到 00431642 处
0167:00431638  pop      edi
0167:00431639  pop      esi
0167:0043163a  pop      ebp
0167:0043163b  mov      eax, 01                            <-- 相等的话就继续执行到这里，并将 eax 置 1，我们已经知道出来后会比较看 eax 是否为 0，是的话就 Game Over
0167:00431640  pop      ebx
0167:00431641  ret
0167:00431642  pop      edi                                 <-- 从前面跳过来的话就…
0167:00431643  pop      esi
0167:00431644  pop      ebp
0167:00431645  xor      eax, eax                           <-- 可恶吧，会将 eax 置 0
0167:00431647  pop      ebx
0167:00431648  ret
```

我认为我有必要讲一下，其实这并不难理解。在 `004315e8` 处的时候我们可以通过 `d` 指令知道 `ebp` 中装的其实就是注册码的地址，在 `004315e8` 处的时候会比较注册码的第 5 位是否为 `s`，是的话就将 `ebp` 加上 1。而到了 `004315ef` 处的时候会将 `ebp` 加上 7，之后就将这个地址压栈了。过了 `004315f3` 处的那个 `CALL` 后 `EAX` 中的值就会改变了，完全不必要追进去，猜也能猜出来了。呵呵，这个 `CALL` 的作用就是将之前 `004315f2` 处压入的 `ebp` 中的地址处开始的值装入 `eax` 中。而 `ebp` 中装的就是注册码的第 7 位的地址，明白过来了吗？注册码从第 7 位开始的值就是程序真正要用的 ;)可问题是第 7 位以后还有几位呢？嘿嘿，这个很容易就能发现，在 `00431617` 的时候程序不是会判断 `esi` 中的值是否大于 6078 吗？而 6078，也就是 6078 只有 4 位，所以嘛，注册码的位数就是 7+4 即 11 位!不过在 `004315e8` 和我们会发现程序会判断注册码的第 5 位是否为 `s`，是的话就接着将 `ebp` 加上 1，所以问题就解决了。正确的注册码位数只有两种情况:一种是 11 位，另一种是 12 位，这个 12 位的第 5 位是一个固定不变的，即 `s`。

呵呵，我们来看一下正确的注册码的格式吧，第一种 11 位的是 `mg37XXXXXXX`，另一种 12 位的是 `mg37sXXXXXXX`。

现在我们知道的已经够多了，那就写注册机吧 ;)

这之前再大概说一遍程序注册码的计算过过程:

首先固定不变的是程序注册码的前 4 位，永远是 `mg37`，然后有一个可有可无的位，即第 5 位，要这一位的话，就让它为 `s` (注意如果注册码的第 5 位是 `s`，那就应该在其后再补上 3 位)，否则就不要它而直接在 `mg37` 后边儿随便补充 3 位，就比如说 `CCG ^_^`。而接下来就是最后 4 位了，这个也是重要的地方所在了 ;) 后 4 位的计算过程其实也不难，就是用你输入的注册名的首位字符的 ASCII 码乘以 `X`(`X` 每计算一位字符就加上 1，如果值后来一直累加到 11，就会还原为 0)。再用这个乘积加上 3039。并将其保存起来，这里称它为 `Y`，之后比较 `Y` 是否大于 6078，如果大于就用 `Y` 减去 6078。接下来第二位字符参加运算，同样用其值乘以 `X`，然后加上 `Y`，再比较 `Y` 是否大于 6078，再接下来第三位字符参加运算，用其乘以 `X`，然后加上 `Y`…直到所有字符均参加完运算…

BTW:如果字符参加完运算后得到的是一个 3 位有效值，比如 123，就在 1 的前边儿补上一个 0，如 0123。

贴的这个是注册机源码是 Delphi 的，直接声明这个函数就能用了:

```delphi
function KeyGen(Name: String): String;

var
  i, Cnt, vai, ASC: integer;
  Serial: String;

begin
  Cnt := 3039;
  vai := 0;
  for i := 1 to Length(Name) do
  begin
    inc(vai);
    ASC := Ord(Name[i]);
    ASC := ASC * vai;
    Cnt := Cnt + ASC;
    if Cnt > 6078 then Cnt := Cnt - 6078;
    if vai > 10 then vai := 0;
  end;
  Serial := inttostr(Cnt);
  Case Length(Serial) of
    3: Serial := '0' + Serial;
    2: Serial := '00' + Serial;
    1: Serial := '000' + Serial;
  end;
  Serial := 'mg37CCG' + Serial;
  Result := Serial;
end;
```

OK，就这些。

`4dd466d0e1e0cc123afbe504da2462ca`
