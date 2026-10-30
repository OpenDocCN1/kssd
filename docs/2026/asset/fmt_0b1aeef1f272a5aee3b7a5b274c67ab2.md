
# [原创]试着破解孔雀石SDR的序列号保护

网上看到俄国这个孔雀石SDR收音机不错，但是有序列号保护，于是想试着看看能不能去掉，由于这个比较贵要上百元，我这样没收入的人根本买不起，所以不保证破解成功。

孔雀石SDR收音机作者提供的资料

[783K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6j5h3c8A6i4K6u0W2M7$3E0Q4x3V1k6V1i4K6u0r3y4q4A6Y4M7%4u0K6N6%4S2k6b7$3I4s2x3g2p5`.](https://bbs.kanxue.com/elink@e7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6j5h3c8A6i4K6u0W2M7$3E0Q4x3V1k6V1i4K6u0r3y4q4A6Y4M7%4u0K6N6%4S2k6b7$3I4s2x3g2p5`.)

作者直接提供了PCB和固件。

去掉这种序列号保护在专业人眼里非常简单，别人要么有硬件要么软件仿真一下就能找到，而我技术比较菜这次只通过看IDA反汇编的代码找。

由于没硬件网上找资料也没找到演示如何输入注册号码的，只找到一个烧写固件的视频，在视频结尾看到了序列号页面。

