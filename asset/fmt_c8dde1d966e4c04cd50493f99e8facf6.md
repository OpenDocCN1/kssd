
# Swf2MP3(国外)保护技术深入剖析

> 原文：[`www.pediy.com/kssd/pediy10/81414.html`](https://www.pediy.com/kssd/pediy10/81414.html)

## 软件信息

软件类别：国外软件/共享版/音频工具

运行环境：`Win9x`/`Me`/`NT`/`2000`/`XP`

开发商：`http://www.hootech.com/`

软件介绍：

`flash`工具，可以释放出`flash`电影中（`swf`文件）所有的声音元素，同时支持突发声音和流声音，支持 2 种格式（`.mp3`和`.wav`).

## 前言

My wife 要大量地从`swf`中取`Mp3`放手机里听(比如新`xx`的课程)，网上正好有这个工具。但无注册码可找，甚感疑惑。况且这个是老外的工具，遂破之。本文主要阐述原理,细节省略。

![](img/2cb85afccbf574a2584f2b0d5e866a99.jpg)

简单流程：运行后读取注册表中一串 60 字节的关键数据进行处理，如果符合注册条件就正常使用，不再修改注册表数据否则会将已使用的次数加密后写入该注册表数据的特定地方，同时写入的还有判断是否注册的条件数据，最后弹出注册框显示使用的次数。该软件爆破很简单，改变一下运行的流程，就会在注册表中生成符合注册条件的数据，以后就可以正常使用了。由于卸载时不清除注册表中该处关键数据，导致在该机重装该软件后仍可正常使用。

## 一、先来看看如何判断是否注册

关键位置在这：

```assembly
0040DE6E   .  52            PUSH EDX                             ;三个外部参数
0040DE6F   .  6A 1E        PUSH 1E                              ;
0040DE71   .  50            PUSH EAX                             ;
0040DE72   .  E8 C9EEFFFF  CALL Swf2MP3(.0040CD40  ；//判断注册 并计算使用次数
0040DE77   .  83C4 0C      ADD ESP,0C                           ; 返回 eax＝1 表示注册
0040DE7A   .  85C0          TEST EAX,EAX                         ;
0040DE7C   .  74 1E        JE SHORT Swf2MP3(.0040DE9C
0040DE7E   .  8B86 8C260000 MOV EAX,DWORD PTR DS:[ESI+268C]     
0040DE84   .  8BCE          MOV ECX,ESI
0040DE86   .  0D 00000096   OR EAX,96000000
0040DE8B   .  8986 8C260000 MOV DWORD PTR DS:[ESI+268C],EAX

;  esi＋268c 处存放初始值 96000000
0040DE91   .  E8 6A270000   CALL Swf2MP3(.00410600
0040DE96   .  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]
0040DE9A   .  EB 13        JMP SHORT Swf2MP3(.0040DEAF
0040DE9C   >  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]    ;  未注册的跳到这
0040DEA0   .  83F8 1E      CMP EAX,1E                              ;  有没有 30 次
0040DEA3   .  72 0A        JB SHORT Swf2MP3(.0040DEAF
0040DEA5   .  818E 8C260000>OR DWORD PTR DS:[ESI+268C],40000000    
;  超过试用次数第 30 位置 1
0040DEAF   >  8B8E 8C260000 MOV ECX,DWORD PTR DS:[ESI+268C]
0040DEB5   .  F7C1 00000002 TEST ECX,2000000        判断 32 位的标志中有无变化
0040DEBB   .  74 18        JE SHORT Swf2MP3(.0040DED5
0040DEBD   .  F7C1 00000004 TEST ECX,4000000
0040DEC3   .  74 10        JE SHORT Swf2MP3(.0040DED5
0040DEC5   .  F7C1 00000010 TEST ECX,10000000
0040DECB   .  74 08        JE SHORT Swf2MP3(.0040DED5
0040DECD   .  F7C1 00000080 TEST ECX,80000000
0040DED3   .  75 37        JNZ SHORT Swf2MP3(.0040DF0C  ;  即注册过的跳走返回 1
0040DED5   >  40            INC EAX                             ;  上次使用的次数 inc 了一下
0040DED6   .  8D4C24 20     LEA ECX,DWORD PTR SS:[ESP+20]
0040DEDA   .  50            PUSH EAX                            ；//使用的次数（含本次）
0040DEDB   .  51            PUSH ECX
0040DEDC   .  894424 14     MOV DWORD PTR SS:[ESP+14],EAX
0040DEE0   .  E8 0BEDFFFF   CALL Swf2MP3(.0040CBF0      ;生成注册表 60 字节数据  
0040DEE5   .  83C4 08        ADD ESP,8
0040DEE8   .  8D5424 20     LEA EDX,DWORD PTR SS:[ESP+20]
0040DEEC   .  8BCE          MOV ECX,ESI
0040DEEE   .  52            PUSH EDX
0040DEEF   .  6A 01         PUSH 1
0040DEF1   .  E8 4A010000   CALL Swf2MP3(.0040E040
0040DEF6   .  8B4424 0C     MOV EAX,DWORD PTR SS:[ESP+C]   
;前面调用返回的使用次数
0040DEFA   .  8B4E 1C       MOV ECX,DWORD PTR DS:[ESI+1C]
0040DEFD   .  6A 00         PUSH 0                 ; /lParam = 0
0040DEFF   .  50            PUSH EAX             //发送使用的次数 显示时用到
0040DF00   .  68 77040000   PUSH 477           ; |Message = MSG(477)
0040DF05   .  51            PUSH ECX             ; |hWnd
0040DF06   .  FF15 ACD44500 CALL DWORD PTR DS:[<&USER32.PostMessageA 0040DF0C   >  5F            POP EDI
0040DF0D   .  5E            POP ESI
0040DF0E   .  B8 01000000   MOV EAX,1
0040DF13   .  5B            POP EBX
0040DF14   .  83C4 50       ADD ESP,50
0040DF17   .  C3            RETN
0040CD40  /$  8B4C24 04     MOV ECX,DWORD PTR SS:[ESP+4]
.......
0040CE7D  |.  33C0          XOR EAX,EAX
0040CE7F  |.  5B            POP EBX
0040CE80  \\.  C3            RETN
```

