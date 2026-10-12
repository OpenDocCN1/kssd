
## 反汇编代码分析

```
00563688  |. 6A 00          PUSH 0                               ; /lParam = 0
0056368A  |. 6A 00          PUSH 0                               ; |wParam = 0
0056368C  |. 68 78070000    PUSH 778                               ; |Message = MSG(778)
00563691  |. A1 84A65800    MOV EAX,DWORD PTR DS:[58A684]          ; |
00563696  |. 8B00           MOV EAX,DWORD PTR DS:[EAX]             ; |
00563698  |. 50             PUSH EAX                               ; |hWnd
00563699  |. E8 9A4CEAFF    CALL <JMP.&user32.SendMessageA>       ; \SendMessageA
0056369E  |> 33C0          XOR EAX,EAX
005636A0  |. 5A             POP EDX
005636A1  |. 59             POP ECX
005636A2  |. 59             POP ECX
005636A3  |. 64:8910        MOV DWORD PTR FS:[EAX],EDX
005636A6  |. 68 11375600    PUSH RegOpt.00563711
005636AB  |> 8D45 8C        LEA EAX,DWORD PTR SS:[EBP-74]
005636AE  |. BA 02000000    MOV EDX,2
005636B3  |. E8 041BEAFF    CALL RegOpt.004051BC
005636B8  |. 8D45 94        LEA EAX,DWORD PTR SS:[EBP-6C]
005636BB  |. BA 06000000    MOV EDX,6
005636C0  |. E8 F71AEAFF    CALL RegOpt.004051BC
005636C5  |. 8D45 AC        LEA EAX,DWORD PTR SS:[EBP-54]
005636C8  |. BA 02000000    MOV EDX,2
005636CD  |. E8 EA1AEAFF    CALL RegOpt.004051BC
005636D2  |. 8D45 B4        LEA EAX,DWORD PTR SS:[EBP-4C]
005636D5  |. E8 BE1AEAFF    CALL RegOpt.00405198
005636DA  |. 8D45 B8        LEA EAX,DWORD PTR SS:[EBP-48]
005636DD  |. BA 02000000    MOV EDX,2
005636E2  |. E8 D51AEAFF    CALL RegOpt.004051BC
005636E7  |. 8D45 C0        LEA EAX,DWORD PTR SS:[EBP-40]
005636EA  |. BA 07000000    MOV EDX,7
005636EF  |. E8 C81AEAFF    CALL RegOpt.004051BC
005636F4  |. 8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
005636F7  |. BA 06000000    MOV EDX,6
005636FC  |. E8 BB1AEAFF    CALL RegOpt.004051BC
00563701  |. 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
00563704  |. E8 8F1AEAFF    CALL RegOpt.00405198
00563709  \\. C3            RETN
```

### 关键比较 CALL

