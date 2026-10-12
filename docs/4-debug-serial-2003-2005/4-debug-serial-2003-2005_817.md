
# Windows 设置大师 2005 版本:6.30 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-124.htm`](https://www.pediy.com/kssd/pediy07/pediy7-124.htm)

Windows 设置大师 2005 版本:6.30 算法分析

日期：2005 年 3 月 11 日 破解人：Baby2008

----------------------------------------

【软件名称】：Windows 设置大师 2005 版本:6.30

【软件大小】：8.7M

【下载地址】：`http://www.szds88.com/showsoft.asp?soft_id=9` (注册后必须进行正版升级?)

【软件限制】：未注册 30 天试用限制

【破解声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：`OLLYDBG 1.1` ,`PEID0.92`

----------------------------------------

【破解过程】：

先用 `PEID 0.93` 汉化增强版查壳，`ASPack 2.12` -> `Alexey Solodovnikov`，用 `PEID 0.93` 插件 `PEiD Generic UNpacker` 轻松脱去，默认另存为

`WinSSM.exe.unpacked\_.exe`，脱壳后再用 `PEID 0.93` 汉化增强版查壳，`Borland Delphi 6.0 - 7.0`，不用重建输入表就能直接运行。真巧，和我前两天练手的《时间提醒助手》一样^\_^,(更巧的是，跟了一下注册算法后发现，连算法都差不多，真的有这么巧?不会是同一个作者吧?我从程序风格上觉得比时间提醒助手强多了!) `delphi` 程序，先用 `Dede` 处理，输出 `map` 文件，用 `od` 打开目标程序：`WinSSM.exe.unpacked\_.exe` ，用 `mapconv` 插件引入 `Dede` 输出的 `map` 文件，替换标签及插件。然后查找字符，可以找到“注册码不正确...”等字样，双击来到 `cpu` 窗口，来到 `00507E43` ,向上翻看可以找到这里：

```
000507DE0 <>/. 55 push ebp ; <-TForm4@suiButton3Click
00507DE1 |. 8BEC mov ebp,esp
00507DE3 |. 33C9 xor ecx,ecx
00507DE5 |. 51 push ecx
00507DE6 |. 51 push ecx
00507DE7 |. 51 push ecx
00507DE8 |. 51 push ecx
00507DE9 |. 53 push ebx
00507DEA |. 56 push esi
00507DEB |. 8BD8 mov ebx,eax
00507DED |. 33C0 xor eax,eax
00507DEF |. 55 push ebp
00507DF0 |. 68 F77E5000 push <WinSSM_e.->System.@HandleFinally;>
00507DF5 |. 64:FF30 push dword ptr fs:[eax]
00507DF8 |. 64:8920 mov dword ptr fs:[eax],esp
00507DFB |. 8D55 F8 lea edx,dword ptr ss:[ebp-8]
00507DFE <>|. 8B83 00030000 mov eax,dword ptr ds:[ebx+300] ; \*TForm4.suiEdit6:TsuiEdit
00507E04 <>|. E8 BFD4F3FF call WinSSM_e.004452C8 ;
\->Controls.TControl.GetText(TControl):TCaption;
00507E09 |. 8B45 F8 mov eax,dword ptr ss:[ebp-8] ; 试炼码
00507E0C |. 50 push eax
00507E0D |. 8D55 F4 lea edx,dword ptr ss:[ebp-C]
00507E10 <>|. 8B83 18030000 mov eax,dword ptr ds:[ebx+318] ; \*TForm4.suiEdit1:TsuiEdit
00507E16 <>|. E8 ADD4F3FF call WinSSM_e.004452C8 ;
\->Controls.TControl.GetText(TControl):TCaption;
00507E1B |. 8B45 F4 mov eax,dword ptr ss:[ebp-C]
00507E1E |. 50 push eax
00507E1F |. 8D55 F0 lea edx,dword ptr ss:[ebp-10]
00507E22 <>|. 8B83 FC020000 mov eax,dword ptr ds:[ebx+2FC] ; \*TForm4.suiEdit4:TsuiEdit
00507E28 <>|. E8 9BD4F3FF call WinSSM_e.004452C8 ;
\->Controls.TControl.GetText(TControl):TCaption;
00507E2D |. 8B55 F0 mov edx,dword ptr ss:[ebp-10] ; 注册姓名
00507E30 <>|. 8B83 20030000 mov eax,dword ptr ds:[ebx+320] ; \*TForm4.RegwareII:TRegwareII
00507E36 |. 59 pop ecx
00507E37 <>|. E8 D4ECFFFF call WinSSM_e.00506B10 ; 关键，跟进
00507E3C |. 84C0 test al,al
00507E3E |. 75 2C jnz short WinSSM_e.00507E6C
00507E40 |. 8D45 FC lea eax,dword ptr ss:[ebp-4]
00507E43 |. BA 0C7F5000 mov edx,WinSSM_e.00507F0C ; 注册码不正确
00507E48 <>|. E8 C7CBEFFF call WinSSM_e.00404A14 ; ->System.@LStrLAsg(void;void;void;void);
00507E4D |. 6A 40 push 40
call WinSSM_e.00506B10
```

