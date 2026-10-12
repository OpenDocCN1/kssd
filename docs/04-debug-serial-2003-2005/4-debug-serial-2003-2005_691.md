
# Ap TIFF to PDF convert 2.3 算法分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-380.htm`](https://www.pediy.com/kssd/pediy07/pediy7-380.htm)

【破文标题】：Ap TIFF to PDF convert 2.3 算法分析 + 注册机

【破文作者】：KuNgBiM\[DFCG\]【作者邮箱】：gb\_1227@163.com

【软件名称】：Ap TIFF to PDF convert 2.3

【开 发 商】：http://www.adultpdf.com/

【下载地址】：http://nj.onlinedown.net/soft/34222.htm

【保护方式】：注册码 + 启动 NAG + 功能限制

【编译语言】：Borland C++ 1999

【调试环境】：WinXP、PEiD、Ollydbg、Importrec New Fix、LordPE

【破解日期】：2005-09-11

【破解目的】：推广使用 ESP 定律脱壳，以及研究算法分析

【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

—————————————————————————————————

【破解过程】：

\\  【脱壳过程】  \\

侦壳脱壳：用 `PEiD` 查壳，`ASPack` 2.12 -> `Alexey Solodovnikov` 加壳

使用法宝：我们既然知道了是 `ASPack` 所加壳保护的，所以拿出 `Ollydbg` 结合文章目的手动脱之~~

————————————————————

`Ollydbg` 载入主程序：

```
00536001 >  60               pushad                                 ; ASPack 2.12 标准壳入口，F8 一次
00536002    E8 03000000      call TIFF_to_.0053600A                 ; 来到这里，这时查看寄存器窗口
00536007  - E9 EB045D45      jmp 45B064F7
0053600C    55               push ebp
0053600D    C3               retn
........
```

\\ 寄存器 \\

```
EAX 00000000
ECX 0012FFB0
EDX 7C92EB94 ntdll.KiFastSystemCallRet
EBX 7FFDA000
ESP 0012FFA4
EBP 0012FFF0
ESI FFFFFFFF
EDI 7C930738 ntdll.7C930738
EIP 00536002 TIFF_to_.00536002
```

\\

根据 `ESP` 定律规则，现在在命令栏中下 `hr 0012ffa4` 命令，回车，`F9` 运行：

```
005363B0   /75 08           jnz short TIFF_to_.005363BA            ; 这里断下，继续 F8
005363B2   |B8 01000000     mov eax,1
005363B7   |C2 0C00         retn 0C
005363BA   \68 A0144000     push TIFF_to_.004014A0                 ; 004014A0 就是 OEP 了！
005363BF    C3               retn                                  ; 再 F8 一次，飞向光明之颠！^_^
........
004014A0   /EB 10           jmp short TIFF_to_.004014B2            ; 在这儿用 LordPE 纠正 ImageSize 后完全 Dump 这个进程
004014A2   |66:623A         bound di,dword ptr ds:[edx]
004014A5   |43               inc ebx
004014A6   |2B2B             sub ebp,dword ptr ds:[ebx]
004014A8   |48               dec eax
004014A9   |4F               dec edi
004014AA   |4F               dec edi
004014AB   |4B               dec ebx
004014AC   |90               nop
004014AD  -|E9 98804D00     jmp 008D954A
004014B2   \A1 8B804D00     mov eax,dword ptr ds:[4D808B]
004014B7    C1E0 02         shl eax,2
004014BA    A3 8F804D00     mov dword ptr ds:[4D808F],eax
004014BF    52               push edx
004014C0    6A 00            push 0
004014C2    E8 395F0D00     call TIFF_to_.004D7400                ; jmp to kernel32.GetModuleHandleA
........
```

*********************************************************************

脱壳修复：运行 `LordPE`，`Dump` 整个进程，然后打开 `ImportREC` 找到该程序对应进程，`OEP` 填 `000014A0`，“自动搜索 IAT”，“获取输入表”，

          唯一的一个指针有效，“修复抓取文件”，`OK`，脱壳修复完成！运行成功！`Fix Dump!!`

用 `PEiD` 再次检测，程序为 `Borland C++` 1999 编译。优化一下，原始：`430 KB` -->脱壳优化：`1.18 MB`

—————————————————————————————————

\\  【破解过程】  \\

注册运行有对话框提示`"Series number error,please check it and try again."`

命令栏内下断：`bpx MessageBoxA`

`F9` 运行，输入试炼信息：

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

E-mail：gb\_1227@163.com