![](https://bbs.kanxue.com/upload/attach/202101/621165_QAPSGJRCV7FQHQC.jpg)

可以看到第2行是序列号，第四行估计应该就是输入的密码

这个SDR收音机是`STM32H7`的单片机做主控，于是下载参考手册，这种序列号保护一般是基于`STM32`上`96`位ID的。

打开手册找寄存器地址

![](https://bbs.kanxue.com/upload/attach/202101/621165_AGBGC4YFBEQSGZ3.jpg)

![](https://bbs.kanxue.com/upload/attach/202101/621165_65SJUQ8QN7E2VB6.jpg)

打开用`IDA` 打开下载到的固件，方法网上和论坛上都有这里就不重复了。

[`FW_1_10a.hex`](https://bbs.pediy.com/upload/attach/202101/621165_3U76KHST2WXK4V4._hex)

搜索`0x1FF1E800`结果找不到。

于是搜索LCD屏幕上的英文`THE`，结果找到了

![](https://bbs.kanxue.com/upload/attach/202101/621165_XSXATZMQWQBW8SK.jpg)

可以知道这个就是序列号显示函数，于是找那里调用了这个函数。

![](https://bbs.kanxue.com/upload/attach/202101/621165_AVY5K7VHS4QY6HX.jpg)

可以看到这个函数调用了ID显示函数

![](https://bbs.kanxue.com/upload/attach/202101/621165_9USVYCVGK4STXXR.jpg)

在这里可以看到`0x1ff1e8xx`,这明显就是读ID的函数，但是这个地址是`0x1FF1E80B`和 `0x1FF1E7FF`，难怪刚才搜索不到，可能作者故意的，地址`8019B12`的循环完成了读`96`位ID并且保存到`0X2001AFC8`

```
19B12 loc_8019B12     

    ADD.W           R2, R3, R8    

        R2等于 R3+0XE00E1801，第一次相加超过32位正好等于0，第二次结果就是1了，结果随着R3自增

    LDRB.W          R1, [R3,#1]!

    读取0X1FF1E800地址里面的8位数据到R1然后地址自增8位

    CMP             R3, R4

和R3和0X1FF1E80B比较看读没读完

    STRB.W          R1, [R9,R2]

    把8位写入0X2001AFC8

    BNE             loc_8019B12

    循环
```

再往下找看`sub_800D748`，可以知道ID号码经过了多次运算

```
seg000:0800D748 ; ID

seg000:0800D748 ; r0=ID地址0x2001AFC8

seg000:0800D748

seg000:0800D748 sub_800D748                             ; CODE XREF: sub_8019AF8+2E↓p

seg000:0800D748                 PUSH            {R4-R6}

seg000:0800D74A                 MOVS            R2, #0  ; R2=0

seg000:0800D74C                 SUBS            R3, R0, #1 ; R3=R0-1

seg000:0800D74E                 ADD.W           R5, R0, #0xB ; R5=R0+0X0B

seg000:0800D752

seg000:0800D752 loc_800D752                             ; CODE XREF: sub_800D748+12↓j

seg000:0800D752                 LDRSB.W         R4, [R3,#1]! ; 读8位ID

seg000:0800D756                 CMP             R5, R3

seg000:0800D758                 ADD             R2, R4  ; ID累加

seg000:0800D75A                 BNE             loc_800D752 ; 读8位ID

seg000:0800D75C                 LDRSB.W         R3, [R0] ; 读ID 最低8位

seg000:0800D760                 UBFX.W          R2, R2, #0, #0xA ; 提取10位

seg000:0800D764                 LDRSB.W         R4, [R0,#0xB] ; 读ID最高8位

seg000:0800D768                 SUBS            R0, R1, #1 ; R0=0X2001AF54-1

seg000:0800D76A                 LDR             R6, =unk_2001AE88

seg000:0800D76C                 ADDS            R1, #7  ; R1=0X200001AF54 + 7

seg000:0800D76E                 ADD             R3, R4  ; ID最高8位+ID最低8位

seg000:0800D770                 LDR             R5, =0x20001AC8

seg000:0800D772                 LDR             R4, =unk_2001AE8C

seg000:0800D774                 ADD             R3, R2  ; 最高8位+最低8位结果+ID累加提取的10位

seg000:0800D776                 STR             R2, [R6] ; ID累加提取的10位

seg000:0800D776                                         ; 保存到

seg000:0800D776                                         ; 0x200001AC8

seg000:0800D778                 UBFX.W          R3, R3, #0, #0xA ; ID最高8位

seg000:0800D778                                         ; 加

seg000:0800D778                                         ; ID最低8位

seg000:0800D778                                         ; 加

seg000:0800D778                                         ; ID累加结果提取的10位

seg000:0800D778                                         ; 的结果

seg000:0800D778                                         ; 提取10位

seg000:0800D77C                 STR             R3, [R4] ; 保存到

seg000:0800D77C                                         ; 0x20001AE8C

seg000:0800D77E

seg000:0800D77E loc_800D77E                             ; CODE XREF: sub_800D748+46↓j

seg000:0800D77E                 LDR.W           R4, [R5,R3,LSL#2] ;

seg000:0800D77E                                         ; r5=0x20001AC8

seg000:0800D77E                                         ; 加

seg000:0800D77E                                         ; r3扩展成32位

seg000:0800D77E                                         ; 地址的数据放到

seg000:0800D77E                                         ; R4

seg000:0800D782                 ADD             R3, R2  ;

seg000:0800D782                                         ; ID最高8位

seg000:0800D782                                         ; +

seg000:0800D782                                         ; ID最低8位

seg000:0800D782                                         ; +

seg000:0800D782                                         ; ID累加结果提取10位

seg000:0800D782                                         ; +

seg000:0800D782                                         ; ID蕾姐结果提取10位

seg000:0800D784                 STRB.W          R4, [R0,#1]! ;

seg000:0800D784                                         ; r4=0x20001ac8里面的数据

seg000:0800D784                                         ; 写到

seg000:0800D784                                         ; R0=0x2001AF54

seg000:0800D788                 CMP             R1, R0  ;

seg000:0800D788                                         ; R0=0x2001af54

seg000:0800D788                                         ; r1=0X20001AF5C

seg000:0800D788                                         ; 复制16位

seg000:0800D78A                 UBFX.W          R3, R3, #0, #0xA ;

seg000:0800D78A                                         ; ID最高8位

seg000:0800D78A                                         ; +

seg000:0800D78A                                         ; ID最低8位

seg000:0800D78A                                         ; +

seg000:0800D78A                                         ; ID累加结果提取10位

seg000:0800D78A                                         ; +

seg000:0800D78A                                         ; ID蕾姐结果提取10位

seg000:0800D78A                                         ; =

seg000:0800D78A                                         ; 结果提取10位

seg000:0800D78E                 BNE             loc_800D77E

seg000:0800D790                 POP             {R4-R6}

seg000:0800D792                 BX              LR
```

这个是比较关键的函数，看完这个就知道了计算后的ID被写入了`0X2001AF54`和`0X2001AF58`，而恰巧输入的密码也是两个`32`位数也就是`16`个`4`位数，正好和密码位数一样。

再看下面这个函数可以看到把`0X2001AF20`的一个数据读到了`R0`

![](https://bbs.kanxue.com/upload/attach/202101/621165_FT8W3JGF56BK74G.jpg)

继续看`800FE30`

![](https://bbs.kanxue.com/upload/attach/202101/621165_C3DRF7EFQAF776X.jpg)

先看`8001148C`

![](https://bbs.kanxue.com/upload/attach/202101/621165_FFFEK9F28RTSPHF.jpg)

可以看到把`0X50020414`的第`12`位清零

到参考手册看这个是什么寄存器

![](https://bbs.kanxue.com/upload/attach/202101/621165_ZPCC6QEYMBBHMV5.jpg)

可以看到从做的是`GPIOB`，打开后面的连接看寄存器页面

![](https://bbs.kanxue.com/upload/attach/202101/621165_ZAHNYKXVVTQBUW7.jpg)

可以看到是输出数据的寄存器，第`12`位也就是`GPIOB`的第`12`个脚

接下来打开`STM32H7`数据手册找这引脚是干什么的，用的是`LQFP100`封装的芯片

![](https://bbs.kanxue.com/upload/attach/202101/621165_UMZMSAM632PU4KD.jpg)

打开作者提供的电路原理图，找`GPIOB12`，也就是上图`PB12`第`51`个引脚

![](https://bbs.kanxue.com/upload/attach/202101/621165_RGWYJGP9QQV7WPM.jpg)

可以看到连接外部`FRAM`的片选引脚

![](https://bbs.kanxue.com/upload/attach/202101/621165_AQS5FNK9K2774KG.jpg)

看这个知道怎么连接外部`RAM`并且知道是通过`SPI2`，从手册可以看到`SPI2`的地址

```

```
![](https://bbs.kanxue.com/upload/attach/202101/621165_AX9ZD5GSGP5GRG7.jpg)

这个是操作外部`FRAM`的方法， 从`sub_800FED0`看到这里知道了这个就是读外部`FRAM`的，读了两个`32`位数

接着往下看