----------------------------------------

```
00506B10 /$ 55 push ebp
00506B11 |. 8BEC mov ebp,esp
00506B13 |. 83C4 F0 add esp,-10
00506B16 |. 53 push ebx
00506B17 |. 33DB xor ebx,ebx
00506B19 |. 895D F0 mov dword ptr ss:[ebp-10],ebx
00506B1C |. 895D F4 mov dword ptr ss:[ebp-C],ebx
00506B1F |. 894D F8 mov dword ptr ss:[ebp-8],ecx
00506B22 |. 8955 FC mov dword ptr ss:[ebp-4],edx ; 注册名
00506B25 |. 8BD8 mov ebx,eax
00506B27 |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
00506B2A |. E8 09E3EFFF call WinSSM_e.00404E38
00506B2F |. 8B45 F8 mov eax,dword ptr ss:[ebp-8]
00506B32 |. E8 01E3EFFF call WinSSM_e.00404E38
00506B37 |. 8B45 08 mov eax,dword ptr ss:[ebp+8] ; 试炼码
00506B3A |. E8 F9E2EFFF call WinSSM_e.00404E38
00506B3F |. 33C0 xor eax,eax
00506B41 |. 55 push ebp
00506B42 |. 68 FA6B5000 push WinSSM_e.00506BFA
00506B47 |. 64:FF30 push dword ptr fs:[eax]
00506B4A |. 64:8920 mov dword ptr fs:[eax],esp
00506B4D |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
00506B50 |. E8 FBE0EFFF call WinSSM_e.00404C50
00506B55 |. 3B43 4C cmp eax,dword ptr ds:[ebx+4C]
00506B58 |. 7F 19 jg short WinSSM_e.00506B73
00506B5A |. 8B45 FC mov eax,dword ptr ss:[ebp-4]
00506B5D |. E8 EEE0EFFF call WinSSM_e.00404C50
00506B62 |. 3B43 50 cmp eax,dword ptr ds:[ebx+50]
00506B65 |. 7C 0C jl short WinSSM_e.00506B73
00506B67 |. 8B45 08 mov eax,dword ptr ss:[ebp+8]
00506B6A |. E8 E1E0EFFF call WinSSM_e.00404C50
00506B6F |. 85C0 test eax,eax
00506B71 |. 75 04 jnz short WinSSM_e.00506B77
00506B73 |> 33DB xor ebx,ebx
00506B75 |. EB 60 jmp short WinSSM_e.00506BD7
00506B77 |> 8D55 F4 lea edx,dword ptr ss:[ebp-C]
00506B7A |. 8B45 08 mov eax,dword ptr ss:[ebp+8]
00506B7D |. E8 8E23F0FF call WinSSM_e.00408F10
00506B82 |. 8B55 F4 mov edx,dword ptr ss:[ebp-C]
00506B85 |. 8D45 08 lea eax,dword ptr ss:[ebp+8]
00506B88 |. E8 87DEEFFF call WinSSM_e.00404A14
00506B8D |. 8D4D F0 lea ecx,dword ptr ss:[ebp-10]
00506B90 |. 8B55 FC mov edx,dword ptr ss:[ebp-4]
00506B93 |. 8BC3 mov eax,ebx
00506B95 |. E8 BAFBFFFF call WinSSM_e.00506754 ；关键,跟进
00506B9A |. 8B45 F0 mov eax,dword ptr ss:[ebp-10]
00506B9D |. 8B55 08 mov edx,dword ptr ss:[ebp+8]
00506BA0 |. E8 E323F0FF call WinSSM_e.00408F88
00506BA5 |. 85C0 test eax,eax
00506BA7 |. 74 04 je short WinSSM_e.00506BAD
00506BA9 |. 33DB xor ebx,ebx
00506BAB |. EB 2A jmp short WinSSM_e.00506BD7
00506BAD |> 8D43 48 lea eax,dword ptr ds:[ebx+48]
00506BB0 |. 8B55 FC mov edx,dword ptr ss:[ebp-4]
00506BB3 |. E8 18DEEFFF call WinSSM_e.004049D0
00506BB8 |. 8D43 54 lea eax,dword ptr ds:[ebx+54]
00506BBB |. 8B55 F8 mov edx,dword ptr ss:[ebp-8]
00506BBE |. E8 0DDEEFFF call WinSSM_e.004049D0
00506BC3 |. 8D43 5C lea eax,dword ptr ds:[ebx+5C]
00506BC6 |. 8B55 08 mov edx,dword ptr ss:[ebp+8]
00506BC9 |. E8 02DEEFFF call WinSSM_e.004049D0
00506BCE |. 8BC3 mov eax,ebx
00506BD0 |. E8 DF010000 call WinSSM_e.00506DB4
```



