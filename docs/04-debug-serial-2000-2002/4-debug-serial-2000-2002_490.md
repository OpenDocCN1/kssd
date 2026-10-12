
# sygate 3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum650.htm`](https://www.pediy.com/kssd/pediy03/forum650.htm)

## SyGate3.0 破解文档

### 使用工具

*   `Softice` 3.0
*   `w32dasm` 8.93

**破解者**: 买草帽

`SyGate3.0` 是一个很好的代理服务软件，它可以让用户用一台 modem 上网共享。这个软件有 31 天的使用限制，也有使用次数的限制，并且它好象在注册表中写入了一个啥，反正我重装不能使用，不爽。兄弟们几个在网上想用一个小猫同时在网上神游的感觉才没几天就要结束了，可不行。得想想办法。看来得跟 `Sygate` 公司请个安了。因为这个软件的注册必须在网上进行，先拨号上网再输入注册码，这个程序跟注册运算相关的，分别是 `Serial No` 有两个输入文本框，还有一个是 `Registration` 文本框。我在它们中输入如下：

*   `Serial No`: `18181818` `41841888`
*   `Registration`: `94189418`

为啥这样输入，呵呵。我没钱钱去注册呀，他们要这样拨号验证。当然先去给 `Sygate` 公司道个吉祥（不是小弟舍不得注册的钱钱，不怕各位笑话，我连真的 $ 都没摸过。哪有钱钱去注册嘛。等二回我手头有 $ 了一定...恭喜发财！），先礼...而后在 `Softice` 的黑脸下面的断点：

```
bpx hmemcpy
```

然后 `Ctrl+D` 切回，按注册按钮，断下了。就按 `F12` 回到程序空间。程序先把你输入的东西转换为其相应的 16 进制表示的有点像 BCD 码的相应表示。然后再走一会可以看见如下代码：

```
:0040F1CE 50                     push eax
:0040F1CF 57                     push edi
; Reference To: SgSrvAd.SgSetLicense, Ord:001Fh
;
:0040F1D0 E883490000             Call 00413B58//这个调用就是调用接口函数在 SgSrvAd.dll 文件中,完成注册
                                         //下面就要进去追注册了!我跳进这个 dll 去追注册码
:0040F1D5 8BF0                   mov esi, eax
:0040F1D7 8B442424               mov eax, dword ptr [esp+24]
:0040F1DB 85C0                   test eax, eax//返回 eax=0 就显示注册成功,
:0040F1DD 7479                   je 0040F258
:0040F1DF 3D37FFFFFF             cmp eax, FFFFFF37
:0040F1E4 7533                   jne 0040F219
; Possible Reference to String Resource ID=02000: "SyGate Message"
;
:0040F1E6 68D0070000             push 000007D0
:0040F1EB 8D4C2414               lea ecx, dword ptr [esp+14]
:0040F1EF E83EC90100             call 0042BB32
```

下面就是在 dll 中的代码部分判断函数在 `SgSrvAd.dll` 中的函数 `SgSetLicense()` 里，慢慢走没多久就可以走到这里了，让我来一点一点地解读它。

