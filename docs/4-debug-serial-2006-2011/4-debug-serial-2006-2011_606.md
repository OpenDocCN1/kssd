
# 我的第一次，写注册机

> 原文：[`www.pediy.com/kssd/pediy12/97480.html`](https://www.pediy.com/kssd/pediy12/97480.html)

【文章标题】: 我的第一次，写注册机【邀请码已发】  

【软件名称】: 房友软件 V9.7  

【保护方式】: ASPack 2.12 -> Alexey Solodovnikov  

【使用工具】: OD, PEID  

【操作平台】: WINDOWS XP SP3  

【作者声明】: 只是感兴趣，练习，没有其他目的。  

首先 `PEID` 查壳,结果为 `ASPack 2.12`，老牌的压缩壳，没什么难度，想怎么搞，随便你了，`ESP` 定律，单步，或者其它的，为了方便，直接弄个脱壳机，这个看雪也有下载，不多说，脱之...命名为 `crack.exe`。  

直接运行 `crack.exe`,无问题，说明脱壳正常。  

接下来，`OD` 载入，`F9` 直接跑起，出现一个小窗口，给出了你硬盘指纹（6 位），提示需要输入 6 位的验证码，OK，随便输，`"123456"`，单击“确定”，提示错误，要的就是这个，切换 `OD`，按“`F12`”，`ALT+K`，右击（一般是）最后一个，选择 `SHOW CALL`，然后直接 `ALT+F9`，这时再跑去点掉刚才的提示，此时 `OD` 停在了用户的代码区，此时走过 2 个 `retn`,再往上翻就看到下面的一段代码。

```
01BB646B    8B02             mov     eax, dword ptr [edx]                ; 硬盘指纹
01BB646D    50               push    eax
01BB646E    FF15 2C10B901    call    dword ptr [0x1B9102C]              ; 获取指纹长度
01BB6474    8B35 9812B901    mov     esi, dword ptr [0x1B91298]         ; msvbvm60.__vbaStrMove
01BB647A    8B3D 2811B901    mov     edi, dword ptr [0x1B91128]         ; msvbvm60.__vbaStrCmp
01BB6480    8B1D 0810B901    mov     ebx, dword ptr [0x1B91008]         ; msvbvm60.__vbaStrI2
01BB6486    8985 7CFFFFFF    mov     dword ptr [ebp-0x84], eax         ; 将长度存入 dword ptr [ebp-0x84]
01BB648C    B8 01000000      mov     eax, 0x1
01BB6491    8945 E8          mov     dword ptr [ebp-0x18], eax
01BB6494    3B85 7CFFFFFF    cmp     eax, dword ptr [ebp-0x84]          ; 开始--根据硬盘指纹获取邀请码.此处为一个 for 循环的开始,
01BB649A    0F8F C9010000    jg      01BB6669                         ; 比如:for(eax =1;eax<=dword ptr [ebp-0x84];eax++)
01BB64A0    8B4D 08          mov     ecx, dword ptr [ebp+0x8]
01BB64A3    8D55 C8          lea     edx, dword ptr [ebp-0x38]
01BB64A6    52               push    edx
01BB64A7    50               push    eax
01BB64A8    894D A0          mov     dword ptr [ebp-0x60], ecx
01BB64AB    8D45 98          lea     eax, dword ptr [ebp-0x68]
01BB64AE    8D4D B8          lea     ecx, dword ptr [ebp-0x48]
01BB64B1    50               push    eax
01BB64B2    51               push    ecx
01BB64B3    C745 D0 0100000> mov     dword ptr [ebp-0x30], 0x1
01BB64BA    C745 C8 0200000> mov     dword ptr [ebp-0x38], 0x2
01BB64C1    C745 98 0840000> mov     dword ptr [ebp-0x68], 0x4008
01BB64C8    FF15 0811B901    call    dword ptr [0x1B91108]              ; 中截取,将指纹中的当前位取出
01BB64CE    8D55 B8          lea     edx, dword ptr [ebp-0x48]
01BB64D1    52               push    edx
01BB64D2    FF15 3410B901    call    dword ptr [0x1B91034]              ; msvbvm60.__vbaStrVarMove
01BB64D8    8BD0             mov     edx, eax
01BB64DA    8D4D E0          lea     ecx, dword ptr [ebp-0x20]
01BB64DD    FFD6             call    esi
01BB64DF    8D45 B8          lea     eax, dword ptr [ebp-0x48]
01BB64E2    8D4D C8          lea     ecx, dword ptr [ebp-0x38]
01BB64E5    50               push    eax
01BB64E6    51               push    ecx
01BB64E7    6A 02            push    0x2
01BB64E9    FF15 4010B901    call    dword ptr [0x1B91040]              ; msvbvm60.__vbaFreeVarList
01BB64EF    8B55 E0          mov     edx, dword ptr [ebp-0x20]
01BB64F2    83C4 0C          add     esp, 0xC
01BB64F5    8D4D 84          lea     ecx, dword ptr [ebp-0x7C]
01BB64F8    FF15 1412B901    call    dword ptr [0x1B91214]              ; msvbvm60.__vbaStrCopy
01BB64FE    8B55 84          mov     edx, dword ptr [ebp-0x7C]         ;以下为判断,根据上面获取的一位数值,获取一对应码
01BB6501    52               push    edx                                 ; 当前数值
01BB6502    68 808AB901      push    01B98A80                           ; 比较码 1
01BB6507    FFD7             call    edi
01BB6509    85C0             test    eax, eax
01BB650B    75 10            jnz     short 01BB651D                     ; 判断这里是不是和 1 相等,如果不是则跳转
01BB650D    6A 04            push    0x4                               ; 如果相等则 返回 4
01BB650F    FFD3             call    ebx
01BB6511    8BD0             mov     edx, eax
01BB6513    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB6516    FFD6             call    esi
01BB6518    E9 1F010000      jmp     01BB663C
01BB651D    8B45 84          mov     eax, dword ptr [ebp-0x7C]         ; dword ptr [ebp-0x7C]中存放的是 当前数值
01BB6520    50               push    eax
01BB6521    68 048BB901      push    01B98B04                          ; 比较码 4
01BB6526    FFD7             call    edi
01BB6528    85C0             test    eax, eax
01BB652A    75 10            jnz     short 01BB653C                     ; 判断这里是不是和 4 相等,如果不是则跳转
01BB652C    6A 07            push    0x7                                ; 如果相等则 返回 7
01BB652E    FFD3             call    ebx
01BB6530    8BD0             mov     edx, eax
01BB6532    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB6535    FFD6             call    esi
01BB6537    E9 00010000      jmp     01BB663C
01BB653C    8B4D 84          mov     ecx, dword ptr [ebp-0x7C]
01BB653F    51               push    ecx
01BB6540    68 9C8BB901      push    01B98B9C                           ; 比较码 7
01BB6545    FFD7             call    edi
01BB6547    85C0             test    eax, eax
01BB6549    75 10            jnz     short 01BB655B
01BB654B    6A 01            push    0x1                                ; 返回 1
01BB654D    FFD3             call    ebx
01BB654F    8BD0             mov     edx, eax
01BB6551    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB6554    FFD6             call    esi
01BB6556    E9 E1000000      jmp     01BB663C
01BB655B    8B55 84          mov     edx, dword ptr [ebp-0x7C]
01BB655E    52               push    edx
01BB655F    68 AC8AB901      push    01B98AAC                            ; 比较码 2
01BB6564    FFD7             call    edi
01BB6566    85C0             test    eax, eax
01BB6568    75 10            jnz     short 01BB657A
01BB656A    6A 05            push    0x5
01BB656C    FFD3             call    ebx
01BB656E    8BD0             mov     edx, eax
01BB6570    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB6573    FFD6             call    esi
01BB6575    E9 C2000000      jmp     01BB663C
01BB657A    8B45 84          mov     eax, dword ptr [ebp-0x7C]
01BB657D    50               push    eax
01BB657E    68 3C8BB901      push    01B98B3C                            ; 比较码 5
01BB6583    FFD7             call    edi
01BB6585    85C0             test    eax, eax
01BB6587    75 10            jnz     short 01BB6599
01BB6589    6A 08            push    0x8
01BB658B    FFD3             call    ebx
01BB658D    8BD0             mov     edx, eax
01BB658F    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB6592    FFD6             call    esi
01BB6594    E9 A3000000      jmp     01BB663C
01BB6599    8B4D 84          mov     ecx, dword ptr [ebp-0x7C]
01BB659C    51               push    ecx
01BB659D    68 E48BB901      push    01B98BE4                            ; 比较码 8
01BB65A2    FFD7             call    edi
01BB65A4    85C0             test    eax, eax
01BB65A6    75 10            jnz     short 01BB65B8
01BB65A8    6A 02            push    0x2
01BB65AA    FFD3             call    ebx
01BB65AC    8BD0             mov     edx, eax
01BB65AE    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB65B1    FFD6             call    esi
01BB65B3    E9 84000000      jmp     01BB663C
01BB65B8    8B55 84          mov     edx, dword ptr [ebp-0x7C]
01BB65BB    52               push    edx
01BB65BC    68 D48AB901      push    01B98AD4                             ; 比较码 3
01BB65C1    FFD7             call    edi
01BB65C3    85C0             test    eax, eax
01BB65C5    75 0D            jnz     short 01BB65D4
01BB65C7    6A 06            push    0x6
01BB65C9    FFD3             call    ebx
01BB65CB    8BD0             mov     edx, eax
01BB65CD    8D4D E4          lea     ecx, dword ptr [ebp-0x1C]
01BB65D0    FFD6             call    esi
```



# 汇编代码分析

```asm
01BB65D2    EB 68              jmp     short 01BB663C  
01BB65D4    8B45 84            mov     eax, dword ptr [ebp-0x7C]  
01BB65D7    50                 push    eax  
01BB65D8    68 708BB901        push    01B98B70               ; 比较码 6  
01BB65DD    FFD7               call    edi  
01BB65DF    85C0               test    eax, eax  
01BB65E1    75 0D              jnz     short 01BB65F0  
01BB65E3    6A 09              push    0x9  
01BB65E5    FFD3               call    ebx  
01BB65E7    8BD0               mov     edx, eax  
01BB65E9    8D4D E4            lea     ecx, dword ptr [ebp-0x1C]  
01BB65EC    FFD6               call    esi  
01BB65EE    EB 4C              jmp     short 01BB663C  
01BB65F0    8B4D 84            mov     ecx, dword ptr [ebp-0x7C]  
01BB65F3    51                 push    ecx  
01BB65F4    68 188CB901        push    01B98C18               ; 比较码 9  
01BB65F9    FFD7               call    edi  
01BB65FB    85C0               test    eax, eax  
01BB65FD    75 0D              jnz     short 01BB660C  
01BB65FF    6A 03              push    0x3  
01BB6601    FFD3               call    ebx  
01BB6603    8BD0               mov     edx, eax  
01BB6605    8D4D E4            lea     ecx, dword ptr [ebp-0x1C]  
01BB6608    FFD6               call    esi  
01BB660A    EB 30              jmp     short 01BB663C  
01BB660C    8B55 84            mov     edx, dword ptr [ebp-0x7C]  
01BB660F    52                 push    edx  
01BB6610    68 A085B901        push    01B985A0               ; 比较码 0  
01BB6615    FFD7               call    edi  
01BB6617    85C0               test    eax, eax  
01BB6619    75 13              jnz     short 01BB662E         ; 如果不为 0 则跳转. 0 的取值最复杂,是根据一系列的运算得出来的,由于本来时间比较紧  
01BB661B    8B45 D8            mov     eax, dword ptr [ebp-0x28]      ; 不作过多尝试,反正也就对应了 10 个数字,我本机,优先出来的是 6,所以在写注册机的时候  
01BB661E    50                 push    eax                ; 我会根据自身数据的判断,写出 0 对应的 数值  
01BB661F    FF15 1810B901      call    dword ptr [0x1B91018]            ; msvbvm60.__vbaStrI4  
01BB6625    8BD0               mov     edx, eax  
01BB6627    8D4D E4            lea     ecx, dword ptr [ebp-0x1C]  
01BB662A    FFD6               call    esi  
01BB662C    EB 0E              jmp     short 01BB663C  
01BB662E    BA A085B901        mov     edx, 01B985A0  
01BB6633    8D4D E4            lea     ecx, dword ptr [ebp-0x1C]  
01BB6636    FF15 1412B901      call    dword ptr [0x1B91214]            ; msvbvm60.__vbaStrCopy  
01BB663C    8B4D DC            mov     ecx, dword ptr [ebp-0x24]  
01BB663F    8B55 E4            mov     edx, dword ptr [ebp-0x1C]  
01BB6642    51                 push    ecx  
01BB6643    52                 push    edx  
01BB6644    FF15 6810B901      call    dword ptr [0x1B91068]            ; 数值连接 就是 把 6 码都连起来  
01BB664A    8BD0               mov     edx, eax  
01BB664C    8D4D DC            lea     ecx, dword ptr [ebp-0x24]  
01BB664F    FFD6               call    esi  
01BB6651    8B4D E8            mov     ecx, dword ptr [ebp-0x18]       ; 取出当前位置 for 语句的循环变量 i  
01BB6654    B8 01000000        mov     eax, 0x1  
01BB6659    03C1               add     eax, ecx                ; i 加 1  
01BB665B    0F80 99000000      jo      01BB66FA  
01BB6661    8945 E8            mov     dword ptr [ebp-0x18], eax  
01BB6664    E9 2BFEFFFF        jmp     01BB6494            ; 这里是前面 for 语句的 回跳部分  
01BB6669    8D4D 98            lea     ecx, dword ptr [ebp-0x68]        ; 结束  
```

# 中间代码省略

```asm
01BB0333    FF15 E010B901      call    dword ptr [0x1B910E0]            ; 01BB0339    8D45 98            lea     eax, dword ptr [ebp-0x68]  
01BB033C    50                 push    eax  
01BB033D    FF15 3410B901      call    dword ptr [0x1B91034]            ; msvbvm60.__vbaStrVarMove  
01BB0343    8BD0               mov     edx, eax  
01BB0345    8D4D D8            lea     ecx, dword ptr [ebp-0x28]  
01BB0348    FF15 9812B901      call    dword ptr [0x1B91298]            ; msvbvm60.__vbaStrMove  
01BB034E    8D4D B8            lea     ecx, dword ptr [ebp-0x48]  
01BB0351    FF15 DC12B901      call    dword ptr [0x1B912DC]            ; msvbvm60.__vbaFreeObj  
01BB0357    8D4D 98            lea     ecx, dword ptr [ebp-0x68]  
01BB035A    51                 push    ecx  
01BB035B    8D55 A8            lea     edx, dword ptr [ebp-0x58]  
01BB035E    52                 push    edx  
01BB035F    6A 02              push    0x2  
01BB0361    FF15 4010B901      call    dword ptr [0x1B91040]            ; msvbvm60.__vbaFreeVarList  
01BB0367    83C4 0C            add     esp, 0xC  
01BB036A    C745 FC 0A00000>   mov     dword ptr [ebp-0x4], 0xA  
01BB0371    8B45 DC            mov     eax, dword ptr [ebp-0x24]  
01BB0374    50                 push    eax                    ; 根据指纹获取的密码压栈"..."  
01BB0375    8B4D D8            mov     ecx, dword ptr [ebp-0x28]  
01BB0378    51                 push    ecx                     ; 客户输入的邀请码"123456"  
01BB0379    FF15 2811B901      call    dword ptr [0x1B91128]            ; 比较注册码  
01BB037F    85C0               test    eax, eax  
01BB0381    0F85 2F020000      jnz     01BB05B6                             ; 关键跳  
01BB0387    C745 FC 0B00000>   mov     dword ptr [ebp-0x4], 0xB  
01BB038E    8D55 C8            lea     edx, dword ptr [ebp-0x38]  
01BB0391    52                 push    edx  
01BB0392    8B45 08            mov     eax, dword ptr [ebp+0x8]  
01BB0395    8B48 38            mov     ecx, dword ptr [eax+0x38]  
01BB0398    51                 push    ecx  
01BB0399    8B55 CC            mov     edx, dword ptr [ebp-0x34]  
01BB039C    8B02               mov     eax, dword ptr [edx]  
01BB039E    8B4D CC            mov     ecx, dword ptr [ebp-0x34]  
```

# 逻辑分析与注册机

根据上面的注释，应该还算清楚了，首先，程序会根据获取的硬盘指纹，一位一位获取对应的“密码”并拼起来。比如是 `1`，则返回 `4`；是 `4` 返回 `7`。10 个数字，除了 `0`，其它都很简单，但为了节省大家和我的时间我们就不去深入研究了，毕竟 `0` 也就对应了键盘上的 10 个数字而已。并且本人多次尝试把本地日期改成不同时间，在断网（懒的开 IRIS）的时候看，`0` 对应的数字是否有变化，结果是一直为 `6`，说明 `0` 的对应码跟日期无关。

根据上面的提示，我把对应关系列一下。

```text
1 4 7 2 5 8 3 6 9 0
4 7 1 5 8 2 6 9 3 未知
```

那好，现在就来写个简单的注册机吧，工具 `VS 2008 C++`

```cpp
int _tmain(int argc, _TCHAR* argv[])
{
  int j=0;
  char chr;
  printf("\n 请输入机器码:\n");
  string s1,result;
  cin>>s1;
  while(s1.length() != 6)
  {
    cout<<"长度必须为 6"<<endl;
    cin>>s1;
  };
  do{
    char c;
    for(int i=0;i<6;i++)
    {
      c = s1[i];
      switch(c)
      {
      case '1':
        result += "4";
        break;
      case '4':
        result += "7";
        break;
      case '7':
        result += "1";
        break;
      case '2':
        result += "5";
        break;
      case '5':
        result += "8";
        break;
      case '8':
        result += "2";
        break;
      case '3':
        result += "6";
        break;
      case '6':
        result += "9";
        break;
      case '9':
        result += "3";
        break;
      case '0':
        switch(j)
        {
        case 0:
          result += "6";
          break;
        case 1:
          result += "5";
          break;
        case 2:
          result += "2";
          break;
        case 3:
          result += "1";
          break;
        case 4:
          result += "3";
          break;
        case 5:
          result += "4";
          break;
        case 6:
          result += "7";
          break;
        case 7:
          result += "8";
          break;
        case 8:
          result += "9";
          break;
        case 9:
          result += "0";
          break;
        }
        break;
      }
    }
    cout<<j<<" 激活码为: "<<result<<endl;
    result.clear();
    cout<<"是否需要继续(y/n)?: ";
    j++;
    cin>>chr;
  }while(chr!='n' && chr!='N' );
  return 0;
}
```

为什么当 `0` 的时候，出现顺序为 `6，5，2，0，1...9`。因为我本机 `0` 出现的是 `6`，后面只是个人喜好![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸") 好了，现在注册机也就有了，缺点就是运行不好的时候，会要运行上 10 次。

至于注释不齐，是因为我 COPY 到记事本里面写的。我的 `OD` 打中文有点问题，所以还请谅解。
