

# Petite 2.x

> 原文：[`www.pediy.com/kssd/pediy02/forum178.htm`](https://www.pediy.com/kssd/pediy02/forum178.htm)

  

 简要说一下：   
1、设断点   
bpx loadlibrarya do "dd esp\->4"   
2、F12 到 EditPad.exe 领空。   
3、向上找到最近的一个 JNZ。我这里跟踪时，显示如下：   
015F:004B6C01  8BD8                MOV      EBX,EAX   
015F:004B6C03  833E00              CMP      DWORD PTR \[ESI\],00   
015F:004B6C06  0F8415020000        JZ        004B6E21    <--注意   
015F:004B6C0C  51                  PUSH      ECX   
015F:004B6C0D  51                  PUSH      ECX   
015F:004B6C0E  FF9590070000        CALL      \[EBP+00000790\]   
015F:004B6C14  85C0                TEST      EAX,EAX    <--你停在这儿   
015F:004B6C16  0F84E6000000        JZ        004B6D02   
015F:004B6C1C  8BF8                MOV      EDI,EAX   
4、禁止原断点，设新断点   
bd \*   
bpx 004B6E21   
5、F5，中断情况如下：   
015F:004B6E1C  83EB06              SUB      EBX,06   
015F:004B6E1F  EBC5                JMP      004B6DE6   
015F:004B6E21  59                  POP      ECX    <--你停在这儿   
015F:004B6E22  5E                  POP      ESI   
015F:004B6E23  FD                  STD   
015F:004B6E24  33C0                XOR      EAX,EAX   
015F:004B6E26  B965030000          MOV      ECX,00000365   
015F:004B6E2B  E8D2720400          CALL      004FE102    <--注意了，按 F8   
015F:004B6E30  0000                ADD      \[EAX\],AL   
015F:004B6E32  0000                ADD      \[EAX\],AL   
6、在 Call 004FE102 语句处 F8 进入，显示如下：   
015F:004FE101  07                  POP      ES   
015F:004FE102  5F                  POP      EDI   
015F:004FE103  F3AA                REPZ STOSB   
015F:004FE105  61                  POPAD   
015F:004FE106  669D                POPF   
015F:004FE108  83C40C              ADD      ESP,0C   
015F:004FE10B  E96860FBFF          JMP      004B4178    <--注意了，真正的 Entry Point   
7、在 JMP 语句处 Dump 内存映象   
/dump 400000 100000 c:\\temp\\dump.exe   
8、修补 dump.exe 的 VA=>RVA，Entry Point 为 004B4178。   
9、运行应该成功。   
这会很幸运，连 import table 都不用修补。我 Dump 出结果 import table 为 0000FF69C。如果不一样，改改试试。   4dd466d0e1e0cc123afbe504da2462ca