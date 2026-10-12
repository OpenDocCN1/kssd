
# IrfanView

> 原文：`www.pediy.com/kssd/pediy05/pediy50035.htm`

初学解密，很想做一个`KeyGen`，这是低级菜鸟向高级菜鸟升级的必经之路。于是驾猫寻得一看图软件 `IrfanView` 效果不错，功能强大，可和`ACDSee`相媲美。这个软件不需要注册即可使用其全部功能，不过他还是设置了注册选项，注册成功后，在关于对话框中就有你的大名了。正好那他开刀。拿宝刀`Trw2000`。

下`bpx hmemcpy`，截下后`bc *`，再`pmodule`，这时已得到输入的名字，`F10`跟下去，又得到输入的假注册码（`d eax`看看，里面放着输入数据的长度）。再跟下去领空会进入系统内核，下`pmodule`跳出来，往下走来到一个循环，别管它，用`g`命令跳过去，接下来就是本文的正题了：（顺便提一下，姓名必须大于 2 各字符且小于 85 个字符。仔细看就会发现）

```
..................................

0167:0044FC1E  LEA       ECX,[ESP+11D0]----放入你输入的假注册码
0167:0044FC25  LEA       EDX,[ESP+10D0]----放入你输入的名字
0167:0044FC2C  PUSH      ECX
0167:0044FC2D  PUSH      EDX
0167:0044FC2E  CALL      00435F40----应该有点感觉吧，真正的注册码就在里面产生
          //这个 CALL 下面的内容我就省略了，应位于主题无关。

...................................

0167:00435F40  MOV        EAX,[ESP+08]----从上面的 CALL 跳过来的。开工哦^o^
0167:00435F44  SUB        ESP,BYTE +14
0167:00435F47  PUSH       EBX
0167:00435F48  PUSH       EBP
0167:00435F49  PUSH       ESI
0167:00435F4A  PUSH       EDI
0167:00435F4B  PUSH       EAX
0167:00435F4C  XOR        EBX,EBX
0167:00435F4E  CALL       004B0863----这个 CALL 可能有用，不过随叫咱还是菜鸟，别管它，抓住关键最重要
0167:00435F53  MOV        ESI,[ESP+2C]
0167:00435F57  MOV        EBP,EAX
0167:00435F59  MOV        EDI,ESI
0167:00435F5B  OR         ECX,BYTE -01
0167:00435F5E  XOR        EAX,EAX
0167:00435F60  ADD        ESP,BYTE +04
0167:00435F63  XOR        EDX,EDX
0167:00435F65  REPNE SCASB
0167:00435F67  NOT        ECX
0167:00435F69  DEC        ECX
0167:00435F6A  TEST       ECX,ECX
0167:00435F6C  JNG        00435F85
0167:00435F6E  MOVSX      ECX,BYTE [EDX+ESI]----从这里开始工作哦
0167:00435F72  ADD        EBX,ECX
0167:00435F74  MOV        EDI,ESI
0167:00435F76  OR         ECX,BYTE -01
0167:00435F79  XOR        EAX,EAX
0167:00435F7B  INC        EDX
0167:00435F7C  REPNE SCASB
0167:00435F7E  NOT        ECX
0167:00435F80  DEC        ECX
0167:00435F81  CMP        EDX,ECX\----EDX 放名字的长度
0167:00435F83  JL         00435F6E----将名字的各个 ASCII 码相加后放入 EBX
0167:00435F85  MOV        EAX,0104----以下各部生成密文
0167:00435F8A  PUSH       BYTE +0A
0167:00435F8C  SUB        EAX,EBX
0167:00435F8E  CDQ
0167:00435F8F  XOR        EAX,EDX
0167:00435F91  SUB        EAX,EDX
0167:00435F93  ADD        EAX,014C
0167:00435F98  LEA        EDX,[EAX*8+00]
0167:00435F9F  SUB        EDX,EAX
0167:00435FA1  LEA        ECX,[EAX+EDX*4]
0167:00435FA4  LEA        EDX,[ESP+14]
0167:00435FA8  PUSH       EDX
0167:00435FA9  LEA        ESI,[EAX+ECX*2]
0167:00435FAC  SHL        ESI,03
0167:00435FAF  PUSH       ESI\----到此为止输入? esi 后看到的十进制数即为密文
0167:00435FB0  CALL       004BB8D4----处理密文的有效性，并且将十进制密文转换为 ASCII 码
0167:00435FB5  ADD        ESP,BYTE +0C
0167:00435FB8  CMP        ESI,000F423F----查看十进制的密文是否大于 999999 按它的算法大于 999999 的
              //可能性不大。不过我还是写了 C 源代码。你可以忽略不管。
0167:00435FBE  JA         NEAR 004360B3----大就跳，一般是小
0167:00435FC4  MOV        CL,[ESP+14]----开始进行密文数据的移位，大致顺序是：（第一位记为 0）
              //第 4 位到第 6 位，第 5 位到第 8 位，第 3 位到第 5 位，
              //第 2 位到第 3 位，第 1 位到第 2 位
0167:00435FC8  MOV        AL,[ESP+15]
0167:00435FCC  MOV        DL,[ESP+13]
0167:00435FD0  MOV        [ESP+16],CL
0167:00435FD4  MOV        CL,[ESP+11]
0167:00435FD8  MOV        [ESP+18],AL
0167:00435FDC  MOV        AL,[ESP+12]
0167:00435FE0  MOV        [ESP+15],DL
0167:00435FE4  MOV        [ESP+12],CL
0167:00435FE8  MOV        ECX,[ESP+14]
0167:00435FEC  AND        ECX,FF----和上面的一句加起来其实就是一句 MOV CL,[ESP+14]。这样的语句后面还
              //有，要注意哦。
0167:00435FF2  MOV        [ESP+13],AL\-------数据移位结束，多看看就明白了^\_^
0167:00435FF6  MOV        EAX,ECX\----ECX 里放的是第 4 位，不了解就再仔细看看上面的数据移位方法
0167:00435FF8  SHL        EAX,05----接下来是一大堆无聊的操作，大家都是菜鸟，每句用 C 语言表示就可以了
0167:00435FFB  SUB        EAX,ECX
0167:00435FFD  MOV        ECX,[ESP+18]
0167:00436001  AND        ECX,FF
0167:00436007  LEA        EDX,[EAX+EAX*2]\----无聊，不就是 EAX*3 吗!\_^
0167:0043600A  LEA        EAX,[ECX+ECX*4]
0167:0043600D  SHL        EAX,03
0167:00436010  SUB        EAX,ECX
0167:00436012  SUB        EAX,EDX
0167:00436014  CDQ
          //把 EAX 的符号扩展到 EDX 中去（若 EAX 为负，则 EDX 为-1，否则为 0）后面
0167:00436015  MOV        ECX,EAX
0167:00436017  XOR        ECX,EDX
0167:00436019  SUB        ECX,EDX
0167:0043601B  LEA        EAX,[ECX+ECX*4]又来了，写成 C 语言的时候可别范傻
0167:0043601E  SHL        EAX,03
0167:00436021  SUB        EAX,ECX
0167:00436023  MOV        ECX,09
0167:00436028  CDQ
0167:00436029  IDIV       ECX
0167:0043602B  MOV        EAX,[ESP+13]
0167:0043602F  AND        EAX,FF
0167:00436034  ADD        DL,30
0167:00436037  MOV        [ESP+17],DL
0167:0043603B  LEA        EDX,[EAX+EAX*2]
0167:0043603E  SHL        EDX,04
0167:00436041  SUB        EDX,EAX
0167:00436043  MOV        EAX,[ESP+15]
0167:00436047  AND        EAX,FF
0167:0043604C  LEA        ECX,[EAX+EAX*8]
0167:0043604F  LEA        EAX,[EAX+ECX*4]
0167:00436052  LEA        EAX,[EDX+EAX*2]
0167:00436055  CDQ
0167:00436056  XOR        EAX,EDX
0167:00436058  SUB        EAX,EDX
0167:0043605A  LEA        ECX,[EAX+EAX*8]
0167:0043605D  LEA        EAX,[EAX+ECX*4]
0167:00436060  MOV        ECX,09
0167:00436065  SHL        EAX,1
0167:00436067  CDQ
0167:00436068  IDIV       ECX
0167:0043606A  MOV        ECX,[ESP+10]
0167:0043606E  AND        ECX,FF
0167:00436074  LEA        EAX,[ECX+ECX*2]
0167:00436077  LEA        EAX,[EAX+EAX*8]
0167:0043607A  SHL        EAX,1
0167:0043607C  SUB        EAX,ECX
0167:0043607E  ADD        DL,30
0167:00436081  MOV        [ESP+14],DL
0167:00436085  MOV        ECX,[ESP+11]
0167:00436089  AND        ECX,FF
0167:0043608F  LEA        EDX,[ECX*8+00]
0167:00436096  SUB        EDX,ECX
0167:00436098  LEA        EDX,[EDX+EDX*4]
0167:0043609B  SUB        EAX,EDX
0167:0043609D  CDQ
0167:0043609E  MOV        ECX,EAX
0167:004360A0  XOR        ECX,EDX
0167:004360A2  SUB        ECX,EDX
0167:004360A4  LEA        EAX,[ECX+ECX*2]
0167:004360A7  LEA        EAX,[EAX+EAX*8]
0167:004360AA  SHL        EAX,1
0167:004360AC  SUB        EAX,ECX\----慢慢长路。吾等菜鸟，就把上面的汇编语句
          //一句一句翻成 C 语句吧！！！\_!
0167:004360AE  JMP        004361A8----跳过去检测生成的注册码的正确性
0167:004360B3  MOV        AL,[ESP+15]----密文大于 999999 时的处理方法，和前面的有些相识
0167:004360B7  MOV        DL,[ESP+16]
0167:004360BB  MOV        CL,[ESP+14]
0167:004360BF  MOV        [ESP+16],AL
0167:004360C3  MOV        AL,[ESP+11]
0167:004360C7  MOV        [ESP+18],DL
0167:004360CB  MOV        DL,[ESP+12]
0167:004360CF  MOV        [ESP+12],AL
0167:004360D3  MOV        EAX,[ESP+16]
0167:004360D7  MOV        [ESP+15],CL
0167:004360DB  AND        EAX,FF
0167:004360E0  MOV        [ESP+13],DL
0167:004360E4  MOV        ECX,EAX
0167:004360E6  SHL        ECX,06
0167:004360E9  SUB        ECX,EAX
0167:004360EB  MOV        EAX,[ESP+18]
0167:004360EF  AND        EAX,FF
0167:004360F4  LEA        EAX,[EAX+EAX*8]
0167:004360F7  SHL        EAX,02
0167:004360FA  SUB        EAX,ECX
0167:004360FC  MOV        ECX,09
0167:00436101  CDQ
0167:00436102  XOR        EAX,EDX
0167:00436104  SUB        EAX,EDX
0167:00436106  LEA        EAX,[EAX+EAX*8]
0167:00436109  SHL        EAX,02
0167:0043610C  CDQ
0167:0043610D  IDIV       ECX
0167:0043610F  ADD        DL,30
0167:00436112  MOV        [ESP+17],DL
0167:00436116  MOV        EAX,[ESP+14]
0167:0043611A  AND        EAX,FF
0167:0043611F  ADD        EAX,BYTE +20
0167:00436122  LEA        EDX,[EAX*8+00]
0167:00436129  SUB        EDX,EAX
0167:0043612B  LEA        EAX,[EAX+EDX*4]
0167:0043612E  LEA        ECX,[EAX+EAX*2]
```



