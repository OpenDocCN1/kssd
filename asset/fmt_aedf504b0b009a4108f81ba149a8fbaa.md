
```
:00534086 BA0D000000                mov edx, 0000000D  
:0053408B 8B45FC                   mov eax, dword ptr [ebp-04]     \这里是假注册码！  
:0053408E E82D0BEDFF               call 00404BC0                   \这里是取得假注册码第 13、14 位数的值！  
:00534093 FF75DC                   push [ebp-24]                   \假注册码第 13、14 位数的值压栈！  
:00534096 8D45D8                   lea eax, dword ptr [ebp-28]  
:00534099 50                       push eax  
:0053409A B902000000               mov ecx, 00000002  
:0053409F BA12000000               mov edx, 00000012  
:005340A4 8B45FC                   mov eax, dword ptr [ebp-04]     \这里是假注册码！  
:005340A7 E8140BEDFF               call 00404BC0                   \这里是取得假注册码第 18、19 位数的值！  
:005340AC FF75D8                   push [ebp-28]                   \假注册码第 18、19 位数的值压栈！  
:005340AF 8D45E8                   lea eax, dword ptr [ebp-18]  
:005340B2 BA04000000               mov edx, 00000004  
:005340B7 E86C09EDFF               call 00404A28                   \这里是把假注册码第 3、4、8、9、13、14、18、19 位连在一起！  
:005340BC 8D4DD4                   lea ecx, dword ptr [ebp-2C]  
:005340BF BA08000000               mov edx, 00000008  
:005340C4 8B45EC                   mov eax, dword ptr [ebp-14]     \这里是 00534046 的 CALL 把机器码连在一起后的值!  
:005340C7 E828C8F0FF               call 004408F4                   \这里是把机器码计算后的值取其后八位！  
:005340CC 8B45D4                   mov eax, dword ptr [ebp-2C]     \机器码计算后的值的后八位！ 
:005340CF 8B55E8                   mov edx, dword ptr [ebp-18]     \假注册码第 3、4、8、9、13、14、18、19 连在一起! 
:005340D2 E8D509EDFF               call 00404AAC      \这个 CALL 是比较的,有兴趣的朋友可以自己进去转转!  

                                  \我进去后看见有 N 多个地址跳去 404AAC 就晕了!  
:005340D7 7504                     jne 005340DD  
:005340D9 B301                     mov bl, 01  
:005340DB EB02                     jmp 005340DF 

\ Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:005340D7(C)  
:005340DD 33DB                     xor ebx, ebx 
\ Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:005340DB(U)  
:005340DF 33C0                     xor eax, eax  
:005340E1 5A                       pop edx  
:005340E2 59                       pop ecx  
:005340E3 59                       pop ecx  
:005340E4 648910                   mov dword ptr fs:[eax], edx  
:005340E7 6817415300               push 00534117 
\ Referenced by a (U)nconditional or (C)onditional Jump at Address:  
:00534115(U)  
:005340EC 8D45D4                   lea eax, dword ptr [ebp-2C]  
:005340EF BA07000000               mov edx, 00000007  
:005340F4 E8DB05EDFF               call 004046D4  
:005340F9 8D45F8                   lea eax, dword ptr [ebp-08] 

\ Possible StringData Ref from Code Obj -> ".3"  
:005340FC 8B15C03E5300             mov edx, dword ptr [00533EC0]  
:00534102 E8CD1EEDFF               call 00405FD4  
:00534107 8D45FC                   lea eax, dword ptr [ebp-04]  
:0053410A E8A105EDFF               call 004046B0  
:0053410F C3                       ret 

------------------------------------------------------------------  

【总    结】：  

呵呵,终于到总结了,好累!(天都亮了!) 

算法:  

这个程序注册的用户名不做计算,可以说是没有用的!(我个人觉得用户名最大的作用就是显示再关于那里的授权给 "XXX") 

真注册码=机器码的 ASCII 码都乘起来的积-->再加 1 的和-->与 B 或运算的结果-->取得结果的后八位-->与输入的注册码第 3、4、8、9、13、14、18、19 连在一起的值比较,不同就死! 

以我的机器码为例(我的机器码是 `YMDYMH13838`) 

(注:以下括号里的是 16 进制的乘法和加法)  

`59 * 4D * 44 * 59 * 4D * 48 * 31 * 33 * 38 * 33 * 38 + 1` XOR `B` = `FB5E45C45EA3C80A`  

取其后八位 `5EA3C80A` 与注册码第 3、4、8、9、13、14、18、19 连在一起的值比较,不同就死! 

我的机器码是 "YMDYMH13838" 

那么注册码是 "XX5EXXXA3XXXC8XXX0A"(X 为 0-F 间任意) 

终于可以睡觉了!  

最后在这里真心感谢你花了那么多时间看这篇文章!谢谢了... 

                                                                                          2003.05.24 晨于清远
`4dd466d0e1e0cc123afbe504da2462ca`
