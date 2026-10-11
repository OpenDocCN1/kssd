
# 刚接触 1 一个左右，对一个软件的超级简单算法的分析（混个邀请码）

> 原文：[`www.pediy.com/kssd/pediy11/122601.html`](https://www.pediy.com/kssd/pediy11/122601.html)

## 文章信息

- **文章标题**：本人的算法分析笔记
- **软件名字**：`MasmEditor`
- **软件类别**：编程类
- **本文连接**：`http://hi.baidu.com/zzydog/blog/item/3e02c409aa09213ee924880b.html`
- **软件连接**：`http://26.duote.org/masmeditor\_setup.exe`

小弟初学者，刚接触这些 1 个月左右，这是我第一次发这一类的文章（这篇文章在自己的博客发过），07 年的软件，希望能够混个邀请码~

软件加了壳：`ASProtect 2.1x SKE` -> `Alexey Solodovnikov`，可以下载到相关的脱壳脚本

## 注册流程说明

软件采用重启验证的方式，没有注册的情况下，在帮助选项中，会存在注册选项；注册成功后，该选项会消失（下下注册表断点，然后跟踪发现的~）。

## 算法分析

脱壳后发现是 `delphi` 写的，可以通过 `DEDE` 查找帮助按钮的按钮事件（现在才想到的，开始自己是通过下，注册表断点跟踪算法的）。

找到按钮事件的开始代码如下：

```
00506A78  /.  55             PUSH EBP
00506A79  |.  8BEC           MOV EBP,ESP
00506A7B  |.  83C4 C4        ADD ESP,-3C
00506A7E  |.  53             PUSH EBX
00506A7F  |.  56             PUSH ESI
00506A80  |.  33C9           XOR ECX,ECX
00506A82  |.  894D C4        MOV DWORD PTR SS:[EBP-3C],ECX
00506A85  |.  894D C8        MOV DWORD PTR SS:[EBP-38],ECX
00506A88  |.  894D FC        MOV DWORD PTR SS:[EBP-4],ECX
00506A8B  |.  8BF2           MOV ESI,EDX
00506A8D  |.  33C0           XOR EAX,EAX
00506A8F  |.  55             PUSH EBP
00506A90  |.  68 466B5000    PUSH Cracker.00506B46
00506A95  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
00506A98  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
00506A9B  |.  8D45 C8        LEA EAX,DWORD PTR SS:[EBP-38]
00506A9E  |.  E8 69D9F8FF    CALL Cracker.0049440C                     ;  //这里生成机器码的原码
00506AA3  |.  8B45 C8        MOV EAX,DWORD PTR SS:[EBP-38]
00506AA6  |.  8D55 CC        LEA EDX,DWORD PTR SS:[EBP-34]
00506AA9  |.  E8 42E5F8FF    CALL Cracker.00494FF0                     ;  //这里对字符串进行 MD5 加密,生成的是 HEX 数据
00506AAE  |.  8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]
00506AB1  |.  8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]
00506AB4  |.  E8 ABE5F8FF    CALL Cracker.00495064                     ;  //将上面的 HEX 数据转化成字符串，即是生成机器码
00506AB9  |.  8D55 CC        LEA EDX,DWORD PTR SS:[EBP-34]
00506ABC  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00506ABF  |.  E8 2CE5F8FF    CALL Cracker.00494FF0                     ;  //这里对机器码的数据加密
00506AC4  |.  8D45 CC        LEA EAX,DWORD PTR SS:[EBP-34]
00506AC7  |.  8D55 C4        LEA EDX,DWORD PTR SS:[EBP-3C]
00506ACA  |.  E8 95E5F8FF    CALL Cracker.00495064                     ;  //这个 CALL 之后出现了真正的注册码，算法 CALL~
00506ACF  |.  8B55 C4        MOV EDX,DWORD PTR SS:[EBP-3C]
00506AD2  |.  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
00506AD5  |.  E8 56D8EFFF    CALL Cracker.00404330
00506ADA  |.  8D55 EC        LEA EDX,DWORD PTR SS:[EBP-14]
00506ADD  |.  A1 30A35000    MOV EAX,DWORD PTR DS:[50A330]
00506AE2  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]
00506AE4  |.  E8 07E5F8FF    CALL Cracker.00494FF0
00506AE9  |.  8D55 DC        LEA EDX,DWORD PTR SS:[EBP-24]
00506AEC  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00506AEF  |.  E8 FCE4F8FF    CALL Cracker.00494FF0
00506AF4  |.  33C9           XOR ECX,ECX
00506AF6  |.  8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
00506AF9  |.  8D55 DC        LEA EDX,DWORD PTR SS:[EBP-24]
00506AFC  |>  8A18           /MOV BL,BYTE PTR DS:[EAX]               ;  //下面这个循环是注册码的比较
00506AFE  |.  3A1A           |CMP BL,BYTE PTR DS:[EDX]
00506B00      75 08          |JNZ SHORT Cracker.00506B0A
00506B02  |.  41             |INC ECX
00506B03  |.  42             |INC EDX
00506B04  |.  40             |INC EAX
00506B05  |.  83F9 10        |CMP ECX,10
00506B08  |.^ 75 F2          \JNZ SHORT Cracker.00506AFC
00506B0A  |>  83F9 10        CMP ECX,10
00506B0D      75 14          JNZ SHORT Cracker.00506B23               ;  //爆破点
00506B0F  |.  8BC6           MOV EAX,ESI
00506B11  |.  8B15 40214700  MOV EDX,DWORD PTR DS:[472140]            ;  Cracker.0047218C
00506B17  |.  E8 F8CAEFFF    CALL Cracker.00403614
00506B1C  |.  33D2           XOR EDX,EDX
00506B1E  |.  E8 11C7F6FF    CALL Cracker.00473234
00506B23  |>  33C0           XOR EAX,EAX
00506B25  |.  5A             POP EDX
00506B26  |.  59             POP ECX
00506B27  |.  59             POP ECX
00506B28  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX
00506B2B  |.  68 4D6B5000    PUSH Cracker.00506B4D
00506B30  |>  8D45 C4        LEA EAX,DWORD PTR SS:[EBP-3C]
00506B33  |.  BA 02000000    MOV EDX,2
00506B38  |.  E8 7FD7EFFF    CALL Cracker.004042BC
00506B3D  |.  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
00506B40  |.  E8 53D7EFFF    CALL Cracker.00404298
00506B45  \.  C3             RETN
```

跟入算法 CALL：

```
00495064  /$  55             PUSH EBP
00495065  |.  8BEC           MOV EBP,ESP
00495067  |.  83C4 E8        ADD ESP,-18
0049506A  |.  53             PUSH EBX
0049506B  |.  56             PUSH ESI
0049506C  |.  57             PUSH EDI
0049506D  |.  33C9           XOR ECX,ECX
0049506F  |.  894D EC        MOV DWORD PTR SS:[EBP-14],ECX
00495072  |.  894D E8        MOV DWORD PTR SS:[EBP-18],ECX
00495075  |.  8BF0           MOV ESI,EAX
00495077  |.  8D7D F0        LEA EDI,DWORD PTR SS:[EBP-10]
0049507A  |.  A5             MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
0049507B  |.  A5             MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
0049507C  |.  A5             MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
0049507D  |.  A5             MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ES>
0049507E  |.  8BFA           MOV EDI,EDX
00495080  |.  33C0           XOR EAX,EAX
00495082  |.  55             PUSH EBP
00495083  |.  68 FF504900    PUSH Cracker.004950FF
00495088  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
0049508B  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
0049508E  |.  8BC7           MOV EAX,EDI
00495090  |.  E8 03F2F6FF    CALL Cracker.00404298
00495095  |.  B3 10          MOV BL,10
00495097  |.  8D75 F0        LEA ESI,DWORD PTR SS:[EBP-10]
0049509A  |>  FF37           /PUSH DWORD PTR DS:[EDI]                  ;  //这个循环是注册码的生成算法
0049509C  |.  8D45 EC        |LEA EAX,DWORD PTR SS:[EBP-14]
0049509F  |.  33D2           |XOR EDX,EDX
004950A1  |.  8A16           |MOV DL,BYTE PTR DS:[ESI]
004950A3  |.  C1EA 04        |SHR EDX,4
004950A6  |.  83E2 0F        |AND EDX,0F                               ;  //取机器码 MD5 加密后数据的一个字节的高 4 位的做索引~
004950A9  |.  8A92 C4935000  |MOV DL,BYTE PTR DS:[EDX+5093C4]           ;  //索引出一位注册码
004950AF  |.  E8 CCF3F6FF    |CALL Cracker.00404480
004950B4  |.  FF75 EC        |PUSH DWORD PTR SS:[EBP-14]
004950B7  |.  8D45 E8        |LEA EAX,DWORD PTR SS:[EBP-18]
004950BA  |.  8A16           |MOV DL,BYTE PTR DS:[ESI]
004950BC  |.  80E2 0F        |AND DL,0F
004950BF  |.  81E2 FF000000  |AND EDX,0FF                               ;  //取机器码 MD5 加密后数据的一个字节的低 4 位做索引
004950C5  |.  8A92 C4935000  |MOV DL,BYTE PTR DS:[EDX+5093C4]           ;  //索引出一位注册码
004950CB  |.  E8 B0F3F6FF    |CALL Cracker.00404480
004950D0  |.  FF75 E8        |PUSH DWORD PTR SS:[EBP-18]
004950D3  |.  8BC7           |MOV EAX,EDI
004950D5  |.  BA 03000000    |MOV EDX,3
004950DA  |.  E8 39F5F6FF    |CALL Cracker.00404618
004950DF  |.  46             |INC ESI                                     ;   //指针指向下一个字节~
004950E0  |.  FECB           |DEC BL
004950E2  |.^ 75 B6          \JNZ SHORT Cracker.0049509A                 ; //是否已经结素
004950E4  |.  33C0           XOR EAX,EAX
004950E6  |.  5A             POP EDX
004950E7  |.  59             POP ECX
004950E8  |.  59             POP ECX
004950E9  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX
004950EC  |.  68 06514900    PUSH Cracker.00495106
004950F1  |>  8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]
004950F4  |.  BA 02000000    MOV EDX,2
004950F9  |.  E8 BEF1F6FF    CALL Cracker.004042BC
004950FE  \.  C3             RETN
```



在索引的时候，通过跟入 `ESI` 寄存器的地址可以发现，索引字符串为：

```
"0d2b45678af3c1e9"
```

`/*=============================================================*/`

分析到这里，得到是一个非常标准的注册流程，整个算法思路就非常简单和清晰了~

实际上就是先对机器码进行 `MD5` 加密，对加密后的数据从开始到结束，每次取 4 位作为索引号去找 `"0d2b45678af3c1e9"` 对应的字符。

对应算法：

```c
VOID GenerateRegKey(TCHAR *MachineKey, TCHAR *RegKey)             //产生注册码
{
    BYTE KeyMd5Data[16];
    BYTE Index;
    BYTE c=0,j=0;
    CHAR ByteMachineKey[33];

    WideCharToMultiByte(CP_ACP, WC_COMPOSITECHECK, MachineKey, -1, ByteMachineKey, 33, NULL, NULL);

    //MD5 加密机器码
    Md5Struct Context;
    Md5Inist(&Context);
    Md5Proc(&Context, (BYTE *)ByteMachineKey, lstrlen(MachineKey));
    Md5Finish(&Context, KeyMd5Data);

    //生成注册码~
    for(;c<16;c++,j+=2)
    {
        Index=KeyMd5Data[c];
        Index=(Index >> 4) & 0x0f;
        RegKey[j]=IndexFlag[Index];

        Index=KeyMd5Data[c];
        Index=Index & 0x0f;
        RegKey[j+1]=IndexFlag[Index];
    }

    RegKey[j]=NULL;    //结束标志
}
```

自己用 `SDK` 写了一个注册机~，作为附件上传了~

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `8086MasmKeyGen.rar` |
| :--- | :--- |
