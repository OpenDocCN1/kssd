
```asm
00401FF4    .  8B4424 18          mov eax,dword ptr ss:[esp+18]                             ；EAX 00D24908 ASCII "4072869764"

00401FF8    .  8378 F4 15         cmp dword ptr ds:[eax-C],15

；[eax-c]=0000000A，这是"4072869764"的长度，长度不等于21就跳，但是注册码明明是10

；

00401FFC    .  B3 03              mov bl,3                                                  ；

00401FFE    .  885C24 2C          mov byte ptr ss:[esp+2C],bl

00402002    .  0F85 EC000000      jnz RepairRe.004020F4                                     ；跳了

00402008    .  8B5424 10          mov edx,dword ptr ss:[esp+10]

0040200C    .  8B4A F4            mov ecx,dword ptr ds:[edx-C]

0040200F    .  85C9               test ecx,ecx

00402011    .  0F8E DD000000      jle RepairRe.004020F4
```

跳到这：

```asm
004020F4    > \68 50794400        push RepairRe.00447950

;  ASCII "RepairRegistry Pro is running under Trial version.To get the complete functionality of this application,

          buy or register

it by clicking on the Buy/Register Button"

004020F9    .  8D4C24 18          lea ecx,dword ptr ss:[esp+18]

004020FD    .  E8 DEFCFFFF        call RepairRe.00401DE0

00402102    .  C64424 2C 07       mov byte ptr ss:[esp+2C],7

00402107    >  8B06               mov eax,dword ptr ds:[esi]

00402109    .  8D4C24 14          lea ecx,dword ptr ss:[esp+14]
```

进去程序后可以发现，About画面显示有

```text
RepairRegistry Pro is running under Trial version.To get the complete functionality of this application,
          buy or register
it by clicking on the Buy/Register Button
```

但是"Buy/Register Button"却不存在了，程序的功能也可以全部使用。

`cmp dword ptr ds:[eax-C],15` 改为 `cmp dword ptr ds:[eax-C],0A`

通过`00401FF8` 的比较以后，