```
004055A8  /$ 53            PUSH EBX
004055A9  |. 56            PUSH ESI
004055AA  |. 57            PUSH EDI
004055AB  |. 89C6          MOV ESI,EAX
004055AD  |. 89D7          MOV EDI,EDX
004055AF  |. 39D0          CMP EAX,EDX
004055B1  |. 0F84 8F000000 JE RegOpt.00405646
004055B7  |. 85F6          TEST ESI,ESI
004055B9  |. 74 68         JE SHORT RegOpt.00405623
004055BB  |. 85FF          TEST EDI,EDI
004055BD  |. 74 6B         JE SHORT RegOpt.0040562A
004055BF  |. 8B46 FC       MOV EAX,DWORD PTR DS:[ESI-4]
004055C2  |. 8B57 FC       MOV EDX,DWORD PTR DS:[EDI-4]
004055C5  |. 29D0          SUB EAX,EDX
004055C7  |. 77 02         JA SHORT RegOpt.004055CB
004055C9  |. 01C2          ADD EDX,EAX
004055CB  |> 52            PUSH EDX
004055CC  |. C1EA 02      SHR EDX,2
004055CF  |. 74 26         JE SHORT RegOpt.004055F7
004055D1  |> 8B0E         /MOV ECX,DWORD PTR DS:[ESI]
004055D3  |. 8B1F         |MOV EBX,DWORD PTR DS:[EDI]
004055D5  |. 39D9         |CMP ECX,EBX
004055D7  |. 75 58         |JNZ SHORT RegOpt.00405631
004055D9  |. 4A            |DEC EDX
004055DA  |. 74 15         |JE SHORT RegOpt.004055F1
004055DC  |. 8B4E 04      |MOV ECX,DWORD PTR DS:[ESI+4]
004055DF  |. 8B5F 04      |MOV EBX,DWORD PTR DS:[EDI+4]
004055E2  |. 39D9         |CMP ECX,EBX
004055E4  |. 75 4B        |JNZ SHORT RegOpt.00405631
004055E6  |. 83C6 08      |ADD ESI,8
004055E9  |. 83C7 08      |ADD EDI,8
004055EC  |. 4A            |DEC EDX
004055ED  |.^ 75 E2       \\JNZ SHORT RegOpt.004055D1
004055EF  |. EB 06         JMP SHORT RegOpt.004055F7
004055F1  |> 83C6 04      ADD ESI,4
004055F4  |. 83C7 04      ADD EDI,4
004055F7  |> 5A            POP EDX
004055F8  |. 83E2 03      AND EDX,3
004055FB  |. 74 22         JE SHORT RegOpt.0040561F
004055FD  |. 8B0E          MOV ECX,DWORD PTR DS:[ESI]
004055FF  |. 8B1F          MOV EBX,DWORD PTR DS:[EDI]
00405601  |. 38D9          CMP CL,BL
00405603  |. 75 41         JNZ SHORT RegOpt.00405646
00405605  |. 4A            DEC EDX
00405606  |. 74 17         JE SHORT RegOpt.0040561F
00405608  |. 38FD          CMP CH,BH
0040560A  |. 75 3A         JNZ SHORT RegOpt.00405646
0040560C  |. 4A            DEC EDX
0040560D  |. 74 10         JE SHORT RegOpt.0040561F
0040560F  |. 81E3 0000FF00 AND EBX,0FF0000
00405615  |. 81E1 0000FF00 AND ECX,0FF0000
0040561B  |. 39D9          CMP ECX,EBX
0040561D  |. 75 27         JNZ SHORT RegOpt.00405646
0040561F  |> 01C0          ADD EAX,EAX
00405621  |. EB 23         JMP SHORT RegOpt.00405646
00405623  |> 8B57 FC       MOV EDX,DWORD PTR DS:[EDI-4]
00405626  |. 29D0          SUB EAX,EDX
00405628  |. EB 1C         JMP SHORT RegOpt.00405646
0040562A  |> 8B46 FC       MOV EAX,DWORD PTR DS:[ESI-4]
0040562D  |. 29D0          SUB EAX,EDX
0040562F  |. EB 15         JMP SHORT RegOpt.00405646
00405631  |> 5A            POP EDX
00405632  |. 38D9          CMP CL,BL
00405634  |. 75 10         JNZ SHORT RegOpt.00405646
00405636  |. 38FD          CMP CH,BH
00405638  |. 75 0C         JNZ SHORT RegOpt.00405646
0040563A  |. C1E9 10       SHR ECX,10
0040563D  |. C1EB 10       SHR EBX,10
00405640  |. 38D9          CMP CL,BL
00405642  |. 75 02         JNZ SHORT RegOpt.00405646
00405644  |. 38FD          CMP CH,BH
00405646  |> 5F            POP EDI
00405647  |. 5E            POP ESI
00405648  |. 5B            POP EBX
00405649  \\. C3           RETN
```

## 算法总结

1. 注册码分为 5 段，分别记为 `SN1`, `SN2`, `SN3`, `SN4`, `SN5`；
2. 分别对各段注册码进行合法性检查；
3. 第四段注册码由第一段和第二段注册码计算出来：结果是从 `SN2*SN1-3` 中取出后 4 位；
4. 第五段注册码由第二段和第三段计算出来：结果是从 `(SN3 + 0XEB) * SN2 - 5` 中取出后 4 位；
5. 最后组成形式是 `SN1-SN2-SN3-SN4-SN5`。

## 算法注册机

```c
#include <stdio.h>

#include <string.h>

#include <math.h>

main()
{ int SN1, SN2, SN3, SN4,SN5;
printf("////////////////////////////////////////////////////\n");
printf("//   WinASO Registry Optimizer V2.6 Key Generator //\n");
printf("//                                                //\n");
printf("//          Author: ocnzhao[OCN]                 //\n");
printf("//                                                //\n");
printf("//          E-mail: ocnzhao@163.com               //\n");
printf("//                                                //\n");
printf("//       OS  : WinXP, PEiD, Ollydbg, C-Free3.5    //\n");
printf("//                                                //\n");
printf("//          Date  :  2006-04-25                   //\n");
printf("//////////////////////////////////////////////////\n\n");
printf("请输入注册码第一段 SN1[4 位]: ");
scanf("%d",&SN1);
printf("请输入注册码第一段 SN2[4 位]: ");
scanf("%d",&SN2);
printf("请输入注册码第一段 SN3[4 位]: ");
scanf("%d",&SN3);
SN4=(SN2 * SN1-0x3);
if(SN4 < 0X2710)
  {
   printf("请重新输入，不符合要求");
  }
else
SN4=(SN2 * SN1-0x3)%0x2710;
SN5=((SN3 + 0XEB) * SN2 - 5)%0x2710;
 printf("注册码是%d-%d-%d-%d-%d",SN1,SN2,SN3,SN4,SN5);
 printf("\n");
}
```

## 【破解总结】

`SN:1234-2345-3456-3727-5390`

## 【版权声明】

本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