```
; Referenced by a CALL at Addresses:
; :100056A5  , :10009C85
;
:10005630 53                     push ebx
:10005631 8B5C240C               mov ebx, dword ptr [esp+0C]
:10005635 57                     push edi
:10005636 8B7C240C               mov edi, dword ptr [esp+0C]
:1000563A 53                     push ebx//第一部分输入的字符长度
:1000563B 57                     push edi//这里是输入的 serial No 第一部分输入转换成相应 16 进制 BCD 码
:1000563C E86FFFFFFF             call 100055B0//马上跟进望下边看，这个调用是验证第一部分输入的有效性和生成第二部分注册码的
:10005641 83C408                 add esp, 00000008
:10005644 85C0                   test eax, eax//回来后这个要为 0 才让你一路顺风哈
:10005646 753B                   jne 10005683
:10005648 837C241804             cmp dword ptr [esp+18], 00000004
:1000564D 752C                   jne 1000567B
:1000564F 8B442414               mov eax, dword ptr [esp+14]
:10005653 56                     push esi//Registration 部分输入的 BCD 码转换,94189418
:10005654 53                     push ebx//第一部分输入的字符长度
:10005655 57                     push edi//其内是 16 进制 BCD 表示的 18181818
:10005656 8B30                   mov esi, dword ptr [eax]
:10005658 E8B3FFFFFF             call 10005610//这个也要跟进你先往下跳哈,遇到 ret 才回来(生成 Registration 部分注册码)
:1000565D 83C408                 add esp, 00000008
:10005660 3BC6                   cmp eax, esi//这两个要相等,才行哟,哪个为准?当然是 eax 中的,它才是真的哟!(esi 中的 94189418Sygate 不收哈)
:10005662 5E                     pop esi
:10005663 7516                   jne 1000567B//当然这里可不能跳的哈
:10005665 8B0F                   mov ecx, dword ptr [edi]//还有个机关哟...对 18181818 这段输入有一个很迷信的要求哈,Sygate 公司制订的哈.
:10005667 BA00000040             mov edx, 40000000//第一部分第一个字符，一定不能大于 4(看来 Sygate 确实有点忌讳的哈,输入第一个大于 4 的话它不高兴的哟)，也玩完！！！
:1000566C 81E1000000F0           and ecx, F0000000
:10005672 5F                     pop edi
:10005673 3BD1                   cmp edx, ecx//大于 4 就是影响 S 标志
:10005675 5B                     pop ebx
:10005676 1BC0                   sbb eax, eax//这条指令要被 S 标志影响
:10005678 F7D8                   neg eax//第一个输入字符大于 4,就米嘎了
:1000567A C3                     ret

; Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
; :1000564D(C), :10005663(C)
;
:1000567B 5F                     pop edi
:1000567C B802000000             mov eax, 00000002
:10005681 5B                     pop ebx
:10005682 C3                     ret

; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :10005646(C)
;
:10005683 5F                     pop edi
:10005684 B801000000             mov eax, 00000001
:10005689 5B                     pop ebx
:1000568A C3                     ret
```

**第二部分生成代码区**

```
; Referenced by a CALL at Address:
; :1000563C
;
:100055B0 837C240808             cmp dword ptr [esp+08], 00000008//第一部分必须为 8 个字符
:100055B5 7406                   je 100055BD //不跳就玩完
:100055B7 B801000000             mov eax, 00000001//你不相信到了这里返回 eax=1 就要洗白
:100055BC C3                     ret//知道了把,为啥输入'18181818',第一部分输入必须是 8 个字符才让你过哈

; Referenced by a (U)nconditional or (C)onditional Jump at Address:
; :100055B5(C)
;
:100055BD 56                     push esi
:100055BE 8B742408               mov esi, dword ptr [esp+08]
:100055C2 57                     push edi
:100055C3 8B06                   mov eax, dword ptr [esi]//BCD 码 18181818 注册码第一部分
:100055C5 50                     push eax
:100055C6 E805FFFFFF             call 100054D0//生成注册码第二部分，跟进哈
:100055CB 8BC8                   mov ecx, eax//返回值给 ecx
:100055CD 83C404                 add esp, 00000004
:100055D0 8BD1                   mov edx, ecx
:100055D2 C1E834                 shr eax, 34
:100055D5 C1EA28                 shr edx, 28
:100055D8 8D0440                 lea eax, dword ptr [eax+2*eax]
:100055DB 8D1492                 lea edx, dword ptr [edx+4*edx]
:100055DE 03C2                   add eax, edx
:100055E0 8BD1                   mov edx, ecx
:100055E2 C1EA1C                 shr edx, 1C
:100055E5 C1E90E                 shr ecx, 0E
:100055E8 8D3C92                 lea edi, dword ptr [edx+4*edx]
:100055EB 8D147A                 lea edx, dword ptr [edx+2*edi]
:100055EE 5F                     pop edi
:100055EF 03C2                   add eax, edx
:100055F1 03C1                   add eax, ecx
:100055F3 8B4E04                 mov ecx, dword ptr [esi+04]//'34343434'
:100055F6 25FFFF0F00             and eax, 000FFFFF//这个值就是第一部分生成的
                                          //第二部分，一定要等于 esi+04
:100055FB 81E1FFFF0F00           and ecx, 000FFFFF
:10005601 2BC1                   sub eax, ecx
:10005603 5E                     pop esi
:10005604 C3                     ret
```

