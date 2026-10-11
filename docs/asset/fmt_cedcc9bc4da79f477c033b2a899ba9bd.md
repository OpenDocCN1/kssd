
# 汇编代码分析：循环与字符串转换逻辑

## 1. 字符串操作准备

```
004AD105   . 8D55 80        LEA EDX,DWORD PTR SS:[EBP-80]  
004AD108   . 52             PUSH EDX  
004AD109   . 8D45 B0        LEA EAX,DWORD PTR SS:[EBP-50]  
004AD10C   . 50             PUSH EAX  
004AD10D   . 895D 88        MOV DWORD PTR SS:[EBP-78],EBX  ; 机器码与注册名连接字符串送[EBP-78]  
004AD110   . C745 80 084000>MOV DWORD PTR SS:[EBP-80],4008  
004AD117   . FF15 AC134000  CALL DWORD PTR DS:[<&MSVBVM60.#619>]            ;  MSVBVM60.rtcRightCharVar  
```
`; 取机器码与注册名连接字符串右边 5 个字符组成字符串`
`; (为叙述方便设为 R5)`

```
004AD11D   . 8D4D A0        LEA ECX,DWORD PTR SS:[EBP-60]  
004AD120   . 51             PUSH ECX  
004AD121   . 56             PUSH ESI                     ；初始为 1  
004AD122   . 8D55 B0        LEA EDX,DWORD PTR SS:[EBP-50]  
004AD125   . 52             PUSH EDX                      ;R5 入栈  
004AD126   . 8D45 90        LEA EAX,DWORD PTR SS:[EBP-70]    ;[EBP-70]为返回值地址  
004AD129   . 50             PUSH EAX  
004AD12A   . C745 A8 010000>MOV DWORD PTR SS:[EBP-58],1  
004AD131   . C745 A0 020000>MOV DWORD PTR SS:[EBP-60],2  
```

## 2. 循环体：字符提取与数值计算

```
004AD138   . FF15 84114000  CALL DWORD PTR DS:[<&MSVBVM60.#632>]            ;  MSVBVM60.rtcMidCharVar  
```
`; 依次取 R5 每个字符`

```
004AD13E   . 8D4D 90        LEA ECX,DWORD PTR SS:[EBP-70]  
004AD141   . 51             PUSH ECX  
004AD142   . 8D55 C4        LEA EDX,DWORD PTR SS:[EBP-3C]  
004AD145   . 52             PUSH EDX  
004AD146   . FF15 94124000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVarVal>]   ;  MSVBVM60.__vbaStrVarVal  
```
`; 转字符`

```
004AD14C   . 50             PUSH EAX  
004AD14D   . FF15 20104000  CALL DWORD PTR DS:[<&MSVBVM60.#693>]            ;  MSVBVM60.rtcByteValueBstr  
```
`; 求字符字节 ASCII 值 (当为汉字时取的是低位字节)`

```
004AD153   . DB85 58FFFFFF  FILD DWORD PTR SS:[EBP-A8]       ; [EBP-A8]装入浮点寄存器  
004AD159   . 83EC 08        SUB ESP,8  
004AD15C   . 8885 6CFFFFFF  MOV BYTE PTR SS:[EBP-94],AL      ;L5 每个字符字节 ASCII 值依次送[EBP-94]  
004AD162   . DD1C24         FSTP QWORD PTR SS:[ESP]  
004AD165   . 68 00002440    PUSH 40240000  
004AD16A   . 6A 00          PUSH 0  
```

## 3. 幂运算与除法逻辑

```
004AD16C   . FF15 18134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaPowerR8>]     ;  MSVBVM60.__vbaPowerR8  
```
`; 求 10 的乘方.`

```
004AD172   . 8B85 6CFFFFFF  MOV EAX,DWORD PTR SS:[EBP-94]  
004AD178   . 25 FF000000    AND EAX,0FF  
004AD17D   . 99             CDQ  
004AD17E   . B9 0A000000    MOV ECX,0A  
004AD183   . F7F9           IDIV ECX                     ; 除 10  
004AD185   . 8995 50FFFFFF  MOV DWORD PTR SS:[EBP-B0],EDX  
004AD18B   . DB85 50FFFFFF  FILD DWORD PTR SS:[EBP-B0]  
004AD191   . DEC9           FMULP ST(1),ST               ; 余数与上面 10 的乘方相乘  
004AD193   . DA45 D8        FIADD DWORD PTR SS:[EBP-28]  ; 循环累加到[EBP-28]  
```

## 4. 类型转换与清理

```
004AD196   . FF15 6C134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFpI4>]        ;  MSVBVM60.__vbaFpI4  
```
`; 转整数`

```
004AD19C   . 8D4D C4        LEA ECX,DWORD PTR SS:[EBP-3C]  
004AD19F   . 8945 D8        MOV DWORD PTR SS:[EBP-28],EAX    ;  整数值送[EBP-28]  
004AD1A2   . FF15 E8134000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeStr>]     ;  MSVBVM60.__vbaFreeStr  
004AD1A8   . 8D55 90        LEA EDX,DWORD PTR SS:[EBP-70]  
004AD1AB   . 52             PUSH EDX  
004AD1AC   . 8D45 A0        LEA EAX,DWORD PTR SS:[EBP-60]  
004AD1AF   . 50             PUSH EAX  
004AD1B0   . 8D4D B0        LEA ECX,DWORD PTR SS:[EBP-50]  
004AD1B3   . 51             PUSH ECX  
004AD1B4   . 6A 03          PUSH 3  
004AD1B6   . FF15 44104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeVarList>] ;  MSVBVM60.__vbaFreeVarList  
```