```
00506BD5   |.  B3 01               mov bl,1  

00506BD7   |>  33C0               xor eax,eax  

00506BD9   |.  5A                pop edx  

00506BDA   |.  59                pop ecx  

00506BDB   |.  59                pop ecx  

00506BDC   |.  64:8910          mov dword ptr fs:[eax],edx  

00506BDF   |.  68 016C5000      push WinSSM_e.00506C01  

00506BE4   |>  8D45 F0          lea eax,dword ptr ss:[ebp-10]  

00506BE7   |.  BA 04000000      mov edx,4  

00506BEC   |.  E8 AFDDEFFF      call WinSSM_e.004049A0  

00506BF1   |.  8D45 08          lea eax,dword ptr ss:[ebp+8]  

00506BF4   |.  E8 83DDEFFF      call WinSSM_e.0040497C  

00506BF9   \.  C3               retn  

———————————————————————————————————————————  

call WinSSM_e.00506754  

———————————————————————————————————————————  

00506754   /$  55               push ebp  

00506755   |.  8BEC             mov ebp,esp  

00506757   |.  83C4 E0          add esp,-20  

0050675A   |.  53               push ebx  

0050675B   |.  56               push esi  

0050675C   |.  57               push edi  

0050675D   |.  33DB             xor ebx,ebx  

0050675F   |.  895D EC          mov dword ptr ss:[ebp-14],ebx                 ;  试炼码  

00506762   |.  895D E8          mov dword ptr ss:[ebp-18],ebx  

00506765   |.  895D E4          mov dword ptr ss:[ebp-1C],ebx  

00506768   |.  895D E0          mov dword ptr ss:[ebp-20],ebx  

0050676B   |.  8BF9             mov edi,ecx  

0050676D   |.  8955 FC          mov dword ptr ss:[ebp-4],edx                ;  注册姓名  

00506770   |.  8BF0             mov esi,eax  

00506772   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00506775   |.  E8 BEE6EFFF      call WinSSM_e.00404E38  

0050677A   |.  33C0             xor eax,eax  

0050677C   |.  55               push ebp  

0050677D   |.  68 ED685000      push WinSSM_e.005068ED  

00506782   |.  64:FF30          push dword ptr fs:[eax]  

00506785   |.  64:8920          mov dword ptr fs:[eax],esp  

00506788   |.  8D45 EC          lea eax,dword ptr ss:[ebp-14]  

0050678B   |.  BA 04695000      mov edx,WinSSM_e.00506904                   ;  ASCII "gf258369gf"  

00506790   |.  E8 7FE2EFFF      call WinSSM_e.00404A14  

00506795   |.  837D EC 00       cmp dword ptr ss:[ebp-14],0  

00506799   |.  75 0D            jnz short WinSSM_e.005067A8  

0050679B   |.  8D45 E0          lea eax,dword ptr ss:[ebp-20]  

0050679E   |.  8B55 FC          mov edx,dword ptr ss:[ebp-4]  

005067A1   |.  E8 6EE2EFFF      call WinSSM_e.00404A14  

005067A6   |.  EB 5D            jmp short WinSSM_e.00506805  

005067A8   |>  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

005067AB   |.  E8 A0E4EFFF      call WinSSM_e.00404C50  

005067B0   |.  8BD8             mov ebx,eax  

005067B2   |.  8D45 E8          lea eax,dword ptr ss:[ebp-18]  

005067B5   |.  50               push eax  

005067B6   |.  8BCB             mov ecx,ebx  

005067B8   |.  D1F9             sar ecx,1  

005067BA   |.  79 03            jns short WinSSM_e.005067BF  

005067BC   |.  83D1 00          adc ecx,0  

005067BF   |>  BA 01000000      mov edx,1  

005067C4   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

005067C7   |.  E8 DCE6EFFF      call WinSSM_e.00404EA8  

005067CC   |.  8D45 E4          lea eax,dword ptr ss:[ebp-1C]               ;  取'gf258369gf'前 5 位  

005067CF   |.  50               push eax  

005067D0   |.  8BC3             mov eax,ebx  

005067D2   |.  D1F8             sar eax,1  

005067D4   |.  79 03            jns short WinSSM_e.005067D9  

005067D6   |.  83D0 00          adc eax,0  

005067D9   |>  8BCB             mov ecx,ebx  

005067DB   |.  2BC8             sub ecx,eax  

005067DD   |.  8BD3             mov edx,ebx  

005067DF   |.  D1FA             sar edx,1  

005067E1   |.  79 03            jns short WinSSM_e.005067E6  

005067E3   |.  83D2 00          adc edx,0  

005067E6   |>  42               inc edx  

005067E7   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

005067EA   |.  E8 B9E6EFFF      call WinSSM_e.00404EA8  

005067EF   |.  FF75 E8          push dword ptr ss:[ebp-18]                 ;  前 5 位  

005067F2   |.  FF75 FC          push dword ptr ss:[ebp-4]                  ;  注册姓名  

005067F5   |.  FF75 E4          push dword ptr ss:[ebp-1C]                 ;  后 5 位  

005067F8   |.  8D45 E0          lea eax,dword ptr ss:[ebp-20]  

005067FB   |.  BA 03000000      mov edx,3  

00506800   |.  E8 0BE5EFFF      call WinSSM_e.00404D10  

00506805   |>  C745 F0 00000000 mov dword ptr ss:[ebp-10],0  

0050680C   |.  C745 F4 00000000 mov dword ptr ss:[ebp-C],0  

00506813   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00506816   |.  E8 35E4EFFF      call WinSSM_e.00404C50  

0050681B   |.  3B46 4C          cmp eax,dword ptr ds:[esi+4C]  

0050681E   |.  7F 0D            jg short WinSSM_e.0050682D  

00506820   |.  8B45 FC          mov eax,dword ptr ss:[ebp-4]  

00506823   |.  E8 28E4EFFF      call WinSSM_e.00404C50  

00506828   |.  3B46 50          cmp eax,dword ptr ds:[esi+50]  

0050682B   |.  7D 0C            jge short WinSSM_e.00506839  

0050682D   |>  8BC7             mov eax,edi  

0050682F   |.  E8 48E1EFFF      call WinSSM_e.0040497C  

00506834   |.  E9 91000000      jmp WinSSM_e.005068CA  

00506839   |>  8B45 E0          mov eax,dword ptr ss:[ebp-20]              ;  Name=gf258+注册姓名+369gf  

0050683C   |.  E8 0FE4EFFF      call WinSSM_e.00404C50  

00506841   |.  8BD8             mov ebx,eax                                ;  求 Name 长度  

00506843   |.  EB 37            jmp short WinSSM_e.0050687C  

00506845   |>  8B46 68          /mov eax,dword ptr ds:[esi+68]             ;  常数 1B382F6B 记为 Key  

00506848   |.  8B56 6C          |mov edx,dword ptr ds:[esi+6C]             ;  0  

0050684B   |.  0345 F0          |add eax,dword ptr ss:[ebp-10]             ;  Key+Serail  

0050684E   |.  1355 F4          |adc edx,dword ptr ss:[ebp-C]              ;  带进位加法  

00506851   |.  52               |push edx                                   ;  入栈  

00506852   |.  50               |push eax  

00506853   |.  8B45 E0          |mov eax,dword ptr ss:[ebp-20]              ;  Name=gf258+注册姓名+369gf  

00506856   |.  0FB64418 FF      |movzx eax,byte ptr ds:[eax+ebx-1]          ;  取字符 Name[i]  

0050685B   |.  50               |push eax  

0050685C   |.  B8 59040000      |mov eax,459                                ;  常数 459  

00506861   |.  5A               |pop edx                                    ;  Name[i]  

00506862   |.  8BCA             |mov ecx,edx  

00506864   |.  33D2             |xor edx,edx  

00506866   |.  F7F1             |div ecx                                    ;  459/Name[i]  

00506868       8BC2             mov eax,edx                                 ;  取余  

0050686A   |.  33D2             |xor edx,edx  

0050686C   |.  290424           |sub dword ptr ss:[esp],eax                 ;  Key+Serail-459/Name[i]取余  

0050686F   |.  195424 04        |sbb dword ptr ss:[esp+4],edx  

00506873   |.  58               |pop eax  

00506874   |.  5A               |pop edx  

00506875   |.  8945 F0          |mov dword ptr ss:[ebp-10],eax              ;  Key+Serail-459/Name[i]取余  

00506878   |.  8955 F4          |mov dword ptr ss:[ebp-C],edx  

0050687B   |.  4B               |dec ebx                                    ;  i=i-1  

0050687C   |>  8B45 E0          mov eax,dword ptr ss:[ebp-20]              ;  fg258+注册姓名+369gf  

0050687F   |.  E8 CCE3EFFF      |call WinSSM_e.00404C50                    ;  求 Name 长度
```



