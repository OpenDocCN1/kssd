
# 超级通讯王 1.83 注册简单分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-209.htm`](https://www.pediy.com/kssd/pediy07/pediy7-209.htm)

【软件名称】   超级通讯王 1.83

【加密方式】   序列号/注册码

【破解工具】   `ollydbg`/`ida`

【破解难度】   +++初级+++

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

----------------------------------------------------------------------------------------------

【软件简介】

一款多种功能的优秀个人信息管理软件.

----------------------------------------------------------------------------------------------

【破解过程】

```
0041FB00  >   6A FF           push -1
0041FB02    68 A8BE4D00     push <SuperPIM.unknown_libname_477>
0041FB07    64:A1 00000000   mov eax,dword ptr fs:[0]
0041FB0D    50               push eax
0041FB0E    64:8925 0000000>mov dword ptr fs:[0],esp
0041FB15    83EC 08          sub esp,8
0041FB18    53               push ebx
0041FB19    56               push esi
0041FB1A    57               push edi
0041FB1B    6A 01           push 1
0041FB1D    8BF1            mov esi,ecx
0041FB1F    E8 DC530800     call <SuperPIM.sub_4A4F00>
0041FB24    8D4424 0C       lea eax,dword ptr ss:[esp+C]
0041FB28    50               push eax
0041FB29    E8 322A0400     call <SuperPIM.sub_462560>
0041FB2E    83C4 04          add esp,4
0041FB31    8B4E 74          mov ecx,dword ptr ds:[esi+74]                       ;取输入注册码
0041FB34    8B41 F4          mov eax,dword ptr ds:[ecx-C]
0041FB37    85C0            test eax,eax
0041FB39    8D7E 74          lea edi,dword ptr ds:[esi+74]
0041FB3C    C74424 1C 00000>mov dword ptr ss:[esp+1C],0
0041FB44    0F84 D6000000   je <SuperPIM.loc_41FC20>                            ; 输入注册码长度为 0?
0041FB4A    6A 28           push 28
0041FB4C    8D5424 14       lea edx,dword ptr ss:[esp+14]
0041FB50    57               push edi
0041FB51    52               push edx
0041FB52    E8 B97E0300     call <SuperPIM.sub_457A10>
0041FB57    8B4C24 18       mov ecx,dword ptr ss:[esp+18]
0041FB5B    8B00            mov eax,dword ptr ds:[eax]
0041FB5D    51               push ecx
0041FB5E    50               push eax
0041FB5F    E8 790D0700     call <SuperPIM.__mbscmp>                            ; 比较机器码变换后的字符串
0041FB64    8BD8            mov ebx,eax                                         ; 和输入的注册码变换后的字符串
0041FB66    8B4424 24       mov eax,dword ptr ss:[esp+24]                       ; 相同则注册成功
0041FB6A    83C4 14          add esp,14
0041FB6D    F7DB            neg ebx                                             ;相同则 bl 为 0
0041FB6F    1ADB            sbb bl,bl
0041FB71    83C0 F0          add eax,-10
0041FB74    FEC3            inc bl
0041FB76    8D50 0C          lea edx,dword ptr ds:[eax+C]
0041FB79    83C9 FF          or ecx,FFFFFFFF
0041FB7C    F0:0FC10A       lock xadd dword ptr ds:[edx],ecx
0041FB80    49               dec ecx
0041FB81    85C9            test ecx,ecx
0041FB83    7F 08           jg short <SuperPIM.loc_41FB8D>
0041FB85    8B08            mov ecx,dword ptr ds:[eax]                          ; SuperPIM.0052C3B8
0041FB87    8B11            mov edx,dword ptr ds:[ecx]
0041FB89    50               push eax
0041FB8A    FF52 04          call dword ptr ds:[edx+4]
0041FB8D  >   84DB            test bl,bl
0041FB8F    0F84 8B000000   je <SuperPIM.loc_41FC20>
0041FB95    8D4424 10       lea eax,dword ptr ss:[esp+10]
0041FB99    6A 71           push 71                                             ; 注册成功
0041FB9B    50               push eax
0041FB9C    E8 AF270400     call <SuperPIM.sub_462350>                          ;注册成功的消息框
```

跟进关键 call,`0041FB52    E8 B97E0300     call <SuperPIM.sub_457A10>`