```
; Referenced by a CALL at Address:
; :100055C6
;
//算法部分:文本的第二部分跟 18181818 相匹配的注册码就由这里运算得来,你可以把它当做一个黑箱.在这里外
//界输入只有一个 18181818,(当然你也可以把它当做一头奶牛,吃的是 18181818,挤出的就是......:),其实程序
//在这里做的也就是做的几个异或加上小学就会的四则混合运算拉,我懒!就不分析了哈,有数学兴趣的漫漫看.
:100054D0 8B4C2404               mov ecx, dword ptr [esp+04] \ BCD 18181818
:100054D4 56                     push esi
:100054D5 8BC1                   mov eax, ecx
:100054D7 3543512400             xor eax, 00245143
:100054DC 0FAFC1                 imul eax, ecx
```



## 第一部分：生成前两部分注册码

```
:100054DF 0FAFC1                imul eax, ecx   

:100054E2 8BD0                  mov edx, eax   

:100054E4 C1E20B                shl edx, 0B   

:100054E7 2BD0                  sub edx, eax   

:100054E9 8D1492                lea edx, dword ptr [edx+4*edx]   

:100054EC 8D1490                lea edx, dword ptr [eax+4*edx]   

:100054EF 8D1452                lea edx, dword ptr [edx+2*edx]   

:100054F2 8DB450777A0600        lea esi, dword ptr [eax+2*edx+00067A77]   

:100054F9 B865A62573            mov eax, 7325A665   

:100054FE 0FAFF1                imul esi, ecx   

:10005501 F7E1                  mul ecx   

:10005503 0FAFF1                imul esi, ecx   

:10005506 81EE6E1A0000          sub esi, 00001A6E   

:1000550C 0FAFF1                imul esi, ecx   

:1000550F C1EA11                shr edx, 11   

:10005512 8BC2                  mov eax, edx   

:10005514 C1E008                shl eax, 08   

:10005517 03C2                  add eax, edx   

:10005519 8D0440                lea eax, dword ptr [eax+2*eax]   

:1000551C 8D8406B1110000        lea eax, dword ptr [esi+eax+000011B1] \\ 第二部分挤出来的在 eax 中,   

:10005523 5E                    pop esi   

:10005524 C3                    ret
```

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

## 第三部分：生成代码区

```
10005610 837C240808            cmp dword ptr [esp+08], 00000008 //第二次判断你输入是字符数是 8 吗?   

:10005615 7406                  je 1000561D //这里要跳过去哈,不然.....   

:10005617 B801000000            mov eax, 00000001   

:1000561C C3                    ret // 呵呵~看来 Sygate 公司跟我们一样,对这样的吉祥数值 8 也是喜欢哈.但你可不要把代码走到这一行哟!
```

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:10005615(C)`

`|`

```
:1000561D 8B442404              mov eax, dword ptr [esp+04]   

:10005621 8B08                  mov ecx, dword ptr [eax]   

:10005623 51                    push ecx //根据第一部分生成   

10005624 E807FFFFFF            call 10005530 //跟进就是生成 Registration 的算法了哈!    

:10005629 83C404                add esp, 00000004   

:1000562C C3                    ret
```

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

## 算法部分

`//算法部分:文本的第三部分跟 18181818 相匹配的 Registration 注册码就由这里运算得来,你可以把它当做一个`
`//黑箱.在这里外界输入只有一个 18181818,(当然你也可以还是把它当做一头奶牛,吃的是 18181818,挤出的就是......:),`

