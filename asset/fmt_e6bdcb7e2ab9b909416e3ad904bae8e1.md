
```
      0052B3D9  |. E8 2E99EDFF    CALL zhdm.00404D0C                       ;DL 值加入 Blue2 串的第 7 个
位置(从 1 开始)形成新串值 Blue2

      0052B3DE  |. 8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]

      0052B3E1  |. 8A55 FB        MOV DL,BYTE PTR SS:[EBP-5]               ;DL=S4;

      0052B3E4  |. E8 6B95EDFF    CALL zhdm.00404954

      0052B3E9  |. 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]

      0052B3EC  |. 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]

      0052B3EF  |. B9 03000000    MOV ECX,3

      0052B3F4  |. E8 1399EDFF    CALL zhdm.00404D0C                       ;DL 值加入 Blue2 串的第 3 个
位置(从 1 开始)形成新串值 Blue2

      0052B3F9  |. 8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]

      0052B3FC  |. 8A55 F9        MOV DL,BYTE PTR SS:[EBP-7]               ;DL=S2;

      0052B3FF  |. E8 5095EDFF    CALL zhdm.00404954

      0052B404  |. 8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]

      0052B407  |. 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]

      0052B40A  |. B9 05000000    MOV ECX,5

      0052B40F  |. E8 F898EDFF    CALL zhdm.00404D0C                       ;DL 值加入 Blue2 串的第 5 个
位置(从 1 开始)形成新串值 Blue2

      0052B414  |. 8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]

      0052B417  |. 8A55 FA        MOV DL,BYTE PTR SS:[EBP-6]               ;DL=S3

      0052B41A  |. E8 3595EDFF    CALL zhdm.00404954

      0052B41F  |. 8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]

      0052B422  |. 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]

      0052B425  |. B9 09000000    MOV ECX,9

      0052B42A  |. E8 DD98EDFF    CALL zhdm.00404D0C                       ;DL 值加入 Blue2 串的第 9 个
位置(从 1 开始)形成新串值 Blue2

      0052B42F  |. 8BC6           MOV EAX,ESI

      0052B431  |. 8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]            ;最后生成的 Blue2 串放入
EDX

      0052B434  |. E8 8F93EDFF    CALL zhdm.004047C8

      0052B439  |. 33C0           XOR EAX,EAX

      0052B43B  |. 5A             POP EDX

      0052B43C  |. 59             POP ECX

      0052B43D  |. 59             POP ECX

      0052B43E  |. 64:8910        MOV DWORD PTR FS:[EAX],EDX

      0052B441  |. 68 63B45200    PUSH zhdm.0052B463

      0052B446  |> 8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]

      0052B449  |. BA 04000000    MOV EDX,4

      0052B44E  |. E8 4593EDFF    CALL zhdm.00404798

      0052B453  |. 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]

      0052B456  |. E8 1993EDFF    CALL zhdm.00404774

      0052B45B  \. C3             RETN

      0052B45C   .^E9 5B8CEDFF    JMP zhdm.004040BC

      0052B461   .^EB E3          JMP SHORT zhdm.0052B446

      0052B463   . 5E             POP ESI

      0052B464   . 5B             POP EBX

      0052B465   . 8BE5           MOV ESP,EBP

      0052B467   . 5D             POP EBP

      0052B468   . C3             RETN
}******************************CALL zhdm.0052B310
```

```
0052D182  |. FFB5 5CFFFFFF  PUSH DWORD PTR SS:[EBP-A4]

0052D188  |. 68 10D35200    PUSH zhdm.0052D310

0052D18D  |. 8D85 68FFFFFF  LEA EAX,DWORD PTR SS:[EBP-98]

0052D193  |. BA 03000000    MOV EDX,3

0052D198  |. E8 4F79EDFF    CALL zhdm.00404AEC                     ;生成正解的注册码
```

