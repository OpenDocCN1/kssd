
# 源码备份 V2.20

> 原文：[`www.pediy.com/kssd/pediy06/pediy6133.htm`](https://www.pediy.com/kssd/pediy06/pediy6133.htm)

## 软件介绍

如果你是一个编程爱好者，或是网页设计师，或是一名“闪客”，那么你一定会经常与各种各样的源代码打交道。如果你以前有过因为忘记备份源代码而造成源代码丢失的惨痛经历，那么你一定知道及时备份源代码的重要性。可是市面上的备份软件要么设置复杂，要么对各种文件类型照单全收，无法适应源代码类型众多，又有很多临时文件掺杂其中的特点。于是，我就写了这个源代码备份工具。

它是专门为了备份源代码而设计的，可以定时自动备份，并对文件进行过滤，只备份源代码，而将临时文件等过滤掉。它的过滤器完全开放的，通过自定义过滤器，可以用来备份各种程序源代码。

## 下载地址

[`www.softreg.com.cn/download.asp?id=/6FAADF76-F16C-4D33-A75F-386C2499F26F/`](http://www.softreg.com.cn/download.asp?id=/6FAADF76-F16C-4D33-A75F-386C2499F26F/)

## 软件限制

还没注意。

## 作者声明

初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！如果你有条件，请支持国产软件。

## 作者

abcde12345【DFCG】

## 破解工具

`Ollydbg1.09`、`Language2K`、`Api32`

---

## 过程

1. 用 `Language2K` 检查 `"源码备份 V2.20"`，`AsPack` `1.2.12` 壳，用 `unpack` 脱壳再查为 `Delphi` 程序。

2. 运行 `Api32` 调试程序，检查会用到哪些函数，并分析可疑对象。

```
0046EA9A:RegOpenKeyExA 
(HANDLE:80000002,LPSTR:004B8094:"Software\\AIS\\DPB",DWORD:00000000,DWORD:000F003F,LPDATA:0072FC28)
```

3. 用 `Ollydbg1.09` 调试该程序，在得到的地址 `0046EA9A` 处设置断点。

5. `F9` 运行后，中断在我们设的断点上。按 `F8` 数次后，终于出现了 `RegName` 字样，等你等到我心痛。

```
004B7F0C  /.  55             PUSH EBP  
004B7F0D  |.  8BEC           MOV EBP,ESP  
004B7F0F  |.  33C9           XOR ECX,ECX  
004B7F11  |.  51             PUSH ECX  
004B7F12  |.  51             PUSH ECX  
004B7F13  |.  51             PUSH ECX  
004B7F14  |.  51             PUSH ECX  
004B7F15  |.  51             PUSH ECX  
004B7F16  |.  8945 FC        MOV DWORD PTR SS:[EBP-4],EAX  
004B7F19  |.  33C0           XOR EAX,EAX  
004B7F1B  |.  55             PUSH EBP  
004B7F1C  |.  68 7E804B00    PUSH CRDPB.004B807E  
004B7F21  |.  64:FF30        PUSH DWORD PTR FS:[EAX]  
004B7F24  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP  
004B7F27  |.  33C0           XOR EAX,EAX  
004B7F29  |.  55             PUSH EBP  
004B7F2A  |.  68 BE7F4B00    PUSH CRDPB.004B7FBE  
004B7F2F  |.  64:FF30        PUSH DWORD PTR FS:[EAX]  
004B7F32  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP  
004B7F35  |.  B2 01          MOV DL,1  
004B7F37  |.  A1 1CE84600    MOV EAX,DWORD PTR DS:[46E81C]  
004B7F3C  |.  E8 DB69FBFF    CALL CRDPB.0046E91C  
004B7F41  |.  8945 F8        MOV DWORD PTR SS:[EBP-8],EAX  
004B7F44  |.  BA 02000080    MOV EDX,80000002  
004B7F49  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004B7F4C  |.  E8 6B6AFBFF    CALL CRDPB.0046E9BC  
004B7F51  |.  33C9           XOR ECX,ECX  
004B7F53  |.  BA 94804B00    MOV EDX,CRDPB.004B8094             ;  ASCII "Software\\AIS\\DPB"  
004B7F58  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004B7F5B  |.  E8 C06AFBFF    CALL CRDPB.0046EA20  
004B7F60  |.  8D4D F4        LEA ECX,DWORD PTR SS:[EBP-C]  
004B7F63  |.  BA B0804B00    MOV EDX,CRDPB.004B80B0             ;  ASCII "RegName"  
004B7F68  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004B7F6B  |.  E8 986CFBFF    CALL CRDPB.0046EC08  
004B7F70  |.  8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]  
004B7F73  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
004B7F76  |.  05 34040000    ADD EAX,434  
004B7F7B  |.  E8 C0C7F4FF    CALL CRDPB.00404740  
004B7F80  |.  8D4D F0        LEA ECX,DWORD PTR SS:[EBP-10]  
004B7F83  |.  BA C0804B00    MOV EDX,CRDPB.004B80C0             ;  ASCII "RegCode"  
004B7F88  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004B7F8B  |.  E8 786CFBFF    CALL CRDPB.0046EC08  
004B7F90  |.  8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10]  
004B7F93  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
004B7F96  |.  05 38040000    ADD EAX,438  
004B7F9B  |.  E8 A0C7F4FF    CALL CRDPB.00404740  
004B7FA0  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004B7FA3  |.  E8 E469FBFF    CALL CRDPB.0046E98C  
004B7FA8  |.  33C0           XOR EAX,EAX  
004B7FAA  |.  5A             POP EDX  
004B7FAB  |.  59             POP ECX  
004B7FAC  |.  59             POP ECX  
004B7FAD  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX  
004B7FB0  |.  68 C57F4B00    PUSH CRDPB.004B7FC5  
004B7FB5  |>  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004B7FB8  |.  E8 C3B9F4FF    CALL CRDPB.00403980  
004B7FBE   .^\\E9 51C1F4FF    JMP CRDPB.00404114  
004B7FC3   .^ EB F0           JMP SHORT CRDPB.004B7FB5  
004B7FC5   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
004B7FC8   .  83B8 34040000>CMP DWORD PTR DS:[EAX+434],0  
004B7FCF   .  74 0C          JE SHORT CRDPB.004B7FDD  
004B7FD1   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
004B7FD4   .  83B8 38040000>CMP DWORD PTR DS:[EAX+438],0  
004B7FDB   .  75 0C          JNZ SHORT CRDPB.004B7FE9  
004B7FDD   >  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
004B7FE0   .  C680 30040000>MOV BYTE PTR DS:[EAX+430],0  
004B7FE7   .  EB 21          JMP SHORT CRDPB.004B800A  
004B7FE9   >  55             PUSH EBP  
004B7FEA   .  E8 71FEFFFF    CALL CRDPB.004B7E60      ；在这里进入算法关键 

///////////////////////  

//进入算法分析 Call:  

///////////////////////  

004B7E60  /$  55             PUSH EBP  
004B7E61  |.  8BEC           MOV EBP,ESP  
004B7E63  |.  6A 00          PUSH 0  
004B7E65  |.  53             PUSH EBX  
004B7E66  |.  56             PUSH ESI  
004B7E67  |.  8B75 08        MOV ESI,DWORD PTR SS:[EBP+8]  
004B7E6A  |.  83C6 FC       ADD ESI,-4  
004B7E6D  |.  33C0           XOR EAX,EAX  
004B7E6F  |.  55             PUSH EBP  
004B7E70  |.  68 FD7E4B00    PUSH CRDPB.004B7EFD  
004B7E75  |.  64:FF30        PUSH DWORD PTR FS:[EAX]  
004B7E78  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP  
004B7E7B  |.  8B06           MOV EAX,DWORD PTR DS:[ESI]  
004B7E7D  |.  8B80 34040000 MOV EAX,DWORD PTR DS:[EAX+434]  
004B7E83  |.  E8 1CCBF4FF    CALL CRDPB.004049A4  
004B7E88  |.  83F8 09        CMP EAX,9               ；这里用户名与 9 比较  
004B7E8B  |.  7C 07          JL SHORT CRDPB.004B7E94  
004B7E8D  |.  B8 09000000    MOV EAX,9  
004B7E92  |.  EB 0D          JMP SHORT CRDPB.004B7EA1  
004B7E94  |>  8B06           MOV EAX,DWORD PTR DS:[ESI]  
004B7E96  |.  8B80 34040000 MOV EAX,DWORD PTR DS:[EAX+434]  
004B7E9C  |.  E8 03CBF4FF    CALL CRDPB.004049A4  
004B7EA1  |>  33DB           XOR EBX,EBX  
004B7EA3  |.  85C0           TEST EAX,EAX  
004B7EA5  |.  7E 18          JLE SHORT CRDPB.004B7EBF  
004B7EA7  |.  BA 01000000    MOV EDX,1  
004B7EAC  |>  8B0E           /MOV ECX,DWORD PTR DS:[ESI]  
004B7EAE  |.  8B89 34040000 |MOV ECX,DWORD PTR DS:[ECX+434]  
004B7EB4  |.  0FB64C11 FF   |MOVZX ECX,BYTE PTR DS:[ECX+EDX-1]  
004B7EB9  |.  03D9           |ADD EBX,ECX           ；这里进行用户名累加到 ebx  
004B7EBB  |.  42             |INC EDX  
004B7EBC  |.  48             |DEC EAX               ；用户名只加到前 9 位，后面不管了  
004B7EBD  |.^ 75 ED          \JNZ SHORT CRDPB.004B7EAC  
004B7EBF  |>  81C3 5D4F0000 ADD EBX,4F5D            ；再把 累加和 + 4f5d  
004B7EC5  |.  8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]  
004B7EC8  |.  8BC3           MOV EAX,EBX  
004B7ECA  |.  E8 3111F5FF    CALL CRDPB.00409000  
004B7ECF  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]  
004B7ED2  |.  8B06           MOV EAX,DWORD PTR DS:[ESI]  
004B7ED4  |.  8B80 38040000 MOV EAX,DWORD PTR DS:[EAX+438]  
004B7EDA  |.  E8 09CCF4FF    CALL CRDPB.00404AE8     ；真假码在这里比较  
004B7EDF  |.  75 04          JNZ SHORT CRDPB.004B7EE5  
004B7EE1  |.  B3 01          MOV BL,1  
004B7EE3  |.  EB 02          JMP SHORT CRDPB.004B7EE7  
004B7EE5  |>  33DB           XOR EBX,EBX  
004B7EE7  |>  33C0           XOR EAX,EAX  
004B7EE9  |.  5A             POP EDX  
004B7EEA  |.  59             POP ECX  
004B7EEB  |.  59             POP ECX  
004B7EEC  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX  
004B7EEF  |.  68 047F4B00    PUSH CRDPB.004B7F04  
004B7EF4  |>  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]  
004B7EF7  |.  E8 F0C7F4FF    CALL CRDPB.004046EC  
004B7EFC  \\.  C3             RETN
```

## 源码备份 V2.20 Java 注册机

```java
public class SourceBakKey  

{  

  public  static void main(String[] args)  

      {  

              String name = new String("abcde12345[DFCG]");  
```



```java
            long sum = 0; 

            for(int i=0;i<name.length();i++)  

            {  

                            char c = name.charAt(i);  

                            sum = sum + c ;  

                            if(i==8)  

                            {  

                                break;  

                            }  

            } 

            long sn  = sum + 0x4F5D; 

            System.out .println("源码备份 V2.20  注册机　Cracked   By   CrazyXY\[DFCG\]－－－");  

            System.out .println("用户名：" +name);  

            System.out .println("注册码：" +sn);  

            System.out .println("－－－--------------------------－－－---------------------");  

      }  

} 
```

`4dd466d0e1e0cc123afbe504da2462ca`