```
:10005530 8B4C2404              mov ecx, dword ptr [esp+04]   

:10005534 56                    push esi   

:10005535 8BC1                  mov eax, ecx //ecx=(BCD 码)18181818   

:10005537 3545099294            xor eax, 94920945   

:1000553C 0FAFC1                imul eax, ecx   

:1000553F 0FAFC1                imul eax, ecx   

:10005542 0FAFC1                imul eax, ecx   

:10005545 0FAFC1                imul eax, ecx   

:10005548 8D14C500000000        lea edx, dword ptr [8*eax+00000000]   

:1000554F 2BD0                  sub edx, eax   

:10005551 8D1490                lea edx, dword ptr [eax+4*edx]   

:10005554 8DB4D061050000        lea esi, dword ptr [eax+8*edx+00000561]   

:1000555B B8CF4D0FC2            mov eax, C20F4DCF   

:10005560 F7E1                  mul ecx   

:10005562 0FAFF1                imul esi, ecx   

:10005565 8BC1                  mov eax, ecx   

:10005567 2BC2                  sub eax, edx   

:10005569 D1E8                  shr eax, 1   

:1000556B 03C2                  add eax, edx   

:1000556D C1E811                shr eax, 11   

:10005570 8BD0                  mov edx, eax   

:10005572 C1E204                shl edx, 04   

:10005575 03D0                  add edx, eax   

:10005577 8D04D2                lea eax, dword ptr [edx+8*edx]   

:1000557A 8D04C0                lea eax, dword ptr [eax+8*eax]   

:1000557D 03F0                  add esi, eax   

:1000557F 0FAFF1                imul esi, ecx   

:10005582 81EE1C82B302          sub esi, 02B3821C   

:10005588 0FAFF1                imul esi, ecx   

:1000558B C1E91E                shr ecx, 1E   

:1000558E 81F1FAFAFAFA          xor ecx, FAFAFAFA   

:10005594 8D0489                lea eax, dword ptr [ecx+4*ecx]   

:10005597 8D0480                lea eax, dword ptr [eax+4*eax]   

:1000559A 8D0480                lea eax, dword ptr [eax+4*eax]   

:1000559D 8D1480                lea edx, dword ptr [eax+4*eax]   

:100055A0 C1E204                shl edx, 04   

:100055A3 2BD1                  sub edx, ecx   

:100055A5 8D8416E3140000        lea eax, dword ptr [esi+edx+000014E3] //挤出来第三部分的就是这个 eax 的拉!   

:100055AC 5E                    pop esi   

:100055AD C3                    ret
```

## 后记

这个软件已经是我很久跟的了,不过它确实很好,现在它的版本也升高了.我这人既笨又懒.新的 4.0 注册方法
好象是改了,好象不是通过调用 dll 实现的注册码验证的了.我也不敢去追了,在汇编代码森林里真的好可怕.还是
上街去打望的好,呜呜......

//很久前以写的了，哪时候还有班上，现在我又下岗了。真怀恋有工作的时光

```c
#include<windows.h>   

#include<stdio.h>   

#include<math.h>   

#include<string.h>   

#include<stdlib.h>   

#include<time.h>   

char *point,*point1,charstr[100],covert[100],input_no[100],char1;   

DWORD number,i,first,second,third,random;   

void main(void)   
{ printf("SyGate 3.0 proxy registry http://www.sygate.com \\n");   
  printf("crack by:machoman\\n");   
 printf("please input number \\n");   
 scanf("%s",charstr);   
 getchar();   
 strcpy(input_no,charstr);
```

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

`//这部分 C 是把输入的转换成对应的类 BCD 表示，如果输入'18181818'把它转换为一个 DWORD 的 0x18181818`