```
    {******************************CALL zhdm.00404AEC

     00404AEC   $ 53             PUSH EBX

     00404AED   . 56             PUSH ESI

     00404AEE   . 57             PUSH EDI

     00404AEF   . 52             PUSH EDX

     00404AF0   . 50             PUSH EAX

     00404AF1   . 89D3           MOV EBX,EDX

     00404AF3   . 31FF           XOR EDI,EDI

     00404AF5   . 8B4C94 14      MOV ECX,DWORD PTR SS:[ESP+EDX*4+14]    ;Yushu

     00404AF9   . 85C9           TEST ECX,ECX

     00404AFB   . 74 06          JE SHORT zhdm.00404B03

     00404AFD   . 3908           CMP DWORD PTR DS:[EAX],ECX

     00404AFF   . 75 02          JNZ SHORT zhdm.00404B03

     00404B01   . 89C7           MOV EDI,EAX

     00404B03   > 31C0           XOR EAX,EAX

     00404B05   > 8B4C94 14      MOV ECX,DWORD PTR SS:[ESP+EDX*4+14]    ;Blue2

     00404B09   . 85C9           TEST ECX,ECX

     00404B0B   . 74 09          JE SHORT zhdm.00404B16

     00404B0D   . 0341 FC        ADD EAX,DWORD PTR DS:[ECX-4]           ;EAX 存放 Yushu 十进制串长度
,[ECX-4]存放 Blue2 串长度

     00404B10   . 39CF           CMP EDI,ECX

     00404B12   . 75 02          JNZ SHORT zhdm.00404B16

     00404B14   . 31FF           XOR EDI,EDI

     00404B16   > 4A             DEC EDX

     00404B17   .⁷⁵ EC           JNZ SHORT zhdm.00404B05

     00404B19   . 85FF           TEST EDI,EDI

     00404B1B   . 74 14          JE SHORT zhdm.00404B31

     00404B1D   . 89C2           MOV EDX,EAX

     00404B1F   . 89F8           MOV EAX,EDI

     00404B21   . 8B37           MOV ESI,DWORD PTR DS:[EDI]

     00404B23   . 8B76 FC        MOV ESI,DWORD PTR DS:[ESI-4]

     00404B26   . E8 85020000    CALL zhdm.00404DB0

     00404B2B   . 57             PUSH EDI

     00404B2C   . 0337           ADD ESI,DWORD PTR DS:[EDI]

     00404B2E   . 4B             DEC EBX

     00404B2F   . EB 08          JMP SHORT zhdm.00404B39

     00404B31   > E8 02FDFFFF    CALL zhdm.00404838

     00404B36   . 50             PUSH EAX

     00404B37   . 89C6           MOV ESI,EAX

     00404B39   > 8B449C 18      MOV EAX,DWORD PTR SS:[ESP+EBX*4+18]   ;依次取 Yush 和 Blue2 十进制串

     00404B3D   . 89F2           MOV EDX,ESI

     00404B3F   . 85C0           TEST EAX,EAX

     00404B41   . 74 0A          JE SHORT zhdm.00404B4D

     00404B43   . 8B48 FC        MOV ECX,DWORD PTR DS:[EAX-4]         ;Yush 和 Blue2 串长度

     00404B46   . 01CE           ADD ESI,ECX

     00404B48   . E8 C7DEFFFF    CALL zhdm.00402A14                   ;串联接后尾部插入字'^'

     00404B4D   > 4B             DEC EBX

     00404B4E   .⁷⁵ E9           JNZ SHORT zhdm.00404B39

     00404B50   . 5A             POP EDX

     00404B51   . 58             POP EAX

     00404B52   . 85FF           TEST EDI,EDI

     00404B54   . 75 0C          JNZ SHORT zhdm.00404B62

     00404B56   . 85D2           TEST EDX,EDX

     00404B58   . 74 03          JE SHORT zhdm.00404B5D

     00404B5A   . FF4A F8        DEC DWORD PTR DS:[EDX-8]

     00404B5D   > E8 66FCFFFF    CALL zhdm.004047C8

     00404B62   > 5A             POP EDX

     00404B63   . 5F             POP EDI

     00404B64   . 5E             POP ESI

     00404B65   . 5B             POP EBX

     00404B66   . 58             POP EAX

     00404B67   . 8D2494         LEA ESP,DWORD PTR SS:[ESP+EDX*4]

     00404B6A   . FFE0           JMP EAX

     00404B6C   . C3             RETN
    }******************************CALL zhdm.00404AEC
```

```
0052D19D  |. 8B95 68FFFFFF  MOV EDX,DWORD PTR SS:[EBP-98]             ;正确的注册码,这里可以作内存
注册机

0052D1A3  |. 58             POP EAX                                  ;输入的试炼码

0052D1A4  |. E8 C779EDFF    CALL zhdm.00404B70                       ;正确的注册码与试炼码的比较

0052D1A9  |. 0F85 95000000  JNZ zhdm.0052D244                         ;不相等跳到出错信息,这里可以
爆破
```

## 算法描述

1. 设 `ddh` 为输入的订单号
2. `yushu=ddh%0x315;`
3. `blue=ddh⁰xB25F1;` 转换成串
4. `blue=blue[(blue[0]+blue[1])%5+0x34]+(blue[2]+blue[3])%5+0x33];` 转换成整数
5. `blue1=blue^DDFB7687;`
6. `s1=(blue1[0]+blue1[1])%5+0x66;`
   `s2=(blue1[2]+blue1[3])%5+0x75;`
   `s3=(blue1[4]+blue1[5])%5+0x7a;`
   `s4=(blue1[6]+blue1[7]+blue1[8])%5+0x69;`
7. `blue1` 转换成串;
   串 `blue1` 的第 7 个位置插入 `S1`(从 1 开始)生成新串 `blue1`;
   串 `blue1` 的第 3 个位置插入 `S4`(从 1 开始)生成新串 `blue1`;
   串 `blue1` 的第 5 个位置插入 `S2`(从 1 开始)生成新串 `blue1`;
   串 `blue1` 的第 9 个位置插入 `S3`(从 1 开始)生成新串 `blue1`;
8. `yushu` 转换成串
   设注册码为 `zcm`
   `zcm=yush+blue1+'^'`

由于版权问题我就不附注册机了，注册机源程序也就不附了，有兴趣的可以编写一下.

不过我还是给出一个注册号:

订单号:`12345678`

注册码:`19525l9u087{i883`

不过该软件注册成功后，如果订单号短的话，就会重启后显示没有注册，最好是 8 位的订单号