# 汇编分析

```
0167:00436131  MOV     EAX,[ESP+13]   
0167:00436135  AND     EAX,FF   
0167:0043613A  LEA     EDX,[EAX+EAX*4]   
0167:0043613D  SHL     EDX,03   
0167:00436140  SUB     EDX,EAX   
0167:00436142  LEA     EAX,[ECX+EDX*2]   
0167:00436145  CDQ    
0167:00436146  XOR     EAX,EDX   
0167:00436148  SUB     EAX,EDX   
0167:0043614A  LEA     ECX,[EAX*8+00]   
0167:00436151  SUB     ECX,EAX   
0167:00436153  LEA     EAX,[EAX+ECX*4]   
0167:00436156  MOV     ECX,09   
0167:0043615B  LEA     EAX,[EAX+EAX*2]   
0167:0043615E  CDQ    
0167:0043615F  IDIV    ECX   
0167:00436161  MOV     EAX,[ESP+10]   
0167:00436165  AND     EAX,FF   
0167:0043616A  ADD     DL,30   
0167:0043616D  MOV     [ESP+14],DL   
0167:00436171  LEA     EDX,[EAX*8+00]   
0167:00436178  SUB     EDX,EAX   
0167:0043617A  LEA     EAX,[EAX+EDX*4]   
0167:0043617D  MOV     EDX,[ESP+11]   
0167:00436181  AND     EDX,FF   
0167:00436187  MOV     ECX,EDX   
0167:00436189  SHL     ECX,04   
0167:0043618C  ADD     ECX,EDX   
0167:0043618E  SHL     EAX,1   
0167:00436190  LEA     ECX,[ECX+ECX*4]   
0167:00436193  SUB     EAX,ECX   
0167:00436195  CDQ    
0167:00436196  XOR     EAX,EDX   
0167:00436198  SUB     EAX,EDX   
0167:0043619A  LEA     EDX,[EAX*8+00]   
0167:004361A1  SUB     EDX,EAX   
0167:004361A3  LEA     EAX,[EAX+EDX*4]   
0167:004361A6  SHL     EAX,1   
0167:004361A8  CDQ        ---上面的正确性检测跳转语句跳过来的。这里也是一条语句别忘哦！   
0167:004361A9  MOV     ECX,09   
0167:004361AE  MOV     BYTE [ESP+19],00----加上字符串结尾符'\0'，这里可知注册码必为 10 位   
0167:004361B3  IDIV    ECX   
0167:004361B5  ADD     DL,30   
0167:004361B8  MOV     [ESP+11],DL   
0167:004361BC  LEA     EDX,[ESP+10]   
0167:004361C0  PUSH    EDX----一般来说到这里就能找到正确的注册码了，打 d esp+10 看看   
0167:004361C1  CALL    004B0863----这里开始对生成的注册码进行正确性检测。不过用处不大，不需要      
    //为他编写源程序，通常都能校验通过（随叫我是菜鸟，只要把问题大
    //致解决就可以了，想要进一步了解的话，自己进去就是了吧）
0167:004361C6  ADD     ESP,BYTE +04----下面的对于注册机源文件已无多大用处   
0167:004361C9  XOR     ECX,ECX   
0167:004361CB  CMP     EBP,EAX   
0167:004361CD  POP     EDI   
0167:004361CE  POP     ESI   
0167:004361CF  SETZ    CL   
0167:004361D2  POP     EBP   
0167:004361D3  MOV     EAX,ECX   
0167:004361D5  POP     EBX   
0167:004361D6  ADD     ESP,BYTE +14   
0167:004361D9  RET      ----返回（什么，你还不知道注册码是多少。拜托，在 EDX 里啦。你是不是没再看）
```

