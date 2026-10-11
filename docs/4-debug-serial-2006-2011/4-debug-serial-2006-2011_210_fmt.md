
# 谁不想理财

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-11.htm`](https://www.pediy.com/kssd/pediy08/pediy8-11.htm)

## 基本信息

【破文作者】你说呢?\[感谢 `DFCG` 的学习环境。  
【软件名称】谁不想理财！  
【破解工具】`PEID` `OD`  
【软件限制】60 天试用  
【破解平台】`Win9x/NT/2000/XP/XP SP2`  

【文章简介】
力求多学习点汇编代码，培养反汇编思维的能力。

---

## 破解过程

1. 启动软件，任意输入注册码后注册，提示“注册失败。请确认输入的注册码无误”。
   
   有点门道感觉可以使用对话框类 `API` 拦截。

2. 启动 `PEID` 侦察，没有伏击，显示 `Microsoft Visual C++ 6.0`。动用算法扫描插件。
   
   没有发现间谍类。省了一道手续。感谢自信的作者。（反正我是没能攻克他的心理防线^\_^）

3. 启动 `OD`。打开可执行文件。插件－超级字符参考。发现许多中文字符（比乱码好看许多^\_^）。其中有一句”注册成功。谢谢你使用本软件。“ 双击后来到这里：

```
0043EBF6   .   8B7D DC       MOV EDI,DWORD PTR SS:[EBP-24]
0043EBF9   .   8B4D D4       MOV ECX,DWORD PTR SS:[EBP-2C]
0043EBFC   .   6A 40         PUSH 40
0043EBFE   .   68 888D4500   PUSH FamFinan.00458D88              ;  谁不想理财！
0043EC03   .   68 30A34500   PUSH FamFinan.0045A330              ;  注册成功。谢谢你使用本软件。
0043EC08   .   C687 E4000000>MOV BYTE PTR DS:[EDI+E4],1
0043EC0F   .   C687 D0000000>MOV BYTE PTR DS:[EDI+D0],1
0043EC16   .   E8 173B0000   CALL <JMP.&MFC42.#4224_?MessageBoxA@CWnd>
0043EC1B   .   8D85 40FAFFFF LEA EAX,DWORD PTR SS:[EBP-5C0]
0043EC21   .   68 FF000000   PUSH 0FF                  ; /BufSize = FF (255.)
0043EC26   .   50            PUSH EAX                  ; |Buffer
0043EC27   .   FF15 CCA14400 CALL DWORD PTR DS:[<&KERNEL32.GetSystemD>; \GetSystemDirectoryA
```

想了解算法就一致向上拉到代码起始处：

```
0043E760   $   55            PUSH EBP                  ;  (初始 cpu 选择)
0043E761   .   8BEC          MOV EBP,ESP
0043E763   .   6A FF        PUSH -1
0043E765   .   68 739C4400   PUSH FamFinan.00449C73    ;  SE 处理程序安装
0043E76A   .   64:A1 0000000>MOV EAX,DWORD PTR FS:[0]
0043E770   .   50            PUSH EAX
0043E771   .   64:8925 00000>MOV DWORD PTR FS:[0],ESP
0043E778   .   81EC B4050000 SUB ESP,5B4
0043E77E   .   53            PUSH EBX
0043E77F   .   56            PUSH ESI
0043E780   .   57            PUSH EDI
0043E781   .   8BF9          MOV EDI,ECX
0043E783   .   8965 F0       MOV DWORD PTR SS:[EBP-10],ESP
0043E786   .   897D D4       MOV DWORD PTR SS:[EBP-2C],EDI
0043E789   .   8B87 AC040000 MOV EAX,DWORD PTR DS:[EDI+4AC]    ;  将注册码地址放到 EAX
0043E78F   .   8B40 F8       MOV EAX,DWORD PTR DS:[EAX-8]      ;  将注册码位数放到 EAX
0043E792   .   85C0          TEST EAX,EAX
0043E794   .   0F84 08060000 JE FamFinan.0043EDA2               ;  如果为空则跳走
0043E79A   .   E8 DF3E0000   CALL <JMP.&MFC42.#1168_?AfxGetModuleStat>
0043E79F   .   8B70 04       MOV ESI,DWORD PTR DS:[EAX+4]
0043E7A2   .   8D4D C4       LEA ECX,DWORD PTR SS:[EBP-3C]
0043E7A5   .   8975 DC       MOV DWORD PTR SS:[EBP-24],ESI
0043E7A8   .   E8 E1400000   CALL <JMP.&MFC42.#354_??0CFile@@QAE@XZ>
0043E7AD   .   8D86 E8000000 LEA EAX,DWORD PTR DS:[ESI+E8]
0043E7B3   .   68 708D4500   PUSH FamFinan.00458D70            ;  下面打开\famfinan.bik
0043E7B8   .   8D4D EC       LEA ECX,DWORD PTR SS:[EBP-14]
0043E7BB   .   50            PUSH EAX
0043E7BC   .   51            PUSH ECX
0043E7BD   .   C745 FC 00000>MOV DWORD PTR SS:[EBP-4],0
0043E7C4   .   E8 553E0000   CALL <JMP.&MFC42.#924_??H@YG?AVCString@@>
0043E7C9   .   8B00          MOV EAX,DWORD PTR DS:[EAX]
```

。。。。。中间检测文件中字符名称和版本号。

```
0043E969   .   8B97 AC040000 MOV EDX,DWORD PTR DS:[EDI+4AC]    ;  注册码地址入 EDX
0043E96F   .   8B45 BC       MOV EAX,DWORD PTR SS:[EBP-44]
0043E972   .   8DB7 AC040000 LEA ESI,DWORD PTR DS:[EDI+4AC]
0043E978   .   8B4A F8       MOV ECX,DWORD PTR DS:[EDX-8]      ;  注册码位数去 ECX
0043E97B   .   3BC1          CMP EAX,ECX
0043E97D   .   74 0E         JE SHORT FamFinan.0043E98D        ;  与 23 位比较。相等就跳
0043E97F   .   66:C787 B2040>MOV WORD PTR DS:[EDI+4B2],2
0043E988   .   E9 F6030000   JMP FamFinan.0043ED83
0043E98D   >   83C0 10       ADD EAX,10                        ;  加 10
0043E990   .   99            CDQ                               ;  双字扩展把 EAX 扩展到 EDX 中去
0043E991   .   83E2 0F       AND EDX,0F                        ;  EDX 与 0F 做与运算
0043E994   .   03C2          ADD EAX,EDX                       ;  EAX 相加 EDX
0043E996   .   C1F8 04       SAR EAX,4                         ;  右移 4 位
0043E999   .   8BD8          MOV EBX,EAX                       ;  移动到 EBX
0043E99B   .   8945 E8       MOV DWORD PTR SS:[EBP-18],EAX
0043E99E   .   C1E3 04       SHL EBX,4                         ;  EBX2 进制值左移 4 位
0043E9A1   .   53            PUSH EBX                          ;  入栈
0043E9A2   .   E8 653C0000   CALL <JMP.&MFC42.#823_??2@YAPAXI@Z>
0043E9A7   .   53            PUSH EBX                          ;  再入
0043E9A8   .   8945 EC       MOV DWORD PTR SS:[EBP-14],EAX
0043E9AB   .   E8 5C3C0000   CALL <JMP.&MFC42.#823_??2@YAPAXI@Z>
0043E9B0   .   8BF8          MOV EDI,EAX
0043E9B2   .   53            PUSH EBX
0043E9B3   .   897D D8       MOV DWORD PTR SS:[EBP-28],EDI
0043E9B6   .   E8 513C0000   CALL <JMP.&MFC42.#823_??2@YAPAXI@Z>
0043E9BB   .   83C4 0C       ADD ESP,0C                        ;  堆碱指针加 12
0043E9BE   .   8945 E0       MOV DWORD PTR SS:[EBP-20],EAX
0043E9C1   .   85DB          TEST EBX,EBX
0043E9C3   .   7E 12         JLE SHORT FamFinan.0043E9D7      ;  若小于或等于则跳
0043E9C5   .   8BCB          MOV ECX,EBX                       ;  ECX=20
0043E9C7   .   33C0          XOR EAX,EAX
0043E9C9   .   8BD1          MOV EDX,ECX                       ;  EDX=20
0043E9CB   .   C1E9 02       SHR ECX,2                         ;  右移 2 位
0043E9CE   .   F3:AB         REP STOS DWORD PTR ES:[EDI]
0043E9D0   .   8BCA          MOV ECX,EDX
0043E9D2   .   83E1 03       AND ECX,3                         ;  和 3 做与运算
0043E9D5   .   F3:AA         REP STOS BYTE PTR ES:[EDI]
0043E9D7   >   8B3E          MOV EDI,DWORD PTR DS:[ESI]        ;  移动注册码地址到 EDI
0043E9D9   .   83C9 FF       OR ECX,FFFFFFFF                   ;  或运算
0043E9DC   .   33C0          XOR EAX,EAX
0043E9DE   .   F2:AE         REPNE SCAS BYTE PTR ES:[EDI]
0043E9E0   .   F7D1          NOT ECX                           ;  取反
0043E9E2   .   2BF9          SUB EDI,ECX                       ;  相减
0043E9E4   .   8BC1          MOV EAX,ECX
0043E9E6   .   8BF7          MOV ESI,EDI
0043E9E8   .   8B7D D8       MOV EDI,DWORD PTR SS:[EBP-28]
0043E9EB   .   C1E9 02       SHR ECX,2                         ;  右移 2
0043E9EE   .   F3:A5         REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>;  双字节重复
0043E9F0   .   8BC8          MOV ECX,EAX
0043E9F2   .   83E1 03       AND ECX,3                         ;  与 3 运算
0043E9F5   .   33C0          XOR EAX,EAX
0043E9F7   .   F3:A4         REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>;  单字节重复
0043E9F9   .   C785 C0FEFFFF>MOV DWORD PTR SS:[EBP-140],FamFinan.0044>
0043EA03   >   888405 40FEFF>MOV BYTE PTR SS:[EBP+EAX-1C0],AL
0043EA0A   .   40            INC EAX                           ;  自加 1
0043EA0B   .   3D 80000000   CMP EAX,80                        ;  80 个常用字符 0-}移动过来
0043EA10   .^  7C F1         JL SHORT FamFinan.0043EA03        ;  小于 80H 重复
0043EA12   .   8D8D 40FEFFFF LEA ECX,DWORD PTR SS:[EBP-1C0]    ;  移动过来字符的 0 位置
0043EA18   .   68 80000000   PUSH 80                           ;  入
0043EA1D   .   51            PUSH ECX                          ;  入
0043EA1E   .   8D8D C0FEFFFF LEA ECX,DWORD PTR SS:[EBP-140]
0043EA24   .   E8 37250000   CALL FamFinan.00440F60           －－－－这里挺特别 进去看看
0043EA29   .   8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]
0043EA2C   .   85C0          TEST EAX,EAX
```



```
0043EA2E   .   7E 27            JLE SHORT FamFinan.0043EA57  