## 5. 循环控制与跳转

```
004AD1BC   . 8B4D DC        MOV ECX,DWORD PTR SS:[EBP-24]   ;[EBP-24]中值送[EBP-24]  
004AD1BF   . 83C4 10        ADD ESP,10  
004AD1C2   . B8 01000000    MOV EAX,1  
004AD1C7   . 46             INC ESI                    ;esi 加 1  
004AD1C8   . 03F8           ADD EDI,EAX                ;edi 加 1  
004AD1CA   . ^E9 5FFEFFFF   JMP unpacked.004AD02E      ; 跳向循环开头  
```

### 循环结束后的处理

循环结束后来到这里:

```
004AD1CF   > 8B55 D8        MOV EDX,DWORD PTR SS:[EBP-28]  
004AD1D2   . 81F1 CADE0000  XOR ECX,0DECA      ;  上面第一部分累加值与 57034(十进制)作异或  
004AD1D8   . 81F2 13700000  XOR EDX,7013       ;  上面第二部分累加值与 28691(十进制)作异或  
004AD1DE   . 81F9 9F860100  CMP ECX,1869F      ;  与 99999 作比较,若大于就转为小于 99999 的数  
004AD1E4   . 8955 D8        MOV DWORD PTR SS:[EBP-28],EDX ; 异或值[EBP-28]  
004AD1E7   . 7E 16          JLE SHORT unpacked.004AD1FF ;  
```

______________________________________________________________________________

```
004AD1E9   . B8 67666666    MOV EAX,66666667    ;这段是把 ECX 中数转为小于 99999(十进制)的数  
004AD1EE   . F7E9           IMUL ECX  
004AD1F0   . C1FA 02        SAR EDX,2  
004AD1F3   . 8BC2           MOV EAX,EDX  
004AD1F5   . C1E8 1F        SHR EAX,1F  
004AD1F8   . 03D0           ADD EDX,EAX  
004AD1FA   . 8955 DC        MOV DWORD PTR SS:[EBP-24],EDX  
004AD1FD   . 8BCA           MOV ECX,EDX  
```

______________________________________________________________________________

```
004AD1FF   > BE 10270000    MOV ESI,2710      ;ESI=10000(十进制)  
004AD204   . 3BCE           CMP ECX,ESI       ;与 10000(十进制)作比较,若小于就加 10000  
004AD206   . 7D 02          JGE SHORT unpacked.004AD20A  
004AD208   . 03CE           ADD ECX,ESI       ;加 10000(十进制)  
```

\----------------------------------------------对第二部分异或值作同样处理(保证数为 5 位,在 10000--99999 之间)

```
004AD20A   > 8B55 D8        MOV EDX,DWORD PTR SS:[EBP-28]  
004AD20D   . 81FA 9F860100  CMP EDX,1869F  
004AD213   . 7E 14          JLE SHORT unpacked.004AD229  
004AD215   . B8 67666666    MOV EAX,66666667  
004AD21A   . F7EA           IMUL EDX  
004AD21C   . C1FA 02        SAR EDX,2  
004AD21F   . 8BC2           MOV EAX,EDX  
004AD221   . C1E8 1F        SHR EAX,1F  
004AD224   . 03D0           ADD EDX,EAX  
004AD226   . 8955 D8        MOV DWORD PTR SS:[EBP-28],EDX  
```

```
004AD229   > 8B45 D8        MOV EAX,DWORD PTR SS:[EBP-28]  
004AD22C   . 3BC6           CMP EAX,ESI  
004AD22E   . 7D 05          JGE SHORT unpacked.004AD235  
004AD230   . 03C6           ADD EAX,ESI  
004AD232   . 8945 D8        MOV DWORD PTR SS:[EBP-28],EAX  
```

\-------------------------------------------------------------

```
004AD235   > 8B3D 18104000  MOV EDI,DWORD PTR DS:[<&MSVBVM60.__vbaStrI4>]    ;  MSVBVM60.__vbaStrI4  
004AD23B   . 51             PUSH ECX  
004AD23C   . FFD7           CALL EDI                     ;  <&MSVBVM60.__vbaStrI4>  
```
`; 整数转字符串`

```
004AD23E   . 8B35 98134000  MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaStrMove>]  ;  MSVBVM60.__vbaStrMove  
004AD244   . 8BD0           MOV EDX,EAX  
004AD246   . 8D4D C4        LEA ECX,DWORD PTR SS:[EBP-3C]  
004AD249   . FFD6           CALL ESI                     ;  <&MSVBVM60.__vbaStrMove>  
```

```
004AD24B   . 8B4D D8        MOV ECX,DWORD PTR SS:[EBP-28]  
004AD24E   . 50             PUSH EAX  
004AD24F   . 51             PUSH ECX  
004AD250   . FFD7           CALL EDI                     ;整数转字符串  
004AD252   . 8BD0           MOV EDX,EAX  
004AD254   . 8D4D C0        LEA ECX,DWORD PTR SS:[EBP-40]  
004AD257   . FFD6           CALL ESI  
004AD259   . 50             PUSH EAX  
```