Serial number：9876543210

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
0040D4F8    55               push ebp                               ; 再次运行输入试炼信息后，不断下！
0040D4F9    8BEC             mov ebp,esp
0040D4FB    83C4 B4          add esp,-4C
0040D4FE    53               push ebx
0040D4FF    56               push esi
0040D500    57               push edi
0040D501    8BD8             mov ebx,eax
0040D503    BE E3D14D00      mov esi,dumped_.004DD1E3
0040D508    8D7D C8          lea edi,dword ptr ss:[ebp-38]
0040D50B    B8 30D44D00      mov eax,dumped_.004DD430
0040D510    E8 4BE00B00      call dumped_.004CB560
0040D515    66:C747 10 1400  mov word ptr ds:[edi+10],14
0040D51B    33D2             xor edx,edx
0040D51D    8955 FC          mov dword ptr ss:[ebp-4],edx
0040D520    8D55 FC          lea edx,dword ptr ss:[ebp-4]
0040D523    FF47 1C          inc dword ptr ds:[edi+1C]
0040D526    8B83 28050000    mov eax,dword ptr ds:[ebx+528]
0040D52C    E8 5B570600      call dumped_.00472C8C                   ; 获取假注册码
0040D531    66:C747 10 0800  mov word ptr ds:[edi+10],8
0040D537    66:C747 10 2000  mov word ptr ds:[edi+10],20
0040D53D    33C9             xor ecx,ecx
0040D53F    894D F8          mov dword ptr ss:[ebp-8],ecx
0040D542    8D55 F8          lea edx,dword ptr ss:[ebp-8]
0040D545    FF47 1C          inc dword ptr ds:[edi+1C]
0040D548    8B83 08050000    mov eax,dword ptr ds:[ebx+508]
0040D54E    E8 39570600      call dumped_.00472C8C                   ; 获取 E-mail
0040D553    66:C747 10 0800  mov word ptr ds:[edi+10],8
0040D559    837D FC 00       cmp dword ptr ss:[ebp-4],0              ; 判断 E-mail 是否为空?不是则继续！
0040D55D    8D4D B4          lea ecx,dword ptr ss:[ebp-4C]
0040D560    74 05            je short dumped_.0040D567
0040D562    8B55 FC          mov edx,dword ptr ss:[ebp-4]            ; 把 gb_1227@163.com 送到 edx 里
0040D565    EB 03            jmp short dumped_.0040D56A
0040D567    8D56 1C          lea edx,dword ptr ds:[esi+1C]
0040D56A    8BC3             mov eax,ebx
0040D56C    E8 DF030000      call dumped_.0040D950                   ; ★算法 CALL 跟进 F7★
0040D571    837D F8 00       cmp dword ptr ss:[ebp-8],0              ; 判断试炼码是否为空？
0040D575    74 05            je short dumped_.0040D57C
0040D577    8B55 F8          mov edx,dword ptr ss:[ebp-8]
0040D57A    EB 03            jmp short dumped_.0040D57F
0040D57C    8D56 1D          lea edx,dword ptr ds:[esi+1D]
0040D57F    8BC3             mov eax,ebx
0040D581    E8 AA060000      call dumped_.0040DC30
0040D586    84C0             test al,al
0040D588    0F85 97000000    jnz dumped_.0040D625
0040D58E    6A 10            push 10
0040D590    837D F8 00       cmp dword ptr ss:[ebp-8],0
0040D594    74 05            je short dumped_.0040D59B
0040D596    8B4D F8          mov ecx,dword ptr ss:[ebp-8]
0040D599    EB 03            jmp short dumped_.0040D59E
0040D59B    8D4E 1E          lea ecx,dword ptr ds:[esi+1E]
0040D59E    51               push ecx
0040D59F    8D45 B4          lea eax,dword ptr ss:[ebp-4C]
0040D5A2    50               push eax                                  ; 真码出现在 eax 中，★内存注册机★
0040D5A3    E8 6CDE0B00      call dumped_.004CB414
0040D5A8    83C4 0C          add esp,0C
0040D5AB    85C0             test eax,eax
0040D5AD    74 76            je short dumped_.0040D625                ; 真假码比较，不跳则挂！
0040D5AF    66:C747 10 2C00  mov word ptr ds:[edi+10],2C
0040D5B5    8D56 1F          lea edx,dword ptr ds:[esi+1F]
0040D5B8    8D45 F4          lea eax,dword ptr ss:[ebp-C]
0040D5BB    E8 88920C00      call dumped_.004D6848
0040D5C0    FF47 1C          inc dword ptr ds:[edi+1C]
0040D5C3    8B10             mov edx,dword ptr ds:[eax]
0040D5C5    8B83 14050000    mov eax,dword ptr ds:[ebx+514]
0040D5CB    E8 EC560600      call dumped_.00472CBC
0040D5D0    FF4F 1C          dec dword ptr ds:[edi+1C]
0040D5D3    8D45 F4          lea eax,dword ptr ss:[ebp-C]
0040D5D6    BA 02000000      mov edx,2
0040D5DB    E8 C8930C00      call dumped_.004D69A8
```



```
0040D5E0    6A 10             push 10  