0043EA30   .   8B75 D8         MOV ESI,DWORD PTR SS:\[EBP-28\]  

0043EA33   .   8B7D E0         MOV EDI,DWORD PTR SS:\[EBP-20\]  

0043EA36   .   2BFE          SUB EDI,ESI  

0043EA38   .   8945 E8       MOV DWORD PTR SS:\[EBP-18\],EAX  

0043EA3B   >   8D1437        LEA EDX,DWORD PTR DS:\[EDI+ESI\]  

0043EA3E   .   8D8D C0FEFFFF LEA ECX,DWORD PTR SS:\[EBP-140\]  

0043EA44   .   52            PUSH EDX                               ; /Arg2  
0043EA45   .   56            PUSH ESI                               ; |Arg1  
0043EA46   .   E8 95260000  CALL FamFinan.004410E0                 ; \\FamFinan.004410E0  
0043EA4B   .   8B45 E8       MOV EAX,DWORD PTR SS:\[EBP-18\]  

0043EA4E   .   83C6 10       ADD ESI,10  

0043EA51   .   48            DEC EAX  

0043EA52   .   8945 E8       MOV DWORD PTR SS:\[EBP-18\],EAX  

0043EA55   .^  75 E4         JNZ SHORT FamFinan.0043EA3B  

CALL FamFinan.00440F60 ：－－－这里面把我搞晕了。  