由于算法比较复杂，用`hexrays`转换成伪`c`代码分析。

外部参数`a1`指向读取的注册表数据的起始处，`a2`是`1Eh`即`30`，`a3`地址处存放计算得到的已使用的次数（不包括本次）。

```c
signed int __cdecl sub_40CD40(int a1, int a2, int a3)  
{  //内部变量定义省略
   ....
  v4 = a1;
  v3 = *(_DWORD *)(a1 + 12);
  if ( (v3 ^ *(_DWORD *)(a1 + 56) ^ 0xB927102E) != 698926849
    || (v5 = *(_DWORD *)(a1 + 4), v6 = *(_DWORD *)a1, a1 = *(_DWORD *)(a1 + 8), v7 = *(_DWORD *)(v4 + 16),  
(v3 ^ 0xB927102E) + (v7 ^ 0xB927102E) + (a1 ^ 0xB927102E) + (v5 ^ 0xB927102E) + (v6 ^ 0xB927102E) != ~*
(_DWORD *)(v4 + 24))
    || (v8 = *(_DWORD *)(v4 + 36), (v8 ^ 0x28A79C95) * (*(_DWORD *)(v4 + 32) ^ 0x28A79C95) != ~*(_DWORD *)(v4  
+ 48)) )
  {
    *(_DWORD *)a3 = a2 + 1;
    result = 0;
  }
  else
  {
    v9 = v7 ^ *(_DWORD *)(v4 + 20) ^ 0xB927102E;
    if ( v9 != -1875289631    //即 0x903959E1  
      || (v6 ^ *(_DWORD *)(v4 + 28) ^ 0xB927102E) != 1686250269
      || (v5 ^ *(_DWORD *)(v4 + 40) ^ 0xB927102E) != -1988021778
      || (v8 ^ *(_DWORD *)(v4 + 44) ^ 0x28A79C95) != -1740411931
      || (*(_DWORD *)(v4 + 32) ^ *(_DWORD *)(v4 + 52) ^ 0x28A79C95) != 374450403 )
    {
      if ( v9 == -1989516033 )
      {   //未注册过的到这 此时 a1 是 *(_DWORD *)(a1 + 8) 注意前面有重新赋值
          //已使用的次数（不含本次）＝偏移 8 处的^偏移 40 处的异⁰xB927102E  
        *(_DWORD *)a3 = a1 ^ *(_DWORD *)(v4 + 40) ^ 0xB927102E;  
        result = 0;
      }
      else
      {
        *(_DWORD *)a3 = a2 + 1;  //31 次
        result = 0;
      }
    }
    else
    { //注册过的到这执行
      result = 1;
    }
  }
  return result;
}
```