```asm
00402027    .  83C0 10            add eax,10

0040202A    .  8907               mov dword ptr ds:[edi],eax

0040202C    .  8B4424 18          mov eax,dword ptr ss:[esp+18]

00402030    .  83C0 F0            add eax,-10

00402033    .  896424 28          mov dword ptr ss:[esp+28],esp

00402037    .  8BFC               mov edi,esp

00402039    .  50                 push eax

0040203A    .  C64424 38 04       mov byte ptr ss:[esp+38],4

0040203F    .  E8 CCF7FFFF        call RepairRe.00401810

00402044    .  83C0 10            add eax,10

00402047    .  83C4 04            add esp,4

0040204A    .  8907               mov dword ptr ds:[edi],eax                             ；EAX 00D24940 ASCII "CNwinndy"

0040204C    .  885C24 34          mov byte ptr ss:[esp+34],bl                            ；ESP+34=0012ADF0  bl=03

00402050    .  E8 BB380100        call RepairRe.00415910                                 ；跟进去，见后面[重启分析一]

00402055    .  83C4 08            add esp,8

00402058    .  84C0               test al,al                                              ；这里跳了，去清除注册信息

0040205A    .  74 58              je short RepairRe.004020B4                              ；这里不跳，则正确注册

0040205C    .  E8 826E0300        call RepairRe.00438EE3

00402061    .  8B10               mov edx,dword ptr ds:[eax]

00402063    .  8BC8               mov ecx,eax

00402065    .  FF52 0C            call dword ptr ds:[edx+C]

00402068    .  83C0 10            add eax,10

0040206B    .  894424 14          mov dword ptr ss:[esp+14],eax

0040206F    .  8B4424 10          mov eax,dword ptr ss:[esp+10]

00402073    .  50                 push eax

00402074    .  8D4C24 18          lea ecx,dword ptr ss:[esp+18]

00402078    .  68 287A4400        push RepairRe.00447A28                                    ;  ASCII "This version of Repair Registry is Licenced to %s."

0040207D    .  51                 push ecx

0040207E    .  C64424 38 05       mov byte ptr ss:[esp+38],5

00402083    .  E8 F8FCFFFF        call RepairRe.00401D80

00402088    .  8B16               mov edx,dword ptr ds:[esi]

0040208A    .  83C4 0C            add esp,0C

0040208D    .  8D4424 14          lea eax,dword ptr ss:[esp+14]

00402091    .  50                 push eax

00402092    .  8BCE               mov ecx,esi

00402094    .  FF92 40010000      call dword ptr ds:[edx+140]

0040209A    .  8B4424 14          mov eax,dword ptr ss:[esp+14]

0040209E    .  83C0 F0            add eax,-10

004020A1    .  885C24 2C          mov byte ptr ss:[esp+2C],bl

004020A5    .  8D48 0C            lea ecx,dword ptr ss:[eax+C]

004020A8    .  83CA FF            or edx,FFFFFFFF

004020AB    .  F0:0FC111          lock xadd dword ptr ds:[ecx],edx

004020AF    .  4A                 dec edx

004020B0    .  85D2               test edx,edx

004020B2    .  EB 7A              jmp short RepairRe.0040212E

004020B4    > \68 8E784400        push RepairRe.0044788E

004020B9    .  68 E4784400        push RepairRe.004478E4                                    ;  ASCII "regcode"

004020BE    .  68 EC784400        push RepairRe.004478EC                                    ;  ASCII "Repair Registry Pro"

004020C3    .  E8 B83C0100        call RepairRe.00415D80                                    ；这一句执行后，发现注册表的regcode被清除

004020C8    .  68 8E784400        push RepairRe.0044788E

004020CD    .  68 00794400        push RepairRe.00447900                                    ;  ASCII "username"

004020D2    .  68 EC784400        push RepairRe.004478EC                                    ;  ASCII "Repair Registry Pro"

004020D7    .  E8 A43C0100        call RepairRe.00415D80                                    ； ；这一句执行后，发现注册表的username被清除

004020DC    .  83C4 18            add esp,18

004020DF    .  68 F4794400        push RepairRe.004479F4                                    ;  ASCII "RepairRegistry Pro is running under Trial version."

004020E4    .  8D4C24 18          lea ecx,dword ptr ss:[esp+18]

004020E8    .  E8 F3FCFFFF        call RepairRe.00401DE0

004020ED    .  C64424 2C 06       mov byte ptr ss:[esp+2C],6

004020F2    .  EB 13              jmp short RepairRe.00402107
```

注册信息被清除后，进入后，就变成未注册的了

### 重启校验小结

输入注册码后，注册码长度为10，但是在重启校验的时候却要长度为21，我们清楚地看到注册码只与MAC地址有关，但是重启校验的时候还要计算用户名和注册码的ascii码和，然后还进行一段运算，得出三个值，最后再与用户名的第一个，第二个和最后一个字符比较，相符则校验成功。

疑是程序BUG，失误之处还往指出。

关于重启的分析，见附件的txt，加到这里，文件太大，篇幅太长了。加上疑是BUG，有兴趣的读者可以看附件。

---

## 算法总结

注册码与用户名(但用户名不能为空)无关，由网卡物理地址字符串(长12)，再连接常数串`"1ou6#uk&akietoe"`(长15)，组成27位的字符串，然后设一个`regcode`初值`FFFFFFFF`，更觉字符串来查密码表，取相应的双字来和`FFFFFFFF`运算，得到最终的`regcode`后，再转化为无符号长整型，即为注册码。

用VB6写了注册机，04.15写了框架，今天调试完毕。

## Greetings

看雪论坛,FCG论坛,DFCG论坛等

注册机源码中的MAC地址以及`Flat Button`控件是从网上下载而来，稍加修改，感谢原作者！

## 完稿时间等

2005.04.28，凌晨1:31,天气:小雨,广州

2005.04.28，11:58，重启校验分析

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