00440F60  /$  53            PUSH EBX  

00440F61  |.  8BD9          MOV EBX,ECX  

00440F63  |.  8B4C24 0C     MOV ECX,DWORD PTR SS:\[ESP+C\]  

00440F67  |.  C1E9 05       SHR ECX,5                              ;  右移 5 位  

00440F6A  |.  83F9 04       CMP ECX,4  

00440F6D  |.  0F82 62010000 JB FamFinan.004410D5                   ;  小于转移  

00440F73  |.  83F9 0E       CMP ECX,0E                             ;  大于转移  

00440F76  |.  0F87 59010000 JA FamFinan.004410D5  

00440F7C  |.  55            PUSH EBP                               ;  基址  

00440F7D  |.  56            PUSH ESI                               ;  源址  

00440F7E  |.  85C9          TEST ECX,ECX  

00440F80  |.  57            PUSH EDI                               ;  目的变址  

00440F81  |.  76 18         JBE SHORT FamFinan.00440F9B  

00440F83  |.  8B5424 14     MOV EDX,DWORD PTR SS:\[ESP+14\]  

00440F87  |.  B8 74AF4500   MOV EAX,FamFinan.0045AF74  

00440F8C  |.  2BD0          SUB EDX,EAX  

00440F8E  |.  8BF1          MOV ESI,ECX  

00440F90  |>  8B3C02        /MOV EDI,DWORD PTR DS:\[EDX+EAX\]      ;  两者地址送 EDI  
00440F93  |.  8938          |MOV DWORD PTR DS:\[EAX\],EDI  

00440F95  |.  83C0 04       |ADD EAX,4  

00440F98  |.  4E            |DEC ESI  

00440F99  |.^  75 F5        \\JNZ SHORT FamFinan.00440F90  

00440F9B  |>  8D41 01       LEA EAX,DWORD PTR DS:\[ECX+1\]  

00440F9E  |.  890C8D 74AF45>MOV DWORD PTR DS:\[ECX\*4+45AF74\],ECX  

00440FA5  |.  83F8 0F       CMP EAX,0F  

00440FA8  |.  73 12         JNB SHORT FamFinan.00440FBC  

00440FAA  |.  B9 0F000000   MOV ECX,0F  

00440FAF  |.  8D3C85 74AF45>LEA EDI,DWORD PTR DS:\[EAX\*4+45AF74\]  

00440FB6  |.  2BC8          SUB ECX,EAX  

00440FB8  |.  33C0          XOR EAX,EAX  

00440FBA  |.  F3:AB         REP STOS DWORD PTR ES:\[EDI\]  

00440FBC  |>  33D2          XOR EDX,EDX  

00440FBE  |.  8D4B 04       LEA ECX,DWORD PTR DS:\[EBX+4\]  

00440FC1  |>  33C0          /XOR EAX,EAX  

00440FC3  |>  8BB0 40AD4500 |/MOV ESI,DWORD PTR DS:\[EAX+45AD40\]    ;  OD 入 ESI 源变址  
00440FC9  |.  8BB8 7CAD4500 ||MOV EDI,DWORD PTR DS:\[EAX+45AD7C\]    ;  08 入 EDI 目的变址  
00440FCF  |.  8B34B5 74AF45>||MOV ESI,DWORD PTR DS:\[ESI\*4+45AF74\]  
00440FD6  |.  8B2CBD 74AF45>||MOV EBP,DWORD PTR DS:\[EDI\*4+45AF74\]  
00440FDD  |.  33F5          ||XOR ESI,EBP  
00440FDF  |.  8BA8 74AF4500 ||MOV EBP,DWORD PTR DS:\[EAX+45AF74\]  
00440FE5  |.  C1C6 03       ||ROL ESI,3                              ;  循环左移 3 位  
00440FE8  |.  8D3C10        ||LEA EDI,DWORD PTR DS:\[EAX+EDX\]  
00440FEB  |.  83C0 04       ||ADD EAX,4  
00440FEE  |.  33F7          ||XOR ESI,EDI  
00440FF0  |.  33EE          ||XOR EBP,ESI                            ;  异或运算  
00440FF2  |.  83F8 3C       ||CMP EAX,3C  
00440FF5  |.  89A8 70AF4500 ||MOV DWORD PTR DS:\[EAX+45AF70\],EBP  
00440FFB  |.^  7C C6        |\\JL SHORT FamFinan.00440FC3  

00440FFD  |.  BE 04000000   |MOV ESI,4  
00441002  |>  33C0          |/XOR EAX,EAX  
00441004  |>  8BB8 04AD4500 ||/MOV EDI,DWORD PTR DS:\[EAX+45AD04\]  
0044100A  |.  8BA8 74AF4500 |||MOV EBP,DWORD PTR DS:\[EAX+45AF74\]  
00441010  |.  83C0 04       |||ADD EAX,4  
00441013  |.  8B3CBD 74AF45>|||MOV EDI,DWORD PTR DS:\[EDI\*4+45AF74\]  
0044101A  |.  81E7 FF010000 |||AND EDI,1FF  
00441020  |.  8B3CBD 04A545>|||MOV EDI,DWORD PTR DS:\[EDI\*4+45A504\]  
00441027  |.  03FD          |||ADD EDI,EBP  
00441029  |.  C1C7 09       |||ROL EDI,9  
0044102C  |.  89B8 70AF4500 |||MOV DWORD PTR DS:\[EAX+45AF70\],EDI  
00441032  |.  83F8 3C       |||CMP EAX,3C  
00441035  |.^  7C CD        ||\\JL SHORT FamFinan.00441004  
00441037  |.  4E            ||DEC ESI  
00441038  |.^  75 C8        |\\JNZ SHORT FamFinan.00441002  