```
00457AD2    8A140A          mov dl,byte ptr ds:[edx+ecx]                       ; strart 待处理的字符串
00457AD5    885424 20       mov byte ptr ss:[esp+20],dl
00457AD9    8B4424 20       mov eax,dword ptr ss:[esp+20]
00457ADD    50               push eax
00457ADE    E8 8DFDFFFF     call <SuperPIM.sub_457870>
00457AE3    33D2            xor edx,edx                                         ;把计算结果转换成可显示字符
00457AE5    B9 2B000000     mov ecx,2B
00457AEA    F7F1            div ecx
00457AEC    83C4 08          add esp,8
00457AEF    8BDA            mov ebx,edx
00457AF1    80C3 30          add bl,30
00457AF4    80FB 39          cmp bl,39
00457AF7    7E 08           jle short <SuperPIM.loc_457B01>
00457AF9    80FB 41          cmp bl,41
00457AFC    7D 03           jge short <SuperPIM.loc_457B01>
00457AFE    80C3 F6          add bl,0F6
00457B01  >   3BEF            cmp ebp,edi
00457B03    7D 3B           jge short <SuperPIM.loc_457B40>
00457B05    8B46 FC          mov eax,dword ptr ds:[esi-4]
00457B08    8B56 F4          mov edx,dword ptr ds:[esi-C]
00457B0B    B9 01000000     mov ecx,1
00457B10    2BC8            sub ecx,eax
00457B12    8B46 F8          mov eax,dword ptr ds:[esi-8]
00457B15    8D7A 01          lea edi,dword ptr ds:[edx+1]
00457B18    2BC7            sub eax,edi
00457B1A    0BC1            or eax,ecx
00457B1C    895424 20       mov dword ptr ss:[esp+20],edx
00457B20    7D 12           jge short <SuperPIM.loc_457B34>                    ;比较循环记数
```

待处理的字符串分别为:硬盘物理序列号+硬盘厂商标识,序列号,输入的注册码等等，通过变换分别得到变换后的字符串.

设转换函数为 `F(source,counter)` 注册验证过程为:

`serial number=F(硬盘物理序列号+硬盘厂商标识),`

`tmpstr1=F(serial number)`

`tmpstr2=F(tmpstr1)`

`tmpstr3=F(input code)`

其中变换函数 `F` 相同，只是其中循环次数不一样。

测试是否 `tmp3=tmp2`，结果为真则注册成功，所以，只要 `input code=tmpstr1(F(serial number))` 就可以了，而这个中间字符串 `tmpstr1` 是以明码状态出现的 ，也就是说这个软件其实还是属于明码比较的.

变换的具体实现:

```
00457870  >   51               push ecx
00457871    0FB64424 08     movzx eax,byte ptr ss:[esp+8]                    ; 字符在串中的序数（由 0 开始）
00457876    894424 00       mov dword ptr ss:[esp],eax
0045787A    8B4424 0C       mov eax,dword ptr ss:[esp+C]
0045787E    8BC8            mov ecx,eax
00457880    0FAFC8          imul ecx,eax
00457883    8D4C49 07       lea ecx,dword ptr ds:[ecx+ecx*2+7]
00457887    0FAFC8          imul ecx,eax
0045788A    83C1 0D          add ecx,0D
0045788D    0FAFC8          imul ecx,eax                                      ;预计算结果
00457890    56               push esi
00457891    8D70 05          lea esi,dword ptr ds:[eax+5]
00457894    56               push esi
00457895    8D5424 10       lea edx,dword ptr ss:[esp+10]
00457899    6A 04           push 4
0045789B    52               push edx
0045789C    894C24 18       mov dword ptr ss:[esp+18],ecx
004578A0    E8 CBFEFFFF     call <SuperPIM.sub_457770>
004578A5    56               push esi
004578A6    8D4424 14       lea eax,dword ptr ss:[esp+14]
004578AA    6A 04           push 4
004578AC    50               push eax
004578AD    E8 3EFFFFFF     call <SuperPIM.sub_4577F0>
004578B2    8B4424 24       mov eax,dword ptr ss:[esp+24]
004578B6    8B4C24 1C       mov ecx,dword ptr ss:[esp+1C]
004578BA    83C4 18          add esp,18
004578BD    33C1            xor eax,ecx
004578BF    5E               pop esi
004578C0    59               pop ecx
004578C1    C3               retn

004577D0  >   57               push edi                                      ;外循环
004577D1    53               push ebx
004577D2    E8 09FFFFFF     call <SuperPIM.sub_4576E0>
004577D7    83C4 08          add esp,8
004577DA    4E               dec esi
004577DB   ^ 75 F3           jnz short <SuperPIM.loc_4577D0>                ; 初始为 5,按照序数递增

004576E0  >   8B4424 08       mov eax,dword ptr ss:[esp+8]                  ;内循环,次数为固定值:4
004576E4    56               push esi
004576E5    8B7424 08       mov esi,dword ptr ss:[esp+8]
004576E9    33D2            xor edx,edx
004576EB    8A5406 FF       mov dl,byte ptr ds:[esi+eax-1]                 
004576EF    8BCE            mov ecx,esi
```



