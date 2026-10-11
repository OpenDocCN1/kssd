
# QuickCHM2.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50469.htm`](https://www.pediy.com/kssd/pediy05/pediy50469.htm)

\*\*\* The Tutorial For QuickCHM ver 2.01 By LostInUnASM 2003.3.6 \*\*\*

**声明：**

今天想用 `QuickCHM` 把以前搜集的材料整理一下，才发现由于没有注册，不能编辑和编译大于 15 个节点的项目 :(于是决定自己动手丰衣足食 ![](img/ed938650fb57c1336c009c22f429c285.jpg) 在这里先向吴新兵老兄陪个不是：呵呵，大哥你也喜欢用 `DELPHI` 写程式，小弟也一样，有空可以交流经验 ![](img/ed938650fb57c1336c009c22f429c285.jpg)这个 `QuickCHM` 的版本好象不是最新的吧，不知道你有何新的研究成果 ![](img/ed938650fb57c1336c009c22f429c285.jpg)

**程序：** `QuickChM` 2.01

**类型：** `Delphi` Aspack

**工具：** `W32Dasm`、`ProDump`、`Dede`、`TRW2000` //高手完全可以不用那么多吧

**分析：** 先用 `PRODUMP` 脱壳，再用 `W32DASM` 反汇编，结合 `DEDE` 可以找到具体比较注册码的关键地方是

```
:004F246D 8B4334                   mov eax, dword ptr [ebx+34]
:004F2470 8B55F8                   mov edx, dword ptr [ebp-08]
:004F2473 E83429F1FF               call 00404DAC  ==》这里是具体比较的 CALL LstrCmp Call
:004F2478 0F85BA000000             jne 004F2538    ==》在这里修改就可以暴力破解了 ：P
```

不过只要你重新启动程序，依然会出现 `UnRegister` 的字样，SO 继续跟踪程序启动时调用了哪些过程

**发现：**

```
* Possible StringData Ref from Code Obj -> "F0E1"  ==》注意：这是一个很有用的字符串
                                     |
:004EDC9D BAB8E04E00                 mov edx, 004EE0B8
:004EDCA2 8BC7                       mov eax, edi   
:004EDCA4 E8DBABFFFF                 call 004E8884  ==》这是一个特别重要的 CALL 哦！
:004EDCA9 8BC7                       mov eax, edi
:004EDCAB E8CCAEFFFF                 call 004E8B7C  ==》这也很重要哦
:004EDCB0 8B4734                     mov eax, dword ptr [edi+34]  ==》存储正确的注册码
:004EDCB3 8B55F0                     mov edx, dword ptr [ebp-10]
:004EDCB6 E8F170F1FF                 call 00404DAC ==》显然这个 CALL 和上面那个所调用的过程 a 一样
:004EDCBB 7532                       jne 004EDCEF  ==》修改这里就可去除 UnRegister 的字样了
```

**用户名：** `L o s t I n U n A s m`

**注册码：** `64 F6 56 8C 30 3F 9D B7 17 1E A1 4A`

使用 `TRW` 在 `004EDCB0` 处设端点，再按了无数个 `F10` 后，终于有所发现。

**跟踪后发现其计算规律如下：**

1. 首字母的 `ASCII` 值 `X` 加 100 除 255 得余数 `X1`，`X1` 再 `XOR` 字符串 `F0E1` 中相对应的第一个字母的 `ASCII` 值得出 数 `X2` 再把该数转换为 16 进制就可以了。
2. 把第二字母的 `ASCII` 值 `Y` 加上 `X2`，取得 `Y1`  `Y1` 再 `XOR` 字符串 `F0E1` 中想对应的第二个字母的 `ASCII` 值得出 数 `X3` 再把该数转换为 16 进制就可以了。 依次类推.

`EG:` `Name:` `F` `U` `C` `K` `Key:` `68487498` (随便输入)

```
      70     85      67      75
  Add  100   236     114     240
    ---------------------------------
      170    321     181     315
  Mod  255   255     255     255   
    --------------------------------- 
      170     66     181     60
  XOR    70     48      69     49   ====>70 48 69 49 分别为 F0E1 的 ASCII 值
    ---------------------------------
      236    114     240     13
 InttoHex   EC      72      F0     0D

      70    48    69    49   ===》正确的注册码是：  64EC72F00D  (不要漏了‘64’哦)
      F     0     E     1  
```

于是顺利地写出了注册机，用 `DELPHI` 6.0

```pascal
program KeygenForQChm;  

{$APPTYPE CONSOLE}  

Uses sysutils;  

Var  
   Name,XorStr,Key:String;  
   LastModResult,Mr:Integer;  
   StrLen,I,J,IntXor:Integer;  

begin  
 writeln('===========================================================');  
 writeln('This is a Keygen For QuickCHM ver 2.01   ----By LostInUnAsm');  
 writeln('===========================================================');  
 Write('Please Enter your Name(Length>=4):');  
 LastModResult:=100;  
 XorStr:='F0E1';  
 J:=1;  
 Read(Name);  
 StrLen:=Length(Name);

 For I:=1 to StrLen do  
    Begin  
       Mr:=(Ord(Name[i])+LastModResult) Mod 255;  
       IntXor:=Mr Xor Ord(XorStr[J]);  
       Key:=Key+InttoHex(IntXor,2);  
       LastModResult:=IntXor;  
       If J<4 then Inc(J) Else J:=1;  
    End;  
 Writeln('Correct Serial Is: 64',Key);  //注意，每个注册码都是以 64 开头的 ![](img/ed938650fb57c1336c009c22f429c285.jpg)  
 Readln;  
 Readln;  
end.
```

\===================================忙了好几个小时！累死我了！=============================

`PS`：破解完全不用拘泥于形式，如果你想简单得到注册码，完全可以使用 `WINHEX`，  
 用她来查看 `QuickCHM` 的内存，几秒就可以找到正确的注册码（好象是根据谁的理论，正确的注册码一定在错误的注册码附近） `4dd466d0e1e0cc123afbe504da2462ca`