0044103A  |.  B8 B8AD4500   |MOV EAX,FamFinan.0045ADB8  
0044103F  |>  8B30          |/MOV ESI,DWORD PTR DS:\[EAX\]  
00441041  |.  83C0 04       ||ADD EAX,4  
00441044  |.  83C1 04       ||ADD ECX,4  
00441047  |.  3D E0AD4500   ||CMP EAX,FamFinan.0045ADE0  
0044104C  |.  8B34B5 74AF45>||MOV ESI,DWORD PTR DS:\[ESI\*4+45AF74\]  
00441053  |.  8971 FC       ||MOV DWORD PTR DS:\[ECX-4\],ESI  
00441056  |.^  7C E7        |\\JL SHORT FamFinan.0044103F  

00441058  |.  42            |INC EDX  
00441059  |.  83FA 04       |CMP EDX,4  
0044105C  |.^  0F82 5FFFFFFF \\JB FamFinan.00440FC1  

00441062  |.  8D73 18       LEA ESI,DWORD PTR DS:\[EBX+18\]  
00441065  |.  BF 10000000   MOV EDI,10  
0044106A  |>  8B0E          /MOV ECX,DWORD PTR DS:\[ESI\]  
0044106C  |.  8BD1          |MOV EDX,ECX  
0044106E  |.  83CA 03       |OR EDX,3  
00441071  |.  8BC2          |MOV EAX,EDX  
00441073  |.  8BDA          |MOV EBX,EDX  
00441075  |.  F7D0          |NOT EAX  
00441077  |.  25 FFFFFFFF   |AND EAX,7FFFFFFF  
0044107C  |.  D1EB          |SHR EBX,1  
0044107E  |.  33C3          |XOR EAX,EBX  
00441080  |.  8BD8          |MOV EBX,EAX  
00441082  |.  8BE8          |MOV EBP,EAX  
00441084  |.  C1EB 02       |SHR EBX,2  
00441087  |.  D1ED          |SHR EBP,1  
00441089  |.  23DD          |AND EBX,EBP  
0044108B  |.  23C3          |AND EAX,EBX  
0044108D  |.  8BD8          |MOV EBX,EAX  
0044108F  |.  8BE8          |MOV EBP,EAX  
00441091  |.  C1EB 06       |SHR EBX,6  
00441094  |.  C1ED 03       |SHR EBP,3  
00441097  |.  23DD          |AND EBX,EBP  
00441099  |.  23C3          |AND EAX,EBX  
0044109B  |.  74 2D         |JE SHORT FamFinan.004410CA  
0044109D  |.  D1E0          |SHL EAX,1  
0044109F  |.  83E1 03       |AND ECX,3  
004410A2  |.  8D1C00        |LEA EBX,DWORD PTR DS:\[EAX+EAX\]  
004410A5  |.  0BC3          |OR EAX,EBX  
004410A7  |.  8D1C85 000000>|LEA EBX,DWORD PTR DS:\[EAX\*4\]  
004410AE  |.  0BC3          |OR EAX,EBX  
004410B0  |.  8B1C8D F4AD45>|MOV EBX,DWORD PTR DS:\[ECX\*4+45ADF4\]  
004410B7  |.  8B4E FC       |MOV ECX,DWORD PTR DS:\[ESI-4\]  
004410BA  |.  D3C3          |ROL EBX,CL  
004410BC  |.  8BC8          |MOV ECX,EAX  
004410BE  |.  C1E1 04       |SHL ECX,4  
004410C1  |.  0BC8          |OR ECX,EAX  
004410C3  |.  23D9          |AND EBX,ECX  
004410C5  |.  83E3 FC       |AND EBX,FFFFFFFC  
004410C8  |.  33D3          |XOR EDX,EBX  
004410CA  |>  8916          |MOV DWORD PTR DS:\[ESI\],EDX  
004410CC  |.  83C6 08       |ADD ESI,8  
004410CF  |.  4F            |DEC EDI  
004410D0  |.^  75 98        \\JNZ SHORT FamFinan.0044106A  
004410D2  |.  5F            POP EDI  
004410D3  |.  5E            POP ESI  
004410D4  |.  5D            POP EBP  
004410D5  |>  5B            POP EBX  
004410D6  \\ . C2 0800      RETN 8  
0043EA68   .   C745 E8 00000>MOV DWORD PTR SS:\[EBP-18\],0  
0043EA6F   .   7E 27         JLE SHORT FamFinan.0043EA98  
0043EA71   >   8B7D EC       MOV EDI,DWORD PTR SS:\[EBP-14\]          ;  (初始 cpu 选择)  
0043EA74   .   53            PUSH EBX  
0043EA75   .   57            PUSH EDI  
0043EA76   .   8D4D C4       LEA ECX,DWORD PTR SS:\[EBP-3C\]  
0043EA79   .   E8 F83D0000   CALL <JMP.&MFC42.#5442\_?Read@CFile@@UAEI>
```



## 汇编代码段 1

```
0043EA7E   .  8B4D BC       MOV ECX,DWORD PTR SS:[EBP-44]
0043EA81   .  8B75 E0       MOV ESI,DWORD PTR SS:[EBP-20]
0043EA84   .  33D2         XOR EDX,EDX
0043EA86   .  F3:A6        REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS:>;  应该是查表的地方。我没搞明白呢
0043EA88     74 28        JE SHORT FamFinan.0043EAB2            ;  这里爆破后，可以提示注册成功。但有时间
0043EA8A   .  8B45 E8     MOV EAX,DWORD PTR SS:[EBP-18]        限制
0043EA8D   .  8B4D B8     MOV ECX,DWORD PTR SS:[EBP-48]
0043EA90   .  40          INC EAX
0043EA91   .  3BC1        CMP EAX,ECX
0043EA93   .  8945 E8     MOV DWORD PTR SS:[EBP-18],EAX
0043EA96   .^ 7C D9       JL SHORT FamFinan.0043EA71
0043EA98   >  8B45 D4     MOV EAX,DWORD PTR SS:[EBP-2C]
0043EA9B   .  6A 00       PUSH 0
0043EA9D   .  6A 01       PUSH 1
0043EA9F   .  6A 02       PUSH 2
0043EAA1   .  66:C780 B2040>MOV WORD PTR DS:[EAX+4B2],2
0043EAAA   .  8B40 20     MOV EAX,DWORD PTR DS:[EAX+20]
0043EAAD   .  E9 DA020000   JMP FamFinan.0043ED8C
0043EAB2   >  8B4D EC     MOV ECX,DWORD PTR SS:[EBP-14]
0043EAB5   .  51          PUSH ECX
0043EAB6   .  E8 2D3B0000   CALL <JMP.&MFC42.#825_??3@YAXPAX@Z>
```

---

## 破解心得

本想努力把这个软件拿下并编辑好后发的，可因为算法功底不够。再加上年前需要帮家人干活，只好仓促先发出来。希望会的朋友。给指点一下。。

---

## 声明与版权

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享:)

**【版权声明】** 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

文章写于 2006-01-16 11:55:07

---

## 汇编代码段 2

```
0043E760     55           push ebp
0043E761     8BEC         mov ebp,esp
0043E763     6A FF        push -1
0043E765     68 739C4400  push FamFinan.00449C73
0043E76A     64:A1 00000000 mov eax,dword ptr fs:[0]
0043E770     50           push eax
0043E771     64:8925 00000000 mov dword ptr fs:[0],esp
0043E778     81EC B4050000 sub esp,5B4
0043E77E     53           push ebx
0043E77F     56           push esi
0043E780     57           push edi
0043E781     8BF9         mov edi,ecx
0043E783     8965 F0      mov dword ptr ss:[ebp-10],esp
0043E786     897D D4      mov dword ptr ss:[ebp-2C],edi
0043E789     8B87 AC040000 mov eax,dword ptr ds:[edi+4AC]
0043E78F     8B40 F8      mov eax,dword ptr ds:[eax-8]
0043E792     85C0         test eax,eax
0043E794     0F84 08060000 je FamFinan.0043EDA2
0043E79A     E8 DF3E0000  call <jmp.&MFC42.#1168_AfxGetModuleState>
0043E79F     8B70 04      mov esi,dword ptr ds:[eax+4]
0043E7A2     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E7A5     8975 DC      mov dword ptr ss:[ebp-24],esi
0043E7A8     E8 E1400000  call <jmp.&MFC42.#354_CFile::CFile>
0043E7AD     8D86 E8000000 lea eax,dword ptr ds:[esi+E8]
0043E7B3     68 708D4500  push FamFinan.00458D70                  ; ASCII "\FamFinan.bik"
0043E7B8     8D4D EC      lea ecx,dword ptr ss:[ebp-14]
0043E7BB     50           push eax
0043E7BC     51           push ecx
0043E7BD     C745 FC 00000000 mov dword ptr ss:[ebp-4],0
0043E7C4     E8 553E0000  call <jmp.&MFC42.#924_operator+>
0043E7C9     8B00         mov eax,dword ptr ds:[eax]
0043E7CB     6A 00        push 0
0043E7CD     6A 00        push 0
0043E7CF     50           push eax
0043E7D0     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E7D3     C645 FC 01   mov byte ptr ss:[ebp-4],1
0043E7D7     E8 A6400000  call <jmp.&MFC42.#5186_CFile::Open>      ; 打开 FamFinan.bik 文件
0043E7DC     85C0         test eax,eax
0043E7DE     8D4D EC      lea ecx,dword ptr ss:[ebp-14]
0043E7E1     C645 FC 00   mov byte ptr ss:[ebp-4],0
0043E7E5     0F94C3       sete bl
0043E7E8     E8 CB3D0000  call <jmp.&MFC42.#800_CString::~CString>
0043E7ED     84DB         test bl,bl
0043E7EF     74 18        je short FamFinan.0043E809
0043E7F1     8B57 20      mov edx,dword ptr ds:[edi+20]
0043E7F4     6A 00        push 0
0043E7F6     6A 01        push 1
0043E7F8     6A 02        push 2
0043E7FA     66:C787 B2040000 >mov word ptr ds:[edi+4B2],1
0043E803     52           push edx
0043E804     E9 84050000  jmp FamFinan.0043ED8D
0043E809     8D45 E4      lea eax,dword ptr ss:[ebp-1C]
0043E80C     6A 04        push 4
0043E80E     50           push eax
0043E80F     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E812     E8 5F400000  call <jmp.&MFC42.#5442_CFile::Read>    ; 读入四个字节去缓冲区 1a 00 00 00
0043E817     8B4D E4      mov ecx,dword ptr ss:[ebp-1C]
0043E81A     8D95 40FBFFFF lea edx,dword ptr ss:[ebp-4C0]
0043E820     51           push ecx
0043E821     52           push edx
0043E822     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E825     E8 4C400000  call <jmp.&MFC42.#5442_CFile::Read>    ; 再次读入：长度(第一次读入的数值)
0043E82A     8B45 E4      mov eax,dword ptr ss:[ebp-1C]
0043E82D     8D4D E4      lea ecx,dword ptr ss:[ebp-1C]
0043E830     6A 04        push 4
0043E832     51           push ecx
0043E833     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E836     C68405 40FBFFFF 0>mov byte ptr ss:[ebp+eax-4C0],0
0043E83E     E8 33400000  call <jmp.&MFC42.#5442_CFile::Read>    ; 读入长度值:0c 00 00 00
0043E843     8B55 E4      mov edx,dword ptr ss:[ebp-1C]
0043E846     8D85 40FDFFFF lea eax,dword ptr ss:[ebp-2C0]
0043E84C     52           push edx
0043E84D     50           push eax
0043E84E     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E851     E8 20400000  call <jmp.&MFC42.#5442_CFile::Read>
0043E856     8B4D E4      mov ecx,dword ptr ss:[ebp-1C]
0043E859     8D55 E4      lea edx,dword ptr ss:[ebp-1C]
0043E85C     6A 04        push 4
0043E85E     52           push edx
0043E85F     C6840D 40FDFFFF 0>mov byte ptr ss:[ebp+ecx-2C0],0
0043E867     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E86A     E8 07400000  call <jmp.&MFC42.#5442_CFile::Read>
0043E86F     8B45 E4      mov eax,dword ptr ss:[ebp-1C]
0043E872     8D8D 40FCFFFF lea ecx,dword ptr ss:[ebp-3C0]
0043E878     50           push eax
0043E879     51           push ecx
0043E87A     8D4D C4      lea ecx,dword ptr ss:[ebp-3C]
0043E87D     E8 F43F0000  call <jmp.&MFC42.#5442_CFile::Read>
0043E882     8B55 E4      mov edx,dword ptr ss:[ebp-1C]
0043E885     BE 3C8D4500  mov esi,FamFinan.00458D3C               ; ASCII "k blueice e use y file 1.0"
0043E88A     8D85 40FBFFFF lea eax,dword ptr ss:[ebp-4C0]
0043E890     C68415 40FCFFFF 0>mov byte ptr ss:[ebp+edx-3C0],0
0043E898     8A10         mov dl,byte ptr ds:[eax]
0043E89A     8A1E         mov bl,byte ptr ds:[esi]
0043E89C     8ACA         mov cl,dl
0043E89E     3AD3         cmp dl,bl
0043E8A0     75 1E        jnz short FamFinan.0043E8C0
0043E8A2     84C9         test cl,cl
0043E8A4     74 16        je short FamFinan.0043E8BC
0043E8A6     8A50 01      mov dl,byte ptr ds:[eax+1]
0043E8A9     8A5E 01      mov bl,byte ptr ds:[esi+1]
0043E8AC     8ACA         mov cl,dl
0043E8AE     3AD3         cmp dl,bl
0043E8B0     75 0E        jnz short FamFinan.0043E8C0
```



```
0043E8B2     83C0 02           add eax,2  
0043E8B5     83C6 02           add esi,2  
0043E8B8     84C9             test cl,cl  
0043E8BA   ^ 75 DC            jnz short FamFinan.0043E898                     ; 比较读入了版本号和文件的版本号  
0043E8BC     33C0             xor eax,eax                                 ; 也就是对第一次读入的 1a 个字符进行比较  
0043E8BE     EB 05            jmp short FamFinan.0043E8C5  
0043E8C0     1BC0             sbb eax,eax  
0043E8C2     83D8 FF          sbb eax,-1  
0043E8C5     85C0             test eax,eax  
0043E8C7     0F85 AD040000    jnz FamFinan.0043ED7A  
0043E8CD     BE 888D4500      mov esi,FamFinan.00458D88  
0043E8D2     8D85 40FDFFFF    lea eax,dword ptr ss:\[ebp-2C0\]  
0043E8D8     8A10             mov dl,byte ptr ds:\[eax\]  
0043E8DA     8A1E             mov bl,byte ptr ds:\[esi\]  
0043E8DC     8ACA             mov cl,dl  
0043E8DE     3AD3             cmp dl,bl  
0043E8E0     75 1E            jnz short FamFinan.0043E900  
0043E8E2     84C9             test cl,cl  
0043E8E4     74 16            je short FamFinan.0043E8FC  
0043E8E6     8A50 01          mov dl,byte ptr ds:\[eax+1\]  
0043E8E9     8A5E 01          mov bl,byte ptr ds:\[esi+1\]  
0043E8EC     8ACA             mov cl,dl  
0043E8EE     3AD3             cmp dl,bl  
0043E8F0     75 0E            jnz short FamFinan.0043E900  
0043E8F2     83C0 02          add eax,2  
0043E8F5     83C6 02          add esi,2  
0043E8F8     84C9             test cl,cl  
0043E8FA   ^ 75 DC            jnz short FamFinan.0043E8D8                     ; 对第二次读入的 0c 个字符进行比较  
0043E8FC     33C0             xor eax,eax  
0043E8FE     EB 05            jmp short FamFinan.0043E905  
0043E900     1BC0             sbb eax,eax  
0043E902     83D8 FF          sbb eax,-1  
0043E905     85C0             test eax,eax  
0043E907     0F85 6D040000    jnz FamFinan.0043ED7A  
0043E90D     BE 808D4500      mov esi,FamFinan.00458D80                     ; ASCII "V1.00"  
0043E912     8D85 40FCFFFF    lea eax,dword ptr ss:\[ebp-3C0\]  
0043E918     8A10             mov dl,byte ptr ds:\[eax\]  
0043E91A     8A1E             mov bl,byte ptr ds:\[esi\]  
0043E91C     8ACA             mov cl,dl  
0043E91E     3AD3             cmp dl,bl  
0043E920     75 1E            jnz short FamFinan.0043E940  
0043E922     84C9             test cl,cl  
0043E924     74 16            je short FamFinan.0043E93C  
0043E926     8A50 01          mov dl,byte ptr ds:\[eax+1\]  
0043E929     8A5E 01          mov bl,byte ptr ds:\[esi+1\]  
0043E92C     8ACA             mov cl,dl  
0043E92E     3AD3             cmp dl,bl  
0043E930     75 0E            jnz short FamFinan.0043E940  
0043E932     83C0 02          add eax,2  
0043E935     83C6 02          add esi,2  
0043E938     84C9             test cl,cl  
0043E93A   ^ 75 DC            jnz short FamFinan.0043E918                     ; 对第二次读入的 05 个字符进行比较  
0043E93C     33C0             xor eax,eax  
0043E93E     EB 05            jmp short FamFinan.0043E945  
0043E940     1BC0             sbb eax,eax  
0043E942     83D8 FF          sbb eax,-1  
0043E945     85C0             test eax,eax  
0043E947     0F85 2D040000    jnz FamFinan.0043ED7A  
0043E94D     8D45 B8          lea eax,dword ptr ss:\[ebp-48\]  
0043E950     6A 04            push 4  
0043E952     50               push eax  
0043E953     8D4D C4          lea ecx,dword ptr ss:\[ebp-3C\]  
0043E956     E8 1B3F0000      call <jmp.&MFC42.#5442\_CFile::Read>  
0043E95B     8D4D BC          lea ecx,dword ptr ss:\[ebp-44\]  
0043E95E     6A 04            push 4  
0043E960     51               push ecx  
0043E961     8D4D C4          lea ecx,dword ptr ss:\[ebp-3C\]  
0043E964     E8 0D3F0000      call <jmp.&MFC42.#5442\_CFile::Read>  
0043E969     8B97 AC040000    mov edx,dword ptr ds:\[edi+4AC\]              ; edx=假码  
0043E96F     8B45 BC          mov eax,dword ptr ss:\[ebp-44\]               ; eax=读入的四个字节 17 00 00 00  
0043E972     8DB7 AC040000    lea esi,dword ptr ds:\[edi+4AC\]              ; esi=假码指针  
0043E978     8B4A F8          mov ecx,dword ptr ds:\[edx-8\]                ; ecx=假码长度  
0043E97B     3BC1             cmp eax,ecx  
0043E97D     74 0E            je short FamFinan.0043E98D                     ; 如果假码长度 != 0x17，则跳  
0043E97F     66:C787 B2040000 mov word ptr ds:\[edi+4B2\],2  
0043E988     E9 F6030000      jmp FamFinan.0043ED83  
0043E98D     83C0 10          add eax,10  
0043E990     99               cdq                                           ; 符号扩展(edx=00)  
0043E991     83E2 0F          and edx,0F  
0043E994     03C2             add eax,edx  
0043E996     C1F8 04          sar eax,4                                      ; 符号扩展右移 4 位  
0043E999     8BD8             mov ebx,eax  
0043E99B     8945 E8          mov dword ptr ss:\[ebp-18\],eax  
0043E99E     C1E3 04          shl ebx,4  
0043E9A1     53               push ebx  
0043E9A2     E8 653C0000      call <jmp.&MFC42.#823\_operator new>           ; 分配内存  
0043E9A7     53               push ebx  
0043E9A8     8945 EC          mov dword ptr ss:\[ebp-14\],eax               ; \[ebp-14\]指向新分配的内存 AcMem1=new byte\[32\]  
0043E9AB     E8 5C3C0000      call <jmp.&MFC42.#823\_operator new>  
0043E9B0     8BF8             mov edi,eax  
0043E9B2     53               push ebx  
0043E9B3     897D D8          mov dword ptr ss:\[ebp-28\],edi               ; \[ebp-28\]指向新分配的内存 AcMem2=new byte\[32\]  
0043E9B6     E8 513C0000      call <jmp.&MFC42.#823\_operator new>  
0043E9BB     83C4 0C          add esp,0C  
0043E9BE     8945 E0          mov dword ptr ss:\[ebp-20\],eax               ; \[ebp-20\]指向新分配的内存 AcMem3=new byte\[32\]  
0043E9C1     85DB             test ebx,ebx  
0043E9C3     7E 12            jle short FamFinan.0043E9D7  
0043E9C5     8BCB             mov ecx,ebx  
0043E9C7     33C0             xor eax,eax  
0043E9C9     8BD1             mov edx,ecx  
0043E9CB     C1E9 02          shr ecx,2                                      ; 初使化第二次分配的内存  
0043E9CE     F3:AB            rep stos dword ptr es:\[edi\]                   ; edi=\[ebp-28\]=第二次分配的内存  
0043E9D0     8BCA             mov ecx,edx  
0043E9D2     83E1 03          and ecx,3  
0043E9D5     F3:AA            rep stos byte ptr es:\[edi\]                    ; AcMem2={NUll};  
0043E9D7     8B3E             mov edi,dword ptr ds:\[esi\]                   ; edi=指向假码  
0043E9D9     83C9 FF          or ecx,FFFFFFFF  
0043E9DC     33C0             xor eax,eax  
0043E9DE     F2:AE            repne scas byte ptr es:\[edi\]  
0043E9E0     F7D1             not ecx  
0043E9E2     2BF9             sub edi,ecx  
0043E9E4     8BC1             mov eax,ecx  
0043E9E6     8BF7             mov esi,edi  
0043E9E8     8B7D D8          mov edi,dword ptr ss:\[ebp-28\]               ; edi=第二次分配的内存  
0043E9EB     C1E9 02          shr ecx,2  
0043E9EE     F3:A5            rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  
0043E9F0     8BC8             mov ecx,eax  
0043E9F2     83E1 03          and ecx,3  
0043E9F5     33C0             xor eax,eax  
0043E9F7     F3:A4            rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\] ; strcpy(AcMem2,假码)  
0043E9F9     C785 C0FEFFFF 04 mov dword ptr ss:\[ebp-140\],FamFinan.0044BD04 ; \[ebp-140\]=0044bd04  
0043EA03     888405 40FEFFFF  mov byte ptr ss:\[ebp+eax-1C0\],al  
0043EA0A     40               inc eax  
0043EA0B     3D 80000000      cmp eax,80  
0043EA10   ^ 7C F1            jl short FamFinan.0043EA03                     ; \[ebp-1c0\]\[80\]={0...7F}  
0043EA12     8D8D 40FEFFFF    lea ecx,dword ptr ss:\[ebp-1C0\]              ; ecx=数组\[ebp-1c0\]  
0043EA18     68 80000000      push 80                                        ; 长度
```



```
0043EA1D     51                    push ecx                                    ; 数组指针  