0040D5E2    8D4E 60           lea ecx,dword ptr ds:[esi+60]  

0040D5E5    51               push ecx  

0040D5E6    8D46 2D          lea eax,dword ptr ds:[esi+2D]  

0040D5E9    50               push eax  

0040D5EA    8BC3             mov eax,ebx  

0040D5EC    E8 AFBD0600      call dumped_.004793A0  

0040D5F1    50               push eax  

0040D5F2    E8 C5A50C00      call <jmp.&USER32.MessageBoxA>               ; 这里被断下，向上看，在 0040D4F8 处下断，重新运行！  

........  

========================= 跟进 0040D56C    E8 DF030000     call dumped_.0040D950 =========================  

0040D950    53               push ebx  

0040D951    56               push esi  

0040D952    57               push edi  

0040D953    55               push ebp  

0040D954    81C4 68FFFFFF    add esp,-98  

0040D95A    8BE9             mov ebp,ecx  

0040D95C    8BDA             mov ebx,edx  

0040D95E    85DB             test ebx,ebx  

0040D960    0F84 A3000000    je dumped_.0040DA09  

0040D966    85ED             test ebp,ebp  

0040D968    0F84 9B000000    je dumped_.0040DA09  

0040D96E    6A 11            push 11  

0040D970    6A 00            push 0  

0040D972    8D8424 8C000000  lea eax,dword ptr ss:[esp+8C]  

0040D979    50               push eax  

0040D97A    E8 59D70B00      call dumped_.004CB0D8  

0040D97F    83C4 0C          add esp,0C  

0040D982    BE BFD14D00      mov esi,dumped_.004DD1BF                           ; ASCII "PX4VUTE8Q1YONML6GIHJZFSDCBA9R7K5W32"  

0040D987    8D7C24 08        lea edi,dword ptr ss:[esp+8]                         ; 上面的字符串为查表用的密码表  

0040D98B    B9 09000000      mov ecx,9  

0040D990    F3:A5            rep movs dword ptr es:[edi],dword ptr ds:[esi]  

0040D992    8D4424 2C        lea eax,dword ptr ss:[esp+2C]  

0040D996    50               push eax  

0040D997    E8 C80D0000      call dumped_.0040E764                               ; ★算法 CALL 继续 F7 跟进★  

0040D99C    59               pop ecx  

........  

========================= 跟进 0040D997    E8 C80D0000     call dumped_.0040E764 =========================  

0040E764    55               push ebp                                            ; 我跟踪很多 MD5 加密的软件，一看就知道是 MD5 算法常数  

0040E765    8BEC             mov ebp,esp  

0040E767    8B45 08          mov eax,dword ptr ss:[ebp+8]  

0040E76A    33D2             xor edx,edx  

0040E76C    8950 14          mov dword ptr ds:[eax+14],edx  

0040E76F    8950 10          mov dword ptr ds:[eax+10],edx  

0040E772    C700 01234567    mov dword ptr ds:[eax],67452301                      ; MD5 算法常数 1  

0040E778    C740 04 89ABCDE  mov dword ptr ds:[eax+4],EFCDAB89                   ; MD5 算法常数 2  

0040E77F    C740 08 FEDCBA9  mov dword ptr ds:[eax+8],98BADCFE                   ; MD5 算法常数 3  

0040E786    C740 0C 7654321  mov dword ptr ds:[eax+C],10325476                   ; MD5 算法常数 4  

0040E78D    5D               pop ebp  

0040E78E    C3               retn  

........  

==========================================================================================================  

继续跟踪：  

0040D99D    53               push ebx                                            ; 取 gb_1227@163.com 压栈  

0040D99E    E8 69D80B00      call dumped_.004CB20C  

0040D9A3    59               pop ecx  

0040D9A4    50               push eax  

0040D9A5    53               push ebx  

0040D9A6    8D5424 34        lea edx,dword ptr ss:[esp+34]  

0040D9AA    52               push edx  

0040D9AB    E8 E00D0000      call dumped_.0040E790  

0040D9B0    83C4 0C          add esp,0C  

0040D9B3    8D4C24 2C        lea ecx,dword ptr ss:[esp+2C]  

0040D9B7    51               push ecx  

0040D9B8    8D8424 88000000  lea eax,dword ptr ss:[esp+88]  

0040D9BF    50               push eax  

0040D9C0    E8 630E0000      call dumped_.0040E828  

0040D9C5    83C4 08          add esp,8                                             ; 上面几个 CALL 是用 MD5 把“gb_1227@163.com”进行转化。  