# 汇编分析与逆向工程

## 循环 1 汇编代码

```
004576F1    81E2 80000000   and edx,80
004576F7    85C0            test eax,eax
004576F9    8BF2            mov esi,edx
004576FB    74 22           je short <SuperPIM.loc_45771F>
004576FD    57              push edi
004576FE    8BF8            mov edi,eax
00457700 >  8A01            mov al,byte ptr ds:[ecx]                    ;
00457702    33D2            xor edx,edx
00457704    8AD0            mov dl,al
00457706    D0E0            shl al,1
00457708    8801            mov byte ptr ds:[ecx],al
0045770A    81E2 80000000   and edx,80
00457710    85F6            test esi,esi
00457712    74 04           je short <SuperPIM.loc_457718>
00457714    0C 01           or al,1                                       ;
00457716    8801            mov byte ptr ds:[ecx],al
00457718 >  41              inc ecx
00457719    4F              dec edi
0045771A    8BF2            mov esi,edx
0045771C  ^ 75 E2          jnz short <SuperPIM.loc_457700>
0045771E    5F              pop edi                                       ;
0045771F >  5E              pop esi
00457720    C3              retn
```

以上为循环 1，设这个结果为 `tmp1`。

## 循环 2 相关汇编

### 外部循环

```
00457850 >  57              push edi                                      ;外循环
00457851    53              push ebx
00457852    E8 D9FEFFFF     call <SuperPIM.sub_457730>
00457857    83C4 08         add esp,8
0045785A    4E              dec esi                                       ; 初始循环 5 次，按照字符在串中序数依次加 1
0045785B  ^ 75 F3          jnz short <SuperPIM.loc_457850>
0045785D >  5F              pop edi
0045785E    5E              pop esi
0045785F    5B              pop ebx
00457860 >  C3              retn
```

### 内部循环

```
00457730 >  8B4C24 04       mov ecx,dword ptr ss:[esp+4]                ;内循环，次数为固定值:4
00457734    33C0            xor eax,eax
00457736    8A01            mov al,byte ptr ds:[ecx]
00457738    56              push esi
00457739    83E0 01         and eax,1
0045773C    8BF0            mov esi,eax
0045773E    8B4424 0C       mov eax,dword ptr ss:[esp+C]
00457742    85C0            test eax,eax
00457744    8D4C01 FF       lea ecx,dword ptr ds:[ecx+eax-1]
00457748    74 22           je short <SuperPIM.loc_45776C>
0045774A    57              push edi
0045774B    8BF8            mov edi,eax
0045774D    8D49 00         lea ecx,dword ptr ds:[ecx]
00457750 >  8A01            mov al,byte ptr ds:[ecx]                    ;
00457752    33D2            xor edx,edx
00457754    8AD0            mov dl,al
00457756    D0E8            shr al,1
00457758    8801            mov byte ptr ds:[ecx],al
0045775A    83E2 01         and edx,1
0045775D    85F6            test esi,esi
0045775F    74 04           je short <SuperPIM.loc_457765>
00457761    0C 80           or al,80
00457763    8801            mov byte ptr ds:[ecx],al
00457765 >  49              dec ecx
00457766    4F              dec edi                                       ; 循环 4 次
00457767    8BF2            mov esi,edx
00457769  ^ 75 E5          jnz short <SuperPIM.loc_457750>
0045776B    5F              pop edi
0045776C >  5E              pop esi
0045776D    C3              retn
```