0043EA1E     8D8D C0FEFFFF       lea ecx,dword ptr ss:[ebp-140]              ; ecx=某个类的指针  

0043EA24     E8 37250000         call FamFinan.00440F60                      ; 生成 key 表  

0043EA29     8B45 E8           mov eax,dword ptr ss:[ebp-18]  

0043EA2C     85C0                test eax,eax  

0043EA2E     7E 27               jle short FamFinan.0043EA57  

0043EA30     8B75 D8           mov esi,dword ptr ss:[ebp-28]                ; esi=假码  

0043EA33     8B7D E0           mov edi,dword ptr ss:[ebp-20]                ; edi=AcMem3  第三次分配的内存  

0043EA36     2BFE                sub edi,esi  

0043EA38     8945 E8           mov dword ptr ss:[ebp-18],eax  

0043EA3B     8D1437            lea edx,dword ptr ds:[edi+esi]              ; edx=AcMem3  第三次分配的内存  

0043EA3E     8D8D C0FEFFFF     lea ecx,dword ptr ss:[ebp-140]  

0043EA44     52                  push edx                                    ; AcMem3 入栈  (临时分配的缓冲区)  

0043EA45     56                  push esi                                    ; 假码入栈  

0043EA46     E8 95260000       call FamFinan.004410E0                      ; 加密运算，结果存入 2 参 AcMem3  