0040D9C8    33FF             xor edi,edi                                           ; 转化后的数值为：001BC8640A1A1446F51DF6184B6294EE  

0040D9CA    8BF5             mov esi,ebp                                           ; 下面开始用 MD5 转化后的数值进行运算了。  

0040D9CC    8D9C24 84000000  lea ebx,dword ptr ss:[esp+84]                           ; 数据值：00 1B C8 64 0A 1A 14 46 F5 1D F6 18 4B 62 94 EE  

0040D9D3    33C0             xor eax,eax  

0040D9D5    8A03             mov al,byte ptr ds:[ebx]                                ; 取 00 送到 al 里  

0040D9D7    B9 23000000      mov ecx,23                                              ; 把 0x23 送到 ecx 里  

0040D9DC    99               cdq  

0040D9DD    F7F9             idiv ecx                                              ; s[i]%=0x23;对 0x23 取余,余数在 edx  

0040D9DF    33C0             xor eax,eax                                           ; eax 异或清零  

0040D9E1    8A4414 08        mov al,byte ptr ss:[esp+edx+8]                         ; 根据上面提到的字符串，用余数查表  

0040D9E5    50               push eax  

0040D9E6    68 4CD34D00      push dumped_.004DD34C                                  ; ASCII "%c"  

0040D9EB    8D5424 08        lea edx,dword ptr ss:[esp+8]  

0040D9EF    52               push edx                                              ; 然后转化成相应字符  

0040D9F0    E8 29A30C00      call <jmp.&USER32.wsprintfA>  

0040D9F5    83C4 0C          add esp,0C  

0040D9F8    8A0C24           mov cl,byte ptr ss:[esp]  

0040D9FB    880E             mov byte ptr ds:[esi],cl                                ; 把得到的字符存起来  

0040D9FD    47               inc edi  

0040D9FE    46               inc esi  

0040D9FF    43               inc ebx  

0040DA00    83FF 10          cmp edi,10  

0040DA03  ^ 7C CE            jl short dumped_.0040D9D3                               ; 这里就是循环，取数直到取完 16 个数为止  

0040DA05    C645 11 00       mov byte ptr ss:[ebp+11],0  

0040DA09    81C4 98000000    add esp,98  

0040DA0F    5D               pop ebp                                               ; 真码出现：P9BKYAZPP7XCTRQR  

0040DA10    5F               pop edi  

0040DA11    5E               pop esi  

0040DA12    5B               pop ebx  

0040DA13    C3               retn                                                  ; 返回程序  

........  

-------------------------------------------------------------------------------------------------------------------------
```

### 【算法总结】

注册验证比较简单：

1. 把 `E-mail` 转换为 `MD5` 值，然后每 2 个单位的 `MD5` 值为分隔一次，共分成为 16 段数据值
2. 然后每段数据值分别与 `0x23` 取余
3. 最后以求得的余数分别查表（密码表：`PX4VUTE8Q1YONML6GIHJZFSDCBA9R7K5W32`）
4. 查表完毕后连接起来作为注册码

### 【VC6 注册机主要代码】

(最近在学 `VC`，所以将就练习一下吧，这次没帖出 `MD5` 模板，不过去网上随便找一个就行了)

```c
#include <stdio.h>

main()
{
    char KEY[255]={"PX4VUTE8Q1YONML6GIHJZFSDCBA9R7K5W32"};

    int s[255]={
                    0xFD,0x39,0x3F,0x19,0x6B,0xB6,0x0E,0xAB,
                    0xD6,0x10,0xAC,0x80,0x97,0x25,0x39,0xC0,
                    };

    int i,n,j;

    printf("Cracked by KuNgBiM[DFCG]\n");

    printf("\n");

    printf("You serial number is:");

             for(i=0;i<16;i++)
                {
                  s[i]%=0x23;

                  j=s[i];

                  printf("%c",KEY[j]);

                 }
}
```

### 【内存注册机】

*   **中断地址**：`40D5A2`
*   **中断次数**：`1`
*   **第一字节**：`50`
*   **指令长度**：`1`
*   **内存方式**：`EAX`

### 【完美爆破验证点】

```
0040D76F    0F84 01010000    je dumped_.0040D876                            ; nop 掉！
```

### 【注册信息】

*   **E-mail**：`gb_1227@163.com`
*   **Serial number**：`P9BKYAZPP7XCTRQR`

--------------------------------------------------------------------------------------------

版权所有（C）2005 `KuNgBiM[DFCG]`           Copyright (C) 2005 `KuNgBiM[DFCG]`

--------------------------------------------------------------------------------------------

      `UnPacked.Cracked By KuNgBiM[DFCG]`

                  2005-09-11

                  04:49:35 AM

`4dd466d0e1e0cc123afbe504da2462ca`
