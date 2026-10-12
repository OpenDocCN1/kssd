
```
00442F84   .  8D85 4CFFFFFF LEA     EAX, DWORD PTR SS:[EBP-B4]   

00442F8A   .  50            PUSH    EAX   

00442F8B   .  E8 DCF7FBFF   CALL    <JMP.&msvbvm50.__vbaStrVarMove>   

00442F90   .  8BD0          MOV     EDX, EAX   

00442F92   .  8D4D DC       LEA     ECX, DWORD PTR SS:[EBP-24]   

00442F95   .  E8 4EF7FBFF   CALL    <JMP.&msvbvm50.__vbaStrMove>   

00442F9A   .  8D85 4CFFFFFF LEA     EAX, DWORD PTR SS:[EBP-B4]   

00442FA0   .  50            PUSH    EAX   

00442FA1   .  8D85 5CFFFFFF LEA     EAX, DWORD PTR SS:[EBP-A4]   

00442FA7   .  50            PUSH    EAX   

00442FA8   .  6A 02         PUSH    2   

00442FAA   .  E8 FDF6FBFF   CALL    <JMP.&msvbvm50.__vbaFreeVarList>   

00442FAF   .  83C4 0C       ADD     ESP, 0C   

00442FB2   .  8B45 E0       MOV     EAX, DWORD PTR SS:[EBP-20]   

00442FB5   .  40            INC     EAX   

00442FB6   .  0F80 37020000 JO      unpack_.004431F3   

00442FBC   MOV     DWORD PTR SS:[EBP-20], EAX   

00442FBF   JMP     unpack_.00442D1D********跳回!循环，直到取出的 4 位试验码计算完毕!   

00442FC4   PUSH    DWORD PTR SS:[EBP-24]   

00442FC7   CALL    <JMP.&msvbvm50.__vbaLenBstr>   

00442FCC   CMP     EAX, 4********************EAX 的值是上述运算的结果的位数：C(即 12)位!   

00442FCF   JLE     unpack_.00443075****************小于等于就跳!   

00442FD5   PUSH    DWORD PTR SS:[EBP-24]**********计算的值 408494616710 入栈!   

00442FD8   CALL    <JMP.&msvbvm50.__vbaLenBstr>   

00442FDD   SUB     EAX, 5**********************位数 C 再减去 5，EAX=C-5=7   

00442FE0   JO      unpack_.004431F3   

00442FE6   MOV     DWORD PTR SS:[EBP-9C], EAX********把运算的结果：7 保存!   

00442FEC   MOV     DWORD PTR SS:[EBP-A4], 3   

00442FF6   LEA     EAX, DWORD PTR SS:[EBP-A4]   

00442FFC   PUSH    EAX   

00442FFD   PUSH    4   

00442FFF   PUSH    DWORD PTR SS:[EBP-24]**********计算的值 408494616710 入栈!   

00443002   CALL    <JMP.&msvbvm50.rtcMidCharBstr>********此 CALL 对上述结果再计算，计算是这样：把前 3 位去掉，   

*************************再反后两位去掉，得到中间的 7 位值是：4946167!   

00443007   MOV     EDX, EAX***********把这个值移入 EDX 中!   

00443009   LEA     ECX, DWORD PTR SS:[EBP-70]   

0044300C   CALL    <JMP.&msvbvm50.__vbaStrMove>   

00443011   LEA     ECX, DWORD PTR SS:[EBP-A4]   

00443017   CALL    <JMP.&msvbvm50.__vbaFreeVar>   

0044301C   PUSH    DWORD PTR SS:[EBP-5C]********把试验码的前 4 位：2345 入栈!   

0044301F   PUSH    DWORD PTR SS:[EBP-70]********上述结果：4946167 入栈!   

00443022   CALL    <JMP.&msvbvm50.__vbaStrCat>***把两个值连起来：23454946167，放 EAX   

00443027   MOV     EDX, EAX*******************把这个值移入 EDX   

00443029   LEA     ECX, DWORD PTR SS:[EBP-90]   

0044302F   CALL    <JMP.&msvbvm50.__vbaStrMove>   

00443034   PUSH    EAX   

00443035   MOV     EAX, DWORD PTR SS:[EBP+8]*****23454946167 入 EAX   

00443038   PUSH    DWORD PTR DS:[EAX]******试验码：234567765432 入栈!               

0044303A   PUSH    0   

0044303C   CALL    <JMP.&msvbvm50.__vbaStrComp>**比较试验码：234567765432 与 23454946167   

00443041   NEG     AX********************************不相等赋 AX 的值为 FFFF   

00443044   SBB     EAX, EAX   

00443046   NEG     EAX   

00443048   NEG     EAX   

0044304A   MOV     WORD PTR SS:[EBP-120], AX   

00443051   LEA     ECX, DWORD PTR SS:[EBP-90]   

00443057   CALL    <JMP.&msvbvm50.__vbaFreeStr>   

0044305C   MOVSX   EAX, WORD PTR SS:[EBP-120]   

00443063   TEST    EAX, EAX*******************测试 EAX 的值是否是 0   

00443065   JE      SHORT unpack_.0044306E********相等就跳，一跳就成功!   

00443067   AND     WORD PTR SS:[EBP-3C], 0   

0044306C   JMP     SHORT unpack_.00443073
```

### 算法小结

这个软件的注册算法有点怪，没有用用户名或者用户单位计算注册码，而是用试验码的前 `4` 位经过计算后得到后 `7` 位注册码，再把前 `4` 位试验码和后 `7` 位得到的注册码联接起来就是软件的注册码!

其计算过程是这样：

取前 `4` 位试验码，设取出的试验码的位数是 `X`，取出的试验码的 `ASCII` 码值是 `Y`，则 `(X*2+6)*Y^(X*8)`，再把它转化成十进制值，联接起来得到一组 `12` 位的新值，这个新值去掉前 `3` 位，再去掉后两位，得到中间的 `7` 位数值，它的前面再加上前 `4` 位试验码就是注册码!

一组可用的注册码：

*   `Serial Number: 23454946167`
*   `Your full name: yzez`
*   `Your company: [DFCG][BCG][FCG]`

`4dd466d0e1e0cc123afbe504da2462ca`
