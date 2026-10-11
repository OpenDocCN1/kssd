
我们把上面的函数参数表示为 `K1`, `K2`, `K3`, `K4`, `K5`。`K1` 和 `K2` 每次去的是数组 `a` 中的两个。

那么剩下的两个我们表示为 `ecx`，和 `edx`。

上面的 `call`：

```
00468640  /$  55              PUSH EBP     入栈据我估计他使用递归的很佩服作者
00468641  |.  8BEC            MOV EBP,ESP
00468643  |.  53              PUSH EBX
00468644  |.  56              PUSH ESI
00468645  |.  8BF1            MOV ESI,ECX  保存上面窜来的 ecx 设为 M
00468647  |.  8BDA            MOV EBX,EDX  保存上面传来的 edx 摄为 N
00468649  |.  8B06            MOV EAX,DWORD PTR DS:[ESI]
   设 TEMP1=（eax=esi=ecx）=M
0046864B  |.  8B55 18         MOV EDX,DWORD PTR SS:[EBP+18]
设 TEMP2=K1 函数的第一个参数
0046864E  |.  2302            AND EAX,DWORD PTR DS:[EDX]
   TEMP1=TEMP2 AND TEMP2     (与运算)
00468650  |.  8B16            MOV EDX,DWORD PTR DS:[ESI]
TEMP2=ecx
00468652  |.  F7D2            NOT EDX    TEMP2=~TEMP2 求反
00468654  |.  8B4D 14         MOV ECX,DWORD PTR SS:[EBP+14]    函数的第二个参数
00468657  |.  2311            AND EDX,DWORD PTR DS:[ECX]
TEMP2=TEMP2 AND K2   (与运算)
00468659  |.  0BC2            OR EAX,EDX
TEMP1=TEMP1 | TEMP2    或运算
0046865B  |.  0303            ADD EAX,DWORD PTR DS:[EBX]
TEMP1+=N
0046865D  |.  8B55 10         MOV EDX,DWORD PTR SS:[EBP+10]
这是第三个参数 对 c++中函数参数传递不清楚地这下该明白了吧 K3
00468660  |.  0302            ADD EAX,DWORD PTR DS:[EDX]
TEMP1+=K3
00468662  |.  0345 08         ADD EAX,DWORD PTR SS:[EBP+8]
第五个参数 K5    TEMP1+=K5;
00468665  |.  8903            MOV DWORD PTR DS:[EBX],EAX
N=TEMP1;
00468667  |.  8B03            MOV EAX,DWORD PTR DS:[EBX]
00468669  |.  8A55 0C         MOV DL,BYTE PTR SS:[EBP+C]
第四个参数 K4 ,移位
0046866C  |.  E8 8FFFFFFF     CALL ip.00468600 这个 call 的作用是 TEMP1 循环左移 K4 位
TEMP1<<K4;
00468671  |.  8903            MOV DWORD PTR DS:[EBX],EAX
N=TEMP1
00468673  |.  8B06            MOV EAX,DWORD PTR DS:[ESI]  TEMP1=M
00468675  |.  0103            ADD DWORD PTR DS:[EBX],EAX  N+=TEMP1;
00468677  |.  5E              POP ESI
00468678  |.  5B              POP EBX
00468679  |.  5D              POP EBP
0046867A  .   C2 1400         RETN 14
```

注意每次改变的不是函数中传过来的参数而是 `N` 得值，仔细观察内存窗口。

上面的 `call`：

```
00468600  /$  55              PUSH EBP
00468601  |.  8BEC            MOV EBP,ESP
00468603  |.  83C4 F4         ADD ESP,-0C
00468606  |.  8855 FF         MOV BYTE PTR SS:[EBP-1],DL
00468609  |.  8945 F8         MOV DWORD PTR SS:[EBP-8],EAX
0046860C  |.  8D55 F4         LEA EDX,DWORD PTR SS:[EBP-C]
0046860F  |.  8A4D FF         MOV CL,BYTE PTR SS:[EBP-1]
00468612  |.  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]
00468615  |.  D3C0            ROL EAX,CL  循环左移
00468617  |.  8945 F4         MOV DWORD PTR SS:[EBP-C],EAX
0046861A  |.  8B02            MOV EAX,DWORD PTR DS:[EDX]
0046861C  |.  8BE5            MOV ESP,EBP
0046861E  |.  5D              POP EBP
0046861F  .   C3              RETN
```

经过上面的 `n` 个函数后，我们数组中的值是：

```
00CE3B9C   E2F51816
00CE3BA0   9F37DE76
00CE3BA4   CE9B93F9
00CE3BA8   AD29046F
```

别急噩梦还没结束。`F8` 单步执行到这里，怎么知道的，呵呵慢慢式来的。

```
00469253  |>  8D4D E0         LEA ECX,DWORD PTR SS:[EBP-20]
00469256  |.  33C0            XOR EAX,EAX
00469258  |.  8A03            MOV AL,BYTE PTR DS:[EBX]
0046925A  |.  BA 02000000     MOV EDX,2
0046925F  |.  E8 08F9F9FF     CALL ip.00408B6C
00469264  |.  8B55 E0         MOV EDX,DWORD PTR SS:[EBP-20]
00469267  |.  8D45 F4         LEA EAX,DWORD PTR SS:[EBP-C]
0046926A  |.  E8 8DB9F9FF     CALL ip.00404BFC
0046926F  |.  43              INC EBX
00469270  |.  4E              DEC ESI
00469271  |.  75 E0           JNZ SHORT ip.00469253   循环
00469273  |.  8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]
```

这个循环的作用是将上面的数组转化为字符串形式，只有两个 `call` 我不贴代码了。

转化的结果为 `0012F0EC 00CE3BF4 ASCII "1618F5E276DE379FF9939BCE6F0429AD"`。

总结一下吧：我想没人愿意写这个软件的破解思路吧！也没人愿意写他的注册机，我从早上 8 点分析到下午 5 点，加上这篇文章。赫赫作者的算法很麻烦，谁写个注册机气气作者。作者写了这么多算法好像只要爆破一个点，就行了。失败吧！

注册码：就是取上面字串的前 16 位 `1618f5e276de379f`，在前面加 `#`。

一个可用的注册码：`#1618f5e276de379f` 用户名 `mejy[BCG][FCG]`

算法实在太烦写得不好，不好意思。写不出注册机。

```
4dd466d0e1e0cc123afbe504da2462ca
```