0043EA4B     8B45 E8           mov eax,dword ptr ss:[ebp-18]  

0043EA4E     83C6 10           add esi,10  

0043EA51     48                  dec eax  

0043EA52     8945 E8           mov dword ptr ss:[ebp-18],eax  

0043EA55   ^ 75 E4             jnz short FamFinan.0043EA3B  

0043EA57     8B45 D8           mov eax,dword ptr ss:[ebp-28]  

0043EA5A     50                  push eax  

0043EA5B     E8 883B0000       call <jmp.&MFC42.#825_operator delete>      ; delete AcMem2 释放第二次分配的内存  

0043EA60     8B45 B8           mov eax,dword ptr ss:[ebp-48]  

0043EA63     83C4 04           add esp,4  

0043EA66     85C0                test eax,eax  

0043EA68     C745 E8 00000000  mov dword ptr ss:[ebp-18],0  

0043EA6F     7E 27               jle short FamFinan.0043EA98  

0043EA71     8B7D EC           mov edi,dword ptr ss:[ebp-14]  

0043EA74     53                  push ebx  

0043EA75     57                  push edi  

0043EA76     8D4D C4           lea ecx,dword ptr ss:[ebp-3C]  

0043EA79     E8 F83D0000       call <jmp.&MFC42.#5442_CFile::Read>        ; 读入 0x20 个字节到第一次申请的内存中 AcMme1  

