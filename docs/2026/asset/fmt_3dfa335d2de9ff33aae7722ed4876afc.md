
```asm
0054BD92   8B45F4                 mov     eax, [ebp-$0C]          //新sn=11405-11405-11405-11405-11445

0054BD95   0FB600                 movzx   eax, byte ptr [eax]     //sn[0]

0054BD98   8B55F4                 mov     edx, [ebp-$0C]

0054BD9B   0FB65201               movzx   edx, byte ptr [edx+$01] //sn[1]

0054BD9F   2BC2                   sub     eax, edx                //sn[0]-sn[1]

.........

.........

0054BDBB   E8748EEBFF             call    00404C34

0054BDC0   88581B                 mov     [eax+$1B], bl           //bl相减结果=>sn[27]

0054BDC3   8B45F4                 mov     eax, [ebp-$0C]          //新sn=11405-11405-11405-11405-11405

0054BDC6   8B55FC                 mov     edx, [ebp-$04]          //原sn=12345-12345-12345-12345-12345

\* Reference to: System.@LStrCmp;           //比较函数

|

0054BDC9   E85A8DEBFF             call    00404B28

0054BDCE   7504                   jnz     0054BDD4  不相等就完了`~~

0054BDD0   C645FB01               mov     byte ptr [ebp-$05], $01

0054BDD4   33C0                   xor     eax, eax
```

*******

### 算法总结

先把输入的注册码连接起来

然后经过一系列的相减运算就完事~~

过程比较复杂点，但是不难 跟一下就完了

可以编写内存注册机

- 中断地址：`0054BDC6`
- 中断次数：`1`
- 第一个字节：`8B`
- 指令长度：`3`
- 内存方式：`EAX`

象我这般菜鸟 可以去下载来练练

by 微笑刺客~~

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