```
00506884   |.   3BD8                |cmp ebx,eax  
00506886   |.   7F 04               |jg short WinSSM_e.0050688C  
00506888   |.   85DB               |test ebx,ebx  
0050688A   |.^ 7F B9               \jg short WinSSM_e.00506845  
0050688C   |>   8B5E 60            mov ebx,dword ptr ds:[esi+60]  
0050688F   |.   85DB               test ebx,ebx  
00506891        7F 11               jg short WinSSM_e.005068A4                    ; 输出 Serail 的 10 位 16 进制字符串即为注册码  
00506893   |.   FF75 F4            push dword ptr ss:[ebp-C]                      ; /Arg2  
00506896   |.   FF75 F0            push dword ptr ss:[ebp-10]                      ; |Arg1  
00506899   |.   8BD7               mov edx,edi                                     ; |  
0050689B   |.   33C0               xor eax,eax                                     ; |  
0050689D   |.   E8 7A2BF0FF         call WinSSM_e.0040941C                        ; \WinSSM_e.0040941C  
005068A2   |.   EB 26               jmp short WinSSM_e.005068CA  
005068A4   |>   FF75 F4            push dword ptr ss:[ebp-C]                      ; /Arg2  
005068A7   |.   FF75 F0            push dword ptr ss:[ebp-10]                      ; |Arg1  
005068AA   |.   8BD7               mov edx,edi                                     ; |  
005068AC   |.   8BC3               mov eax,ebx                                     ; |  
005068AE   |.   E8 692BF0FF         call WinSSM_e.0040941C                        ; \  
005068B3   |.   8B07               mov eax,dword ptr ds:[edi]  
005068B5   |.   E8 96E3EFFF         call WinSSM_e.00404C50                      ；出现注册码,可制作内存注册机  
005068BA   |.   8BC8               mov ecx,eax  
005068BC   |.   2B4E 60            sub ecx,dword ptr ds:[esi+60]  
005068BF   |.   8B56 60            mov edx,dword ptr ds:[esi+60]  
005068C2   |.   42                 inc edx  
005068C3   |.   8BC7               mov eax,edi  
005068C5   |.   E8 1EE6EFFF         call WinSSM_e.00404EE8  
005068CA   |>   33C0               xor eax,eax  
005068CC   |.   5A                 pop edx  
005068CD   |.   59                 pop ecx  
005068CE   |.   59                 pop ecx  
005068CF   |.   64:8910             mov dword ptr fs:[eax],edx  
005068D2   |.   68 F4685000         push WinSSM_e.005068F4  
005068D7   |>   8D45 E0            lea eax,dword ptr ss:[ebp-20]  
005068DA   |.   BA 04000000         mov edx,4  
005068DF   |.   E8 BCE0EFFF         call WinSSM_e.004049A0  
005068E4   |.   8D45 FC            lea eax,dword ptr ss:[ebp-4]  
005068E7   |.   E8 90E0EFFF         call WinSSM_e.0040497C  
005068EC   \\.   C3                   retn  
```