0043EA7E     8B4D BC           mov ecx,dword ptr ss:[ebp-44]  

0043EA81     8B75 E0           mov esi,dword ptr ss:[ebp-20]  

0043EA84     33D2                xor edx,edx  

0043EA86     F3:A6             repe cmps byte ptr es:[edi],byte ptr ds:[esi]   ; strcmp(AcMem1,AcMem3)  

0043EA88     74 28               je short FamFinan.0043EAB2                ; 如果读入数据与[ebp-20]相等相等，则跳  

0043EA8A     8B45 E8           mov eax,dword ptr ss:[ebp-18]  

0043EA8D     8B4D B8           mov ecx,dword ptr ss:[ebp-48]  

0043EA90     40                  inc eax  

0043EA91     3BC1                cmp eax,ecx  

0043EA93     8945 E8           mov dword ptr ss:[ebp-18],eax  

0043EA96   ^ 7C D9             jl short FamFinan.0043EA71                ; 比较 0xc8 次  

0043EA98     8B45 D4           mov eax,dword ptr ss:[ebp-2C]  

0043EA9B     6A 00               push 0  

0043EA9D     6A 01               push 1  

0043EA9F     6A 02               push 2  

0043EAA1     66:C780 B2040000 >mov word ptr ds:[eax+4B2],2  

