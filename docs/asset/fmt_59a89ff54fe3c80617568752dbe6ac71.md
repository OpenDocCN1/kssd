
# BuddyPhone

> 原文：[`www.pediy.com/kssd/pediy02/forum132.htm`](https://www.pediy.com/kssd/pediy02/forum132.htm)

软件名称：`BuddyPhone`

保护：注册码

下载：`http://ilike.myrice.com/soft_chat/buddyphone_1.html`

软件简介：

一款小巧玲珑，性能优异，操作简单的网络电话。其优异而平滑的通话质量，对硬件的低要求，清晰明白的界面，与`ICQ`天衣无缝的链接，简单顺手的操作。

这软件要移去广告条必需要输入注册码，我试过破解，己找到注册码比较处，但其算法我看不懂，请高手指点为感。

地址：`4135DB` 附近的 `CALL` 是比较核心处，在这代码下面还有一处比较。

不过注册机还是很好写的。

它先检查注册码中是否有横杠字符，即注册码是分成两部分的。

```
015F:004135C7  PUSH       2D              //横杠字符
015F:004135C9  LEA        ECX,[ESP+18]
015F:004135CD  MOV        [ESP+30],EDI
015F:004135D1  CALL       00440EB8        //检查有无横杠
015F:004135D6  MOV        EDI,EAX
015F:004135D8  CMP        EDI,-01
015F:004135DB  JZ         004137A1        //jump if bad guy
```

然后就根据`FirstName`、`LastName`、`ProductID`和注册码的前半部分计算出注册码的后半部分。

```
015F:0041368A  MOV        EDX,[ESP+24]
015F:0041368E  MOV        ECX,[ESP+20]
015F:00413692  MOV        AL,[EDX]        //FirstName 的首字符（大写）
015F:00413694  MOV        DL,[ECX]        //Lastname 的首字符（大写）
015F:00413696  MOV        [ESP+13],AL
015F:0041369A  MOV        EAX,[ESP+1C]
015F:0041369E  PUSH       EAX
015F:0041369F  MOV        [ESP+16],DL
015F:004136A3  CALL       0042B330                    //将注册码前半部分转换成整数
015F:004136A8  MOVSX      ECX,BYTE PTR [ESP+16]        //LastName 的首字符(大写）
015F:004136AD  MOVSX      EDX,BYTE PTR [ESP+17]        //FirstName 的首字符（大写）
015F:004136B2  SHL        ECX,08
015F:004136B5  ADD        ECX,EDX
015F:004136B7  MOV        EDX,[0046B250]        //取出 ProcuctID
015F:004136BD  ADD        EAX,EDX              //和注册码前半部分相加
015F:004136BF  LEA        EDI,[EAX+ECX+FFFF7BD8] //edi 中放的就是注册码的后半部分的正确值
015F:004136C6  MOV        EAX,[ESP+1C]
015F:004136CA  PUSH       EAX
015F:004136CB  CALL       0042B330            //将你输入的注册码的后半部分转换成整数
015F:004136D0  ADD        ESP,08
015F:004136D3  CMP        EDI,EAX             //比较后半部分
015F:004136D5  MOV        EDI,00000001        //good guy
015F:004136DA  JZ         004136E0            //jump if bad guy
015F:004136DC  MOV        EDI,[ESP+2C]        //bad guy
015F:004136E0  LEA        ECX,[ESP+18]
015F:004136E4  MOV        BYTE PTR [ESP+38],03
015F:004136E9  CALL       00440A3D
015F:004136EE  LEA        ECX,[ESP+1C]
015F:004136F2  MOV        BYTE PTR [ESP+38],02
015F:004136F7  CALL       00440A3D
015F:004136FC  TEST       EDI,EDI
015F:004136FE  JZ         004137A1            //jump if bad guy
4dd466d0e1e0cc123afbe504da2462ca
```