.....................................

好了，解说完毕。下面是我的 C 源代码，写的有些傻，就是一句一句的直译汇编语句（你可别笑，通俗易懂吗）

```c
#include "stdio.h"   
#include "string.h"   

void main()   
{   
    static char name[100]={0},sn[10]={0};//初始化   

    int i=0,len;   
    long sum=0,eax=0,ebx=0,ecx=0,edx=0,esi=0,esitemp=0;   

    printf("Please Input Your Name:");   
    gets(name);   

    len=strlen(name);   

    for(i=0;i<len;i++)//累加姓名的 ASCII 码   
        sum+=name[i];   

    ebx=sum;   

    eax=0x104;//算得密文   

    eax-=ebx;   

    if(eax<0)   
        edx=-1;   
    else   
        edx=0;   

    eax^=edx;   

    eax-=edx;   

    eax+=0x014c;   

    edx-=eax;   

    edx=eax*8;   

    edx-=eax;   

    ecx=eax+edx*4;   

    esi=eax+ecx*2;   

    esi<<=0x3;   

    i=5;   

    esitemp=esi;   

    while(esitemp)//将密文转为字符串   
    {   
        sn[i--]=esitemp%10+'0';   

        esitemp/=10;   
    }   

    if(esi<=999999)//开始处理密文   
    {   
        sn[6]=sn[4];//移位（在了解了移位过程后的简化步骤）    
        sn[8]=sn[5];   

        sn[5]=sn[3];   

        sn[3]=sn[2];   

        sn[2]=sn[1];   

        eax=ecx=sn[4];//开始计算注册码   

        eax<<=0x05;   

        eax-=ecx;   

        ecx=sn[8];   

        edx=eax*3;   

        eax=ecx*5;   

        eax<<=0x03;   

        eax-=ecx;   

        eax-=edx;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        ecx=eax;   

        ecx^=edx;   

        ecx-=edx;   

        eax=ecx*5;   

        eax<<=0x03;   

        eax-=ecx;   

        ecx=0x09;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        edx=eax%ecx;   

        eax/=ecx;   

        eax=sn[3];   

        edx+='0';   

        sn[7]=edx;   

        edx=eax*3;   

        edx<<=0x04;   

        edx-=eax;   

        eax=sn[5];   

        ecx=eax*9;   

        eax=eax+ecx*4;   

        eax=edx+eax*2;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        eax^=edx;   

        eax-=edx;   

        ecx=eax*9;   

        eax=eax+ecx*4;   

        ecx=9;   

        eax<<=0x01;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        edx=eax%ecx;   

        eax/=ecx;   

        ecx=sn[0];   

        eax=ecx*3;   

        eax*=9;   

        eax<<=0x01;   

        eax-=ecx;   

        edx+='0';   

        sn[4]=edx;   

        ecx=sn[1];   

        edx=ecx*8;   

        edx-=ecx;   

        edx*=5;   

        eax-=edx;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        ecx=eax;   

        ecx^=edx;   

        ecx-=edx;   

        eax=ecx*3;   

        eax*=9;   

        eax<<=0x01;   

        eax-=ecx;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   
    }   

    else//这里是大于 999999 的处理   
    {   
        sn[8]=sn[6];   

        sn[6]=sn[5];   

        sn[5]=sn[4];   

        sn[3]=sn[2];   

        sn[2]=sn[1];   

        ecx=eax=sn[6];   

        ecx<<=0x06;   

        ecx-=eax;   

        eax=sn[8];   

        eax*=9;   

        eax<<=0x02;   

        eax-=ecx;   

        ecx=0x09;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        eax^=edx;   

        eax-=edx;   

        eax*=9;   

        eax<<=0x02;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        edx=eax%ecx;   

        eax/=ecx;   

        edx+='0';   

        sn[7]=edx;   

        eax=sn[4];   

        eax+=0x20;   

        edx=eax*8;   

        edx-=eax;   

        eax=eax+edx*4;   

        ecx=eax*3;   

        eax=sn[3];   

        edx=eax*5;   

        edx<<=0x03;   

        edx-=eax;   

        eax=ecx+edx*2;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        eax^=edx;   

        eax-=edx;   

        ecx=eax*8;   

        ecx-=eax;   

        eax=eax+ecx*4;   

        ecx=0x09;   

        eax*=3;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        edx=eax%ecx;   

        eax/=ecx;   

        eax=sn[0];   

        edx+='0';   

        sn[4]=edx;   

        edx=eax*8;   

        edx-=eax;   

        eax=eax+edx*4;   

        edx=sn[1];   

        ecx=edx;   

        ecx<<=0x04;   

        ecx+=edx;   

        eax<<=0x01;   

        ecx*=5;   

        eax-=ecx;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   

        eax^=edx;   

        eax-=edx;   

        edx=eax*8;   

        edx-=eax;   

        eax=eax+edx*4;   

        eax<<=0x01;   

        if(eax<0)   
            edx=-1;   
        else   
            edx=0;   
    }   

    ecx=0x09;//这一段为大于 999999 和小于 999999 都要处理的部分（看一下汇编源程序就明白了）    
    edx=eax%ecx;   

    eax/=ecx;   

    edx+='0';   

    sn[1]=edx;   

    printf("Your Serial Numbers:%s\n",sn);//当然是梦寐以求的啦！   
}
```

为了能够方便的输入中文，可用用 VC 作了一个图形界面的 KeyGen（可能有点大材小用了^\_^）。

Note：用户名别输的太长，否则算出的注册码不正确（水平有限?_?）。

写这篇文只是希望能起到抛砖引玉的作用，用 C 直译汇编语言（很傻不过很有用）。

谢谢观赏（初次露面，混个脸熟 o_o） 4dd466d0e1e0cc123afbe504da2462ca