0043EAAA     8B40 20           mov eax,dword ptr ds:[eax+20]  

0043EAAD     E9 DA020000       jmp FamFinan.0043ED8C  

0043EAB2     8B4D EC           mov ecx,dword ptr ss:[ebp-14]  

0043EAB5     51                  push ecx  

0043EAB6     E8 2D3B0000       call <jmp.&MFC42.#825_operator delete>      ; delete AcMem1  

0043EABB     8B55 E0           mov edx,dword ptr ss:[ebp-20]  

0043EABE     52                  push edx  

0043EABF     E8 243B0000       call <jmp.&MFC42.#825_operator delete>      ; delete AcMem3  

0043EAC4     83C4 08           add esp,8  

0043EAC7     8D4D A8           lea ecx,dword ptr ss:[ebp-58]  

0043EACA     E8 BF3D0000       call <jmp.&MFC42.#354_CFile::CFile>  

0043EACF     8B45 DC           mov eax,dword ptr ss:[ebp-24]  

0043EAD2     68 AC8D4500       push FamFinan.00458DAC                    ; ASCII "\FamFinan.biu"  

0043EAD7     05 E8000000       add eax,0E8  

0043EADC     8D4D EC           lea ecx,dword ptr ss:[ebp-14]  

0043EADF     50                  push eax  

0043EAE0     51                  push ecx  

0043EAE1     C645 FC 02        mov byte ptr ss:[ebp-4],2  

0043EAE5     E8 343B0000       call <jmp.&MFC42.#924_operator+>  

0043EAEA     8B00                mov eax,dword ptr ds:[eax]  

0043EAEC     6A 00               push 0  

0043EAEE     68 01100000       push 1001  

0043EAF3     50                  push eax  

0043EAF4     8D4D A8           lea ecx,dword ptr ss:[ebp-58]  

0043EAF7     C645 FC 03        mov byte ptr ss:[ebp-4],3  

0043EAFB     E8 823D0000       call <jmp.&MFC42.#5186_CFile::Open>      ; 打开 FamFinan.biu 文件  

0043EB00     85C0                test eax,eax  

0043EB02     8D4D EC           lea ecx,dword ptr ss:[ebp-14]
```
