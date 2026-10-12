
# 象棋大战

> 原文：[`www.pediy.com/kssd/pediy03/forum1158.htm`](https://www.pediy.com/kssd/pediy03/forum1158.htm)

下载:`www.newhua.com` 7 月 17 日更新中.

追`flax`无功而返,疲劳无聊郁闷受挫,华军软件`DOWN`一个象棋大战游戏,想大战他几盘,不料居然出现注册窗口,我`CG`子弟可是人人可侮之辈?拿出`trw`、`w32dasm`,出征!

结果是:

```
:0040720E FFD6                  call esi      ;//getdlgitemtexta,username and serail
:00407210 8D7C2440              lea edi, dword ptr [esp+40]
:00407214 83C9FF                or ecx, FFFFFFFF
:00407217 33C0                  xor eax, eax
:00407219 F2                    repnz
:0040721A AE                    scasb
:0040721B F7D1                  not ecx       ;//计算 user 长度
:0040721D 49                    dec ecx
:0040721E 0F84E5010000          je 00407409
:00407224 8D7C2420              lea edi, dword ptr [esp+20]
:00407228 83C9FF                or ecx, FFFFFFFF
:0040722B F2                    repnz
:0040722C AE                    scasb
:0040722D F7D1                  not ecx
:0040722F 49                    dec ecx
:00407230 0F84D3010000          je 00407409
:00407236 8D7C2420              lea edi, dword ptr [esp+20]
:0040723A 83C9FF                or ecx, FFFFFFFF
:0040723D F2                    repnz
:0040723E AE                    scasb
:0040723F F7D1                  not ecx
:00407241 49                    dec ecx       ;//计算 serial 长度,必须等于 9
:00407242 83F909                cmp ecx, 00000009
:00407245 0F85A9010000          jne 004073F4
:0040724B 8A442420              mov al, byte ptr [esp+20]
:0040724F 3C4D                  cmp al, 4D      ;//第一位为?4D,41,4O ->"MAO"之一
:00407251 740C                  je 0040725F
:00407253 3C41                  cmp al, 41
:00407255 7408                  je 0040725F
:00407257 3C4F                  cmp al, 4F
:00407259 0F8595010000          jne 004073F4
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00407251(C), :00407255(C)
:0040725F 8A442421              mov al, byte ptr [esp+21]
:00407263 3C57                  cmp al, 57      ;//第二位为?57,45,4n,4G ->"WENG"之一
:00407265 7410                  je 00407277
:00407267 3C45                  cmp al, 45
:00407269 740C                  je 00407277
:0040726B 3C4E                  cmp al, 4E
:0040726D 7408                  je 00407277
:0040726F 3C47                  cmp al, 47
:00407271 0F857D010000          jne 004073F4
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00407265(C), :00407269(C), :0040726D(C)
:00407277 0FBE442422            movsx eax, byte ptr [esp+22]
:0040727C 99                    cdq
:0040727D B911000000            mov ecx, 00000011     ;//第三位 %0x11==0
:00407282 F7F9                  idiv ecx
:00407284 85D2                  test edx, edx
:00407286 0F8568010000          jne 004073F4
:0040728C 8A442423              mov al, byte ptr [esp+23]
:00407290 3C4A                  cmp al, 4A      ;//第四位为?4A,49,41,4n->"JIAN"之一
:00407292 7410                  je 004072A4
:00407294 3C49                  cmp al, 49
:00407296 740C                  je 004072A4
:00407298 3C41                  cmp al, 41
:0040729A 7408                  je 004072A4
:0040729C 3C4E                  cmp al, 4E
:0040729E 0F8550010000          jne 004073F4
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00407292(C), :00407296(C), :0040729A(C)
:004072A4 8A442424              mov al, byte ptr [esp+24]
:004072A8 3C58                  cmp al, 58
:004072AA 740C                  je 004072B8 ;//第五位为?58,49,55 ->"XIU"之一
:004072AC 3C49                  cmp al, 49
:004072AE 7408                  je 004072B8
:004072B0 3C55                  cmp al, 55
:004072B2 0F853C010000          jne 004073F4
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004072AA(C), :004072AE(C)
:004072B8 0FBE442425            movsx eax, byte ptr [esp+25]
:004072BD 99                    cdq
:004072BE B903000000            mov ecx, 00000003 ;//第六位 %3==0
:004072C3 F7F9                  idiv ecx
:004072C5 85D2                  test edx, edx
:004072C7 0F8527010000          jne 004073F4
:004072CD 8A442426              mov al, byte ptr [esp+26]
:004072D1 3C51                  cmp al, 51      ;//第七位为?51,49, ->"QI"之一
:004072D3 7408                  je 004072DD
:004072D5 3C49                  cmp al, 49
:004072D7 0F8517010000          jne 004073F4
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004072D3(C)
:004072DD 8A442427              mov al, byte ptr [esp+27]
:004072E1 3C59                  cmp al, 59
:004072E3 740C                  je 004072F1      ;//第八位 59,55,4e "YUN"之一
:004072E5 3C55                  cmp al, 55
:004072E7 7408                  je 004072F1
:004072E9 3C4E                  cmp al, 4E
:004072EB 0F8503010000          jne 004073F4
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004072E3(C), :004072E7(C)
:004072F1 0FBE442428            movsx eax, byte ptr [esp+28]
:004072F6 99                    cdq
:004072F7 B90D000000            mov ecx, 0000000D  ;//第 9 位 %0xd==0
:004072FC F7F9                  idiv ecx
:004072FE 85D2                  test edx, edx
:00407300 0F85EE000000          jne 004073F4
:00407306 8D4C2410              lea ecx, dword ptr [esp+10]
:0040730A E8F6960000            call 00410A05
* Possible StringData Ref from Data Obj ->"chess.mjq"
```

嘿嘿,连起来我们看到什么?"`MAO WENG JIAN XIU QI YUN`",回头再看看`readme`毛建琦,翁秀云 是作者和他的女友的名字,好浪漫啊,不过注册程序简单了点,如果再来电加密,岂不更有意思?

一个可用的注册码:`MW3JX0QY4`,注册机,还值得写吗?

程序很糟糕,特别是在高级模式下会向 90 老头一样缓慢思考,喝杯茶回来还没想出来呢,比起`将族 III`来,差远了!不过到也可以看到电脑活活受罪正如自己在`将族`里一样很是开心...

虽然程序注册被破掉,想来作者和他的女友也可以幸福的微笑了.

如果我有`GF`,我拿什么献给她呢?

孤独的`冷雨飘心[BCG]`

`2001.7.19` `4dd466d0e1e0cc123afbe504da2462ca`