```c
  number=lstrlen(input_no);   
          if(number<8)   
     {printf("the  input number  can 't lower 8 NULL\\n");   
      return;   
         }   
     point=&char1;   
     random=rand();     
 for(i=0;i<8;i++)   
     {     
    
       point=&input_no[i];   
        if(((*point)<0x30)|(((*point)>0x39)&&((*point)<0x41))|(((*point)>0x5a)&&((*point)<0x61))|   
     ((*point)>0x66)|(((*point)>0x46)&&((*point)<0x61))|(input_no[0]>0x34))   
     {printf("input must area in 0-9,A-F and first character must lower 4");   
      return;   
     }   
     if(*point>=0x30&&*point<=0x39)   
     {*point=*point-0x30;   
       first=first<<4;   
       first=first|(*point);   
     }   
     else   
       if((*point)>=0x41&&(*point)<=0x46)   
     {*point=*point-0x37;   
       first=first<<4;   
       first=first|(*point);   
     }   
       else   
       if((*point)>=0x61&&(*point)<=0x66)   
     {*point=*point-0x57;   
       first=first<<4;   
       first=first|(*point);   
     }   
           point1=point1+1;   
     }
```

/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/

`  //这部分汇编代码就是在程序中粘贴过来的，我懒！不想写成 C 了`

```c
   _asm{   push  eax   
     push  ebx   
     push  ecx   
     push  edx   
     push  esi   
     push  edi   
     push  ebp   
        mov ecx, first//(BCD)18181818   
        mov eax, ecx   
        xor eax, 0x245143   
        imul eax, ecx   
        imul eax, ecx   
        mov edx, eax   
        shl edx, 0x0B   
        sub edx, eax
```



```assembly
    lea edx, dword ptr [edx+4*edx]
    lea edx, dword ptr [eax+4*edx]
    lea edx, dword ptr [edx+2*edx]
    lea esi, dword ptr [eax+2*edx+0x67A77]
    mov eax, 0x7325A665
    imul esi, ecx
    mul ecx
    imul esi, ecx
    sub esi, 0x00001A6E
    imul esi, ecx
    shr edx, 0x11
    mov eax, edx
    shl eax, 0x08
    add eax, edx
    lea eax, dword ptr [eax+2*eax]
    lea eax, dword ptr [esi+eax+0x11B1]
    mov ecx, eax
    mov edx, ecx
    shr eax, 0x34
    shr edx, 0x28
    lea eax, dword ptr [eax+2*eax]
    lea edx, dword ptr [edx+4*edx]
    add eax, edx
    mov edx, ecx
    shr edx, 0x1C
    shr ecx, 0xE
    lea edi, dword ptr [edx+4*edx]
    lea edx, dword ptr [edx+2*edi]
    add eax, edx
    add eax, ecx
    and eax, 0xFFFFF // 这个值就是后半部分 serial no:的低 4 位，高 4 位可为随机数
    mov second,eax
    mov ecx, first//(BCD)18181818
    mov eax, ecx
    xor eax, 0x94920945
    imul eax, ecx
    imul eax, ecx
    imul eax, ecx
    imul eax, ecx
    lea edx, dword ptr [8*eax+00000000]
    sub edx, eax
    lea edx, dword ptr [eax+4*edx]
    lea esi, dword ptr [eax+8*edx+0x0000561]
    mov eax, 0xC20F4DCF
    mul ecx
    imul esi, ecx
    mov eax, ecx
    sub eax, edx
    shr eax, 1
    add eax, edx
    shr eax, 0x11
    mov edx, eax
    shl edx, 04
    add edx, eax
    lea eax, dword ptr [edx+8*edx]
    lea eax, dword ptr [eax+8*eax]
    add esi, eax
    imul esi, ecx
    sub esi, 0x2B3821C
    imul esi, ecx
    shr ecx, 0x1E
    xor ecx, 0xFAFAFAFA
    lea eax, dword ptr [ecx+4*ecx]
    lea eax, dword ptr [eax+4*eax]
    lea eax, dword ptr [eax+4*eax]
    lea edx, dword ptr [eax+4*eax]
    shl edx, 04
    sub edx, ecx
    lea eax, dword ptr [esi+edx+0x0014E3]
    mov third,eax//这里就是 Registration:
    pop ebp
    pop edi
    pop esi
    pop edx
    pop ecx
    pop ebx
    pop eax
}
```

```c
printf("output of registry key is:\n");
printf("Serial No:%x",first);
printf("-");
printf("%d0",random);
printf("%x\n",second);
printf("Registration:%x\n",third);
}
```

4dd466d0e1e0cc123afbe504da2462ca