### 【Crack 总结】

作者的算法不难，且明码比较，但注册成功仅提示"注册信息通过验证"，需要到网站上进行正版升级。

算法用到两个常数：

`Key＝$1B382F6B;`

`NameMark='gf258369gf';`

1. 将注册姓名字符串插入到 `NameMark` 中，记为 `Name=gf258+注册姓名+369gf`。设注册码 `Serail=0`
2. 循环从 `Name` 后面开始取字符，记为 `Name[i]`
3. 常数 `Serail＝Key＋Serail－$459/Name[i]` 的 `ASCII` 值取余数
4. 循环 `Name` 结束
5. 将 `Serail` 转换为 10 位 16 进制字符串即为注册码。

贴出注册机 delphi 源码：

```delphi
Procedure TForm1.btn1Click(Sender: TObject);

Const
  Key: Integer = $1B382F6B;

Var
  Serail: Int64;
  i: Integer;
  RegName: String;

Begin
  RegName := 'gf258' + edt1.Text + '369gf';
  Serail := 0;
  For i := 1 To Length(RegName) Do
  Begin
    Serail := Key + Serail - $459 Mod Ord(RegName[Length(RegName) - i + 1]);
    edt2.Text := IntToHex(Serail, 10);
  End;
End;
```

内存注册机：

中断地址:`005068B5`
中断次数:`1`
第一字节:`E8`
指令长度:`5`
注册码:`内存方式 寄存器 EAX`

大家可以验证一下：

注册姓名：`baby2008`
激活密码：`01E9F352A3`

第 3 次写破文了，与上次写的差不多，本不想写的，只是觉的太巧了（前文提及），而且个人觉得这软件附带了很多好用的小工具，有用的着的地方，所以决定继续谢丑。

### 【遗留问题】

按作者说明，注册验证通过后需到网站上进行正版升级，不知不升级会怎样，我把时间调后 3 个月也能正常使用，没有提示 30 天的试用期之类的。

`4dd466d0e1e0cc123afbe504da2462ca`