//生成 60 字节注册表数据  `a2` 是已使用的次数（含本次）  `a1` 指向 60 字节的数据 buffer 起始地址

```c
signed int __cdecl sub_40CBF0(int a1, int a2)  
{  //内部变量定义省略
   ....
  v3 = time(0);
  v4 = GetTickCount();
  srand(v4 * v3);
  v5 = rand();
  v6 = a1;
  *(_DWORD *)a1 = rand() * v5;
  v7 = rand();
  *(_DWORD *)(v6 + 4) = rand() * v7;
  v8 = rand();
  *(_DWORD *)(v6 + 8) = rand() * v8;                  // 偏移 8h 处＝rand1
  v9 = rand();
  *(_DWORD *)(v6 + 12) = rand() * v9;
  v10 = rand();
  v11 = rand() * v10;
  v12 = *(_DWORD *)v6 + *(_DWORD *)(v6 + 12) + *(_DWORD *)(v6 + 8);
  *(_DWORD *)(v6 + 16) = v11;
  *(_DWORD *)(v6 + 24) = ~(*(_DWORD *)(v6 + 4) + v11 + v12);
  v13 = rand();
  *(_DWORD *)(v6 + 32) = rand() * v13;
  v14 = rand();
  v15 = rand() * v14;
  *(_DWORD *)(v6 + 36) = v15;
  v16 = *(_DWORD *)(v6 + 12) ^ 0x29A8C701;
  v17 = *(_DWORD *)(v6 + 16) ^ 0x896A64FF;
  *(_DWORD *)(v6 + 48) = ~(*(_DWORD *)(v6 + 32) * v15);
  *(_DWORD *)(v6 + 56) = v16;
  *(_DWORD *)(v6 + 20) = v17;
  v18 = rand();
  v19 = a2;                      //注意 v19 变成了已使用的次数（含本次）        
  *(_DWORD *)(v6 + 28) = v18;
  *(_DWORD *)(v6 + 40) = v19 ^ *(_DWORD *)(v6 + 8);    // 偏移 40 处＝已使用的次数（含本次）^rand1
  *(_DWORD *)(v6 + 44) = rand();
  v20 = rand();
  v21 = *(_DWORD *)v6;
  v22 = *(_DWORD *)(v6 + 4);
  *(_DWORD *)(v6 + 52) = v20;
  *(_DWORD *)v6 = v21 ^ 0xB927102E;
  v23 = *(_DWORD *)(v6 + 12);
  v24 = *(_DWORD *)(v6 + 8) ^ 0xB927102E;    //v24=rand1^ 0xB927102E
  *(_DWORD *)(v6 + 4) = v22 ^ 0xB927102E;
  v25 = *(_DWORD *)(v6 + 16);
  *(_DWORD *)(v6 + 8) = v24;         //偏移 8 处的=rand1^ 0xB927102E
  v26 = *(_DWORD *)(v6 + 32);
  *(_DWORD *)(v6 + 12) = v23 ^ 0xB927102E;
  v27 = *(_DWORD *)(v6 + 36);
  *(_DWORD *)(v6 + 16) = v25 ^ 0xB927102E;
  *(_DWORD *)(v6 + 32) = v26 ^ 0x28A79C95;
  *(_DWORD *)(v6 + 36) = v27 ^ 0x28A79C95;
  return 1;
}
```

到此根据第一个伪代码中的：已使用的次数（不含本次）＝偏移 8 处的^偏移 40 处的⁰xB927102E  