以上为循环 2，设结果为 `tmp2`。

## 结果合并

```
004578B2    8B4424 24       mov eax,dword ptr ss:[esp+24]               ;tmp1
004578B6    8B4C24 1C       mov ecx,dword ptr ss:[esp+1C]               ;tmp2
004578BA    83C4 18         add esp,18
004578BD    33C1            xor eax,ecx                                   ;tmp1 xor tmp2
004578BF    5E              pop esi
004578C0    59              pop ecx
004578C1    C3              retn
```

## 结果转换为可显示字符

```
00457AE3    33D2            xor edx,edx
00457AE5    B9 2B000000     mov ecx,2B
00457AEA    F7F1            div ecx
00457AEC    83C4 08         add esp,8
00457AEF    8BDA            mov ebx,edx
00457AF1    80C3 30         add bl,30
00457AF4    80FB 39         cmp bl,39
00457AF7    7E 08           jle short <SuperPIM.loc_457B01>
00457AF9    80FB 41         cmp bl,41
00457AFC    7D 03           jge short <SuperPIM.loc_457B01>
00457AFE    80C3 F6         add bl,0F6
00457B01 >  3BEF            cmp ebp,edi
00457B03    7D 3B           jge short <SuperPIM.loc_457B40>
```

这就是对待处理字符串中一个字符进行处理的完整过程。这两段循环计算开始看时候很乱，花了很久才搞明白是怎么回事，其实就是很简单的 `dword` 乘除计算。

归根到底还是自己的汇编基础太差了，对汇编中的一些常用计算都不能很快识别。

我说的也不清楚，具体的实现，大家还是自己看下面的注册机代码吧。

## 注册机代码

```cpp
//---------------------key.cpp---------------------
#include <iostream>
#include <string>
using namespace std;

//循环 1
long change1(char input, int count)
{
    long tmpCode;
    int flag;
    tmpCode = count - 5;
    tmpCode *= tmpCode;
    tmpCode = tmpCode * 3 + 7;
    tmpCode *= (count - 5);
    tmpCode += 0xd;
    tmpCode *= (count - 5);
    //tmpCode *= pow(2,count);
    for(int i =0; i < count;i++)
    {
        flag = 0;
        if(tmpCode>=0x80000000)
        {
            flag = 1;
        }
        tmpCode *=2;
        if(flag)
        {
            tmpCode += 1;
        }
    }
    return tmpCode;
}

//循环 2
long change2(char input, int count)
{
    unsigned long tmp,tmpCode;
    bool flag;
    tmp = input;
    tmpCode = 0;
    while(count)
    {
        flag = false;
        if(tmp & 1)
        {
            flag = true;
        }
        tmp /=2;
        tmpCode /=2;
        if(flag)
        {
            tmpCode |= 0x80000000;
        }
        count --;
    }
    tmpCode += tmp;
    return tmpCode;
}

//把计算结果转换成可显示字符
long change(char input, int count)
{
    unsigned long code,tmp;
    code = change1(input, count) ^ change2(input, count);
    tmp = code % 0x2b;
    tmp += 0x30;
    if((tmp > 0x39) && (tmp <0x41) )
    {
        tmp = (tmp + 0xf6) - 0x100;
    }
    code = tmp;
    return code;
}

int main()
{
    char *strMcode,*tmpStr,*code;
    strMcode = new char [20];
    tmpStr = new char [40];
    code = new char [21];
    cout << "Serial Number:";
    cin >> strMcode;
    cout << endl;
    strcpy(tmpStr,strMcode);
    while(strlen( tmpStr ) < 20)
    {
        strcat(tmpStr,strMcode);
    }
    for(int i = 0,j = 5; i < 20 ; i++)
    {
        code[i] = change(tmpStr[i], j);
        j++;
    }
    code[i] = '\0';
    cout << "Reg Code:" << code << endl;
    cout << "Press any key to exit." << endl;
    cin.get();  cin.get();
    delete [] strMcode;
    delete [] tmpStr;
    delete [] code;
    return 0;
}
//-------------------key.cpp----------------------------
```

## 例子

serial number:

```
BZIR21250R4LKPF
```

reg code:

```
3WDNL0511KM52E5W063Y
```

----------------------------------------------------------

by ikki[D.4s]

`4dd466d0e1e0cc123afbe504da2462ca`
