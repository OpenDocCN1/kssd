
# 反三国志

> 原文：[`www.pediy.com/kssd/pediy06/pediy6755.htm`](https://www.pediy.com/kssd/pediy06/pediy6755.htm)

反三国志我方无敌加敌方一碰即死代码

妈的安个宽带居然还要等一周才能给安,上不了网只好在家里玩 RPG,这两天玩反三国志,战斗真没意思一场要半天才能结束,一怒之下修改成无敌模式了!~^\_^

先用金山游侠搞定 `HP` 地址,存放在 `728450` 开始的一段内存中(包括所有人物的...)然后利用 `bpm` 找到减血的代码,因为很简单所以不再罗嗦...代码就在下面了...

```
0041FEEF  |. 56               PUSH ESI
0041FEF0  |. 03D1             ADD EDX,ECX //原来的血减去应掉的血
0041FEF2  |. 8990 50847200   MOV DWORD PTR DS:[EAX+728450],EDX //保存 HP
0041FEF8  |. 8BF2             MOV ESI,EDX
```

因为这是个公用减血模块,敌方我方都用这块代码,游戏傻啊!还得咱自己判断,靠!唉...先找个地方放自己代码吧...

我找了一块从 `406891` 开始的,里面全是 `03`,挺怪,不过反正没见哪里用到,将就一下先...^\_\*

加个跳板跳到我的代码:

```
0041FEEF   .^E9 9D69FEFF  JMP SANGUO.00406891 //这里原来是负责处理减血的语句,现在我们自己来处理:)
0041FEF4     90                   NOP
0041FEF5     90                   NOP
0041FEF6     90                   NOP
0041FEF7     90                   NOP
0041FEF8   > 8BF2                 MOV ESI,EDX
```

处理的代码:

```
00406891   > 56                   PUSH ESI
00406892   . 50                   PUSH EAX
00406893   . 52                   PUSH EDX
00406894   . 8B15 F0FCB600  MOV EDX,DWORD PTR DS:[B6FCF0] //B6FCF0 里面存放的是我方人员数目,游戏利用这个数来对我方和敌方排序,
0040689A   . 69D2 E40A0000  IMUL EDX,EDX,0AE4 //我们利用这一点来确保在自己人没有不受伤害
004068A0   . 05 E40A0000        ADD EAX,0AE4 //每隔 AE4 个字节是一个新的人物的 HP...
004068A5   . 3BC2                 CMP EAX,EDX //判断是否是自己人^\_\*
004068A7   . 5A                   POP EDX
004068A8   . 58                   POP EAX
004068A9   . 77 0D                JA SHORT SANGUO.004068B8 //如果大于说明是敌军
004068AB   . 33C9                 XOR ECX,ECX //ECX 是应掉的血
004068AD   . 8990 50847200   MOV DWORD PTR DS:[EAX+728450],EDX //这里是我方 HP 所在,将伤害减为 0!
004068B3   . E9 40960100      JMP SANGUO.0041FEF8 //控制权交还给游戏
004068B8   > 33D2                 XOR EDX,EDX //敌方的 HP 减为 0！！！嘿嘿,够狠吧,我 8 级的时候去砍青龙赚了 N 多经验哦！
004068BA   . 8990 50847200   MOV DWORD PTR DS:[EAX+728450],EDX //存入敌方 HP 所在地
004068C0   . E9 33960100      JMP SANGUO.0041FEF8
004068C5     90                   NOP
004068C6     90                   NOP
004068C7     90                   NOP
004068C8     90                   NOP
```

中间用到的 `B6FCF0` 这个地址也很简单可以得到,因为每次战斗的时候游戏负责分配给每个怪物 `HP` 存放空间,并且这个空间还要照顾到我方人员的数目(从我方最后一名人员开始存放)...

而我们知道每个人物的 `HP` 存放特点(每 `AE4` 字节就是一个了),所以我在第一个怪物应用到的地址下个写内存断点,然后随机遇敌一次,然后就能找到游戏排序存放 `HP` 的地方了...

我提供一下具体地址:

```
0041E82D  |REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI] //WE BREAK HERE!!! LOOP ABOVE!
```

上面的代码:

```
0041E7B0  MOV EAX,DWORD PTR DS:[71F838]
0041E7B5  MOV EDI,DWORD PTR DS:[B6FCF0] //这里存放我方人员数目
0041E7BB  MOV EDX,DWORD PTR SS:[ESP+20]
...
0041E82D  |REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI] //WE BREAK HERE!!! LOOP ABOVE!
0041E82F  |MOV DWORD PTR DS:[EBX+9E0],EAX
```

其它物品之类的统统可以利用游侠搞定我就不废话了...

`4dd466d0e1e0cc123afbe504da2462ca`
