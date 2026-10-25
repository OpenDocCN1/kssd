
![](https://bbs.kanxue.com/upload/attach/201402/603227_kw861p87owgzp48.png)

这是我当前寄存器组的情况。

![](https://bbs.kanxue.com/upload/attach/201402/603227_6ek1ylv307c5a3c.png)

按下F7键,看看现在堆栈的情况:

![](https://bbs.kanxue.com/upload/attach/201402/603227_avpa2fucjnbxyqp.png)

看到所有寄存器的值都被压入堆栈了。12FFC4存放的是之前堆栈顶部的值,然后上面就是0(`PUSH EAX`),12FFBC存放的是`ECX`的内容,接下来是`EDX`寄存器的内容,以此类推。

# POPAD

该指令与`PUSHAD`正好相反,它从堆栈中取值,并将它们放到相应的寄存器中。`POPAD`等价于“`POP EDI,POP ESI,POP ESP,POP ESP,POP EBX,POP EDX,POP ECX,POP EAX`”。

![](https://bbs.kanxue.com/upload/attach/201402/603227_kkwik8qsif0rou5.png)

正如前面的例子,我们将第一条指令修改为`POPAD`:

![](https://bbs.kanxue.com/upload/attach/201402/603227_h2dj5sxhp632fi5.png)

由于堆栈中保存的是之前寄存器组的值,执行`POPAD`其返回原来的状态。

![](https://bbs.kanxue.com/upload/attach/201402/603227_wvjkepu9bzj5mef.png)

因为之前执行了`PUSHAD`,所以恢复的寄存器及其值和之前相同。

![](https://bbs.kanxue.com/upload/attach/201402/603227_wvjkepu9bzj5mef.png)

`PUSHAD-POPAD`指令经常被使用,例如:某个时刻你需要保存所有寄存器的内容,然后修改寄存器的值,或者进行堆栈的相关操作，然后使用`POPAD`恢复它们原来的状态。

也有以下的用法:

`PUSHA` 等价于 `'PUSH AX, CX, DX, BX, SP, BP, SI, DI'`。

`POPA` 等价于 `'pop DI, SI, BP, SP, BX, DX, CX, AX'`。

`PUSHA`,`POPA`和`PUSHAD`,`POPAD`就像姐妹一样,只不过它们在16位程序中使用,所以我们不感兴趣,OllyDbg是一个32位程序的调试器。

# 赋值指令的说明

## `MOV`

该指令将第二个操作数赋值给第一个操作数,例如:

`MOV EAX, EBX`

`EBX`值赋值给`EAX`。继续用OD载入CrueHead'а的CrackMe。

![](https://bbs.kanxue.com/upload/attach/201402/603227_yexkx29a4z6pqeo.png)

两个寄存器的值是不一样的,我们只是看寄存器:

![](https://bbs.kanxue.com/upload/attach/201402/603227_bz0bgwsj1wyeyvr.png)

在我的机器上,`EAX`为0,`EBX`为7FFD7000。这些初始值有可能与你机器上的不一样,我们只是看赋值的过程,按F7键,`EBX`的值就赋值给了`EAX`。

![](https://bbs.kanxue.com/upload/attach/201402/603227_app51pmyzban3tu.png)

明白了吗?

`MOV`指令操作数有很多可以选择,例如:

`MOV AL, CL`

这条指令时将`CL`的值赋值给`AL`。在OD中写上这句。

![](https://bbs.kanxue.com/upload/attach/201402/603227_esmwoqovjrk457t.png)

寄存器:

![](https://bbs.kanxue.com/upload/attach/201402/603227_sz528kiud4e50z1.png)

请记住，`AL`-是`EAX`的最后两位数字以及`CL`-是`ECX`的最后两位数字。按F7

![](https://bbs.kanxue.com/upload/attach/201402/603227_bs0yfy66zwxa7ry.png)

可以看到只是将B0赋值给`AL`了,并不改变`EAX`和`ECX`的其他内容,即`EAX`的最后两位数字。

也可以给内存单元赋值,反之亦然一个寄存器的内容。

![](https://bbs.kanxue.com/upload/attach/201402/603227_hee7x67h4768bt9.png)

上图这种情况下,将要给`EAX`给的值是405000这个内存单元中的值,正如前面提到的,`DWORD`意味着你必须移动4个字节。如果该指令给出了一个不存在的内存单元可能会导致错误。我们可以用OD很容易的检查出来。

在数据窗口中右键单击选择Go to-Expression转到405000处.

![](https://bbs.kanxue.com/upload/attach/201402/603227_f8254stxa02uk7u.png)

![](https://bbs.kanxue.com/upload/attach/201402/603227_8m3goc2945u15vc.png)

可以看到内存中的内容是-00100000。由于内存中内容是倒序存放的,那么`EAX`中将被装入00001000,按下F7键,看看会发生什么。

![](https://bbs.kanxue.com/upload/attach/201402/603227_hqxzyuif9i0kavq.png)

这里是1000，是从内存中读取出来的。现在,如果我们想写一个值到这个地址:

`MOV DWORD PTR DS:[400500],EAX`

写入该指令。

![](https://bbs.kanxue.com/upload/attach/201402/603227_ivp2tb1qwnj8bkj.png)

在数据窗口中查看405000:

![](https://bbs.kanxue.com/upload/attach/201402/603227_zz38pcec70rpgry.png)

然后,按F7键发生异常:

![](https://bbs.kanxue.com/upload/attach/201402/603227_8yxdto8umc7qy6p.png)

注意异常的描述,是由于我们要写入400500这个内存地址导致的内存访问异常。

如果移动的是4个字节使用`DWORD`,移动两个字节的话使用`WORD`。

例如:

`MOV AX,WORD PTR DS:[405008]`

将405008内存单元中的两个字节赋值给`AX`。这种情况下,我们不能用`EAX`,这里移动的只有两个字节,所以你必须使用一个16位的寄存器。

在数据窗口中查看405008.

![](https://bbs.kanxue.com/upload/attach/201402/603227_09ie0cl3mi88mvl.png)

按F7赋值给`AX`的只有两个字节。如下:

![](https://bbs.kanxue.com/upload/attach/201402/603227_jyt852ahfmf6n9t.png)

## `MOVSX` (带符号扩展的传送指令)

第二个操作数可能一个寄存器也可能是内存单元,第一个操作数的位数比第二个操作数多,第二个操作数的符号位填充第一个操作数剩余部分。 下面是一个例子。

这里我们还是在OD中载入CrueHead'а的CrackMe。

![](https://bbs.kanxue.com/upload/attach/201402/603227_eurauwx7yg7rf16.png)

这里我不会说太多，因为我想在座各位自己来计算这个操作数求值,嘿嘿。

在OD中，反汇编窗口和数据窗口中间有一个解释窗口。

![](https://bbs.kanxue.com/upload/attach/201402/603227_rpux7h0pyzd8bqu.png)

这里我们可以看到,解释窗口向我们展示了我们操作数里面存放的值。我这里,`BX`存放的是F000。

![](https://bbs.kanxue.com/upload/attach/201402/603227_33jego2x1q02k1o.png)

同时可以看到,`EAX`的值为0,所以这些东西总是可以帮助我们理解OD要执行的指令(我希望你已经掌握了其中的概念和寻址方式,嘿嘿)。

按F7键。

![](https://bbs.kanxue.com/upload/attach/201402/603227_gswrg9ikvv0m4nr.png)

看到`AX`,`BX`中存放的是F000，并且`EAX`剩余部分填充为了FFFF,因为F000是一个负的16位数字。如果`BX`存放1234,`EAX`将等于00001234，即左边的字节将会被0填充,因为1234是一个正的16位数字。

16位数和32位数的正数和负数的概念是一样的,只不过16位数的范围是0000到FFFF。0000到7FFF是正数,8000到FFFF是负数。

如果我们把`BX`修改为7FFF,把`EAX`修改为0，然后执行这条指令会发生什么呢。

![](https://bbs.kanxue.com/upload/attach/201402/603227_zgvuhkk8jdci62n.png)

`AX`被赋值为了7FFF,其余部分被填充为0了-因为7FFF是正数。我们还可以把`BX`修改为8000(负)。

![](https://bbs.kanxue.com/upload/attach/201402/603227_cvljopht1owwkbq.png)

按F7键。

![](https://bbs.kanxue.com/upload/attach/201402/603227_xavdgpxpaarecc3.png)

`AX`被赋值为了8000,剩余的部分为FFFF,因为8000是负数。

## `MOVZX` (带0扩展的传送指令)

`MOVZX`类似于前面的语句,但是这种情况下,剩余的部分不根据第二个操作数的正负来进行填充。我们这里不提供范例,因为和上面是相似的,剩余的部分总是被填充为0。

## `LEA` (取地址指令)

类似于`MOV`指令, 但是第一个操作数是一个通用寄存器,并且第二个操作数是一个内存单元。当计算的时候要依赖于之前的结果的话,那么这个指令就非常有用。

我们在OD中写入以下指令:

![](https://bbs.kanxue.com/upload/attach/201402/603227_mf4avrtxhr84f9l.png)

在这种情况下,有括号,但不需要获取`ECX+38`指向内存的值,只需要计算`ECX+38`的值即可。我这里,`ECX`的值为

12FFB0。

![](https://bbs.kanxue.com/upload/attach/201402/603227_9hd9y3wks9iwjyt.png)

在这个例子中,`LEA`指令就计算`ECX + 38`的值,然后将计算的结果赋值给`EAX`。

解释窗口中显示了两个操作数。

![](https://bbs.kanxue.com/upload/attach/201402/603227_if350wokgjldkkq.png)

它表示,该操作数是12FFE8,也就是`ECX+38`的值,并且`EAX`的值为0。

按F7键。

![](https://bbs.kanxue.com/upload/attach/201402/603227_laox4cmw84mztwa.png)

指定的地址被存放到了`EAX`中，因为完成的是赋值操作,所以我们会认为操作数是内存单元中的值，但是实际上

操作数仅仅是内存单元的地址,而不是里面的内容。

## `XCHG` (交换 寄存器/内存单元 和 寄存器)

该指令交换两个操作数的值,例如:

`XCHG EAX,ECX`

`EAX`的值将被存放到`ECX`中。反之亦然。我们在OD中来验证这一点。

![](https://bbs.kanxue.com/upload/attach/201402/603227_emh37gpbdywx6jt.png)

在我的机器上,`EAX`的值为0,`ECX`的值为12FFB0。

![](https://bbs.kanxue.com/upload/attach/201402/603227_ofv8nzebpzx2tfp.png)

按F7键,看到他们交换了数值。

![](https://bbs.kanxue.com/upload/attach/201402/603227_8ulu7xxfzou1mlb.png)

你也可以使用这个指令来交换寄存器和内存单元的值,本节之前有提到。

![](https://bbs.kanxue.com/upload/attach/201402/603227_cq47832gkvai8oo.png)

按下F7键:

![](https://bbs.kanxue.com/upload/attach/201402/603227_kdx2atuuggz0rxp.png)

这个例子我们在`MOV`指令使用过,这里我们对该内存地址没有写权限。

好了,这就是常用指令的第一部分,是非常有用的以及有趣的,我给出的例子能够体现这一点。在接下来的部分,我们将继续研究指令。

![](https://bbs.kanxue.com/view/img/face/50.gif)这么多图片,传的我手的麻了,神马时候坛子的编辑功能才能改变呀![](https://bbs.kanxue.com/view/img/face/16.gif),忘了吃饭,饿死我了,吃饭去了,闪人...

下面是DOC版以及本章的附件。

[使用OllyDbg从零开始Cracking 第四章.doc](https://bbs.kanxue.com/attach-download-87108.htm)

[ollydbg01-Crackme.7z](https://bbs.kanxue.com/attach-download-87109.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [图片1.png](javascript:void(0);)（94.85kb，38次下载）
- [图片2.png](javascript:void(0);)（6.91kb，11次下载）
- [图片3.png](javascript:void(0);)（6.19kb，7次下载）
- [图片4.png](javascript:void(0);)（3.17kb，6次下载）
- [图片5.png](javascript:void(0);)（63.49kb，11次下载）
- [图片6.png](javascript:void(0);)（6.33kb，6次下载）
- [图片7.png](javascript:void(0);)（4.82kb，4次下载）
- [图片8.png](javascript:void(0);)（4.54kb，5次下载）
- [图片9.png](javascript:void(0);)（7.08kb，4次下载）
- [图片10.png](javascript:void(0);)（4.96kb，7次下载）
- [图片11.png](javascript:void(0);)（2.67kb，5次下载）
- [图片12.png](javascript:void(0);)（4.79kb，7次下载）
- [图片13.png](javascript:void(0);)（8.29kb，5次下载）
- [图片14.png](javascript:void(0);)（6.55kb，4次下载）
- [图片15.png](javascript:void(0);)（5.38kb，4次下载）
- [图片16.png](javascript:void(0);)（2.95kb，6次下载）
- [图片17.png](javascript:void(0);)（7.03kb，4次下载）
- [图片18.png](javascript:void(0);)（4.90kb，4次下载）
- [图片19.png](javascript:void(0);)（6.11kb，4次下载）
- [图片20.png](javascript:void(0);)（5.80kb，8次下载）
- [图片21.png](javascript:void(0);)（3.09kb，4次下载）
- [图片22.png](javascript:void(0);)（6.33kb，5次下载）
- [图片23.png](javascript:void(0);)（6.71kb，4次下载）
- [图片24.png](javascript:void(0);)（3.05kb，4次下载）
- [图片25.png](javascript:void(0);)（4.27kb，4次下载）
- [图片26.png](javascript:void(0);)（4.67kb，5次下载）
- [图片27.png](javascript:void(0);)（4.21kb，4次下载）
- [图片28.png](javascript:void(0);)（5.80kb，4次下载）
- [图片29.png](javascript:void(0);)（6.25kb，4次下载）
- [图片30.png](javascript:void(0);)（4.30kb，4次下载）
- [图片31.png](javascript:void(0);)（3.00kb，4次下载）
- [图片32.png](javascript:void(0);)（9.67kb，6次下载）
- [图片33.png](javascript:void(0);)（10.06kb，4次下载）
- [图片34.png](javascript:void(0);)（5.05kb，4次下载）
- [图片35.png](javascript:void(0);)（3.21kb，4次下载）
- [图片36.png](javascript:void(0);)
