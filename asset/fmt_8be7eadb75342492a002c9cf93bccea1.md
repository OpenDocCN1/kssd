
00407633   .   8B7E 74             MOV     EDI, DWORD PTR [ESI+74]  
00407636   .   E8 6D1E0200         CALL    004294A8  
0040763B   .   8B40 04            MOV     EAX, DWORD PTR [EAX+4]  
0040763E   .   57                 PUSH    EDI                       ; /Arg3  
0040763F   .   68 D8FD4200        PUSH    0042FDD8                  ; |Arg2 = 0042FDD8 ASCII "registration_code"  
00407644   .   68 ECFD4200        PUSH    0042FDEC                  ; |Arg1 = 0042FDEC ASCII "Option"  
00407649   .   8BC8               MOV     ECX, EAX                  ; |  
0040764B   .   E8 F30D0200        CALL    00428443                  ; \FileTree.00428443  
00407650   .   5B                 POP     EBX  
00407651   .   5F                 POP     EDI  
00407652   .   8BCE               MOV     ECX, ESI  
00407654   .   5E                 POP     ESI  
00407655   .   5D                 POP     EBP  
00407656   .   83C4 0C            ADD     ESP, 0C  
00407659   .   E9 9B8F0100        JMP     004205F9  

注册码出错的地方：

0040765E   >   6A 00              PUSH    0  
00407660   .   6A 00              PUSH    0  
00407662   .   68 D0FE4200        PUSH    0042FED0                  ;  ASCII "Registration failed!"  
00407667   .   E8 77100200        CALL    004286E3  

经过上面的分析，算法大致可以描述出来了，伪代码如下：

```c
string username;
string regcode;
int r[4];

r[1] = (username[1] or 0x45) mod 10;
r[2] = (username[2] or 0x42) mod 10;
r[3] = (username[1] or 0x43) mod 10;
r[4] = (username[2] or 0x44) mod 10;

int sum = username 逐位求和;

regcode[1] = r[1] + 0x30;
regcode[2] = r[2] + 0x30;
regcode[3] = r[3] + 0x30;
regcode[4] = r[4] + 0x30;
regcode[5] = (sum mod 10) + 0x30;
regcode[6..8]任意，当然也可以多于 8 位。
```

算法不复杂，但是如果还想偷懒一下的话就直接这样：

```c
printf("RegCode: 83913146\n");
```

--------------------------------------------------------------------------------

**【版权声明】**：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2006 年 07 月 27 日 11:11:11

4dd466d0e1e0cc123afbe504da2462ca
