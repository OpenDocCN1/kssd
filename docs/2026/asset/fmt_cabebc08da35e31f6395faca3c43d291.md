
```asm
00402B52      |.  8A19            |MOV     BL,BYTE PTR DS:[ECX]                 ;  依取出奇数的假码字符！送入到BL

00402B54      |.  C0E2 04         |SHL     DL,4                                 ;  左移，乘2^4(十六进制10)(十进制为16)

00402B57      |.  02D3            |ADD     DL,BL                                ;  DL=左移4位的DL+BL

00402B59      |.  83C1 02         |ADD     ECX,2                                ;  ECX+2做好准备取下一位偶数假码字符！

00402B5C      |.  885404 04       |MOV     BYTE PTR SS:[ESP+EAX+4],DL           ;  累加后保存到0012EE40（21436587A9CBEDFF）

00402B60      |.  40              |INC     EAX                                  ;  EAX加1为计数器器

00402B61      |.  83F8 08         |CMP     EAX,8                                ;  比较EAX是否小于8

00402B64      |.^ 7C E9           \JL      SHORT QQ聊天~1.00402B4F

00402B66      |.  8A4424 07       MOV     AL,BYTE PTR SS:[ESP+7]                ;  AL=等于计算后的4位

00402B6A      |.  8A5C24 04       MOV     BL,BYTE PTR SS:[ESP+4]                ;  BL=等于计算后的1位

00402B6E      |.  8A4C24 0B       MOV     CL,BYTE PTR SS:[ESP+B]                ;  CL=等于计算后的8位

00402B72      |.  8A5424 05       MOV     DL,BYTE PTR SS:[ESP+5]                ;  DL==等于计算后的2位

00402B76      |.  32C3            XOR     AL,BL

00402B78      |.  8A5C24 06       MOV     BL,BYTE PTR SS:[ESP+6]                ;  BL=等于计算后的3位

00402B7C      |.  32CA            XOR     CL,DL

00402B7E      |.  8A5424 09       MOV     DL,BYTE PTR SS:[ESP+9]                ;  DL=等于计算后的6位

00402B82      |.  32D3            XOR     DL,BL

00402B84      |.  8A5C24 08       MOV     BL,BYTE PTR SS:[ESP+8]                ;  BL==等于计算后的5位

00402B88      |.  325C24 0A       XOR     BL,BYTE PTR SS:[ESP+A]                ;  BL和=等于计算后的7位

00402B8C      |.  3C 38           CMP     AL,38                                 ;  比较1 XOR 4是否等于38

00402B8E      |.  75 35           JNZ     SHORT QQ聊天~1.00402BC5                 ;  不满足则跳走失败！！

00402B90      |.  80F9 6E         CMP     CL,6E                                 ;  比较2 XOR 8是否等于6E

00402B93      |.  75 30           JNZ     SHORT QQ聊天~1.00402BC5                 ;  不满足则跳走失败！！

00402B95      |.  80FA 4E         CMP     DL,4E                                 ;  比较6 XOR 3是否等于4E

00402B98      |.  75 2B           JNZ     SHORT QQ聊天~1.00402BC5                 ;  不满足则跳走失败！！

00402B9A      |.  80FB 1A         CMP     BL,1A                                 ;  比较5 XOR 7是否等于1A

00402B9D      |.  75 26           JNZ     SHORT QQ聊天~1.00402BC5                 ;  不满足则跳走失败！！

00402B9F      |.  8D4C24 2C       LEA     ECX,DWORD PTR SS:[ESP+2C]

00402BA3      |.  C74424 24 FFFFF>MOV     DWORD PTR SS:[ESP+24],-1

00402BAB      |.  E8 C2860100     CALL    QQ聊天~1.0041B272

00402BB0      |.  B8 01000000     MOV     EAX,1                                 ;  EAX置1（注册成功标志，这个标志也全局变量）爆破记得这里哦！

00402BB5      |.  5B              POP     EBX

00402BB6      |.  8B4C24 18       MOV     ECX,DWORD PTR SS:[ESP+18]

00402BBA      |.  64:890D 0000000>MOV     DWORD PTR FS:[0],ECX

00402BC1      |.  83C4 24         ADD     ESP,24

00402BC4      |.  C3              RETN

00402BC5      |>  8D4C24 2C       LEA     ECX,DWORD PTR SS:[ESP+2C]

00402BC9      |.  C74424 24 FFFFF>MOV     DWORD PTR SS:[ESP+24],-1

00402BD1      |.  E8 9C860100     CALL    QQ聊天~1.0041B272

00402BD6      |.  8B4C24 1C       MOV     ECX,DWORD PTR SS:[ESP+1C]

00402BDA      |.  33C0            XOR     EAX,EAX

00402BDC      |.  5B              POP     EBX

00402BDD      |.  64:890D 0000000>MOV     DWORD PTR FS:[0],ECX

00402BE4      |.  83C4 24         ADD     ESP,24

00402BE7      \.  C3              RETN
```

分析总结：

好了这个软件的算很简单，说得简单点这个注册方式就一定要16位字符（0~~F）了，

打比方我输入的假码：

第一步：(输入的假码）

```
01 23 45 67 89 AB CD EF
```

第二步：(假码字符排列）

```
10 32 54 76 98 BA DC FE
```

我们来分一下假码排列吧！把假码分为8组！

```
　　  10 32 54 76 98 BA DC FE
　　－－－－－－－－－－－－－－
      1　2　3　4　5　6　7　8
```

第三步:注册码算法就是做一个`XOR`运算了！!!!^\_^!!

```
  1(10) XOR 4(32)　＝　38
  2(54) XOR 8(76)　＝　6E
  6(98) XOR 3(BA)　＝　4E
  5(DC) XOR 7(FE)　＝　1A
```

满足以上等于号的条件就可以注册成功了。。！！直下来就是为真注册码排列注册方式！

第四步：这个是一个真注册码`1274749121908092`

```
           21　47  47　19　 12　 09　08  29　　　　　　　
        ----------------------------------------------
           1　 2   3　 4　　5  　6　 7　 8　　　　
```

然后排列为`1274749121908092`这个就是真注册码了！！

后话：

这个软件算法超简单了合适菜鸟们学习了。。感觉谢你看完这篇破文希望能给刚入门菜哥们能带来一点点的启发！时间比较伧促(难免错漏）写得不好请DX位不要拍砖哦！（我不坐砖机哦！）^\_^!!

22:22 2006-6-6

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
